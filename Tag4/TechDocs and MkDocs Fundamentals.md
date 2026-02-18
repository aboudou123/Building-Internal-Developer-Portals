

# Erweiterte Scaffolder-Muster

Nachdem du die grundlegenden Aktionen kennengelernt hast, werden wir nun fortgeschrittene Muster untersuchen, die Templates leistungsfähig und flexibel machen. Diese Lektion behandelt bedingte Ausführung, Step-Output-Ketten und die Erstellung kompletter Multi-Step-Workflows.

---

## Bedingte Aktionen

Aktionen werden nur ausgeführt, wenn bestimmte Bedingungen erfüllt sind, mithilfe der `if`-Eigenschaft:

### Basis-Beispiel

```yaml
- id: create-cicd
  if: '${{ parameters.enable_cicd }}'
  name: Setup CI/CD Pipeline
  action: fetch:template
  input:
    url: ./cicd-templates
    targetPath: .github/workflows
    values:
      name: '${{ parameters.name }}'
      owner: '${{ parameters.owner }}'
````

**Funktionsweise:**

* `if`: Aktion wird nur ausgeführt, wenn die Bedingung wahr ist
* Boolean-Parameter: Direkt prüfen `$ {{ parameters.enable_cicd }}`
* String-Vergleich: `$ {{ parameters.database_type === "postgresql" }}`
* Übersprungene Aktionen: Wenn Bedingung falsch, wird die Aktion komplett übersprungen

### Mehrfache Bedingungen

```yaml
# PostgreSQL Setup
- id: setup-postgresql
  if: '${{ parameters.database_type === "postgresql" }}'
  name: Setup PostgreSQL Configuration
  action: fetch:template
  input:
    url: ./database-templates/postgresql
    targetPath: ./database
    values:
      name: '${{ parameters.name }}'

# MongoDB Setup
- id: setup-mongodb
  if: '${{ parameters.database_type === "mongodb" }}'
  name: Setup MongoDB Configuration
  action: fetch:template
  input:
    url: ./database-templates/mongodb
    targetPath: ./database
    values:
      name: '${{ parameters.name }}'

# Swagger nur hinzufügen, wenn API-Dokumentation UND REST API
- id: add-swagger
  if: '${{ parameters.include_swagger && parameters.api_type === "rest" }}'
  name: Add Swagger Documentation
  action: fetch:template
  input:
    url: ./swagger-templates
    targetPath: ./docs
```

**Vorteile bedingter Muster:**

* Benutzerwahl: Entwickler wählen nur benötigte Features
* Technologievarianten: Verschiedene Datenbanken, Frameworks, Sprachen
* Optionale Komponenten: Monitoring, Sicherheit, Dokumentation bei Bedarf
* Sauberer Output: Generierte Projekte enthalten nur die gewählten Features

---

## Step Output Chaining

Aktionen können Outputs erzeugen, die nachfolgende Aktionen nutzen. So entstehen Workflows, bei denen jeder Schritt auf dem vorherigen aufbaut.

### Beispiel: Repository veröffentlichen und registrieren

```yaml
- id: publish
  name: Create GitHub Repository
  action: publish:github
  input:
    description: '${{ parameters.description }}'
    repoUrl: 'github.com?owner=${{ parameters.github_owner }}&repo=${{ parameters.repository_name }}'
    defaultBranch: main

- id: register
  name: Register Component
  action: catalog:register
  input:
    repoContentsUrl: '${{ steps.publish.output.repoContentsUrl }}'
    catalogInfoPath: '/catalog-info.yaml'
```

**Wichtige Output-Eigenschaften:**

* `steps.stepId.output.propertyName`: Verweis auf Outputs vorheriger Schritte
* `publish:github` typische Outputs: `remoteUrl`, `repoContentsUrl`, `cloneUrl`
* `catalog:register` typische Outputs: `catalogInfoUrl`, `entityRef`

### Vorteil von Output-Ketten

* Datenfluss: Jeder Schritt liefert Input für den nächsten
* Fehlerbehandlung: `optional: true` verhindert Abbruch bei Fehlern
* Debugging: `debug:log` für Problemlösung
* Benutzerfeedback: Zeigt Details zu erstellten Ressourcen

---

## Beispiel für ein vollständiges Template

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: nodejs-microservice-advanced
  title: Node.js Microservice (Production)
  description: Production-ready Node.js microservice with optional features
spec:
  owner: platform-team
  type: service

  parameters:
    - title: Service Information
      required: [name, description, owner]
      properties:
        name:
          title: Service Name
          type: string
          pattern: '^[a-z0-9-]+$'
        description:
          title: Description
          type: string
        owner:
          title: Team Owner
          type: string
    - title: Technology Choices
      properties:
        database_type:
          title: Database
          type: string
          enum: ['none', 'postgresql', 'mongodb']
          default: 'none'
        include_swagger:
          title: Add API Documentation
          type: boolean
          default: true
        enable_cicd:
          title: Setup CI/CD Pipeline
          type: boolean
          default: true
    - title: Repository Configuration
      required: [github_owner, repository_name]
      properties:
        github_owner:
          title: GitHub Organization
          type: string
        repository_name:
          title: Repository Name
          type: string

  steps:
    - id: fetch-base
      name: Generate Base Project
      action: fetch:template
      input:
        url: ./skeleton
        values:
          name: '${{ parameters.name }}'
          description: '${{ parameters.description }}'
          owner: '${{ parameters.owner }}'
          include_swagger: '${{ parameters.include_swagger }}'

    - id: add-postgresql
      if: '${{ parameters.database_type === "postgresql" }}'
      name: Add PostgreSQL Configuration
      action: fetch:template
      input:
        url: ./database-templates/postgresql
        targetPath: ./database

    - id: add-mongodb
      if: '${{ parameters.database_type === "mongodb" }}'
      name: Add MongoDB Configuration
      action: fetch:template
      input:
        url: ./database-templates/mongodb
        targetPath: ./database

    - id: add-cicd
      if: '${{ parameters.enable_cicd }}'
      name: Setup GitHub Actions
      action: fetch:template
      input:
        url: ./cicd-templates
        targetPath: .github/workflows
        values:
          name: '${{ parameters.name }}'
          has_database: '${{ parameters.database_type !== "none" }}'

    - id: publish
      name: Publish to GitHub
      action: publish:github
      input:
        description: '${{ parameters.description }}'
        repoUrl: 'github.com?owner=${{ parameters.github_owner }}&repo=${{ parameters.repository_name }}'
        defaultBranch: main

    - id: register
      name: Register in Catalog
      action: catalog:register
      optional: true
      input:
        repoContentsUrl: '${{ steps.publish.output.repoContentsUrl }}'
        catalogInfoPath: '/catalog-info.yaml'

  output:
    links:
      - title: Repository
        url: '${{ steps.publish.output.remoteUrl }}'
      - title: Catalog
        url: '${{ steps.register.output.catalogInfoUrl }}'
      - title: CI/CD Pipeline
        url: '${{ steps.publish.output.remoteUrl }}/actions'
        if: '${{ parameters.enable_cicd }}'
```

---

## Custom Actions

* Ermöglichen spezifische Integrationen (Monitoring, Infrastruktur, Sicherheit)
* Struktur:

  * `id`: Eindeutiger Name, z. B. `custom:datadog:dashboard`
  * `description`: Zweck der Aktion
  * `schema.input`: Eingabeparameter
  * `handler`: Async-Funktion mit Logik
  * `ctx.output()`: Übergabe von Outputs an nachfolgende Schritte
  * `ctx.logger`: Logging für Debugging

### Vorteile

* Organisation-spezifisch
* Automatisierung von wiederkehrenden Aufgaben
* Konsistente Konfiguration
* Wiederverwendbar in mehreren Templates

---

## Best Practices

**Action Design:**

* Atomic: Jede Aktion macht genau eine Aufgabe
* Conditionally: Bedingte Logik gezielt einsetzen
* Output chaining: Sauber zwischen Steps weitergeben
* Error handling: `optional: true` für nicht-kritische Aktionen

**Template Organisation:**

* Trenne Skeleton, Templates und Datenbank-Templates
* Parameter dokumentieren
* Conditional Pfade testen
* Debug-Ausgaben bereitstellen

**User Experience:**

* Sinnvolle Standardwerte
* Schrittweise Offenlegung (progressive disclosure)
* Klare Ausgaben für Nutzer
* Fehlerhinweise verständlich machen

---

## Nächste Schritte

* Bedingte Aktionen einsetzen
* Step-Output-Ketten nutzen
* Multi-Step-Produktions-Templates bauen
* Custom Actions entwickeln
* Best Practices umsetzen für wartbare Templates

**Schlüsselgedanken:**

* Conditional Actions ermöglichen flexible Templates
* Step Outputs verbinden Aktionen zu Workflows
* Vollständige Pipelines kombinieren fetch, conditional, publish, register
* Custom Actions für org-spezifische Integrationen
* Production Templates balancieren Flexibilität und Einfachheit

```

---

```

========================================

# TechDocs-Integration und Publishing

=========================================


## TechDocs Plugin-Architektur

TechDocs benötigt **zwei separate Plugins**, die zusammenarbeiten:

### Frontend-Plugin

**Zweck:** Stellt UI-Komponenten bereit, um Dokumentation in der Backstage-Oberfläche anzuzeigen.

**Installation:**

```bash
yarn --cwd packages/app add @backstage/plugin-techdocs
````

**Funktion:**

* Rendert Dokumentationsseiten in der Backstage UI
* Bietet die TechDocs-Indexseite (listet alle Dokumentationen)
* Bietet die TechDocs-Reader-Seite (zeigt einzelne Dokumente)
* Integriert Dokumentationstabs in Entity-Seiten

### Backend-Plugin

**Zweck:** Verantwortlich für serverseitige Dokumentationsverarbeitung und Bereitstellung.

**Installation:**

```bash
yarn --cwd packages/backend add @backstage/plugin-techdocs-backend
```

**Funktion:**

* Generiert Dokumentation aus MkDocs-Quellen
* Stellt die erstellte Dokumentation für das Frontend bereit
* Verwaltet Dokumentationsspeicher (lokal oder Cloud)
* Verarbeitet Builds on-demand oder vorab erstellt

**Backend-Registrierung:**
Im neuen Backend-System wird das Backend-Plugin automatisch registriert:

```ts
// packages/backend/src/index.ts
backend.add(import('@backstage/plugin-techdocs-backend'));
```

---

## TechDocs-Konfiguration

### App-Config Setup (`app-config.yaml`)

Nach Installation beider Plugins konfigurierst du das Verhalten von TechDocs:

```yaml
techdocs:
  builder: 'local'
  generator:
    runIn: 'local'
    mkdocsCommand: '/opt/mkdocs-venv/bin/mkdocs'  # oder einfach 'mkdocs'
  publisher:
    type: 'local'
    local:
      storageDir: '/root/labs/developer-portal/techdocs-storage'
```

**Erklärung:**

* `builder: 'local'` → Docs lokal bauen (statt externer Builder-Service)
* `generator.runIn: 'local'` → MkDocs lokal ausführen (statt Docker)
* `generator.mkdocsCommand` → Pfad zur MkDocs-Ausführungsdatei
* `publisher.type: 'local'` → Speichert Docs lokal (statt S3/GCS)
* `publisher.local.storageDir` → Speicherort der generierten HTML-Dateien

---

### Frontend-Routen (App.tsx)

Nach Installation des Frontend-Plugins füge Routen für Dokumentationsanzeige hinzu:

```tsx
import {
  TechDocsIndexPage,
  TechDocsReaderPage,
} from '@backstage/plugin-techdocs';

const routes = (
  <FlatRoutes>
    {/* ... andere Routen ... */}
    <Route path="/docs" element={<TechDocsIndexPage />} />
    <Route
      path="/docs/:namespace/:kind/:name/*"
      element={<TechDocsReaderPage />}
    />
  </FlatRoutes>
);
```

**Routen-Erklärung:**

* `/docs`: Indexseite mit allen Dokumentationen
* `/docs/:namespace/:kind/:name/*`: Anzeige einzelner Entity-Dokumentationen

---

## Backstage-Integration

### Catalog Entity Konfiguration

Verknüpfe Dokumentation mit Catalog Entities via `backstage.io/techdocs-ref` Annotation:

```yaml
# catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: sample-service
  title: Sample Microservice
  description: Ein Beispiel-Microservice für TechDocs-Tests
  annotations:
    backstage.io/techdocs-ref: dir:.
spec:
  type: service
  owner: platform-team
  lifecycle: experimental
```

**Annotation Optionen:**

* `dir:.` → Suche nach `mkdocs.yml` und `docs/` im Repository-Root
* `dir:./docs` → Suche in Unterverzeichnis `docs/`
* `dir:./subdirectory` → Suche in beliebigem Unterverzeichnis
* `url:https://github.com/org/repo` → Referenz zu externem Repository

---

### Entity im Catalog registrieren

Füge folgendes zu `app-config.yaml` hinzu:

```yaml
catalog:
  locations:
    - type: file
      target: /root/sample-repos/sample-service/catalog-info.yaml
```

Nach Registrierung:

* Service erscheint im Backstage Catalog
* Dokumentations-Tab verfügbar auf Entity-Seite
* Docs erreichbar unter `/docs/default/component/sample-service`
* Dokumentation automatisch für Suche indexiert

---

## Dokumentation bauen und veröffentlichen

### Lokaler Entwicklungs-Workflow

```bash
# ins Service-Verzeichnis wechseln
cd /root/sample-repos/sample-service

# Docs bauen
mkdocs build

# Output in site/ Verzeichnis
ls -la site/
# index.html, getting-started.html, api-reference.html, deployment.html

# Manuelles Publizieren in Backstage Storage (nur Entwicklung)
cp -r site/* /root/labs/developer-portal/techdocs-storage/default/component/sample-service/
```

**Build-Prozess:**

* MkDocs liest `mkdocs.yml`
* Markdown-Dateien im `docs/` Verzeichnis werden verarbeitet
* Generiert statische HTML-Dateien in `site/`
* Material-Theme Styling angewendet
* Navigationsstruktur erstellt
* Interne Links validiert

---

## Publishing-Workflows

### Lokaler Speicher (Entwicklung)

```yaml
techdocs:
  publisher:
    type: 'local'
    local:
      storageDir: '/path/to/techdocs-storage'
```

**Vorteile:**

* Einfache Einrichtung, keine Cloud-Abhängigkeiten
* Schnelle Iteration
* Keine Zusatzkosten

**Nachteile:**

* Nicht produktionsfähig
* Keine Skalierung/Redundanz
* Manuelles File-Management nötig

### Cloud-Speicher (Produktion)

**AWS S3**

```yaml
techdocs:
  publisher:
    type: 'awsS3'
    awsS3:
      bucketName: 'techdocs-storage'
      region: 'us-east-1'
      credentials:
        accessKeyId: ${AWS_ACCESS_KEY_ID}
        secretAccessKey: ${AWS_SECRET_ACCESS_KEY}
```

**Google Cloud Storage**

```yaml
techdocs:
  publisher:
    type: 'googleGcs'
    googleGcs:
      bucketName: 'techdocs-storage'
      projectId: 'my-project'
```

**Azure Blob Storage**

```yaml
techdocs:
  publisher:
    type: 'azureBlobStorage'
    azureBlobStorage:
      containerName: 'techdocs'
      accountName: 'mystorageaccount'
```

**Vorteile von Cloud-Speicher:**

* Skalierbarkeit: Beliebige Menge an Dokumentation
* Zuverlässigkeit: Provider-Redundanz und Backups
* Performance: CDN-Integration für schnellen globalen Zugriff
* Kosten: Nur für tatsächlich genutzten Speicher

---

## Automatisiertes Publishing

### CI/CD Integration (GitHub Actions)

```yaml
# .github/workflows/techdocs.yml
name: Publish TechDocs

on:
  push:
    branches: [main]
    paths:
      - 'docs/**'
      - 'mkdocs.yml'

jobs:
  publish-techdocs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install MkDocs
        run: pip install mkdocs-techdocs-core

      - name: Build Documentation
        run: mkdocs build

      - name: Publish to S3
        uses: jakejarvis/s3-sync-action@master
        with:
          args: --delete
        env:
          AWS_S3_BUCKET: ${{ secrets.TECHDOCS_S3_BUCKET }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          SOURCE_DIR: 'site'
          DEST_DIR: 'default/component/sample-service'
```

**Automatisierungsvorteile:**

* Immer aktuell: Docs rebuilden automatisch bei Änderungen
* Keine manuellen Schritte: Entwickler committen nur
* Versionskontrolle: History in Git nachvollziehbar
* Quality Gates: Validierung, Linting möglich

---

## Publishing-Strategien

**Push-basiert (empfohlen):**

* CI/CD baut und veröffentlicht bei Git Push
* Schnelle Updates
* Funktioniert mit jedem Storage-Backend

**Pull-basiert (on-demand):**

* Backstage baut Docs beim ersten Zugriff
* Praktisch für große Organisationen
* Reduziert CI/CD-Overhead

---

## Erweiterte Features

### Custom Plugins

* MkDocs-Plugins erweitern TechDocs:

```yaml
# mkdocs.yml
plugins:
  - techdocs-core
  - mermaid2  # Diagramme
  - swagger-ui-tag  # Interaktive API-Dokumentation
  - search  # Volltextsuche
```

**Beliebte Plugins:**

* `mermaid2`: Flussdiagramme und Diagramme
* `swagger-ui-tag`: OpenAPI/Swagger Einbindung
* `drawio-exporter`: Diagramme aus draw.io
* `git-revision-date`: Letztes Update anzeigen
* `minify`: HTML/CSS/JS komprimieren

### Mehrsprachigkeit

```yaml
plugins:
  - techdocs-core
  - i18n:
      default_language: en
      languages:
        en: English
        es: Español
        fr: Français
```

### Suche

* Volltextsuche über alle Docs
* Facettierte Suche (Entity-Typ, Owner, Tags)
* Ranking nach Relevanz
* Kontextuelles Snippet-Highlighting

---

## Vorteile der TechDocs-Integration

**Für Entwickler:**

* Bekannter Workflow (Docs like Code)
* Versionskontrolle
* Einfach zu finden (Catalog integriert)
* Immer aktuell durch automatisiertes Publishing

**Für Organisationen:**

* Konsistentes Format über alle Services
* Weniger Wartung durch Automation
* Bessere Auffindbarkeit
* Einfacheres Onboarding

**Für Plattform-Teams:**

* Standardisierung durch Vorgaben
* Automatisierung reduziert manuelles Publishing
* Sichtbarkeit von Dokumentationsabdeckung
* Skalierbar für Tausende Services

---

## Häufige Probleme & Troubleshooting

**Dokumentation wird nicht angezeigt:**

* `backstage.io/techdocs-ref` Annotation fehlt
* `mkdocs.yml` am falschen Ort
* `docs/index.md` fehlt
* TechDocs Storage-Verzeichnis falsch konfiguriert
* Entity nicht im Catalog registriert

**Build-Fehler:**

* Fehlende `index.md`
* Ungültiges YAML in `mkdocs.yml`
* Defekte interne Links
* Fehlende MkDocs-Plugins
* Falsche Dateipfade

**Publishing-Fehler:**

* Storage Credentials prüfen
* Bucket/Container existiert
* Schreibrechte vorhanden
* Netzwerkzugriff zum Storage-Backend

---

## Nächste Schritte

* Frontend- & Backend-Plugins installieren
* TechDocs Builder, Generator, Publisher konfigurieren
* Frontend-Routen hinzufügen
* Dokumentation an Catalog Entities verlinken
* CI/CD Automatisierung einrichten
* Deployment in Produktions-Storage

---

## Schlüsselgedanken

* Zwei Plugins nötig: Frontend (UI) & Backend (Docs-Generierung)
* Frontend: Index- & Reader-Seiten, Backend: MkDocs-Verarbeitung & Storage
* Backend wird automatisch registriert
* `app-config.yaml` steuert Builder, Generator, Publisher
* Entities verbinden sich via `backstage.io/techdocs-ref` Annotation
* Lokaler Speicher für Entwicklung, Cloud-Speicher für Produktion
* CI/CD hält Dokumentation immer aktuell

---

## Zusätzliche Ressourcen

* [TechDocs Documentation](documentation)
* [TechDocs Configuration](documentation)
* [TechDocs CI/CD](documentation)

```

---


```

