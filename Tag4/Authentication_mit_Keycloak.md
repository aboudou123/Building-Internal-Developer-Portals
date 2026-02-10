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
Im kommenden Lab werden Sie alles implementieren, was in dieser Lektion behandelt wurde:

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

## Zusätzliche Ressourcen
*   [Backstage Authentication](https://backstage.io/docs/auth/)
*   [OIDC Authentication Provider](https://backstage.io/docs/auth/oidc/provider/)
*   [Keycloak Documentation](https://www.keycloak.org/documentation)
*   [OpenID Connect Specification](https://openid.net/specs/openid-connect-core-1_0.html)
*   [Keycloak Catalog Backend Module](https://github.com/backstage/backstage/tree/master/plugins/catalog-backend-module-keycloak)

## Lektionsnotizen (Privat)
Nehmen Sie private Notizen während Sie lernen...

## Studiengruppe
Diesem Kurs ist noch keine Studiengruppe zugeordnet.

Bitten Sie Ihren Instruktor, eine Studiengruppe zu verlinken, um Diskussionen zu ermöglichen.
```