

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









