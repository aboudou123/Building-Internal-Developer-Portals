

# 1 - Backstage ist eine Plattform zur Entwicklung und zum Betrieb interner Entwicklerportale

---

### Lernziele

* Backstage lokal, in Docker und in einer produktiven Kubernetes-Umgebung einrichten und betreiben
* Einen Softwarekatalog mit Services, Teams und Dokumentation erstellen und verwalten
* Software-Templates entwickeln, mit denen Entwickler neue Projekte schnell erstellen können
* TechDocs für Documentation-as-Code-Workflows konfigurieren
* Authentifizierung mit OIDC-Providern einrichten, insbesondere mit **Keycloak**
* Ein vollständiges Entwicklerportal mit allen fünf zentralen Backstage-Kernfunktionen aufbauen
* Beispielorganisationen mit korrekten Service-Beziehungen importieren
* „Golden-Path“-Templates für gängige Projekttypen erstellen
* Automatische Discovery aus GitHub-Repositories einrichten
* Grundlegendes Group-Mapping und Berechtigungen konfigurieren
* Einfache Plugins zur Erweiterung der Backstage-Funktionalität entwickeln
* Das Kubernetes-Plugin konfigurieren, um Entwicklern Transparenz über Workloads zu bieten
* Katalog-Entitäten mittels Annotationen auf Kubernetes-Ressourcen abbilden
* Den geschäftlichen Mehrwert von Backstage sowie Strategien zur Erfolgsmessung und Adoption verstehen
* ArgoCD mit Backstage integrieren, um GitOps-Deployments sichtbar zu machen
* Produktive Templates mit vollständigen CI/CD-Pipelines erstellen
* End-to-End-Self-Service-Workflows für Entwickler erleben
* GitHub-Actions-Workflows mit korrekten Berechtigungen für den Zugriff auf Container-Registries konfigurieren
* Multi-Stage-Docker-Builds für mehr Sicherheit und Effizienz in der Produktion implementieren
* Services gemäß Best Practices in dedizierte Kubernetes-Namespaces deployen
* Die GitHub Container Registry (GHCR) zur Speicherung von Docker-Images integrieren
* GitOps-Prinzipien verstehen, mit Git als Single Source of Truth
* Häufige Fehler bei der Katalog-Ingestion und Entitätsvalidierung beheben
* Backstage-Themes anpassen und Material-UI-Komponenten effektiv einsetzen
* Ein produktionsnahes Backstage-Setup auf dem eigenen Laptop, Server oder einer Cloud-VM replizieren
* Umfassende Vorbereitung auf die **CNCF Certified Backstage Associate (CBA)**-Prüfung

---

### Erworbene Fähigkeiten

* Entwicklung interner Entwicklerportale (IDP) mit Backstage
* Verwaltung und Strukturierung von Softwarekatalogen
* Erstellung von Software-Templates und Git-Integration
* Documentation-as-Code mit TechDocs und MkDocs
* OIDC-Authentifizierung und -Autorisierung mit Keycloak
* Grundlagen der React-Plugin-Entwicklung
* Docker-Containerisierung für Backstage
* Kubernetes-Deployment und -Orchestrierung
* Konfiguration des Kubernetes-Plugins für Workload-Transparenz
* Grundlagen des Platform Engineerings
* ArgoCD- und GitOps-Deployment-Patterns
* Erstellung von CI/CD-Pipelines mit GitHub Actions
* Design von Entwickler-Self-Service-Workflows
* Integration und Verwaltung von Images in der GitHub Container Registry
* Multi-Stage-Dockerfile-Patterns für produktive Sicherheit
* Kubernetes-Namespace-Isolation für Multi-Tenant-Cluster
* Berechtigungsmanagement in GitHub-Actions-Workflows
* Keycloak-SSO-Integration mit automatischer Benutzer- und Gruppensynchronisation
* Fehleranalyse bei Katalog-Ingestion und Entitätsverarbeitung
* Material-UI-Theming und responsives UI-Design
* Aufbau und Betrieb einer selbstgehosteten Backstage-Produktionsumgebung


---

### Voraussetzungen

* Grundkenntnisse in JavaScript/TypeScript (Variablen, Funktionen, Objekte)
* Git-Grundlagen (Clone, Commit, Push, Pull, Branching)
* Sicherer Umgang mit der Kommandozeile (Befehle ausführen, Verzeichnisse navigieren)
* Grundlegendes Verständnis von Docker-Containern (von Vorteil)
* Grundlegendes Verständnis von Kubernetes-Konzepten (Pods, Deployments, Services)

===========================================================================================

---

# Was ist Backstage?

Einführung in **Backstage**, die von Spotify entwickelte Open-Source-Plattform für Entwicklerportale, sowie ein grundlegendes Verständnis von **Internal Developer Portals (IDP)**.

---

### Zentrale Punkte

* Internal Developer Portals lösen Produktivitätsprobleme, indem sie Entwickler-Tools und Informationen in einer zentralen Oberfläche bündeln
* Backstage ist eine von Spotify entwickelte Open-Source-Plattform zum Aufbau von Entwicklerportalen
* Fünf zentrale Säulen: **Softwarekatalog**, **Templates**, **TechDocs**, **Authentifizierung** und **Suche**
* Basierend auf modernen Web-Technologien (React, Node.js, PostgreSQL) mit einer erweiterbaren Plugin-Architektur
* Ein Entitätsmodell zur Strukturierung von Komponenten, APIs, Systemen, Benutzern, Gruppen und Ressourcen inklusive ihrer Beziehungen
* Bietet messbare Vorteile wie schnellere Auffindbarkeit, konsistente Workflows und verbessertes Onboarding
* Besonders geeignet für Organisationen mit mehreren Teams, wachsender Anzahl an Services und einer fragmentierten Tool-Landschaft


---

### Projektstruktur und Konfiguration

Verständnis der Monorepo-Architektur von Backstage, des **packages**-Verzeichnisses sowie des leistungsfähigen **app-config.yaml**-Konfigurationssystems.

---

### Zentrale Punkte

* Backstage verwendet eine **Monorepo-Struktur** mit `packages/` für die Applikationslogik, `plugins/` für Erweiterungen und `app-config.yaml` als zentrale Konfigurationsdatei
* Die **Frontend-Anwendung** (`packages/app`) ist eine React-Applikation mit Routing, UI-Komponenten und Theme-Anpassungen
* Der **Backend-Service** (`packages/backend`) ist ein Node.js-basierter API-Server, der Datenverarbeitung und Integrationen übernimmt
* `app-config.yaml` fungiert als zentrale Konfigurationsdrehscheibe zur Steuerung von Applikationsmetadaten, Backend-Einstellungen, Integrationen, TechDocs, Authentifizierung und Katalog
* Die **Katalogkonfiguration** definiert, aus welchen Quellen Backstage Entitätsdefinitionen entdeckt (dateibasierte Locations)
* **Yarn Workspaces** verwalten Abhängigkeiten über alle Pakete hinweg und ermöglichen gemeinsames Tooling sowie konsistente Versionierung

**Projektstruktur und Konfiguration**
Nachdem Backstage nun lokal läuft und du im vorherigen Lab bereits die Projektstruktur gesehen hast, dient diese Lektion vor allem der Wiederholung dieser Struktur. Zusätzlich betrachten wir die wichtigsten Dateien, die Backstage funktionsfähig machen.

---

**High-Level-Architektur**
Backstage folgt einem Monorepo-Ansatz mit klarer Trennung zwischen Frontend und Backend:

```
my-backstage/
├── packages/           # Main application code
├── plugins/           # Custom plugins
├── app-config.yaml    # Configuration
├── package.json       # Dependencies
└── README.md          # Documentation
```

Diese Struktur unterstützt:

Mehrere Pakete in einem einzigen Repository
Geteilte Abhängigkeiten und gemeinsames Tooling
Plugin-Entwicklung parallel zur Kernanwendung
Konsistente Builds und Deployments

---

**Das packages-Verzeichnis**
Das Verzeichnis `packages/` enthält die zentralen Anwendungskomponenten. Schauen wir uns jede davon an, um ihre jeweilige Rolle zu verstehen.

---

**Frontend-App (packages/app/)**
Das React-Frontend, mit dem Benutzer interagieren:

```
packages/app/
├── src/
│   ├── App.tsx              # Main app component with routing
│   ├── components/          # Reusable UI components
│   ├── App.test.tsx         # Automated tests
│   └── index.tsx            # Application entry point
├── public/                  # Static assets and favicon
├── package.json             # Frontend dependencies
└── tsconfig.json            # TypeScript configuration
```

Was du beim Erkunden findest:

App.tsx: Zentrale React-Komponente, die das gesamte Portal mit Navigation, Routing und Theme rendert
components/: Verzeichnis für benutzerdefinierte UI-Komponenten, die auf die Anforderungen deines Portals zugeschnitten sind
App.test.tsx: Automatisierte Tests zur Qualitätssicherung und zur Vermeidung von Regressionen

---

**Backend-Service (packages/backend/)**
Der Node.js-API-Server, der das Portal antreibt:

```
packages/backend/
├── src/
│   ├── index.ts             # Entry point that starts the server
│   ├── plugins/             # Backend plugin configuration
│   └── types.ts             # TypeScript type definitions
├── package.json             # Backend dependencies
└── Dockerfile               # Container build instructions
```

Was du beim Erkunden findest:

index.ts: Einstiegspunkt, der den Backend-Server startet und Plugins lädt
plugins/: Konfiguration der verschiedenen Backstage-Backend-Plugins
types.ts: TypeScript-Definitionen für benutzerdefinierte Datenstrukturen

Diese klare Trennung zwischen Frontend und Backend ermöglicht es Teams, beide unabhängig voneinander zu entwickeln und dennoch über die Monorepo-Struktur synchron zu halten.

---

**app-config.yaml verstehen**
Die Datei `app-config.yaml` ist die zentrale Konfigurationsdrehscheibe für deine gesamte Backstage-Instanz. Sehen wir uns an, was die einzelnen Abschnitte steuern.

---

**Anwendungsmetadaten**
Diese Einstellungen definieren Identität und Branding deines Portals:

```
app:
  title: Scaffolded Backstage App    # Browser tab title and header
  baseUrl: http://YOUR_IP:3000       # Frontend URL

organization:
  name: My Company                    # Organization name displayed throughout
```

---

**Backend-Konfiguration**
Steuert, wie der Backend-API-Server bereitgestellt wird:

```
backend:
  baseUrl: http://YOUR_IP:7007       # Backend API endpoint
  listen:
    port: 7007                        # Port for backend service
    host: 0.0.0.0                     # Listen on all network interfaces
  csp:
    connect-src: ["'self'", 'http:', 'https:']
  cors:
    origin: http://YOUR_IP:3000      # Allow frontend to call backend
    methods: [GET, HEAD, PATCH, POST, PUT, DELETE]
    credentials: true
  database:
    client: better-sqlite3             # Database type
    connection: ':memory:'             # In-memory database for development
```

---

**Integrationskonfiguration**
Externe Service-Integrationen (initial leer oder mit Platzhalter-Tokens):

```
integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}           # Can be set via environment variable
```

---

**TechDocs-Konfiguration**
Einstellungen für die Dokumentationsgenerierung:

```
techdocs:
  builder: 'local'                     # Build docs locally
  generator:
    runIn: 'docker'                    # Use Docker for doc generation
  publisher:
    type: 'local'                      # Store docs locally
```

---

**Authentifizierungskonfiguration**
Die Standardkonfiguration verwendet für die Entwicklung eine Gast-Authentifizierung:

```
auth:
  providers:
    guest: {}                          # Allow guest access without login
```

---

**Katalogkonfiguration**
Steuert die Entitäts-Erkennung und -Validierung:

```
catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
    - allow: [Component, System, API, Resource, Location]
  locations:
    - type: file
      target: ../../examples/entities.yaml
    - type: file
      target: ../../examples/template/template.yaml
      rules:
        - allow: [Template]
    - type: file
      target: ../../examples/org.yaml
      rules:
        - allow: [User, Group]
```

Katalog-Locations teilen Backstage mit, wo Entitätsdefinitionen gefunden werden können. Die Standardkonfiguration umfasst:

Beispiel-Entitäten zu Demonstrationszwecken
Beispiel-Templates für das Scaffolding
Eine Beispiel-Organisationsstruktur (Benutzer und Gruppen)

---

**Paketverwaltung**
Backstage verwendet Yarn Workspaces zur Verwaltung von Abhängigkeiten über mehrere Pakete hinweg.

---

**Root package.json**
Definiert gemeinsame Abhängigkeiten und Skripte:

```
{
  "name": "root",
  "private": true,
  "workspaces": {
    "packages": ["packages/*", "plugins/*"]
  },
  "scripts": {
    "dev": "concurrently \"yarn start\" \"yarn start:backend\"",
    "start": "yarn workspace app start",
    "start:backend": "yarn workspace backend start"
  }
}
```

---

**Vorteile von Workspaces**

Geteilte Abhängigkeiten: Gemeinsame Pakete werden nur einmal installiert
Konsistente Versionen: Gleiche Versionen über alle Pakete hinweg
Einfache Skripte: Befehle über alle Workspaces ausführen
Schnelle Builds: Es wird nur neu gebaut, was sich geändert hat

---

**Nächste Schritte**
Nachdem du nun die Projektstruktur und Konfiguration von Backstage verstehst:

✅ Vertraut mit der Monorepo-Architektur und dem packages-Verzeichnis

✅ Verständnis der Trennung von Frontend und Backend sowie ihrer Kommunikation

✅ Wissen, wie app-config.yaml die gesamte Backstage-Instanz steuert

✅ Bereit, mehr über Plugins, Builds und Entwicklungs-Workflows zu lernen

In der nächsten Lektion beschäftigen wir uns mit dem Plugin-System, dem Build-Prozess und damit, wie du Backstage an deine Anforderungen anpassen und erweitern kannst!

---

**Zentrale Punkte**

Backstage verwendet eine Monorepo-Struktur mit `packages/` für den Anwendungscode, `plugins/` für Erweiterungen und `app-config.yaml` für die Konfiguration
Die Frontend-App (`packages/app`) ist eine React-Anwendung mit Routing, Komponenten und Theme-Anpassungen
Der Backend-Service (`packages/backend`) ist ein Node.js-API-Server, der Datenverarbeitung und Integrationen übernimmt
`app-config.yaml` ist die zentrale Konfigurationsdrehscheibe für Anwendungsmetadaten, Backend-Einstellungen, Integrationen, TechDocs, Authentifizierung und Katalog
Die Katalogkonfiguration definiert, wo Backstage Entitätsdefinitionen über dateibasierte Locations entdeckt
Yarn Workspaces verwalten Abhängigkeiten über alle Pakete hinweg mit gemeinsamem Tooling und konsistenter Versionierung

# Weiteren link

https://backstage.io/docs/overview/architecture-overview/

https://backstage.io/docs/conf/


======================================================
# 2- Plugins, Build und Entwicklung

==================================================
**Plugins, Build und Entwicklung**
Aufbauend auf deinem Verständnis der Struktur und Konfiguration von Backstage schauen wir uns nun an, wie das Plugin-System funktioniert, wie Builds verwaltet werden und welche Best Practices für Entwicklung und Anpassung gelten.

---

## Plugin-System – Deep Dive

Alles in Backstage ist als Plugin umgesetzt, einschließlich der Kernfunktionen.

### Plugin-Typen

**Frontend-Plugins:**

* Stellen React-Komponenten und Seiten bereit
* Verarbeiten Benutzerinteraktionen
* Kommunizieren mit Backend-APIs

**Backend-Plugins:**

* Stellen REST-APIs bereit
* Übernehmen Datenverarbeitung
* Integrieren externe Systeme

---

### Plugin-Erkennung

Backstage erkennt Plugins automatisch in folgenden Verzeichnissen:

* `packages/app/src/plugins/` (Frontend)
* `packages/backend/src/plugins/` (Backend)
* `plugins/` (benutzerdefinierte Plugins)

---

## Datenbank und Storage

### Entwicklungsdatenbank

Standardmäßig verwendet Backstage SQLite für die Entwicklung:

```
backend:
  database:
    client: better-sqlite3
    connection: ':memory:'
```

Dies erzeugt eine In-Memory-Datenbank, die bei jedem Neustart zurückgesetzt wird.

---

### Datenbank-Migrationen

Schemaänderungen werden über Migrationen in `packages/backend/migrations/` verwaltet:

* Automatische Migration beim Start
* Versionskontrollierte Schemaänderungen
* Unterstützung für Rollbacks bei Downgrades

---

## Architektur des Katalogsystems

Der Softwarekatalog ist das Herzstück des Datenmodells von Backstage.

### Speicherung von Entitäten

Entitäten werden als YAML-Dateien gespeichert und vom Katalog verarbeitet:

* **Discovery**: Der Katalog findet Entitätsdateien
* **Processing**: Entitäten werden validiert und angereichert
* **Storage**: Speicherung in der Datenbank
* **API**: Bereitstellung über eine REST-API
* **UI**: Darstellung in Frontend-Komponenten

---

### Beziehungen zwischen Entitäten

Entitäten referenzieren sich gegenseitig über Metadaten:

```
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-service
spec:
  type: service
  owner: team-alpha
  system: payment-system
  dependsOn:
    - component:user-service
```

---

## Build und Deployment

### Development-Builds

Der Befehl `yarn dev`:

* Startet das Backend im Watch-Modus
* Startet das Frontend mit Hot Reloading
* Leitet API-Anfragen zwischen Frontend und Backend weiter

---

### Production-Builds

Der Befehl `yarn build`:

* Baut das Backend nach `packages/backend/dist/`
* Baut das Frontend nach `packages/app/dist/`
* Erstellt optimierte und minimierte Bundles

---

## Anpassungsmöglichkeiten

Ein gutes Verständnis der Struktur erleichtert gezielte Anpassungen.

### Frontend-Anpassung

* **Theme**: Farben, Schriftarten und Abstände in `App.tsx` anpassen
* **Layout**: Navigation und Seitenstruktur ändern
* **Components**: Eigene UI-Komponenten hinzufügen
* **Pages**: Neue Seiten und Routen erstellen

---

### Backend-Anpassung

* **Plugins**: Neue Backend-Plugins hinzufügen
* **APIs**: Eigene REST-Endpunkte erstellen
* **Processors**: Katalog-Prozessoren erweitern
* **Authentifizierung**: Auth-Provider konfigurieren

---

### Konfigurationsanpassung

* **Catalog Locations**: Neue Entitätsquellen hinzufügen
* **Plugin Settings**: Verhalten von Plugins konfigurieren
* **Integrationen**: Externe Tools anbinden
* **Berechtigungen**: Zugriffskontrolle einrichten

---

## Best Practices für die Entwicklung

### Dateiorganisation

* Zusammengehörige Komponenten beieinander halten
* Klare, aussagekräftige Dateinamen verwenden
* TypeScript-Namenskonventionen einhalten
* Business-Logik von der UI trennen

---

### Konfigurationsmanagement

* Umgebungsvariablen für Secrets verwenden
* Entwicklungs-Konfigurationen versionieren
* Separate Konfigurationen für verschiedene Umgebungen nutzen
* Konfigurationsoptionen dokumentieren

---

### Plugin-Entwicklung

* Bestehende Plugins als Vorlage nutzen
* Dem Plugin-Development-Guide folgen
* TypeScript für Typsicherheit einsetzen
* Tests für eigenen Code schreiben

---

## Häufige Dateimuster

### Frontend-Dateien

* `.tsx`: React-Komponenten mit JSX
* `.ts`: TypeScript-Hilfsfunktionen
* `.test.tsx`: Komponententests
* `index.ts`: Barrel-Exports

### Backend-Dateien

* `.ts`: TypeScript-Server-Code
* `.test.ts`: Unit-Tests
* `router.ts`: Express-Routen-Handler
* `service.ts`: Business-Logik

### Konfigurationsdateien

* `.yaml`: Konfigurationsdateien
* `.json`: Package-Manifeste
* `.env`: Umgebungsvariablen
* `.md`: Dokumentation

---

## Debugging und Troubleshooting

### Log-Quellen

* **Frontend**: Browser-Entwicklerkonsole
* **Backend**: Terminal, in dem `yarn dev` läuft
* **Datenbank**: Backend-Logs zeigen SQL-Abfragen
* **Build**: Webpack-Ausgaben in den Frontend-Logs

---

### Source Maps

Development-Builds enthalten Source Maps für das Debugging:

* Breakpoints im ursprünglichen TypeScript setzen
* Originale Dateinamen in Stacktraces sehen
* Debugging über die Browser-Developer-Tools

---

## Abhängigkeiten verstehen

### Kernabhängigkeiten

**Frontend:**

* `@backstage/core-components`: UI-Komponentenbibliothek
* `@backstage/core-app-api`: Frontend-Framework
* `react` und `react-router`: UI-Framework

**Backend:**

* `@backstage/backend-common`: Backend-Framework
* `@backstage/catalog-model`: Entitätsdefinitionen
* `express`: Webserver-Framework

---

### Plugin-Abhängigkeiten

Jedes Plugin deklariert seine eigenen Abhängigkeiten:

* Frontend-Plugins hängen von React-Komponenten ab
* Backend-Plugins hängen von Datenbank-Clients ab
* Manche Plugins besitzen sowohl Frontend- als auch Backend-Anteile

---

## Nächste Schritte

Nachdem du nun das Plugin-System, den Build-Prozess und die Entwicklungs-Workflows von Backstage verstehst:

✅ Verständnis dafür, wie Plugins die gesamte Backstage-Funktionalität ermöglichen

✅ Wissen, wie Builds für Entwicklung und Produktion erstellt werden

✅ Vertraut mit Anpassungsmöglichkeiten für Theme, Layout und Funktionalität

✅ Bereit, mit dem Softwarekatalog zu arbeiten

In der nächsten Lektion tauchen wir tief in den Softwarekatalog ein und lernen die sechs Entitätstypen kennen, mit denen Services, Teams und Dokumentation organisiert werden.

Das Verständnis dieser Entwicklungskonzepte macht dich deutlich effektiver darin, Backstage an die Anforderungen deiner Organisation anzupassen und zu erweitern.

---

## Zentrale Punkte

* Das Plugin-System treibt sämtliche Backstage-Funktionen an – mit Frontend-Plugins (React-Komponenten) und Backend-Plugins (REST-APIs)
* SQLite-In-Memory-Datenbank für die Entwicklung mit automatischen Migrationen und versionskontrollierten Schemaänderungen
* Der Katalog verarbeitet Entitäten über die Pipeline: Discovery → Validierung → Storage → API → UI
* Development-Builds nutzen Hot Reloading und Watch-Modus, Production-Builds erzeugen optimierte Bundles
* Anpassungsmöglichkeiten umfassen Frontend-Theme/Layout, Backend-Plugins/APIs und Konfigurationseinstellungen
* Best Practices: saubere Dateistruktur, Umgebungsvariablen für Secrets, TypeScript für Sicherheit, Tests für Qualität
* Debugging über Browser-Konsole (Frontend), Terminal-Logs (Backend) und Source Maps für Breakpoint-Debugging

---

## Zusätzliche Ressourcen

**Plugin-Entwicklung**
documentation:https://backstage.io/docs/plugins/

**Backstage Architecture Overview**
documentation:https://backstage.io/docs/overview/architecture-overview/

---
====================================================

# 3- Entitätstypen – Components, APIs und Systems

=====================================================

**Entitätstypen – Components, APIs und Systems**
Der Softwarekatalog ist das Fundament von Backstage – hier befinden sich all deine Services, APIs, Teams und Dokumentationen. Beginnen wir mit dem Verständnis der drei zentralen technischen Entitätstypen.

---

## Was ist der Softwarekatalog?

Der Softwarekatalog ist das Fundament von Backstage – ein zentrales Register, das Folgendes bereitstellt:

Single Source of Truth: Alle Services, APIs und Ressourcen an einem Ort
Ownership-Informationen: Klare Zuordnung, wem was gehört
Beziehungsabbildung: Wie Services miteinander verbunden sind und voneinander abhängen
Metadaten-Management: Konsistente Informationen zu jedem Service
Discovery-Oberfläche: Einfache Möglichkeit, Services zu finden und zu erkunden

Man kann ihn sich als einen „digitalen Zwilling“ deiner Softwareorganisation vorstellen – er bildet sowohl die technische Architektur als auch die dafür verantwortliche menschliche Struktur ab.

---

## Entitätstypen

Backstage organisiert alles als Entitäten mit spezifischen Typen. In dieser Lektion behandeln wir die drei zentralen technischen Entitätstypen, die deine Software und Systeme repräsentieren.

---

## 1. Components

**Zweck:** Einzelne Softwarebestandteile (Services, Websites, Bibliotheken)

Components repräsentieren die tatsächliche Software in deiner Organisation:

Services: Backend-APIs, Microservices, Webanwendungen
Websites: Frontend-Anwendungen, Dokumentationsseiten
Libraries: Gemeinsame Codepakete, SDKs, Frameworks

Jede Component sollte einen klaren Owner (User oder Group) haben und kann Teil größerer Systems sein.

**YAML-Struktur:**

```
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: user-auth-service
  title: User Authentication Service
  description: Centralized authentication and authorization service using OAuth2 and JWT
  tags:
    - platform
    - authentication
    - microservice
  annotations:
    github.com/project-slug: company/user-auth-service
spec:
  type: service
  lifecycle: production
  owner: platform-team
  system: identity-system
  dependsOn:
    - component:api-gateway
```

**Erklärung der wichtigsten Felder:**

spec.type: „service“ für Backend-APIs, „website“ für Frontends, „library“ für Pakete
spec.lifecycle: production, experimental, deprecated – zeigt den Reifegrad
spec.owner: Welches Team oder welche Person verantwortlich ist (Referenz auf Group oder User)
spec.system: Logische Gruppierung zusammengehöriger Components
spec.dependsOn: Von welchen anderen Components diese abhängt
annotations: Externe Verweise wie GitHub-Repositories oder Dokumentations-URLs

---

## 2. APIs

**Zweck:** Schnittstellen zwischen Components

API-Entitäten repräsentieren die Schnittstellen, die Components anderen Systemen bereitstellen:

REST-APIs: HTTP-Endpunkte mit definierten Request-/Response-Formaten
GraphQL-APIs: Abfragebasierte APIs mit Schemas
gRPC-Services: Hochperformante RPC-Schnittstellen
Event-Streams: Asynchrone Kommunikationsschnittstellen

APIs helfen Teams zu verstehen, welche Services verfügbar sind, wie sie integriert werden und welche Datenformate existieren.

**YAML-Struktur:**

```
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: user-auth-api
  title: User Authentication API
  description: REST API for user authentication, authorization, and profile management
  tags:
    - rest
    - authentication
    - oauth2
spec:
  type: openapi
  lifecycle: production
  owner: platform-team
  system: identity-system
  definition: |
    openapi: 3.0.0
    info:
      title: User Authentication API
      version: 1.0.0
      description: Handles user login, logout, token management, and profile data
    paths:
      /auth/login:
        post:
          summary: Authenticate user credentials
          description: Validates username/password and returns JWT token
      /auth/logout:
        post:
          summary: Invalidate user session
      /auth/profile:
        get:
          summary: Get current user profile information
      /auth/refresh:
        post:
          summary: Refresh expired authentication token
```

**Erklärung der wichtigsten Felder:**

spec.type: „openapi“ für REST-APIs, „graphql“ für GraphQL, „grpc“ für gRPC
spec.definition: Die eigentliche API-Spezifikation, eingebettet in YAML
Gleiche Ownership-Felder: APIs haben Owner und gehören – wie Components – zu Systems
Eingebettete Dokumentation: Die Definition dient als lebendige, auffindbare Dokumentation

---

## 3. Systems

**Zweck:** Sammlungen von Components, die gemeinsam funktionieren

Systems repräsentieren logische Gruppierungen zusammengehöriger Components, die gemeinsam eine fachliche Fähigkeit oder ein Feature bereitstellen. Sie helfen dabei, die Architektur nach Business-Domänen statt nur nach technischen Grenzen zu strukturieren.

**Warum Systems verwenden?**

Business-Ausrichtung: Abbildung der technischen Architektur auf fachliche Fähigkeiten
Team-Grenzen: Klare Ownership ganzer Funktionsbereiche
Abhängigkeitsvisualisierung: Überblick über alle Services einer Geschäftsfunktion
Onboarding: Unterstützung neuer Entwickler beim Verständnis fachlicher Zusammenhänge

**Beispiele:**

Payment System: payment-service, billing-service, invoice-service, receipt-generator
User Management: user-service, auth-service, profile-service, preferences-service
Content Platform: cms-service, media-service, search-service, recommendation-engine

**YAML-Struktur:**

```
apiVersion: backstage.io/v1alpha1
kind: System
metadata:
  name: payment-system
  title: Payment System
  description: Handles all payment processing and billing operations
  tags:
    - business-critical
    - financial
spec:
  owner: team-backend
  domain: e-commerce
```

**Erklärung der wichtigsten Felder:**

metadata.name: Eindeutiger Bezeichner des Systems (Kleinbuchstaben, Bindestriche)
metadata.description: Welche fachliche Fähigkeit dieses System bereitstellt
spec.owner: Welches Team das gesamte System verantwortet (meist eine Group)
spec.domain: Optionale übergeordnete fachliche Domäne
tags: Nützlich zur Kategorisierung nach Kritikalität oder Geschäftsbereich

**Wie Components mit Systems verknüpft werden:**
Components referenzieren ihr übergeordnetes System über das Feld `system`:

```
# payment-service component
spec:
  system: payment-system  # Links this component to the Payment System
```

Dadurch entsteht eine logische Hierarchie: **Domain → System → Component**

---

## Nächste Schritte

Nachdem du nun die drei zentralen technischen Entitätstypen (Components, APIs und Systems) verstanden hast, bist du bereit, die organisatorischen Entitätstypen (Users, Groups und Resources) kennenzulernen, mit denen Ownership und Infrastrukturabhängigkeiten verwaltet werden.

Diese technischen Entitäten bilden das Fundament deines Softwarekatalogs – sie repräsentieren den tatsächlichen Code und die Systeme, die deine Teams entwickeln und betreiben.

---

## Zentrale Punkte

Components repräsentieren einzelne Softwarebestandteile: Services, Websites und Libraries mit klarer Ownership
APIs definieren Schnittstellen zwischen Components mit eingebetteten Spezifikationen (OpenAPI, GraphQL, gRPC)
Systems gruppieren zusammengehörige Components nach fachlicher Fähigkeit statt rein technischer Grenzen
Jeder Entitätstyp besitzt eine spezifische YAML-Struktur mit metadata- und spec-Bereichen
Components sind mit Systems verknüpft und bilden eine Domain → System → Component-Hierarchie
Sauberes Entitätsmodellieren verbessert Discovery, Ownership-Tracking und das Architekturverständnis


====================================================

# 3- Entitätstypen – Users, Groups und Resources
In der vorherigen Lektion haben wir die zentralen technischen Entitätstypen behandelt. Jetzt schauen wir uns die organisatorischen Entitäten (Users und Groups) an, die Personen und Teams repräsentieren, sowie Resources, die Infrastrukturabhängigkeiten darstellen.

=====================================================

**Entitätstypen – Users, Groups und Resources**
In der vorherigen Lektion haben wir die zentralen technischen Entitätstypen behandelt. Jetzt schauen wir uns die organisatorischen Entitäten (Users und Groups) an, die Personen und Teams repräsentieren, sowie **Resources**, die Infrastrukturabhängigkeiten darstellen.

---

## Entitätstypen

## 4. Users

**Zweck:** Einzelne Entwickler und Stakeholder

Users repräsentieren einzelne Personen in deiner Organisation und erfüllen wichtige Funktionen:

Individual Ownership: Users können Owner von Components, APIs und Dokumentation sein
Teamzugehörigkeit: Users gehören über `memberOf`-Beziehungen zu Groups
Kontaktinformationen: Ermöglichen die Kontaktaufnahme mit der verantwortlichen Person
Profilverwaltung: Zeigt, wer woran in der Organisation arbeitet

**YAML-Struktur:**

```
apiVersion: backstage.io/v1alpha1
kind: User
metadata:
  name: alice-platform
  title: Alice Johnson
  description: Senior Platform Engineer specializing in developer tooling
spec:
  profile:
    displayName: Alice Johnson
    email: alice@company.com
    picture: https://avatars.dicebear.com/api/identicon/alice.svg
  memberOf: [platform-team]
```

**Erklärung der wichtigsten Felder:**

metadata.name: Eindeutiger Bezeichner (oft mit Teamnamen zur besseren Zuordnung)
spec.profile: Kontakt- und Anzeigeinformationen
spec.memberOf: Liste der Groups, denen dieser User angehört
picture: URL zum Profilbild

Users können mehreren Groups angehören, was besonders für teamübergreifende technische Leads oder Manager sinnvoll ist.

---

## 5. Groups

**Zweck:** Teams, Abteilungen oder organisatorische Einheiten

Groups repräsentieren Teams oder Organisationseinheiten und erfüllen mehrere wichtige Aufgaben:

Ownership: Groups können Owner von Components, APIs und anderen Ressourcen sein
Organisation: Sie helfen, die Unternehmenshierarchie abzubilden
Berechtigungen: Groups werden für Zugriffskontrolle verwendet
Discovery: Teams können leicht gefunden und kontaktiert werden

**YAML-Struktur:**

```
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  name: platform-team
  title: Platform Engineering Team
  description: Team responsible for developer experience and infrastructure
  tags:
    - platform
    - infrastructure
spec:
  type: team
  profile:
    displayName: Platform Team
    email: platform@company.com
    picture: https://avatars.dicebear.com/api/identicon/platform-team.svg
  children: []
```

**Erklärung der wichtigsten Felder:**

metadata.name: Eindeutiger Bezeichner (Kleinbuchstaben, Bindestriche), der in Referenzen verwendet wird
spec.type: „team“ für operative Teams, „organization“ für übergeordnete Gruppen
spec.profile: Kontaktinformationen und Anzeigedetails
spec.children: Liste von Unterteams zur Abbildung der Hierarchie

---

## 6. Resources

**Zweck:** Infrastruktur und externe Abhängigkeiten

Resources repräsentieren Infrastruktur und externe Abhängigkeiten, von denen Components abhängen, die jedoch selbst keine Code-Repositories sind. Man kann sich Resources als das „Supporting Cast“ vorstellen – Datenbanken, Queues, Storage und Drittanbieter-Services, die Anwendungen zum Betrieb benötigen.

**Was gilt als Resource?**

Datenbanken: PostgreSQL-Instanzen, MongoDB-Cluster, Redis-Caches
Messaging-Infrastruktur: RabbitMQ-Queues, Kafka-Topics, SQS-Queues
Storage: S3-Buckets, Azure Blob Storage, Dateisysteme
Externe Services: Drittanbieter-APIs (Stripe, SendGrid, Auth0)
Monitoring: Grafana-Dashboards, Datadog-Monitore, CloudWatch-Alarme
Infrastruktur: Kubernetes-Cluster, VPCs, Load Balancer

**Resources vs. Components – was ist der Unterschied?**

Components: Code, den du schreibst, wartest und deployest (Services, Libraries, Websites)
Resources: Infrastruktur, die du bereitstellst und konfigurierst (Datenbanken, Queues, Storage)

Faustregel: Liegt es in einem Git-Repository mit deinem Code → Component.
Ist es provisionierte Infrastruktur → Resource.

**Warum Resources katalogisieren?**

Abhängigkeitsverfolgung: Sehen, welche Services von welchen Datenbanken abhängen
Kostenmanagement: Infrastrukturkosten Systemen zuordnen
Zugriffskontrolle: Dokumentieren, wer Zugriff auf welche Infrastruktur hat
Disaster Recovery: Infrastrukturabhängigkeiten bei Incidents verstehen

**YAML-Struktur:**

```
apiVersion: backstage.io/v1alpha1
kind: Resource
metadata:
  name: user-database
  title: User Database
  description: PostgreSQL database for user data and authentication
  tags:
    - postgresql
    - production
    - pii-data
  annotations:
    cloud.google.com/instance: prod-users-db-001
    backup.schedule: "daily-3am"
spec:
  type: database
  owner: team-backend
  system: user-management
  dependsOn:
    - resource:user-database-replica
```

**Erklärung der wichtigsten Felder:**

spec.type: Typ der Resource – „database“, „queue“, „storage-bucket“, „api“, „dashboard“
spec.owner: Welches Team diese Resource verwaltet (Infrastruktur- oder Applikationsteam)
spec.system: Welches System diese Resource verwendet
spec.dependsOn: Andere Resources, von denen diese abhängt (Replikate, Backups, Monitoring)
annotations: Cloud-Provider-IDs, Backup-Zeitpläne, Connection-Strings

**Häufige Resource-Typen:**

database: PostgreSQL, MySQL, MongoDB, Redis
queue: RabbitMQ, Kafka, SQS, Azure Service Bus
storage-bucket: S3, GCS, Azure Blob
api: Externe Drittanbieter-APIs
dashboard: Grafana, Datadog, CloudWatch
cluster: Kubernetes-, Datenbank-Cluster

---

## Wie Components mit Resources verknüpft werden

Components deklarieren Abhängigkeiten zu Resources über `dependsOn`:

```
# user-service component
spec:
  dependsOn:
    - resource:user-database        # PostgreSQL database
    - resource:user-cache           # Redis cache
    - resource:email-queue          # Message queue
    - resource:s3-profile-pictures  # Storage bucket
```

Dadurch entsteht ein klares Bild der Infrastrukturabhängigkeiten jedes Services.

---

## Beziehungen zwischen Entitäten

Entitäten sind über Referenzen in ihren Metadaten miteinander verbunden.

### Häufige Beziehungstypen

**Ownership:** Wer ist für diese Entität verantwortlich

```
spec:
  owner: team-backend  # References a Group entity
```

**Systemzugehörigkeit:** Zu welchem System diese Component gehört

```
spec:
  system: payment-system  # References a System entity
```

**Abhängigkeiten:** Wovon diese Component abhängt

```
spec:
  dependsOn:
    - component:user-service    # References another Component
    - resource:payment-database # References a Resource
```

**API-Bereitstellung:** Welche APIs diese Component bereitstellt

```
spec:
  providesApis:
    - user-api  # References an API entity
```

**API-Nutzung:** Welche APIs diese Component verwendet

```
spec:
  consumesApis:
    - payment-api  # References an API entity
```

---

## Metadatenstruktur

Alle Entitäten teilen sich eine gemeinsame Metadatenstruktur.

**Pflichtfelder**

```
metadata:
  name: my-service        # Unique identifier (lowercase, hyphens)
  title: My Service       # Display name (human readable)
```

**Optional, aber empfohlen**

```
metadata:
  description: What this service does
  tags:                   # Searchable keywords
    - microservice
    - java
    - production
  annotations:            # Tool-specific metadata
    github.com/project-slug: myorg/my-service
    backstage.io/managed-by-location: file:///catalog/services.yaml
  labels:                 # Key-value pairs for filtering
    environment: production
    team: backend
```

---

## Nächste Schritte

Du verstehst nun alle sechs Entitätstypen in Backstage:

Technische Entitäten: Components, APIs, Systems
Organisatorische Entitäten: Users, Groups
Infrastruktur-Entitäten: Resources

Zusätzlich weißt du, wie sie über Beziehungen miteinander verknüpft sind und eine gemeinsame Metadatenstruktur nutzen.

In der nächsten Lektion schauen wir uns an, wie du deinen Katalog effektiv organisierst, Entitäten über verschiedene Discovery-Methoden registrierst und Best Practices für das Katalogmanagement anwendest.

---

## Zentrale Punkte

Users repräsentieren einzelne Entwickler mit Profilinformationen und Teamzugehörigkeiten
Groups repräsentieren Teams oder Organisationseinheiten mit Ownership-Funktion und Hierarchie-Unterstützung
Resources repräsentieren Infrastrukturabhängigkeiten wie Datenbanken, Queues, Storage und externe Services
Wichtige Unterscheidung: Components sind Code in Git-Repositories, Resources sind provisionierte Infrastruktur
Entitäten sind über Beziehungen verbunden: Ownership, Systemzugehörigkeit, Abhängigkeiten, API-Bereitstellung/-Nutzung
Alle Entitäten teilen eine gemeinsame Metadatenstruktur mit Pflichtfeldern (name, title) und optionalen Feldern (tags, annotations, labels)
Saubere Entitätsbeziehungen ermöglichen Abhängigkeitsverfolgung, Kostenmanagement und Disaster-Recovery-Planung

---

## Zusätzliche Ressourcen

**Backstage Software Catalog**
[documentation](https://backstage.io/docs/features/software-catalog/)

**Entity Relationships**
[documentation](https://backstage.io/docs/features/software-catalog/well-known-relations/)

---

====================================================

# Katalogorganisation und Discovery
Jetzt, da du alle sechs Entitätstypen kennst, schauen wir uns an, wie du deinen Katalog effektiv organisierst, Entitäten über verschiedene Discovery-Methoden registrierst und Best Practices für langfristigen Erfolg anwendest.

=====================================================

**Katalogorganisation und Discovery**
Jetzt, da du alle sechs Entitätstypen kennst, schauen wir uns an, wie du deinen Katalog effektiv organisierst, Entitäten über verschiedene Discovery-Methoden registrierst und Best Practices für langfristigen Erfolg anwendest.

---

## Organisation deines Katalogs

### Nach Teamstruktur

```
# Team-based organization
Groups:
  - team-frontend (Web team)
  - team-backend (API team)
  - team-platform (Infrastructure team)

Components:
  - web-app (owner: team-frontend)
  - user-service (owner: team-backend)
  - monitoring (owner: team-platform)
```

### Nach Business-Domäne

```
# Domain-based organization
Systems:
  - user-management
  - payment-processing
  - content-delivery

Components:
  - user-service (system: user-management)
  - payment-service (system: payment-processing)
  - cdn-service (system: content-delivery)
```

### Nach Technologie-Stack

```
# Technology-based tagging
Components:
  - frontend-app (tags: [react, typescript])
  - api-gateway (tags: [java, spring])
  - worker-service (tags: [python, celery])
```

---

## Registrieren von Entitäten im Katalog

Damit Entitäten in Backstage erscheinen, musst du festlegen, wo Backstage sie finden soll – über sogenannte **Catalog Locations**.

---

## Wie Backstage Entitäten entdeckt

Backstage nutzt Catalog Locations, um Entitäten zu finden:

* YAML-Dateien aus konfigurierten Locations lesen
* Entitätsdefinitionen parsen
* In die Katalogdatenbank importieren
* In der UI durchsuchbar und navigierbar machen

Man kann sich Catalog Locations als „Zeiger“ vorstellen, die Backstage sagen: „Suche hier nach Entitätsdefinitionen.“

---

## Dateibasierte Locations

Der häufigste Einstiegspunkt:

```
# app-config.yaml
catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
    - allow: [Component, System, API, Resource, Location, Group, User]
  locations:
    - type: file
      target: ../../catalog-entities/teams.yaml
    - type: file
      target: ../../catalog-entities/users.yaml
    - type: file
      target: ../../catalog-entities/components.yaml
    - type: file
      target: ../../catalog-entities/apis.yaml
```

**Erklärung der wichtigsten Konfiguration:**

* rules.allow: Welche Entitätstypen im Katalog erlaubt sind
* type: file: Gibt an, dass es sich um eine lokale Datei handelt
* target: Pfad zur Datei relativ zum Backstage-Projektverzeichnis

---

## Neustart zum Laden der Entitäten

Nach dem Ändern von Catalog Locations muss Backstage neu gestartet werden, um die neuen Entitäten zu laden:

```
# If using systemd service
systemctl restart backstage

# If using yarn dev
# Stop with Ctrl+C and run yarn dev again
```

---

## Erweiterte Discovery-Methoden

### Git-Repository-Integration

Verweist auf eine einzelne, spezifische Datei in einem Repository, die manuell konfiguriert wird. Funktioniert sofort ohne zusätzliche Einrichtung – ideal für kleine Teams oder zentral verwaltete Kataloge.

```
catalog:
  locations:
    - type: url
      target: https://github.com/myorg/backstage-catalog/blob/main/catalog.yaml
```

### GitHub Discovery (automatisch)

Durchsucht automatisch alle Repositories einer GitHub-Organisation mithilfe von Wildcards (z. B. `myorg/*/catalog-info.yaml`). Erfordert die Installation des Backend-Moduls `@backstage/plugin-catalog-backend-module-github`. Ideal für große Organisationen mit dezentraler Ownership.

```
catalog:
  locations:
    - type: github-discovery
      target: https://github.com/myorg/*/blob/main/catalog-info.yaml
```

---

## Katalognavigation

### Katalog-Startseite

Die Katalog-Startseite zeigt:

* Kürzlich angesehene Entitäten
* Entitäten, deren Owner der aktuelle User ist
* Favorisierte (markierte) Entitäten
* Suche und Filter zur gezielten Auswahl

### Entitätsseiten

Jede Entität besitzt eine eigene Seite mit Tabs:

* Overview: Basisinformationen und Metadaten
* Relations: Visueller Graph der Abhängigkeiten
* API: API-Dokumentation (für API-Entitäten)
* Docs: TechDocs-Dokumentation
* Dependencies: Welche Abhängigkeiten diese Entität nutzt
* Dependents: Welche Entitäten diese nutzen

### Suche und Filter

Entitäten finden über:

* Textsuche: Namen, Beschreibungen, Tags
* Typfilter: Nur Components, APIs usw. anzeigen
* Owner-Filter: Entitäten bestimmter Teams anzeigen
* Tag-Filter: Nach Technologie- oder Kategorietags filtern
* Label-Filter: Benutzerdefinierte Key-Value-Filter

---

## Best Practices

### Namenskonventionen

* Konsistente Benennung: kebab-case verwenden (`user-service`, nicht `UserService`)
* Aussagekräftige Namen: Zweck aus dem Namen erkennbar
* Abkürzungen vermeiden: `payment-service` statt `pay-svc`
* Umgebung angeben: `user-service-prod` vs. `user-service-dev`

### Metadatenqualität

* Vollständige Beschreibungen: Erklären, was der Service macht
* Korrekte Ownership: Owner-Informationen aktuell halten
* Relevante Tags: Einheitlich über ähnliche Services hinweg nutzen
* Nützliche Annotations: Links zu Repositories, Dashboards etc.

### Beziehungsmodellierung

* Abhängigkeiten dokumentieren: Alle relevanten Dependencies angeben
* Systems sinnvoll nutzen: Zusammengehörige Components gruppieren
* Beziehungen aktuell halten: Bei Architekturänderungen anpassen
* Zirkuläre Abhängigkeiten vermeiden: Saubere Trennung anstreben

### Organisationsstrategie

* Einfach starten: Zunächst nur Components und Groups erfassen
* Schrittweise wachsen: Systems und APIs später ergänzen
* Konsistent bleiben: Gleiche Muster teamübergreifend nutzen
* Regelmäßige Pflege: Entitätsinformationen überprüfen und aktualisieren

---

## Häufige Muster

### Microservices-Architektur

```
# System for business capability
System: user-management

# Components for individual services
Components:
  - user-service (REST API)
  - user-worker (background jobs)
  - user-admin (admin interface)

# APIs for external interfaces
APIs:
  - user-api (public REST API)
  - user-events (internal events)
```

### Frontend-/Backend-Trennung

```
# System for the application
System: e-commerce-platform

# Components for different layers
Components:
  - web-frontend (React app)
  - mobile-app (React Native)
  - api-gateway (backend API)
  - worker-service (background processing)
```

### Gemeinsame Services

```
# Platform services used by multiple systems
Components:
  - auth-service (system: platform-services)
  - logging-service (system: platform-services)
  - metrics-service (system: platform-services)

# Application services that depend on platform
Components:
  - user-service (dependsOn: [auth-service])
  - order-service (dependsOn: [auth-service, logging-service])
```

---

## Entitätsvalidierung

Backstage validiert Entitäten automatisch:

### Schema-Validierung

* Prüft, ob Pflichtfelder vorhanden sind
* Validiert Feldtypen und -formate
* Stellt sicher, dass Referenzen auf existierende Entitäten zeigen

### Beziehungsvalidierung

* Überprüft, ob referenzierte Entitäten existieren
* Erkennt zirkuläre Abhängigkeiten
* Validiert Ownership-Ketten

### Fehlerbehandlung

* Ungültige Entitäten werden mit Fehlermarkierungen angezeigt
* Validierungsfehler sind in den Entitätsdetails sichtbar
* Fehlende Referenzen werden in der UI hervorgehoben

---

## Nächste Schritte

Das Verständnis von Katalogorganisation und Discovery bildet die Grundlage dafür, dass du:

✅ Services und Teams logisch strukturierst
✅ Ownership und Beziehungen klar dokumentierst
✅ Deine Softwarearchitektur effektiv navigierst
✅ Services und APIs schnell findest

Im kommenden Lab erstellen wir eine Beispielorganisation mit allen Entitätstypen und sehen, wie sie in der Praxis zusammenspielen.

Der Katalog ist das Herz von Backstage – wenn du das richtig aufsetzt, wird alles andere deutlich einfacher.

---

## Zentrale Punkte

* Kataloge können nach Teamstruktur, Business-Domäne oder Technologie-Stack organisiert werden
* Catalog Locations definieren, wo Entitäten entdeckt werden: file-basiert lokal, url für einzelne Repos, github-discovery für automatisches Scannen
* File-basierte Locations funktionieren sofort, GitHub Discovery erfordert ein Backend-Modul
* Navigation über Katalog-Startseite, Entitätsseiten mit Tabs sowie leistungsfähige Such- und Filterfunktionen
* Best Practices: konsistente Namensgebung (kebab-case), vollständige Metadaten, korrekte Beziehungen, regelmäßige Pflege
* Häufige Muster: Microservices-Architektur, Frontend/Backend-Trennung, gemeinsame Plattform-Services
* Backstage validiert Entitäten automatisch mit Schema-, Beziehungs- und Fehlerbehandlung

---

## Zusätzliche Ressourcen

**Catalog Configuration**
[documentation](https://backstage.io/docs/features/software-catalog/configuration/)

**GitHub Discovery**
[documentation](https://backstage.io/docs/integrations/github/discovery/)

---














