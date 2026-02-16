# OIDC-Authentifizierung mit Keycloak

Authentifizierung ist für produktive Backstage-Bereitstellungen unerlässlich. Diese Lektion behandelt Authentifizierungsgrundlagen und bereitet Sie auf die Implementierung der Keycloak-Integration mit Benutzeridentität und Gruppenmitgliedschaft vor.

## Authentifizierung vs. Autorisierung

### Authentifizierung (AuthN)
**"Wer sind Sie?"** – Verifizierung der Benutzeridentität
*   Benutzername/Passwort-Validierung
*   Multi-Faktor-Authentifizierung
*   Social Logins (Google, GitHub)
*   Single Sign-On (SSO)
*   OIDC-Provider wie Keycloak

### Autorisierung (AuthZ)
**"Was dürfen Sie tun?"** – Kontrolle des Zugriffs auf Ressourcen
*   Rollenbasierte Zugriffskontrolle (RBAC)
*   Berechtigungsrichtlinien, die Benutzergruppen prüfen
*   Ressourcenebenen-Berechtigungen
*   Gruppenbasierte Zugriffskontrolle

**Schlüsselbeziehung:** Authentifizierung muss zuerst erfolgen, um die Identität festzustellen. Sobald ein Benutzer authentifiziert ist, kann seine Gruppenmitgliedschaft für Autorisierungsentscheidungen verwendet werden (behandelt in der nächsten Lektion).

## OpenID Connect (OIDC)

### Was ist OIDC?
OIDC ist eine Schicht über OAuth 2.0, die Folgendes bietet:
*   **Identitätsverifizierung:** Wer der Benutzer ist (`preferred_username`, `email`)
*   **Standardisierte Claims:** Benutzerprofilinformationen in JWT-Tokens
*   **Token-basierte Sicherheit:** JWT-Tokens für sichere Kommunikation
*   **Discovery-Endpunkte:** Wohlbekannte Konfigurations-URLs für automatisches Setup
*   **Gruppenmitgliedschaft:** Claims, die die Gruppenzugehörigkeit des Benutzers enthalten

## Keycloak als Identity Provider

### Was ist Keycloak?
Keycloak ist eine Open-Source-Identitäts- und Zugriffsmanagement-Lösung, die Folgendes bietet:
*   **Selbstgehostete Kontrolle:** Bereitstellung auf eigener Infrastruktur
*   **Vollständige OIDC- und SAML-Unterstützung:** Standardprotokolle für Authentifizierung
*   **Realm-Isolation:** Separate Sicherheitsdomänen für verschiedene Anwendungen
*   **Benutzer- und Gruppenmanagement:** Integrierte Admin-Konsole für die Verwaltung von Identitäten
*   **Service-Account-Rollen:** Erlauben Anwendungen das Abfragen von Benutzer- und Gruppeninformationen
*   **Client-Konfiguration:** Definiert Anwendungen, die Keycloak zur Authentifizierung verwenden können

### Keycloak-Schlüsselkonzepte
**Realms**
*   Isolierte Sicherheitsdomänen, die ihre eigenen Benutzer, Gruppen und Anwendungen verwalten
*   Beispiel: Erstellen Sie einen "backstage"-Realm getrennt vom Standard-"master"-Realm
*   Jeder Realm hat seine eigene Konfiguration und Benutzerbasis

**Clients**
*   Anwendungen, die Keycloak zur Authentifizierung verwenden
*   Konfiguriert mit Client-ID, Client-Secret und erlaubten Redirect-URIs
*   Unterstützen verschiedene Authentifizierungsflows (Standard Flow, Direct Access Grants)

**Users and Groups**
*   **Users:** Einzelne Konten mit Zugangsdaten und Profilinformationen
*   **Groups:** Sammlungen von Benutzern, die für Berechtigungen verwendet werden können
*   Gruppenmitgliedschaft wird in OIDC-Tokens als Claims übergeben

**Mappers**
*   Konfigurieren, welche Informationen in Tokens enthalten sind
*   Group Membership-Mapper fügt Benutzergruppen zu Token-Claims hinzu
*   Custom Mapper können zusätzliche Claims nach Bedarf hinzufügen
*   **Kritisch für Berechtigungen:** Der Group-Mapper stellt sicher, dass die Gruppenmitgliedschaft für Berechtigungsrichtlinien verfügbar ist

### Andere Identity Provider Optionen
*   **Active Directory / Azure AD:** Microsoft Enterprise-Identität mit Windows-Integration
*   **Okta:** Cloud-basierter Identity Service mit Enterprise-SSO-Funktionen
*   **GitHub:** Entwicklerfreundlich mit organisationsbasierter Zugriffskontrolle
*   **Google:** Google Workspace-Integration mit vertrauter Benutzererfahrung

## Backstage-Authentifizierungsarchitektur

### Backend-Konfiguration

**Installation des OIDC-Moduls**
```bash
yarn add @backstage/plugin-auth-backend-module-oidc-provider
```

**app-config.yaml**
Das Backend muss wissen, wo Ihr Keycloak-Server ist und wie die Verbindung hergestellt wird:
```yaml
auth:
  environment: development
  session:
    secret: backstage-session-secret
  providers:
    oidc:
      development:
        metadataUrl: https://keycloak.example.com/realms/backstage/.well-known/openid-configuration
        clientId: backstage
        clientSecret: ${KEYCLOAK_CLIENT_SECRET}
        prompt: auto
```

**Wichtig:** Verwenden Sie immer Ihre öffentliche Keycloak-URL (nicht localhost), da Ihr Browser zur Authentifizierung zu Keycloak umgeleitet werden muss. In Lab-Umgebungen entdecken Sie diese URL aus einer Konfigurationsdatei.

### Sign-in Resolver
Ein benutzerdefinierter Resolver, der Identitäts- und Gruppeninformationen aus dem OIDC-Token extrahiert:
*   Extrahiert den Benutzernamen aus dem `preferred_username`-Claim
*   Erstellt eine User-Entity-Referenz (z.B. `user:default/alice.admin`)
*   Extrahiert Gruppen aus den Token-Claims (falls Group-Mapper konfiguriert ist)
*   Erstellt Group-Entity-Referenzen (z.B. `group:default/platform-admins`)
*   Gibt sowohl Benutzer als auch Gruppen im `ent`-Claim für Berechtigungsprüfungen zurück

```javascript
async signInResolver(info, ctx) {
  const username = info.result.fullProfile.userinfo.preferred_username;
  const userRef = stringifyEntityRef({ kind: 'User', name: username });

  const keycloakGroups = info.result.fullProfile.userinfo.groups || [];
  const groupRefs = keycloakGroups.map(group =>
    stringifyEntityRef({ kind: 'Group', name: group })
  );

  return ctx.issueToken({
    claims: {
      sub: userRef,
      ent: [userRef, ...groupRefs],
    },
  });
}
```

Der `ent`-Claim (Ownership Entities) ist entscheidend – er enthält sowohl die Benutzeridentität ALS AUCH deren Gruppenmitgliedschaften, welche Berechtigungsrichtlinien für Autorisierungsentscheidungen verwenden werden.

### Frontend-Konfiguration
**API-Referenz und Factory**
*   Erstellen Sie eine API-Referenz für die Keycloak-OIDC-Authentifizierung
*   Verwenden Sie `OAuth2.create()` zur Konfiguration des Providers mit Scopes und Popup-Optionen
*   Standard-Scopes: `openid`, `profile`, `email`

**Sign-In Page**
```jsx
<SignInPage
  providers={[
    {
      id: 'keycloak',
      title: 'Keycloak',
      message: 'Sign in using Keycloak',
      apiRef: keycloakOIDCAuthApiRef,
    },
  ]}
/>
```

## Catalog-Synchronisierung

### Keycloak Catalog Provider
Synchronisiert automatisch Benutzer und Gruppen von Keycloak zum Backstage-Catalog. Dies stellt sicher, dass wenn Benutzer in Backstage auf Benutzer- oder Gruppenreferenzen klicken, sie tatsächliche Catalog-Entities sehen statt "entity not found"-Fehler.

**Installation des Moduls**
```bash
yarn add @backstage-community/plugin-catalog-backend-module-keycloak
```

**Konfiguration des Providers**
```yaml
catalog:
  providers:
    keycloakOrg:
      default:
        baseUrl: http://keycloak:8080
        loginRealm: backstage
        realm: backstage
        clientId: backstage
        clientSecret: ${KEYCLOAK_CLIENT_SECRET}
        schedule:
          frequency: { minutes: 5 }
          timeout: { minutes: 3 }
```

### Service Account Permissions
Der Backstage-Client benötigt diese Keycloak-Rollen zum Lesen von Benutzern und Gruppen:
*   `view-users`
*   `query-groups`
*   `query-users`

Dies ermöglicht es dem Catalog-Provider, Keycloak-Benutzer und -Gruppen automatisch als Backstage-Catalog-Entities zu importieren.

## Sicherheits-Best Practices

### Docker Secrets für Credentials
Verwenden Sie Docker Secrets statt Umgebungsvariablen für sensible Daten. Für Services, die das Lesen aus Dateien unterstützen (wie PostgreSQL), verwenden Sie die nativen Secrets von Docker:
```yaml
services:
  backstage:
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/postgres_password
      KEYCLOAK_CLIENT_SECRET_FILE: /run/secrets/keycloak_client_secret
    secrets:
      - postgres_password
      - keycloak_client_secret

secrets:
  postgres_password:
    file: ./secrets/postgres_password.txt
  keycloak_client_secret:
    file: ./secrets/keycloak_client_secret.txt
```

Für Keycloak, welches das native Lesen von Passwörtern aus Dateien nicht unterstützt, verwenden Sie ein benutzerdefiniertes Entrypoint-Skript mit Bind-Mounts:
```yaml
keycloak:
  entrypoint: /bin/bash
  command:
    - -c
    - |
      export KEYCLOAK_ADMIN_PASSWORD=$$(cat /secrets/keycloak_admin_password | tr -d '[:space:]')
      /opt/keycloak/bin/kc.sh start-dev
  volumes:
    - ./secrets/keycloak_admin_password.txt:/secrets/keycloak_admin_password:ro
```

Dieses Pattern hält Credentials aus `docker-compose.yml`-Dateien und Umgebungsvariablen fern.

### Redirect-URI-Konfiguration
*   Verwenden Sie öffentliche URLs für Redirect-URIs, nicht localhost
*   Konfigurieren Sie gültige Redirect-URIs in den Keycloak-Client-Einstellungen
*   Beispiel: `https://backstage-app.example.com/api/auth/oidc/handler/frame`
*   In Lab-Umgebungen entdecken Sie Ihre öffentliche URL aus Konfigurationsdateien

### HTTPS-Anforderungen
Produktionsbereitstellungen erfordern HTTPS für sichere Authentifizierungsflows:

```yaml
app:
  baseUrl: https://backstage.company.com
backend:
  baseUrl: https://backstage-api.company.com
```

### Session-Sicherheit
```yaml
auth:
  session:
    secret: ${SESSION_SECRET}  # Starkes, zufälliges Secret verwenden
```

## Kompletter Authentifizierungsflow
So arbeiten alle Teile zusammen:

1.  **Benutzer besucht Backstage:** Sieht Keycloak-Sign-in-Option
2.  **Klickt auf Sign-in:** Browser wird zur öffentlichen Keycloak-URL umgeleitet
3.  **Gibt Credentials ein:** Benutzer authentifiziert sich bei Keycloak
4.  **Keycloak leitet zurück:** Mit Autorisierungscode zur Backstage-Callback-URL
5.  **Token-Austausch:** Backstage-Backend tauscht Code gegen Tokens
6.  **Extract claims:** Ruft Benutzername und Gruppen aus Token-Claims ab
7.  **Sign-in resolver:** Erstellt User-Entity-Ref und Group-Refs im `ent`-Claim
8.  **Issue session:** Backstage erstellt Session mit Identität und Ownership-Entities
9.  **User authenticated:** Benutzer kann nun auf Backstage mit etablierter Identität zugreifen
10. **Catalog sync:** Benutzer und Gruppen erscheinen als Catalog-Entities

## Lab-Vorschau: Keycloak-Integration
Im kommenden Lab werden wir alles implementieren, was in dieser Lektion behandelt wurde:

*   Keycloak in Docker neben Backstage mit sicherer Credential-Verwaltung bereitstellen
*   Einen Backstage-Realm mit korrekter OIDC-Client-Konfiguration erstellen
*   Redirect-URIs mit den öffentlichen URLs Ihres Labs konfigurieren (nicht localhost)
*   Group-Mapper einrichten, um Gruppenmitgliedschaft in Tokens einzubeziehen
*   OIDC-Modul im Backstage-Backend installieren
*   Sign-in-Resolver erstellen, um Benutzername und Gruppen aus Tokens zu extrahieren
*   Frontend konfigurieren, um Keycloak-Sign-in-Option anzuzeigen
*   Testbenutzer mit verschiedenen Gruppenmitgliedschaften erstellen (platform-admins, developers, guests)
*   Zum Catalog synchronisieren, damit Benutzer und Gruppen als Backstage-Entities erscheinen

Dies etabliert die Authentifizierungsgrundlage, auf der Sie in der nächsten Lektion aufbauen werden, um rollenbasierte Zugriffskontrolle zu implementieren.

## Kernpunkte

*   Authentifizierung stellt die Benutzeridentität vor der Autorisierung fest – OIDC bietet standardisierte, JWT-Token-basierte Authentifizierung
*   Keycloak-Realms isolieren Sicherheitsdomänen, Clients repräsentieren Anwendungen mit Redirect-URIs und Secrets
*   Group-Mapper in Keycloak ist kritisch – fügt Gruppenmitgliedschaft zu Token-Claims für Berechtigungsrichtlinien hinzu
*   Sign-in-Resolver extrahiert Benutzername und Gruppen aus Tokens, erstellt Entity-Referenzen im `ent`-Claim
*   Immer öffentliche URLs (nicht localhost) für Keycloak und Redirect-URIs in Lab-/Produktionsumgebungen verwenden
*   Catalog-Provider synchronisiert Keycloak-Benutzer und -Gruppen zu Backstage unter Verwendung eines Service-Accounts mit Query-Berechtigungen
*   Sicherheits-Best Practices: Docker Secrets für Credentials, Bind-Mounts für Keycloak-Passwort, HTTPS in der Produktion

```

## Zusätzliche Ressourcen

*   [Backstage Authentication](https://backstage.io/docs/auth/)
*   [OIDC Authentication Provider](https://backstage.io/docs/auth/oidc/provider/)
*   [Keycloak Documentation](https://www.keycloak.org/documentation)
*   [OpenID Connect Specification](https://openid.net/specs/openid-connect-core-1_0.html)
*   [Keycloak Catalog Backend Module](https://github.com/backstage/backstage/tree/master/plugins/catalog-backend-module-keycloak)

```

# Über das Lab

Implementieren Sie produktionsreife Authentifizierung mit Keycloak als OIDC-Provider. Konfigurieren Sie Realms und Clients, richten Sie Benutzerverwaltung ein, integrieren Sie diese mit der Backstage-Authentifizierung und testen Sie komplette Authentifizierungs-Workflows.

**Wir werden lernen:**

*   Keycloak mit Docker und korrekter Konfiguration bereitzustellen
*   Realms, Clients und OIDC-Einstellungen für die Backstage-Integration zu konfigurieren
*   Den Backstage OIDC-Authentifizierungsprovider zu konfigurieren
*   Komplette Authentifizierungs-Workflows und Benutzersitzungen zu testen
*   Best Practices für Authentifizierungssicherheit zu verstehen
*   Benutzergruppen und Rollenzuordnungen für Berechtigungen einzurichten
*   Keycloak-Benutzer und -Gruppen zum Backstage-Catalog zu synchronisieren

Dieses praxisorientierte Lab etabliert Enterprise-fähige Authentifizierung für Ihr Developer Portal.

## Wichtige Ressourcen

*   [Authentifizierungs-Überblick](https://backstage.io/docs/auth/)
*   [OIDC-Provider Setup](https://backstage.io/docs/auth/oidc/provider/)
*   [Keycloak Dokumentation](https://www.keycloak.org/documentation)

## Was Sie lernen werden (6 Aufgaben)

### 1. Keycloak zu Docker Compose hinzufügen
Fügen Sie den Keycloak Identity Provider zu Ihrem bestehenden Docker Compose Stack neben Backstage hinzu.

### 2. Backstage-Realm und OIDC-Client konfigurieren
Erstellen Sie einen dedizierten Realm für Backstage mit korrekter OIDC-Client-Konfiguration in Keycloak.

### 3. Backstage-Backend für OIDC konfigurieren
Konfigurieren Sie das Backstage-Backend, um Benutzer über Keycloak mittels OIDC zu authentifizieren.

### 4. Backstage-Frontend für OIDC konfigurieren
Konfigurieren Sie das Backstage-Frontend, um die Keycloak-Sign-in-Option mit OIDC anzuzeigen.

### 5. Authentifizierung testen und Benutzer erstellen
Testen Sie den kompletten Authentifizierungs-Workflow und erstellen Sie Benutzer für verschiedene Zugriffsszenarien.

### 6. Keycloak-Benutzer und -Gruppen zum Catalog synchronisieren
Importieren Sie Keycloak-Benutzer und -Gruppen als Entities in den Backstage-Catalog.

```
```



==========================================================
# Rollenbasierte Zugriffskontrolle mit Berechtigungen
==========================================================


Nachdem wir nun verstehen, wie  Benutzer mit Keycloak authentifizieren und Gruppenmitgliedschaften erfassen, behandelt diese Lektion, wie wir diese Informationen nutzen können, um über das Berechtigungs-Framework zu steuern, was Benutzer in Backstage tun dürfen.

## Autorisierung: Aufbauend auf Authentifizierung
In der vorherigen Lektion haben Sie gelernt, wie Authentifizierung Identität feststellt. Jetzt werden wir diese Identität (und Gruppenmitgliedschaft) für Autorisierungsentscheidungen nutzen.

**Rückblick aus der Authentifizierungslektion:**

*   Benutzer authentifizieren sich über Keycloak OIDC
*   Sign-in-Resolver extrahiert Benutzername und Gruppen aus Token-Claims
*   Gruppen werden im `ent`-Claim (Ownership Entities) enthalten
*   Beispiel: `['user:default/alice.admin', 'group:default/platform-admins']`

**Autorisierung baut auf diesem Fundament auf:**

*   Berechtigungsrichtlinien (Permission Policies) prüfen den `ent`-Claim, um die Gruppen des Benutzers zu sehen
*   Verschiedene Gruppen erhalten unterschiedliche Zugriffsstufen
*   Richtlinien treffen ALLOW/DENY-Entscheidungen für jede Aktion

## Backstage-Berechtigungs-Framework

### Was sind Berechtigungen?
Berechtigungen kontrollieren, was authentifizierte Benutzer in Backstage tun dürfen. Nachdem die Authentifizierung die Identität festgestellt hat, setzt das Berechtigungs-Framework die Autorisierung durch.

**Häufige Berechtigungsbeispiele:**

*   `catalog.entity.read` – Catalog-Entities ansehen
*   `catalog.entity.create` – Neue Entities erstellen
*   `catalog.entity.update` – Bestehende Entities ändern
*   `catalog.entity.delete` – Entities entfernen
*   `scaffolder.template.execute` – Software-Templates ausführen

### Aktivierung des Berechtigungs-Frameworks in Backstage
Das Berechtigungs-Framework wird in `app-config.yaml` aktiviert:
```yaml
permission:
  enabled: true
```

### Berechtigungsrichtlinien-Architektur

Eine Richtlinie (Policy) ist TypeScript-Code, der Berechtigungsanfragen empfängt und Entscheidungen trifft. Der Ablauf ist:

Benutzeraktion → Plugin → Berechtigungs-Framework → Ihre Richtlinie → ALLOW/DENY → Aktionsergebnis

**Drei Schlüsselkomponenten:**

1.  **PolicyQuery** – Enthält die geprüfte Berechtigung
    *   Berechtigungsname (z.B. `catalog.entity.delete`)
    *   Berechtigungsattribute (z.B. `action: 'delete'`)

2.  **PolicyQueryUser** – Enthält Benutzerinformationen vom Sign-in-Resolver
    *   `user.info.ownershipEntityRefs` – Array von Benutzer- und Group-Entity-Referenzen
    *   Beispiel: `['user:default/alice.admin', 'group:default/platform-admins']`

3.  **PolicyDecision** – Die Antwort Ihrer Richtlinie
    *   `AuthorizeResult.ALLOW` – Berechtigung gewährt
    *   `AuthorizeResult.DENY` – Berechtigung verweigert
    *   `AuthorizeResult.CONDITIONAL` – Berechtigung mit Bedingungen gewährt (fortgeschritten)

## Gruppenbasierte Berechtigungsrichtlinie
Hier ist eine vollständige Berechtigungsrichtlinie, die Keycloak-Gruppenmitgliedschaft prüft:

```typescript
import { createBackendModule } from '@backstage/backend-plugin-api';
import {
  PolicyDecision,
  AuthorizeResult,
} from '@backstage/plugin-permission-common';
import {
  PermissionPolicy,
  PolicyQuery,
  PolicyQueryUser,
} from '@backstage/plugin-permission-node';
import { policyExtensionPoint } from '@backstage/plugin-permission-node/alpha';

class KeycloakGroupPolicy implements PermissionPolicy {
  async handle(
    request: PolicyQuery,
    user?: PolicyQueryUser,
  ): Promise<PolicyDecision> {
    // Verweigern, wenn nicht authentifiziert
    if (!user) {
      return { result: AuthorizeResult.DENY };
    }

    // Gruppen des Benutzers aus dem ent-Claim holen (vom Sign-in-Resolver gesetzt)
    const groups = user.info.ownershipEntityRefs || [];

    // Platform-Admins haben vollen Zugriff
    if (groups.some(g => g === 'group:default/platform-admins')) {
      return { result: AuthorizeResult.ALLOW };
    }

    // Entwickler können alles außer löschen
    if (groups.some(g => g === 'group:default/developers')) {
      const action = request.permission.attributes.action;
      if (action === 'delete') {
        return { result: AuthorizeResult.DENY };
      }
      return { result: AuthorizeResult.ALLOW };
    }

    // Gäste können nur lesen
    if (groups.some(g => g === 'group:default/guests')) {
      if (request.permission.attributes.action === 'read') {
        return { result: AuthorizeResult.ALLOW };
      }
      return { result: AuthorizeResult.DENY };
    }

    // Standard: unbekannte Gruppen verweigern
    return { result: AuthorizeResult.DENY };
  }
}

export const permissionPolicyModule = createBackendModule({
  pluginId: 'permission',
  moduleId: 'keycloak-group-policy',
  register(reg) {
    reg.registerInit({
      deps: { policy: policyExtensionPoint },
      async init({ policy }) {
        policy.setPolicy(new KeycloakGroupPolicy());
      },
    });
  },
});
```

### Wie die Richtlinie funktioniert
**Schritt-für-Schritt-Ablauf:**

1.  Benutzer versucht Aktion (z.B. löschen einer Catalog-Entity)
2.  Plugin ruft Berechtigungs-Framework mit der Berechtigungsanfrage auf
3.  Framework ruft Ihre Richtlinie mit `PolicyQuery` und `PolicyQueryUser` auf
4.  Richtlinie extrahiert Gruppen aus `user.info.ownershipEntityRefs`
5.  Richtlinie prüft Gruppenmitgliedschaft mit `groups.some()`
6.  Richtlinie gibt Entscheidung zurück (ALLOW oder DENY)
7.  Framework setzt Entscheidung durch – Aktion wird fortgesetzt oder blockiert

**Wichtige Implementierungsdetails:**

*   Zuerst prüfen, ob Benutzer existiert – nicht authentifizierte Benutzer werden verweigert
*   `user.info.ownershipEntityRefs` verwenden, um Gruppen zu erhalten (vom Sign-in-Resolver gesetzt)
*   Group-Refs folgen dem Muster: `group:default/group-name`
*   `request.permission.attributes.action` prüfen, um read/create/update/delete zu unterscheiden
*   Explizites ALLOW oder DENY für jeden Fall zurückgeben

## Gruppenbasierte Zugriffskontrollstufen
Entwerfen Sie Ihre Zugriffsebenen basierend auf den Bedürfnissen Ihrer Organisation:

| Gruppe | Zugriffsebene | Anwendungsfall |
| :--- | :--- | :--- |
| `platform-admins` | Vollzugriff (alle Operationen) | Plattformteam, das Backstage verwaltet |
| `developers` | Lesen + Erstellen + Aktualisieren (kein Löschen) | Ingenieure, die Templates und Catalog nutzen |
| `guests` | Nur Lesezugriff | Stakeholder, die Dokumentation einsehen |

**Beispiele aus der Praxis:**

**Platform Admins:**
*   Software-Templates erstellen und ändern
*   Catalog-Entities löschen
*   TechDocs und Plugins verwalten
*   Vollständiger administrativer Zugriff

**Developers:**
*   Software-Templates ausführen, um Projekte zu erstellen
*   Catalog-Entities registrieren und aktualisieren
*   **Können nicht löschen**, um versehentlichen Datenverlust zu verhindern
*   TechDocs erstellen und ansehen

**Guests:**
*   Catalog-Entities und Dokumentation ansehen
*   Verfügbare Templates durchsuchen
*   **Können keine Änderungen vornehmen**
*   Nützlich für Manager, Stakeholder, Auditoren

## Richtlinien-Registrierung
Registrieren Sie Ihre Berechtigungsrichtlinie im Backend:

```typescript
// In packages/backend/src/index.ts
import { permissionPolicyModule } from './permissionPolicy';

// Allow-All-Richtlinie entfernen (nur EINE Richtlinie kann aktiv sein)
// backend.add(import('@backstage/plugin-permission-backend-module-allow-all-policy'));

// Eigene benutzerdefinierte Richtlinie hinzufügen
backend.add(permissionPolicyModule);
```

**Wichtig:** Backstage kann nur eine aktive Berechtigungsrichtlinie haben. Wenn Sie die allow-all-policy aktiviert lassen, wird sie mit Ihrer benutzerdefinierten Richtlinie in Konflikt stehen.

## Sicherheitsüberlegungen

### Verweigern als Standard
Beenden Sie Ihre Richtlinie immer mit `return { result: AuthorizeResult.DENY }`, um sicherzustellen, dass unbekannte Fälle blockiert werden.

### Benutzerobjekt validieren
Prüfen Sie, dass der Benutzer existiert, bevor Sie auf Eigenschaften zugreifen, um unauthentifizierte Anfragen sicher zu handhaben.

### Gruppenreferenz-Format
Group-Entity-Refs folgen dem Muster `group:default/group-name`. Passen Sie dies genau in Ihren Vergleichen an.

### Verweigerte Berechtigungen loggen
Erwägen Sie, verweigerte Berechtigungen zu loggen (besonders in der Entwicklung), um das Richtlinienverhalten zu verstehen:

```typescript
if (result === AuthorizeResult.DENY) {
  console.log(`Permission denied: ${request.permission.name} for user ${user?.entity?.metadata.name}`);
}
```

## Lab-Vorschau: Rollenbasierte Zugriffskontrolle
Im kommenden Lab werden wir alles implementieren, was in dieser Lektion behandelt wurde:

**Lab-Umgebung:**

*   Keycloak-Authentifizierung ist für Sie vorkonfiguriert
*   Drei Testbenutzer existieren bereits mit verschiedenen Gruppen:
    *   `alice.admin` (`platform-admins`)
    *   `bob.developer` (`developers`)
    *   `carol.guest` (`guests`)
*   Sign-in-Resolver extrahiert bereits Gruppen in den `ent`-Claim

**Aufgaben:**

1.  Berechtigungs-Framework in `app-config.yaml` aktivieren
2.  Berechtigungsrichtlinien-Modul erstellen, das Keycloak-Gruppenmitgliedschaft prüft
3.  Zugriffsebenen für die drei Gruppen (admin, developer, guest) implementieren
4.  Allow-All-Richtlinie entfernen, um Ihre benutzerdefinierte Richtlinie zu aktivieren
5.  Mit verschiedenen Benutzern testen, um zu verifizieren, dass jede Gruppe angemessene Berechtigungen hat
6.  Template- und Catalog-Zugriffskontrolle basierend auf Gruppenmitgliedschaft verifizieren

Dieses Lab baut direkt auf dem Authentifizierungsfundament aus dem Keycloak-Integration-Lab auf.

## Kernpunkte
*   Autorisierung baut auf Authentifizierung auf – Berechtigungsrichtlinien verwenden Gruppen aus dem `ent`-Claim des Sign-in-Resolvers
*   `user.info.ownershipEntityRefs` enthält Array von Benutzer- und Group-Entity-Refs aus der Authentifizierung
*   Berechtigungsrichtlinien prüfen `PolicyQuery` (Berechtigung und Aktion) und `PolicyQueryUser` (Identität und Gruppen)
*   Drei Entscheidungstypen: ALLOW gewährt Zugriff, DENY blockiert Zugriff, CONDITIONAL erlaubt mit zusätzlichen Regeln
*   `request.permission.attributes.action` verwenden, um read/create/update/delete-Operationen zu unterscheiden
*   Immer standardmäßig verweigern und Benutzerexistenz validieren – nur eine Richtlinie kann gleichzeitig aktiv sein

## Zusätzliche Ressourcen
*   [Backstage Permissions Framework](https://backstage.io/docs/permissions/overview/)
*   [Permission Concepts](https://backstage.io/docs/permissions/concepts/)
*   [Writing Permission Policies](https://backstage.io/docs/permissions/writing-policies/)
*   [Getting Started with Permissions](https://backstage.io/docs/permissions/getting-started/)

```


```markdown

# Über das Lab

Bauen Sie auf Ihrem Keycloak-Authentifizierungs-Setup auf, um Autorisierung zu implementieren. Dieses Lab kommt mit vollständig konfiguriertem Keycloak (Realm, Benutzer, Gruppen, OIDC), sodass Sie sich vollständig auf das Berechtigungs-Framework konzentrieren können.
```
**Für Sie vorkonfiguriert:**

*   Keycloak mit Backstage-Realm und OIDC-Client
*   Group-Mapper, der Gruppen an Backstage-Tokens sendet
*   Drei Testbenutzer mit verschiedenen Gruppenmitgliedschaften:
    *   `alice.admin` (`platform-admins`) – Vollzugriff
    *   `bob.developer` (`developers`) – Lesen + eingeschränktes Schreiben
    *   `carol.guest` (`guests`) – Nur Lesen
*   Sign-in-Resolver, der Gruppen aus Tokens extrahiert

```
```
**Sie werden lernen:**

*   Das Backstage-Berechtigungs-Framework zu aktivieren und zu konfigurieren
*   Berechtigungsrichtlinien zu schreiben, die Keycloak-Gruppenmitgliedschaft prüfen
*   Unterschiedliche Zugriffsebenen (Admin, Developer, Guest) zu implementieren
*   Software-Templates und Catalog-Aktionen rollenbasiert zu sichern
*   Zu testen, dass verschiedene Benutzer unterschiedliche Berechtigungen haben
*   Richtlinienentscheidungen zu verstehen: ALLOW, DENY, CONDITIONAL
```
Dieses praxisorientierte Lab etabliert rollenbasierte Sicherheitsgrenzen unter Verwendung Ihrer bestehenden Identity-Provider-Gruppen.
```



## Wichtige Ressourcen

*   [Permissions Overview](https://backstage.io/docs/permissions/overview/)
*   [Permission Concepts](https://backstage.io/docs/permissions/concepts/)
*   [Writing Policies](https://backstage.io/docs/permissions/writing-policies/)



