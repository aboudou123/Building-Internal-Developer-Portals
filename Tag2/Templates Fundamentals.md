
===============================================
# 1- Backstage Software Templates: Fundamentals
===============================================

### Problem ohne Templates

Manuelles Erstellen neuer Projekte führt zu:

* **Inkonsistente Struktur**: Jeder Entwickler richtet Projekte anders ein
* **Fehlende Standards**: Linting, Tests, Sicherheitskonfigurationen werden oft vergessen
* **Langsames Onboarding**: Neue Projekte benötigen Stunden oder Tage
* **Dokumentationslücken**: Setup-Anleitungen veraltet oder unvollständig
* **Tool-Fragmentierung**: Unterschiedliche CI/CD, Monitoring, Deployment

**Produktivität:**

* Ohne Templates: 2–5 Tage pro Projekt
* Mit Templates: 15–30 Minuten pro Projekt

---

### Was sind Software Templates?

Templates sind **projekt-generierende Tools**, die:

* **Code generieren**: Vollständige Projektstruktur aus Vorlagen
* **Standards durchsetzen**: Einheitliche Patterns für alle Projekte
* **Tools integrieren**: CI/CD, Monitoring, Deployment automatisch einrichten
* **Ressourcen erstellen**: Repos generieren, Services konfigurieren, Katalog aktualisieren
* **Guidance bereitstellen**: Dokumentation und Best Practices einbinden

**Kurz:** „Project generators“ für production-ready Projekte.

---

### Backstage Scaffolder

**Komponenten:**

| Komponente           | Funktion                                        |
| -------------------- | ----------------------------------------------- |
| Template Definitions | YAML-Dateien, die Projektstruktur beschreiben   |
| Action System        | Pluggable Aktionen für verschiedene Operationen |
| Form Builder         | Dynamische Formulare für Template-Parameter     |
| Execution Engine     | Führt die Template-Schritte aus                 |
| Integration Layer    | Verbindung zu Git, CI/CD, etc.                  |

**Workflow:**

1. **Discover** – Entwickler findet Template im Catalog
2. **Configure** – Formular ausfüllen (Projektname, Team, Repo…)
3. **Generate** – Scaffolder führt Aktionen Schritt für Schritt aus
4. **Integrate** – Repository erstellen, CI/CD einrichten, Katalog aktualisieren
5. **Handoff** – Entwickler erhält sofort einsatzbereites Projekt

---

### Template YAML Struktur

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: nodejs-service-template
  title: Node.js Microservice Template
  description: Create a new Node.js microservice with Express and TypeScript
  tags: [recommended, nodejs, typescript, microservice]
spec:
  owner: platform-team
  type: service
  parameters: # Form fields
  steps:      # Scaffolding actions
  output:     # Links/Messages nach Fertigstellung
```

**Hauptabschnitte:**

* `apiVersion` – immer `scaffolder.backstage.io/v1beta3`
* `kind` – muss `Template` sein
* `metadata` – Entdeckungsinformationen für Catalog
* `spec` – enthält **owner**, **parameters**, **steps**, **output**

---

### Parameter (Form Definition)

```yaml
parameters:
  - title: Project Information
    required: [name, description, owner]
    properties:
      name:
        title: Service Name
        type: string
        pattern: '^[a-z0-9-]+$'
        minLength: 3
        maxLength: 50
        ui:autofocus: true
      description:
        title: Service Description
        type: string
        minLength: 10
        maxLength: 200
        ui:widget: textarea
      owner:
        title: Team Owner
        type: string
        enum: [platform-team, backend-team, frontend-team]
```

* **Multi-page Forms**: Jede Top-Level-Parametergruppe = Form-Seite
* **Validierung**: required, pattern, enum, min/maxLength
* **UI-Hilfen**: autofocus, help, widget

---

### Template Steps (Actions)

```yaml
steps:
  - id: fetch
    name: Fetch Skeleton Files
    action: fetch:template
    input:
      url: ./skeleton
      values:
        name: '${{ parameters.name }}'
        owner: '${{ parameters.owner }}'

  - id: publish
    name: Create GitHub Repository
    action: publish:github
    input:
      description: '${{ parameters.description }}'
      repoUrl: 'github.com?owner=${{ parameters.github_owner }}&repo=${{ parameters.repository_name }}'

  - id: register
    name: Register Service in Backstage Catalog
    action: catalog:register
    optional: true
    input:
      repoContentsUrl: '${{ steps.publish.output.repoContentsUrl }}'
      catalogInfoPath: '/catalog-info.yaml'
```

* `id` – eindeutige Step-ID
* `name` – angezeigter Step-Name
* `action` – welche Aktion ausgeführt wird
* `input` – Parameter für die Aktion
* `optional` – wenn true, bricht Fehler den Template-Lauf nicht ab
* **Outputs zwischen Steps:** `${{ steps.stepId.output.propertyName }}`

---

### Warum Templates wichtig sind

✅ Reduzieren Projektsetup von Tagen auf Minuten

✅ Stellen Konsistenz und Standards sicher

✅ Automatisieren CI/CD, Katalogintegration und Dokumentation

✅ Erleichtern Onboarding und produktive Entwicklung

---
## Zusätzliche Ressourcen

Backstage Software Templates
[documentation](https://backstage.io/docs/features/software-templates/)

Template Writing Guide
[documentation](https://backstage.io/docs/features/software-templates/writing-templates/)




===========================================
 # 2- Template-Aktionen und Skeleton-Inhalte
===========================================

---

## Template Actions & Skeleton Content

### 1. Template Actions – die Bausteine

Backstage Actions führen die Logik eines Templates aus.

**Häufig genutzte eingebaute Actions:**

| Kategorie           | Action             | Zweck                                                                               |
| ------------------- | ------------------ | ----------------------------------------------------------------------------------- |
| File Operations     | `fetch:template`   | Kopiert und verarbeitet Template-Dateien aus Skeleton                               |
|                     | `fs:rename`        | Dateien nach Generation umbenennen                                                  |
| Git Integration     | `publish:github`   | GitHub Repository erstellen                                                         |
|                     | `publish:gitlab`   | GitLab Repository erstellen                                                         |
| Catalog Integration | `catalog:register` | Entity im Backstage Catalog registrieren                                            |
| Custom Actions      | z.B.               | Kubernetes-Manifest erstellen, CI/CD konfigurieren, Cloud Ressourcen provisionieren |

**Merke:** Actions können **optional** sein, d.h. Fehler brechen den Template-Lauf nicht ab (`optional: true`).

---

### 2. Skeleton Content – die Projektvorlage

* Templates leben in einem `skeleton/`-Verzeichnis **neben** der `template.yaml`.
* Beinhaltet **alle Dateien**, die im generierten Projekt entstehen sollen.
* Dateien werden verarbeitet: **Variablen ersetzt, Struktur erhalten, Bedingungskontrolle**.

**Beispielstruktur:**

```
nodejs-service-template/
├── template.yaml
├── skeleton/
│   ├── package.json
│   ├── tsconfig.json
│   ├── Dockerfile
│   ├── .gitignore
│   ├── .env.example
│   ├── README.md
│   ├── catalog-info.yaml
│   └── src/
│       ├── index.ts
│       └── routes/
│           ├── health.ts
│           └── api.ts
├── cicd-templates/
│   └── ci-pipeline.yml
└── docs/
    └── README.md
```

---

### 3. Nunjucks Templating – dynamische Inhalte

Backstage verwendet **Nunjucks** für Variable Substitution, Conditionals, Loops und Filter.

**Variable Substitution:**

```json
{
  "name": "${{ values.name }}",
  "description": "${{ values.description }}",
  "author": "${{ values.owner }}"
}
```

**Conditional Content:**

```ts
{% if parameters.include_healthcheck %}
app.use('/health', healthRoutes);
{% endif %}
```

**Loops:**

```ts
{% for route in parameters.routes %}
app.use('{{ route.path }}', {{ route.handler }});
{% endfor %}
```

**Filters:**

```yaml
metadata:
  name: ${{ values.name | replace("-", "_") }}
  displayName: ${{ values.name | capitalize }}
```

* Nützliche Filter: `replace`, `upper`, `lower`, `capitalize`
* Kombinierbar und für beliebige Dateitypen nutzbar (JSON, YAML, TS, etc.)

---

### 4. Template Output Section

Definiert, was nach erfolgreicher Generierung angezeigt wird:

```yaml
output:
  links:
    - title: View Repository
      url: '${{ steps.publish.output.remoteUrl }}'
      icon: github
    - title: View in Backstage Catalog
      url: '${{ steps.register.output.catalogInfoUrl }}'
      icon: catalog
  text:
    - title: Successfully Created ${{ parameters.name | title }}
      content: |
        Your new Node.js microservice has been generated!
        **Next Steps:**
        1. Clone repository
        2. Install dependencies: npm install
        3. Start development: npm run dev
```

* **Links:** Repository, Catalog, CI/CD, Docs
* **Text:** Markdown-fähige Anweisungen
* **Dynamische Inhalte:** Parameter und Step-Outputs

---

### 5. Best Practices

* Ketten von Actions: Ausgabe eines Steps kann Input für den nächsten Step sein (`${{ steps.stepId.output.prop }}`)
* Verwende Conditionals für optionale Features (Swagger, Healthchecks, DB)
* Halte Skeleton sauber und dokumentiert
* Teste Template lokal mit `npx @backstage/cli create-template`

---

### Zusammenfassung – Key Points

* **Actions:** `fetch:template`, `publish:github/gitlab`, `catalog:register`
* **Skeleton:** Vorlage für alle Projektdateien, wird 1:1 kopiert und verarbeitet
* **Nunjucks:** Variablen, Conditionals, Loops, Filters
* **Output Section:** Benutzerfeedback nach Template-Generierung
* **Chaining:** Steps können Outputs an nachfolgende Steps weitergeben

---

 **Ein anschauliches Diagramm vom gesamten Template-Workflow** – von **Parameters → Skeleton → Actions → Output**, inklusive Nunjucks-Processing. 

```
+---------------------+
|  Parameters (Form)  |
|  - name             |
|  - description      |
|  - owner            |
|  - repo info        |
+---------------------+
          |
          v
+---------------------+
|  Skeleton Directory |
|  - template files   |
|  - catalog-info.yaml|
|  - source code      |
+---------------------+
          |
          v
+---------------------+
|  Nunjucks Templating|
|  - Variable substitution: ${{ values.name }} |
|  - Conditionals: {% if ... %} |
|  - Loops & filters |
+---------------------+
          |
          v
+---------------------+
|      Actions        |
|  - fetch:template   |
|  - fs:rename        |
|  - publish:github   |
|  - catalog:register |
|  - Custom actions   |
+---------------------+
          |
          v
+---------------------+
|   Output Section    |
|  - Links (Repo, CI) |
|  - Instructions     |
|  - Dynamic info     |
+---------------------+
```

# Additional Resources
Scaffolder Actions
[documentation](https://backstage.io/docs/features/software-templates/builtin-actions/)

Nunjucks Templating
[documentation](https://mozilla.github.io/nunjucks/templating.html)

Writing Custom Actions
[documentation](https://backstage.io/docs/features/software-templates/writing-custom-actions/)




==============================================
# 3- Template Integration and Best Practices
==============================================
````md
# Template-Integration und Best Practices

Du hast die Template-Struktur und die Erstellung von Inhalten kennengelernt. Jetzt schauen wir uns an, wie Templates in deine Infrastruktur integriert werden, wie generierte Services automatisch entdeckt werden und welche Best Practices für produktionsreife Templates gelten.

---

## Golden-Path-Templates

Golden-Path-Templates repräsentieren den von deiner Organisation **„empfohlenen“** Weg zur Erstellung von Services.

### Merkmale guter Golden Paths

- **Vollständiges Setup**: Alles, was für den Produktivbetrieb erforderlich ist  
- **Sicherheit standardmäßig**: Security-Scanning und Secrets-Management integriert  
- **Observability**: Logging, Metriken und Tracing konfiguriert  
- **CI/CD-Integration**: Automatisiertes Testen und Deployen  
- **Dokumentation**: README, API-Dokumentation und Runbooks enthalten  

### Beispielhafte Golden-Path-Komponenten

```text
# Node.js Golden Path includes:
- TypeScript configuration
- ESLint and Prettier
- Jest testing framework
- Docker containerization
- GitHub Actions CI/CD
- Prometheus metrics
- OpenAPI documentation
- Security scanning
- Dependency updates
````

### Warum Golden Paths wichtig sind

* Reduzieren Entscheidungsüberlastung für Entwickler
* Stellen sicher, dass alle Services Organisationsstandards erfüllen
* Integrieren Sicherheit und Observability von Anfang an
* Beschleunigen das Onboarding neuer Teammitglieder
* Machen Wartung und Updates skalierbar

---

## Template-Organisation

### Nach Technologie-Stack

```text
templates/
├── nodejs-service/
├── python-service/
├── react-frontend/
└── golang-service/
```

### Nach Zweck

```text
templates/
├── microservice/
├── frontend-app/
├── library/
└── infrastructure/
```

### Nach Team

```text
templates/
├── backend-team/
├── frontend-team/
└── platform-team/
```

---

## GitHub-Integration für Templates

Templates benötigen eine GitHub-Authentifizierung, um Repositories zu erstellen.

### Integrationskonfiguration

```yaml
# app-config.yaml
integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}
```

### GitHub Personal Access Token

* Erforderliche Scopes: `repo`, `workflow`, `delete_repo`
* Als Umgebungsvariable setzen: `GITHUB_TOKEN`
* Wird von der `publish:github`-Action verwendet
* Token muss Berechtigungen für die Zielorganisation besitzen

---

## Template-Registrierung

```yaml
# app-config.yaml
catalog:
  locations:
    - type: file
      target: ../../templates/nodejs-service-template/template.yaml
```

**Ergebnis:**

* Template erscheint in der *Create Component*-Oberfläche
* Benutzer können Templates durchsuchen, finden und ausführen
* Backstage lädt Templates automatisch beim Start

---

## Template-Discovery

Templates erscheinen in der *Create Component*-Oberfläche von Backstage.

### Funktionen

* Nach Kategorie, Technologie oder Team browsen
* Suche nach Name oder Beschreibung
* Anzeige beliebter Templates
* Detailansicht mit Beschreibung, Parametern und Owner

### Template-Metadaten

```yaml
metadata:
  tags:
    - recommended
    - nodejs
    - microservice
  links:
    - url: https://docs.company.com/templates/nodejs
      title: Template Documentation
      icon: docs
```

---

## Automatisierte Discovery mit GitHub Provider

```yaml
# app-config.yaml
catalog:
  providers:
    github:
      backstageProvider:
        organization: 'your-github-org'
        catalogPath: '/catalog-info.yaml'
        filters:
          branch: 'main'
          repository: '.*'
        schedule:
          frequency:
            minutes: 30
          timeout:
            minutes: 3
```

### Funktionsweise

1. Scan aller Repositories über die GitHub API
2. Anwendung von Filtern (Branch, Name)
3. Suche nach `catalog-info.yaml`
4. Automatische Registrierung der Entities
5. Wiederholung alle 30 Minuten

### Vorteile

* Kein manueller Aufwand
* Immer aktuell
* Organisationsweit
* Skalierbar für tausende Services

---

## Backend-Modul-Registrierung

```ts
// packages/backend/src/index.ts
backend.add(import('@backstage/plugin-catalog-backend'));
backend.add(import('@backstage/plugin-catalog-backend-module-github'));
```

* Erforderlich für GitHub Provider
* Modernes Backstage nutzt ein modulares Backend
* Ohne Registrierung wird die Konfiguration ignoriert

---

## Vorteile von Software Templates

### Für Entwickler

* Schnelle Projekterstellung
* Konsistente Projektstruktur
* Best Practices integriert
* Geringere kognitive Last
* Automatische Katalog-Discovery

### Für Organisationen

* Einheitliche Standards
* Security-Compliance by default
* Schnellere Markteinführung
* Einfachere Wartung
* Zentrale Transparenz

### Für Plattform-Teams

* Durchsetzung von Standards
* Wissensverteilung
* Weniger Support-Aufwand
* Zentrale Weiterentwicklung
* Automatisierter Katalog

---

## Template Best Practices

### Design-Prinzipien

* Einfach starten
* Vollständig für Produktion
* Regelmäßig aktualisieren
* Alles dokumentieren

### Parameter-Design

* Sinnvolle Defaults
* Klare Validierung
* Verständliche Beschreibungen
* Smarte UI-Komponenten

### Content-Organisation

* Modulare Struktur
* Kommentare im Template
* Realistische Beispiele
* Cleanup-Hinweise

### Testing

* Nach jeder Änderung testen
* Output validieren
* Edge-Cases prüfen
* Entwickler testen lassen

### Versionskontrolle

* Git für Versionierung
* Changelog pflegen
* Breaking Changes kommunizieren
* Versionen einführen

---

## Häufige Template-Muster

### Microservice

```text
# Includes:
- REST API framework
- Database integration
- Authentication middleware
- Health check endpoints
- Containerization
- CI/CD pipeline
- Monitoring setup
```

### Frontend Application

```text
# Includes:
- React/Vue/Angular framework
- State management
- Routing configuration
- Build optimization
- Testing setup
- Deployment pipeline
- Analytics integration
```

### Library

```text
# Includes:
- Language-specific structure
- Testing framework
- Documentation generation
- Package publishing
- Version management
- Usage examples
```

---

## Enterprise-Template-Strategien

### Schrittweise Einführung

* Klein starten
* Feedback sammeln
* Schnell iterieren
* Abdeckung erweitern

### Template-Governance

* Template-Owner definieren
* Review-Prozess
* Deprecation-Policy
* Support-Kanäle

### Erfolg messen

* Adoptionsrate
* Zeitersparnis
* Entwicklerzufriedenheit
* Compliance-Rate

---

## Nächste Schritte

✅ Golden-Path-Templates implementieren
✅ GitHub-Integration und Discovery konfigurieren
✅ Templates sauber organisieren
✅ Best Practices anwenden
✅ Erfolg messen

Templates sind ein Grundpfeiler des Platform Engineerings – sie verwandeln Wissen in skalierbare Automatisierung.

---

## Zusätzliche Ressourcen

GitHub Discovery Provider
[documentation](https://backstage.io/docs/integrations/github/discovery/)

Backstage Integrations
[documentation](https://backstage.io/docs/integrations/)

Template Best Practices
[documentation](https://backstage.io/docs/features/software-templates/writing-templates/)

Catalog Configuration
[documentation](https://backstage.io/docs/features/software-catalog/configuration/)


====================================

 # 4-Erstellung erste Template  
 
====================================


**Über dieses Lab**

Lerne, Software Templates mit dem Backstage Scaffolder zu erstellen. Entwickle ein Template, das Node.js-Projekte generiert, sich in Git-Provider integriert und neue Services automatisch im Katalog registriert.

Du lernst:

* Die Template-Architektur und Scaffolding-Konzepte zu verstehen
* Template-Metadaten mit korrekten Parametern und Validierung zu definieren
* Benutzer-Eingabeformulare mit dynamischen Feldtypen zu erstellen
* Template-Skeletons mit parametrisierter Dateigenerierung zu bauen
* Scaffolding-Schritte für Projekterstellung und Git-Integration zu definieren
* Die automatische Katalogregistrierung für neue Services zu konfigurieren
* Template-Workflows zu testen und häufige Probleme zu beheben

Dieses praxisorientierte Lab demonstriert die Erstellung von Golden Paths und Self-Service-Entwicklungsworkflows.

**Wichtige Ressourcen:**

* Software Templates
* Writing Templates
* Scaffolder Actions

---

**Was du lernen wirst (7 Aufgaben)**

**1**
**Backstage Software Templates verstehen**
Erkunde bestehende Templates, um zu verstehen, wie der Backstage Scaffolder funktioniert und aus welchen Komponenten ein Software Template besteht.

**2**
**Template-Metadaten definieren**
Definiere den Metadaten-Bereich eines Software Templates, damit es für Entwicklungsteams auffindbar und nutzbar ist.

**3**
**Das Benutzer-Eingabeformular erstellen**
Erstelle den Parameter-Bereich, der das Formular definiert, das Entwickler beim Verwenden deines Templates ausfüllen.

**4**
**Das Template-Skeleton erstellen**
Erstelle die Skeleton-Dateien, die als Vorlagen für die Generierung neuer Projekte mit Platzhalter-Variablen dienen.

**5**
**Die Scaffolding-Schritte definieren**
Definiere die Scaffolding-Schritte, die steuern, wie dein Template Projekte generiert.

**6**
**Die Template-Ausgabe konfigurieren**
Konfiguriere den Output-Bereich, der Benutzern anzeigt, was erstellt wurde und wie sie nach der Generierung darauf zugreifen können.

**7**
**Templates testen und veröffentlichen**
Richte die GitHub-Authentifizierung ein und teste dein vollständiges Template, indem du es Ende-zu-Ende ausführst und ein echtes Repository erstellst.


===================================
- Template Erklärung
===================================

## Verständnis von Backstage Software Templates

**Ziel:** Lernen, wie Templates die Projekterstellung automatisieren.

**Erkunden:**

- YAML-Template-Definitionen (`template.yaml`)
- Skeleton-Verzeichnisse (`skeleton/`)
- Parameter und Formulare
- Integrierte Actions (`fetch:template`, `publish:github`, `catalog:register`)

**Tipp:** Vergleichen Sie mit einem bestehenden Template in Ihrem Katalog, um alle Bestandteile im Kontext zu sehen.

---

## Definition von Template-Metadaten

**Ziel:** Das Template auffindbar und verständlich machen.

**Wichtige Felder:**

- `metadata.name` – Eindeutige Kennung  
- `metadata.title` – Lesbarer Titel  
- `metadata.description` – Zweck des Templates  
- `metadata.tags` – Kategorien (z. B. nodejs, microservice)

**Spec:**

- `spec.owner` – Verantwortliches Team für das Template  
- `spec.type` – z. B. service oder library  

**Tipp:** Korrekte Metadaten stellen sicher, dass das Template in der Create-Component-UI erscheint.

---

## Erstellung des User-Input-Formulars

**Ziel:** Definieren, welche Informationen Entwickler beim Generieren eines Projekts angeben.

**Parameter-Sektion erstellen:**

- Pflichtfelder (z. B. Name, Beschreibung, Owner)  
- Validierung: Min-/Max-Länge, Regex-Patterns, Enums  
- UI-Hinweise: Autofocus, Textarea, Picker  

**Tipp:** Denken Sie an Standardwerte und Validierung, um Fehler zu reduzieren.

---

## Erstellung des Template-Skeletons

**Ziel:** Projektdateien bereitstellen, die kopiert und angepasst werden.

**Beispiel für eine Skeleton-Ordnerstruktur:**

```

skeleton/
├── package.json
├── tsconfig.json
├── Dockerfile
├── src/
│   └── index.ts
├── catalog-info.yaml
└── README.md

```

**Verwenden Sie Nunjucks-Templating für Variablen und Bedingungen:**

```

{{ values.name }} → Projektname
{% if parameters.include_swagger %} → Bedingter Inhalt

```

---

## Definition der Scaffolding-Schritte

**Ziel:** Actions orchestrieren, die Projekte generieren.

**Typische Schritte:**

- `fetch:template` → Skeleton-Dateien kopieren  
- `fs:rename` → Dateien bei Bedarf umbenennen  
- `publish:github` → Repository erstellen  
- `catalog:register` → Entity im Katalog registrieren  

Schritte können Outputs vorheriger Schritte verwenden:

```

${{ steps.publish.output.repoContentsUrl }}

```

---

## Konfiguration der Template-Ausgabe

**Ziel:** Entwicklern Feedback und Links nach der Generierung bereitstellen.

**Beispiel für eine Output-Sektion:**

```

output:
links:
- title: View Repository
url: '${{ steps.publish.output.remoteUrl }}'
icon: github
- title: View in Catalog
url: '${{ steps.register.output.catalogInfoUrl }}'
icon: catalog
text:
- title: Successfully Created ${{ parameters.name | title }}
content: |
Your new Node.js service is ready!
Name: ${{ parameters.name }}
Owner: ${{ parameters.owner }}
Next Steps:
Clone repo
Install dependencies
Start development

```

---

## Testen und Veröffentlichen von Templates

**Ziel:** Sicherstellen, dass Ihr Template Ende-zu-Ende funktioniert.

**Schritte:**

GitHub-Integration in `app-config.yaml` konfigurieren:

```

integrations:
github:
host: github.com
token: ${GITHUB_TOKEN}

```

Template lokal validieren:

```

npx @backstage/cli validate-entity template.yaml

```

Template über die Create-Component-UI ausführen.

**Überprüfen:**

- Repository wird auf GitHub erstellt  
- Katalogeintrag erscheint in Backstage  
- Skeleton-Dateien werden mit korrekten Variablen generiert  

**Tipp:** Testen Sie Edge-Cases wie optionale Parameter und ungültige Eingaben.

---

## Lab-Tipps

- Einfach starten: Zuerst ein grundlegendes Node.js-Projekt, dann Funktionen hinzufügen.  
- Verwenden Sie bedingte Nunjucks-Templates für optionale Funktionalität.  
- Halten Sie Skeleton und Parameter übersichtlich und gut organisiert.  
- Dokumentieren Sie Schritte in der `README.md` im Skeleton, um Entwickler anzuleiten.
```


====================================

# 5 **Erweiterte Scaffolder-Muster**

====================================

*Du hast die grundlegenden Aktionen gelernt – jetzt erkunden wir erweiterte Muster, die Templates leistungsfähig und flexibel machen. Diese Lektion behandelt die bedingte Ausführung, die Verkettung von Schritt-Ausgaben und den Aufbau vollständiger mehrstufiger Workflows*.

## Bedingte Aktionen

Führe Aktionen nur aus, wenn bestimmte Bedingungen erfüllt sind, indem du die Eigenschaft `if` verwendest:

**Grundlegende bedingte Ausführung**

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
```

**So funktioniert die bedingte Ausführung:**

* `if`: Die Aktion wird nur ausgeführt, wenn die Bedingung zu `true` ausgewertet wird
* Boolesche Parameter: Direkt prüfen: `${{ parameters.enable_cicd }}`
* Zeichenkettenvergleiche: Gleichheit verwenden: `${{ parameters.database_type === "postgresql" }}`
* Übersprungene Aktionen: Ist die Bedingung `false`, wird die Aktion vollständig übersprungen

**Mehrere bedingte Muster**

```yaml
# Database-specific setup
- id: setup-postgresql
  if: '${{ parameters.database_type === "postgresql" }}'
  name: Setup PostgreSQL Configuration
  action: fetch:template
  input:
    url: ./database-templates/postgresql
    targetPath: ./database
    values:
      name: '${{ parameters.name }}'

- id: setup-mongodb
  if: '${{ parameters.database_type === "mongodb" }}'
  name: Setup MongoDB Configuration
  action: fetch:template
  input:
    url: ./database-templates/mongodb
    targetPath: ./database
    values:
      name: '${{ parameters.name }}'

# Only add Swagger if both API docs enabled AND REST API type
- id: add-swagger
  if: '${{ parameters.include_swagger && parameters.api_type === "rest" }}'
  name: Add Swagger Documentation
  action: fetch:template
  input:
    url: ./swagger-templates
    targetPath: ./docs
```

**Vorteile bedingter Muster:**

* **Benutzerwahl:** Entwickler wählen nur die Funktionen, die sie benötigen
* **Technologievarianten:** Unterstützung mehrerer Datenbanken, Frameworks und Sprachen
* **Optionale Komponenten:** Monitoring, Sicherheit oder Dokumentation bei Bedarf hinzufügen
* **Saubere Ausgabe:** Generierte Projekte enthalten nur das, was Benutzer ausgewählt haben

---

## Verkettung von Schritt-Ausgaben

Aktionen erzeugen Ausgaben, die nachfolgende Aktionen verwenden können. Dadurch entstehen leistungsstarke Workflows, bei denen jeder Schritt auf den vorherigen Ergebnissen aufbaut.

**Schritt-Ausgaben verstehen**

```yaml
- id: publish
  name: Create GitHub Repository
  action: publish:github
  input:
    description: '${{ parameters.description }}'
    repoUrl: 'github.com?owner=${{ parameters.github_owner }}&repo=${{ parameters.repository_name }}'
    defaultBranch: main

# This action uses outputs from the publish step
- id: register
  name: Register Component
  action: catalog:register
  input:
    repoContentsUrl: '${{ steps.publish.output.repoContentsUrl }}'
    catalogInfoPath: '/catalog-info.yaml'
```

**Wichtige Ausgabe-Eigenschaften:**

* `steps.stepId.output.propertyName`: Referenziert die Ausgabe eines beliebigen vorherigen Schritts
* Häufige `publish:github`-Ausgaben:

  * `remoteUrl`: Vollständige Repository-URL (z. B. [https://github.com/org/repo](https://github.com/org/repo))
  * `repoContentsUrl`: URL der Repository-Inhalte für die Katalogregistrierung
  * `cloneUrl`: Git-Clone-URL
* Häufige `catalog:register`-Ausgaben:

  * `catalogInfoUrl`: URL zur Anzeige der Entität im Backstage-Katalog
  * `entityRef`: Referenzzeichenfolge der Entität

**Erweiterte Ausgabe-Verkettung**

```yaml
steps:
  # Step 1: Generate project from skeleton
  - id: fetch
    name: Fetch Skeleton Files
    action: fetch:template
    input:
      url: ./skeleton
      values:
        name: '${{ parameters.name }}'
        description: '${{ parameters.description }}'
        owner: '${{ parameters.owner }}'
        port_number: '${{ parameters.port_number }}'

  # Step 2: Conditionally add CI/CD
  - id: create-cicd
    if: '${{ parameters.enable_cicd }}'
    name: Setup GitHub Actions Workflow
    action: fetch:template
    input:
      url: ./cicd-templates
      targetPath: .github/workflows
      values:
        name: '${{ parameters.name }}'
        github_owner: '${{ parameters.github_owner }}'
        repository_name: '${{ parameters.repository_name }}'

  # Step 3: Create GitHub repository
  - id: publish
    name: Publish to GitHub
    action: publish:github
    input:
      description: '${{ parameters.description }}'
      repoUrl: 'github.com?owner=${{ parameters.github_owner }}&repo=${{ parameters.repository_name }}'
      defaultBranch: main
      gitCommitMessage: 'Initial commit from ${{ parameters.name }} template'
      gitAuthorName: '${{ parameters.owner }}'
      gitAuthorEmail: 'backstage@company.com'

  # Step 4: Register in catalog (uses publish output)
  - id: register
    name: Register in Backstage Catalog
    action: catalog:register
    optional: true
    input:
      repoContentsUrl: '${{ steps.publish.output.repoContentsUrl }}'
      catalogInfoPath: '/catalog-info.yaml'

  # Step 5: Log completion details (uses multiple outputs)
  - id: log-completion
    name: Log Template Completion
    action: debug:log
    input:
      message: |
        Successfully generated ${{ parameters.name }}
        Repository: ${{ steps.publish.output.remoteUrl }}
        Catalog: ${{ steps.register.output.catalogInfoUrl }}
        Owner: ${{ parameters.owner }}
```

**Vorteile der Pipeline:**

* **Datenfluss:** Jeder Schritt übergibt Informationen an den nächsten
* **Fehlerbehandlung:** `optional: true` verhindert, dass Fehler bei der Katalogregistrierung das Template abbrechen
* **Debugging:** Die Aktion `debug:log` hilft bei der Fehlersuche während der Template-Ausführung
* **Benutzerfeedback:** Detaillierte Abschlussmeldungen zeigen, was erstellt wurde

---

## Vollständiges praxisnahes Template

Hier ist ein produktionsreifes Template, das alle erweiterten Muster kombiniert:

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
    # Base project structure
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

    # Conditional: PostgreSQL setup
    - id: add-postgresql
      if: '${{ parameters.database_type === "postgresql" }}'
      name: Add PostgreSQL Configuration
      action: fetch:template
      input:
        url: ./database-templates/postgresql
        targetPath: ./database

    # Conditional: MongoDB setup
    - id: add-mongodb
      if: '${{ parameters.database_type === "mongodb" }}'
      name: Add MongoDB Configuration
      action: fetch:template
      input:
        url: ./database-templates/mongodb
        targetPath: ./database

    # Conditional: CI/CD pipeline
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

    # Publish to GitHub
    - id: publish
      name: Publish to GitHub
      action: publish:github
      input:
        description: '${{ parameters.description }}'
        repoUrl: 'github.com?owner=${{ parameters.github_owner }}&repo=${{ parameters.repository_name }}'
        defaultBranch: main

    # Register in catalog
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

**Template-Funktionen:**

* **Flexibel:** Benutzer wählen Datenbank, Dokumentation und CI/CD
* **Bedingte Logik:** Fügt nur ausgewählte Funktionen hinzu
* **Ausgabe-Verkettung:** Schritte nutzen vorherige Ergebnisse
* **Fehlerresistent:** Optionale Katalogregistrierung
* **Benutzerfreundlich:** Klare Ausgabe mit relevanten Links

---

## Benutzerdefinierte Aktionen

Während integrierte Aktionen die meisten Anforderungen abdecken, können Organisationen benutzerdefinierte Aktionen für spezielle Integrationen erstellen:

**Häufige Anwendungsfälle für benutzerdefinierte Aktionen**

**Bereitstellung von Infrastruktur:**

* Erstellen von AWS-Ressourcen (S3-Buckets, RDS-Datenbanken, Lambda-Funktionen)
* Bereitstellen von GCP-Projekten und -Services
* Einrichten von Azure-Ressourcengruppen

**Integrations-Setup:**

* Konfigurieren von Datadog-Dashboards
* Erstellen von PagerDuty-Services
* Einrichten von Monitoring-Warnmeldungen

**Workflow-Automatisierung:**

* Erstellen von Jira-Epics und -Tickets
* Posten in Slack-Kanälen
* Versenden von Willkommens-E-Mails

**Sicherheit:**

* Ausführen von Sicherheits-Scans
* Erstellen von Tickets zur Schwachstellenverfolgung
* Konfigurieren des Geheimnismanagements

**Beispiel für eine benutzerdefinierte Aktion**

```ts
// packages/backend/src/plugins/scaffolder/actions/createDatadogDashboard.ts
import { createTemplateAction } from '@backstage/plugin-scaffolder-node';

export const createDatadogDashboard = () => {
  return createTemplateAction({
    id: 'custom:datadog:dashboard',
    description: 'Creates a Datadog monitoring dashboard for a new service',
    schema: {
      input: {
        serviceName: z =>
          z.string({ description: 'The name of the service to monitor' }),
        apiKey: z =>
          z.string({ description: 'Datadog API key for authentication' }),
      },
    },
    async handler(ctx) {
      // Custom logic to create Datadog dashboard
      const dashboard = await createDashboard(ctx.input.serviceName, ctx.input.apiKey);

      // Set output that can be used by subsequent actions
      ctx.output('dashboardUrl', dashboard.url);
      ctx.output('dashboardId', dashboard.id);

      // Log success for debugging
      ctx.logger.info(`Created Datadog dashboard: ${dashboard.url}`);
    },
  });
};
```

**Struktur der benutzerdefinierten Aktion erklärt:**

* `id`: Eindeutiger Bezeichner mit dem Namensschema „namespace:entity:action“ (z. B. `custom:datadog:dashboard`)
* `description`: Für Menschen lesbarer Zweck, der in der Aktionsdokumentation angezeigt wird
* `schema.input`: Zod-Schema zur Definition der Eingabeparameter mit Beschreibungen
* `handler`: Asynchrone Funktion, die ein Kontextobjekt mit Eingaben, Logger und Workspace-Pfad erhält
* `ctx.output()`: Setzt Ausgabewerte, auf die nachfolgende Aktionen zugreifen können
* `ctx.logger`: Stellt Logging für Debugging und Monitoring bereit

**Vorteile benutzerdefinierter Aktionen:**

* **Organisationsspezifisch:** Integration in die eigene Toolchain
* **Automatisierung:** Reduziert manuelle Einrichtungsschritte
* **Konsistenz:** Stellt jedes Mal eine korrekte Konfiguration sicher
* **Produktivität:** Entwickler konzentrieren sich auf Code statt Infrastruktur
* **Wiederverwendbarkeit:** Einmal schreiben, in mehreren Templates nutzen

---

## Best Practices

**Aktionsdesign**

* Aktionen atomar halten: Jede Aktion erledigt eine Sache gut
* Bedingungen sinnvoll einsetzen: Nicht mit zu vielen Verzweigungen überkomplizieren
* Ausgaben effektiv verketten: Daten sauber zwischen Schritten übergeben
* Fehler robust behandeln: `optional: true` für nicht kritische Schritte verwenden

**Template-Organisation**

* Verantwortlichkeiten trennen: Skeleton, `cicd-templates` und `database-templates` getrennt halten
* Parameter dokumentieren: Klare Beschreibungen und Validierung
* Gründlich testen: Alle bedingten Pfade ausprobieren
* Feedback geben: `debug:log` zur Fehleranalyse nutzen

**Benutzererlebnis**

* Sinnvolle Standardwerte: Häufige Werte vorausfüllen
* Progressive Offenlegung: Einfach starten, Optionen schrittweise hinzufügen
* Klare Ausgabe: Zeigen, was erstellt wurde und wie es weitergeht
* Fehlermeldungen: Umsetzbare Fehlerinformationen bereitstellen

---

## Nächste Schritte

Das Verständnis erweiterter Muster bereitet dich darauf vor:

✅ Flexible Templates mit bedingten Funktionen zu erstellen

✅ Aktionen mithilfe von Schritt-Ausgaben zu verketten

✅ Produktionsreife mehrstufige Workflows zu bauen

✅ Benutzerdefinierte Aktionen für Organisationsanforderungen zu entwerfen

✅ Best Practices für wartbare Templates anzuwenden

Im praktischen Lab wirst du ein vollständiges Template mit diesen erweiterten Mustern erstellen, um einen produktionsreifen Projektgenerator zu bauen.

Erweiterte Scaffolder-Muster verwandeln Templates von einfachen Generatoren in ausgefeilte Automatisierungen, die sich an Entwicklerbedürfnisse anpassen und sich nahtlos in deine Entwicklungs-Toolchain integrieren.

---

## Kernaussagen

* Bedingte Aktionen mit der Eigenschaft `if` ermöglichen flexible Templates, die sich an Benutzerentscheidungen anpassen
* Schritt-Ausgaben erlauben Verkettungen, bei denen jede Aktion Ergebnisse vorheriger Schritte nutzt
* Vollständige Pipelines kombinieren Fetch-, bedingte Logik-, Publish- und Register-Aktionen
* Benutzerdefinierte Aktionen erweitern Templates zur Integration organisationsspezifischer Tools und Workflows
* Best Practices umfassen atomare Aktionen, klare Fehlerbehandlung und gründliche Tests
* Produktions-Templates balancieren Flexibilität und Einfachheit für ein optimales Entwicklererlebnis

---

## Zusätzliche Ressourcen

**Built-in Actions Reference**
[documentation](https://backstage.io/docs/features/software-templates/builtin-actions/)

**Writing Custom Actions**
[documentation](https://backstage.io/docs/features/software-templates/writing-custom-actions/)

**Template Writing Guide**
[documentation](https://backstage.io/docs/features/software-templates/writing-templates/)

---


=======================================

# 6 **TechDocs- und MkDocs-Grundlagen**
=======================================


Du hast die grundlegenden Aktionen gelernt – jetzt erkunden wir erweiterte Muster, die Templates leistungsfähig und flexibel machen. Diese Lektion behandelt die bedingte Ausführung, die Verkettung von Schritt-Ausgaben und den Aufbau vollständiger mehrstufiger Workflows.

**TechDocs- und MkDocs-Grundlagen**
TechDocs macht Dokumentation zu Code und erleichtert so, Dokumente aktuell und auffindbar zu halten. Lass uns erkunden, wie Documentation-as-Code funktioniert und wie man mit MkDocs hochwertige Dokumentation schreibt.

---

## Was ist Documentation-as-Code?

Documentation-as-Code behandelt Dokumentation wie Quellcode:

### Grundprinzipien

* **Versionskontrolliert:** Dokumentation liegt zusammen mit dem Code in Git
* **Automatisierte Veröffentlichung:** Dokumente werden automatisch gebaut und bereitgestellt
* **Single Source of Truth:** Ein zentraler Ort für maßgebliche Dokumentation
* **Entwickler-Workflow:** Schreiben von Doku mit vertrauten Tools und Prozessen

### Vorteile

* **Immer aktuell:** Dokumente werden zusammen mit Codeänderungen aktualisiert
* **Einfach zu schreiben:** Markdown ist simpel und vertraut
* **Durchsuchbar:** Integriert in die Backstage-Suche
* **Auffindbar:** Automatisch mit Katalog-Entitäten verknüpft

---

## TechDocs-Überblick

TechDocs ist das Dokumentationssystem von Backstage, das auf MkDocs basiert:

### Architektur

Markdown-Dateien → MkDocs-Build → Statisches HTML → Backstage-UI

### Zentrale Komponenten

* **MkDocs:** Statischer Site-Generator, der Markdown in HTML umwandelt
* **mkdocs.yml:** Konfigurationsdatei zur Definition von Seitenstruktur und Navigation
* **Markdown-Dateien:** Dokumentationsinhalte im Verzeichnis `docs/`
* **Material Theme:** Modernes, responsives Dokumentations-Theme
* **TechDocs Core Plugin:** Backstage-spezifische MkDocs-Erweiterungen

### Warum MkDocs?

* **Einfache Konfiguration:** YAML-basierte Konfigurationsdatei
* **Markdown-Unterstützung:** Schreiben in vertrauter Markdown-Syntax
* **Integrierte Themes:** Material Theme ist in TechDocs enthalten
* **Plugin-Ökosystem:** Funktionalität durch Plugins erweiterbar
* **Schnelle Builds:** Zügige Generierung von statischem HTML

---

## MkDocs-Konfiguration

### Grundlegende `mkdocs.yml`

```yaml
site_name: 'Sample Service Documentation'
site_description: 'Documentation for the Sample Microservice'

nav:
  - Introduction: index.md
  - Getting Started: getting-started.md
  - API Reference: api-reference.md
  - Deployment: deployment.md

plugins:
  - techdocs-core

theme:
  name: material
  features:
    - navigation.tabs
    - navigation.sections
    - toc.integrate
    - navigation.top
```

**Konfigurationsbereiche:**

* `site_name`: Wird als Titel der Dokumentation angezeigt
* `site_description`: Metadaten für Suche und SEO
* `nav`: Navigationsmenü-Struktur und Seitenreihenfolge
* `plugins`: TechDocs-Core-Plugin für die Backstage-Integration erforderlich
* `theme`: Material Theme mit Navigationsfunktionen

---

## Navigationsstruktur

Der Abschnitt `nav` definiert dein Dokumentationsmenü:

```yaml
nav:
  - Home: index.md
  - Getting Started:
    - Overview: getting-started/index.md
    - Installation: getting-started/installation.md
    - Configuration: getting-started/configuration.md
  - API:
    - Authentication: api/authentication.md
    - Endpoints: api/endpoints.md
  - Deployment: deployment.md
```

**Navigationsfunktionen:**

* **Verschachtelte Bereiche:** Verwandte Seiten gruppieren
* **Benutzerdefinierte Titel:** Anzeigename unterscheidet sich vom Dateinamen
* **Reihenfolgensteuerung:** Seiten erscheinen in der angegebenen Reihenfolge
* **Erforderliche `index.md`:** Jede Dokumentationssammlung benötigt eine Startseite

---

## Dokumentationsstruktur

```
my-service/
├── docs/
│   ├── index.md              # Homepage - required
│   ├── getting-started.md    # Setup instructions
│   ├── api-reference.md      # API documentation
│   └── deployment.md         # Deployment guide
├── mkdocs.yml                # MkDocs configuration
├── catalog-info.yaml         # Backstage entity definition
└── src/                      # Application source code
```

**Wichtige Dateien:**

* `docs/index.md`: Erforderliche Startseite, die erste Seite, die Nutzer sehen
* `mkdocs.yml`: Muss im Repository-Root (oder im `docs`-Verzeichnis) liegen
* `catalog-info.yaml`: Verknüpft die Backstage-Entität mit der Dokumentation

---

## Dokumentation schreiben

### Markdown-Grundlagen

````markdown
# Main Heading

## Section Heading

Regular paragraph text with **bold** and *italic* formatting.

- Bulleted list item
- Another item

1. Numbered list item
2. Another numbered item

```code blocks```

[Link text](https://example.com)
````

### Code-Dokumentation

APIs und Code mit syntaxhervorgehobenen Blöcken dokumentieren:

````markdown
## API Endpoints

### Get User

Retrieve user information by ID.

```http
GET /api/users/{id}
Request Parameters:

id (path, required): User ID
Response:

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}
Status Codes:

200 OK: User found
404 Not Found: User doesn't exist
````

### Diagramme mit Mermaid

Visuelle Diagramme direkt in Markdown erstellen:

````markdown
## System Architecture

```mermaid
graph LR
  A[Client] --> B[API Gateway]
  B --> C[User Service]
  B --> D[Order Service]
  C --> E[Database]
  D --> E
````

**Mermaid-Diagrammtypen:**

* **Flussdiagramme:** Prozessabläufe und Entscheidungsbäume
* **Sequenzdiagramme:** Service-Interaktionen
* **Entitätsbeziehungen:** Datenbankschemata
* **Gantt-Diagramme:** Projektzeitpläne

### Tabellen und Listen

```markdown
## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| port | number | 3000 | Server port |
| host | string | localhost | Server host |
| debug | boolean | false | Debug mode |

## Prerequisites

Before starting, ensure you have:

- Node.js 18 or higher
- Docker Desktop installed
- Git configured locally
- Access to GitHub repository
```

---

## Admonitions (Hinweise/Callouts)

Wichtige Informationen mit Admonitions hervorheben:

```markdown
!!! note "Installation Note"
    Make sure to install dependencies before running the service.

!!! warning "Breaking Change"
    Version 2.0 introduces breaking API changes. Review migration guide.

!!! tip "Performance Tip"
    Enable caching for better performance in production.

!!! danger "Security Warning"
    Never commit API keys or secrets to version control.
```

---

## Best Practices für Dokumentation

### Struktur-Richtlinien

* Mit einer Übersicht beginnen: Was macht dieser Service?
* Getting-Started-Anleitung: Wie lokal starten
* API-Dokumentation: Endpunkte und Beispiele
* Deployment-Guide: Bereitstellung und Konfiguration
* Troubleshooting: Häufige Probleme und Lösungen

### Schreibtipps

* **Einfach halten:** Klare, präzise Sprache verwenden
* **Beispiele einfügen:** Zeigen statt nur erklären
* **Regelmäßig aktualisieren:** Mit Codeänderungen überprüfen und anpassen
* **Diagramme nutzen:** Visuelle Hilfen verbessern das Verständnis
* **Doku testen:** Den eigenen Anleitungen folgen

---

## Häufige Dokumentationsmuster

### Service-Dokumentation:

```
docs/
├── index.md              # Service overview
├── getting-started.md    # Local development setup
├── api/
│   ├── authentication.md
│   ├── users.md
│   └── orders.md
├── deployment/
│   ├── docker.md
│   └── kubernetes.md
└── troubleshooting.md
```

### Architektur-Dokumentation:

```
docs/
├── index.md              # System overview
├── architecture/
│   ├── overview.md
│   ├── components.md
│   └── data-flow.md
├── services/
│   ├── user-service.md
│   └── payment-service.md
└── deployment/
    └── production.md
```

---

## Lokaler Entwicklungs-Workflow

### MkDocs installieren

```bash
# Install MkDocs with TechDocs extensions
pip install mkdocs-techdocs-core

# This includes:
# - mkdocs (core)
# - mkdocs-material (theme)
# - mkdocs-mermaid2-plugin (diagrams)
# - Other TechDocs plugins
```

### Schreiben und Vorschau

```bash
# Navigate to your service directory
cd my-service/

# Serve documentation locally with live reload
mkdocs serve

# Documentation available at http://127.0.0.1:8000
# Auto-refreshes when you save changes
```

### Dokumentation bauen

```bash
# Build static HTML files
mkdocs build

# Output appears in site/ directory
ls -la site/
# index.html, getting-started.html, api-reference.html, deployment.html
```

**Was `mkdocs build` macht:**

* Liest die Konfiguration aus `mkdocs.yml`
* Verarbeitet Markdown-Dateien im Verzeichnis `docs/`
* Generiert statische HTML-Dateien im Verzeichnis `site/`
* Wendet das Material-Theme-Styling an
* Erstellt die Navigationsstruktur
* Validiert interne Links

---

## Nächste Schritte

Das Verständnis von MkDocs und Documentation-as-Code bereitet dich darauf vor:

✅ Wartbare Dokumentation in Markdown zu schreiben

✅ MkDocs für deine Services zu konfigurieren

✅ Dokumentation für gute Auffindbarkeit zu strukturieren

✅ Diagramme und Codebeispiele effektiv zu nutzen

✅ Dokumentation lokal vor der Veröffentlichung zu prüfen

In der nächsten Lektion werden wir erkunden, wie man TechDocs in Backstage integriert, Dokumentation veröffentlicht und den Workflow automatisiert.

Großartige Dokumentation ist ein Multiplikator für Entwicklungsteams – sie reduziert Supportaufwand, beschleunigt das Onboarding und macht Wissen für alle zugänglich.

---

## Kernaussagen

* Documentation-as-Code behandelt Doku als versionskontrollierten Quellcode mit automatisierter Veröffentlichung
* TechDocs nutzt den statischen Site-Generator MkDocs, um Markdown in HTML-Dokumentation umzuwandeln
* `mkdocs.yml` definiert Seitenstruktur, Navigation, Theme und Plugin-Konfiguration
* Markdown unterstützt Überschriften, Listen, Codeblöcke, Tabellen und Mermaid-Diagramme
* Best Practices umfassen klare Struktur, praxisnahe Beispiele, regelmäßige Updates und visuelle Hilfsmittel
* Der lokale Entwicklungs-Workflow nutzt `mkdocs serve` für Live-Vorschau und `mkdocs build` für die statische Generierung

---

## Zusätzliche Ressourcen

**MkDocs User Guide**
[documentation](https://www.mkdocs.org/user-guide/)

**Material for MkDocs**
[documentation](https://squidfunk.github.io/mkdocs-material/)

**Markdown Guide**
[documentation](https://www.markdownguide.org/)

---

====================================================

# 7-**TechDocs-Integration und Veröffentlichung**

======================================================

## TechDocs-Plugin-Architektur

TechDocs benötigt zwei separate Plugins, die zusammenarbeiten:

### Frontend-Plugin

**Zweck:** Stellt UI-Komponenten zum Anzeigen von Dokumentation in der Backstage-Oberfläche bereit.

**Installation:**

```
yarn --cwd packages/app add @backstage/plugin-techdocs
```

**Funktionen:**

* Rendert Dokumentationsseiten in der Backstage-UI
* Stellt die TechDocs-Indexseite bereit (listet alle Dokumentationen auf)
* Stellt die TechDocs-Reader-Seite bereit (zeigt einzelne Dokumentationen an)
* Integriert Dokumentations-Tabs in Entitätsseiten

### Backend-Plugin

**Zweck:** Verarbeitet und stellt Dokumentation serverseitig bereit.

**Installation:**

```
yarn --cwd packages/backend add @backstage/plugin-techdocs-backend
```

**Funktionen:**

* Generiert Dokumentation aus MkDocs-Quellen
* Liefert gebaute Dokumentation an das Frontend aus
* Verwaltet die Speicherung der Dokumentation (lokal oder in der Cloud)
* Verarbeitet Dokumentations-Builds bei Bedarf oder vorab

**Backend-Registrierung:**
Im neuen Backend-System wird das TechDocs-Backend-Plugin nach der Installation automatisch registriert:

```ts
// packages/backend/src/index.ts
backend.add(import('@backstage/plugin-techdocs-backend'));
```

---

## TechDocs-Konfiguration

### App-Konfiguration (app-config.yaml)

Nach der Installation beider Plugins wird das Verhalten von TechDocs konfiguriert:

```yaml
techdocs:
  builder: 'local'
  generator:
    runIn: 'local'
    mkdocsCommand: '/opt/mkdocs-venv/bin/mkdocs'  # or just 'mkdocs'
  publisher:
    type: 'local'
    local:
      storageDir: '/root/labs/developer-portal/techdocs-storage'
```

**Konfiguration erklärt:**

* `builder: 'local'`: Dokumentation lokal bauen (statt externer Builder-Service)
* `generator.runIn: 'local'`: MkDocs lokal ausführen (statt Docker-Container)
* `generator.mkdocsCommand`: Pfad zur MkDocs-Executable
* `publisher.type: 'local'`: Dokumentation lokal im Dateisystem speichern (statt S3/GCS)
* `publisher.local.storageDir`: Speicherort für generierte HTML-Dateien

---

## Frontend-Routen-Konfiguration (App.tsx)

Nach der Installation des Frontend-Plugins müssen Routen hinzugefügt werden, um die Anzeige der Dokumentation zu aktivieren:

```ts
import {
  TechDocsIndexPage,
  TechDocsReaderPage,
} from '@backstage/plugin-techdocs';

const routes = (
  <FlatRoutes>
    {/* ... other routes ... */}
    <Route path="/docs" element={<TechDocsIndexPage />} />
    <Route
      path="/docs/:namespace/:kind/:name/*"
      element={<TechDocsReaderPage />}
    />
  </FlatRoutes>
);
```

**Routen erklärt:**

* `/docs`: Dokumentations-Indexseite mit allen verfügbaren Dokumentationen
* `/docs/:namespace/:kind/:name/*`: Viewer für die Dokumentation einer einzelnen Entität
* Die Routen müssen zur `FlatRoutes`-Komponente in `App.tsx` hinzugefügt werden

---

## Backstage-Integration

### Konfiguration der Katalog-Entität

Verknüpfe Dokumentation mit Katalog-Entitäten über die Annotation `backstage.io/techdocs-ref`:

```yaml
# catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: sample-service
  title: Sample Microservice
  description: A sample microservice for TechDocs testing
  annotations:
    backstage.io/techdocs-ref: dir:.
spec:
  type: service
  owner: platform-team
  lifecycle: experimental
```

**TechDocs-Annotationen:**

* `dir:.`: Sucht nach `mkdocs.yml` und `docs/` im Repository-Root
* `dir:./docs`: Sucht nach Dokumentation im Unterverzeichnis `docs/`
* `dir:./subdirectory`: Sucht in einem benutzerdefinierten Unterverzeichnis
* `url:https://github.com/org/repo`: Referenziert ein externes Repository

### Registrierung der Entität im Katalog

Eintrag in `app-config.yaml` hinzufügen:

```yaml
catalog:
  locations:
    - type: file
      target: /root/sample-repos/sample-service/catalog-info.yaml
```

**Nach der Registrierung:**

* Der Service erscheint im Backstage-Katalog
* Ein Dokumentations-Tab ist auf der Entitätsseite verfügbar
* Dokumentation ist unter `/docs/default/component/sample-service` erreichbar
* Dokumentation wird automatisch für die Suche indexiert

---

## Dokumentation bauen und veröffentlichen

### Lokaler Entwicklungs-Workflow

```bash
# Navigate to service directory
cd /root/sample-repos/sample-service

# Build documentation
mkdocs build

# Output appears in site/ directory
ls -la site/
# index.html, getting-started.html, api-reference.html, deployment.html

# Manually publish to Backstage storage (development only)
cp -r site/* /root/labs/developer-portal/techdocs-storage/default/component/sample-service/
```

**Was beim Build passiert:**

* MkDocs liest die Konfiguration aus `mkdocs.yml`
* Verarbeitet Markdown-Dateien im Verzeichnis `docs/`
* Generiert statische HTML-Dateien im Verzeichnis `site/`
* Wendet das Material-Theme-Styling an
* Erstellt die Navigationsstruktur
* Validiert interne Links

---

## Veröffentlichungs-Workflows

### Lokaler Speicher (Entwicklung)

Für Entwicklung und Tests:

```yaml
techdocs:
  publisher:
    type: 'local'
    local:
      storageDir: '/path/to/techdocs-storage'
```

**Vorteile:**

* Einfache Einrichtung ohne Cloud-Abhängigkeiten
* Schnelle Iteration während der Entwicklung
* Keine zusätzlichen Kosten

**Nachteile:**

* Nicht für den Produktivbetrieb geeignet
* Keine Skalierbarkeit oder Redundanz
* Manuelle Dateiverwaltung erforderlich

### Cloud-Speicher (Produktion)

**AWS S3:**

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

**Google Cloud Storage:**

```yaml
techdocs:
  publisher:
    type: 'googleGcs'
    googleGcs:
      bucketName: 'techdocs-storage'
      projectId: 'my-project'
```

**Azure Blob Storage:**

```yaml
techdocs:
  publisher:
    type: 'azureBlobStorage'
    azureBlobStorage:
      containerName: 'techdocs'
      accountName: 'mystorageaccount'
```

**Vorteile von Cloud-Speicher:**

* **Skalierbarkeit:** Bewältigt jede Menge an Dokumentation
* **Zuverlässigkeit:** Redundanz und Backups des Cloud-Anbieters
* **Performance:** CDN-Integration für schnellen globalen Zugriff
* **Kosteneffizienz:** Bezahlung nur für genutzten Speicher

---

## Automatisierte Veröffentlichung

### CI/CD-Integration

Automatisiere Build und Veröffentlichung der Dokumentation mit GitHub Actions:

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

**Vorteile der Automatisierung:**

* **Immer aktuell:** Dokumentation wird bei Änderungen automatisch neu gebaut
* **Keine manuellen Schritte:** Entwickler committen lediglich die Doku
* **Versionskontrolle:** Dokumentationshistorie wird in Git verfolgt
* **Qualitätssicherung:** Validierungs- und Linting-Schritte möglich

---

## Veröffentlichungsstrategien

**Push-basiert (empfohlen):**

* CI/CD baut und veröffentlicht bei jedem Git-Push
* Schnelle Updates, sofortige Veröffentlichung
* Funktioniert mit jedem Storage-Backend

**Pull-basiert (On-Demand):**

* Backstage baut Dokumentation beim ersten Zugriff
* Geeignet für große Organisationen
* Reduziert CI/CD-Overhead

---

## Erweiterte Funktionen

### Benutzerdefinierte Plugins

Erweitere TechDocs mit MkDocs-Plugins:

```yaml
# mkdocs.yml
plugins:
  - techdocs-core
  - mermaid2  # Diagram support
  - swagger-ui-tag  # Interactive API docs
  - search  # Full-text search
```

**Beliebte Plugins:**

* `mermaid2`: Flussdiagramme und Diagramme
* `swagger-ui-tag`: Einbettung von OpenAPI-/Swagger-Spezifikationen
* `drawio-exporter`: Diagramme aus draw.io-Dateien
* `git-revision-date`: Anzeige des letzten Aktualisierungsdatums
* `minify`: Komprimierung von HTML/CSS/JS für schnellere Ladezeiten

### Mehrsprachige Unterstützung

Unterstützung mehrerer Sprachen mit MkDocs i18n:

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

### Suchintegration

TechDocs-Dokumentation wird automatisch von der Backstage-Suche indexiert:

* Volltextsuche über die gesamte Dokumentation
* Facettierte Suche nach Entitätstyp, Owner und Tags
* Relevanzbasierte Ergebnisreihung
* Kontextuelle Treffer mit passenden Textausschnitten

---

## Vorteile der TechDocs-Integration

### Für Entwickler

* Vertrauter Workflow: Dokumentation wie Code schreiben
* Versionskontrolliert: Änderungen über die Zeit nachverfolgen
* Leicht auffindbar: In den Service-Katalog integriert
* Immer aktuell: Automatisierte Veröffentlichung hält Doku frisch

### Für Organisationen

* Einheitliches Format: Gleiche Struktur für alle Services
* Geringerer Wartungsaufwand: Automatisierung reduziert manuelle Arbeit
* Bessere Auffindbarkeit: Zentrales Dokumentationsportal
* Verbessertes Onboarding: Neue Entwickler finden Informationen schneller

### Für Plattform-Teams

* Standardisierung: Durchsetzung von Dokumentationsstandards
* Automatisierung: Reduzierter manueller Veröffentlichungsaufwand
* Transparenz: Überblick über Dokumentationsabdeckung
* Skalierbarkeit: Dokumentation für Tausende von Services handhabbar

---

## Fehlerbehebung häufiger Probleme

### Dokumentation wird nicht angezeigt

Prüfen:

* Annotation `backstage.io/techdocs-ref` in `catalog-info.yaml` vorhanden
* `mkdocs.yml` existiert am richtigen Ort
* `docs/index.md` existiert (erforderlich)
* TechDocs-Speicherverzeichnis korrekt konfiguriert
* Entität im Katalog registriert

### Build-Fehler

Häufige Ursachen:

* Fehlende `index.md`
* Ungültiges YAML in `mkdocs.yml`
* Defekte interne Links
* Fehlende MkDocs-Plugins
* Falsche Dateipfade in der Navigation

### Veröffentlichungsfehler

Überprüfen:

* Storage-Zugangsdaten korrekt konfiguriert
* Bucket/Container existiert und ist erreichbar
* Dateiberechtigungen erlauben Schreibzugriff
* Netzwerkverbindung zum Storage-Backend

---

## Nächste Schritte

Das Verständnis der TechDocs-Integration ermöglicht dir:

✅ Installation von Frontend- und Backend-TechDocs-Plugins

✅ Konfiguration von Builder-, Generator- und Publisher-Einstellungen

✅ Hinzufügen von Routen zur Dokumentationsanzeige im Frontend

✅ Verknüpfung von Dokumentation mit Katalog-Entitäten über Annotationen

✅ Automatisierung der Dokumentationsveröffentlichung mit CI/CD

✅ Bereitstellung von Dokumentation auf produktiven Storage-Backends

Im kommenden Lab installierst du TechDocs-Plugins, konfigurierst die lokale Dokumentationsgenerierung, fügst Frontend-Routen hinzu und veröffentlichst Dokumentation für einen Beispiel-Service.

Die Integration von TechDocs verwandelt Dokumentation von einer Wartungslast in ein automatisiertes, auffindbares Asset, das mit deiner Organisation skaliert.

---

## Kernaussagen

* TechDocs benötigt zwei Plugins: ein Frontend-Plugin für UI-Komponenten und ein Backend-Plugin für Generierung und Auslieferung
* Das Frontend stellt Index- und Reader-Seiten bereit, während das Backend MkDocs-Verarbeitung und Speicherung übernimmt
* Das Backend-Plugin wird im neuen Backend-System automatisch registriert
* Die Konfiguration in `app-config.yaml` steuert Builder-, Generator- und Publisher-Verhalten
* Katalog-Entitäten werden über die Annotation `backstage.io/techdocs-ref` mit Dokumentation verknüpft
* Lokaler Speicher eignet sich für Entwicklung, Cloud-Speicher (S3/GCS/Azure) ist für Produktion erforderlich
* Automatisierte Veröffentlichung mit CI/CD stellt sicher, dass Dokumentation stets mit dem Code aktuell bleibt

---

## Zusätzliche Ressourcen

**TechDocs Documentation**
[documentation](https://backstage.io/docs/features/techdocs/)

**TechDocs Configuration**
[documentation](https://backstage.io/docs/features/techdocs/configuration/)

**TechDocs CI/CD**
[documentation](https://backstage.io/docs/features/techdocs/configuring-ci-cd/#steps)

---

---

### Über dieses Lab

In diesem Lab konfigurierst du **TechDocs**, um **Documentation-as-Code-Workflows** in Backstage zu ermöglichen. Du lernst, wie du **MkDocs integrierst**, **Speicher-Backends einrichtest** und Dokumentation erstellst, die automatisch zusammen mit deinen Services veröffentlicht wird.

**Du wirst lernen:**

* TechDocs in Backstage für die Veröffentlichung von Dokumentation zu konfigurieren
* Das TechDocs-Backend mit lokalem Speicher einzurichten
* MkDocs zu installieren und mit dem **Material-Theme** zu konfigurieren
* Dokumentationsseiten mit sinnvoller Struktur und Navigation zu erstellen
* Dokumentation mit Katalog-Entitäten zu integrieren
* Workflows für die Veröffentlichung und Validierung der Dokumentation zu testen
* Docs-as-Code-Praktiken und Templates zu etablieren

Dieses praktische Lab etabliert **Dokumentations-Workflows**, die sich nahtlos in dein Entwickler-Portal integrieren.

---

# **TechDocs-Setup**

Richte TechDocs für **Documentation-as-Code-Workflows** ein – inklusive **MkDocs-Integration** und **automatischer Veröffentlichung**.

### Wichtige Ressourcen

* **[TechDocs Übersicht](https://backstage.io/docs/features/techdocs/)**
* **[TechDocs Einstieg](https://backstage.io/docs/features/techdocs/getting-started/)**
* **[MkDocs Dokumentation](https://www.mkdocs.org/)**

---


==============================================
### Lernziele (8 Aufgaben)
===========================================


1. **TechDocs Frontend Plugin installieren**

   Installiere das TechDocs-Frontend-Plugin, um die Anzeige von Dokumentation in Backstage zu ermöglichen.

3. **TechDocs Backend Plugin installieren**

   Installiere das TechDocs-Backend-Plugin, um die Generierung und Bereitstellung von Dokumentation zu ermöglichen.

4. **TechDocs für lokale Generierung konfigurieren**
   
   Konfiguriere TechDocs so, dass Dokumentation lokal generiert wird – ohne Docker-Container.

5. **MkDocs lokal für TechDocs installieren**

   Installiere MkDocs und die benötigten Plugins lokal, um die Dokumentation zu erstellen.

7. **TechDocs-Routen im Frontend hinzufügen**

   Füge die TechDocs-Routen und Komponenten in die Backstage-Frontend-Anwendung ein.

9. **Dokumentation für Katalog-Entitäten erstellen**

   Erstelle die MkDocs-Dokumentationsstruktur und Inhalte für die Katalog-Entitäten.

11. **Dokumentation testen und vorschauen**

    Teste die vollständige TechDocs-Konfiguration und schau die Dokumentation in Backstage an.

---


---
=====================================
# 9 **Automatisierte Discovery**
=====================================

Richte eine **automatisierte Discovery** ein, um Services aus GitHub-Repositories zu importieren – inklusive realer Repository-Erstellung und Tests.

**Über dieses Lab**
Setze die automatisierte Service-Discovery um, um manuelle Pflege des Katalogs zu reduzieren. Konfiguriere die GitHub-Integration, richte Discovery-Provider ein, erstelle echte Test-Repositories und beobachte, wie Backstage diese automatisch entdeckt und registriert.

**Du wirst lernen, wie man:**

* Die GitHub-Integration für automatisiertes Repository-Scanning konfiguriert
* Das GitHub-Discovery-Provider-Modul im Backend registriert
* Echte GitHub-Repositories mit Catalog-Entity-Definitionen erstellt
* Die GitHub CLI nutzt, um Repository-Erstellung und -Verwaltung zu automatisieren
* Die automatisierte Discovery beobachtet, wie sie deine Entities findet und registriert
* Die Discovery mit realen Repositories in deinem GitHub-Account testet
* Testressourcen bereinigt und Zugriffs-Tokens sicher widerruft

Dieses praxisnahe Lab automatisiert die Katalogverwaltung und demonstriert skalierbare Service-Discovery mit echten GitHub-Repositories.

**Wichtige Ressourcen:**

* [Discovery Overview](https://backstage.io/docs/auth/microsoft/provider/#configuration)
* [GitHub Discovery](https://backstage.io/docs/integrations/github/discovery/)
* [GitHub Provider](https://backstage.io/docs/integrations/github/org/#installation)

**Was du lernen wirst (3 Aufgaben):**

1. **GitHub-Integration und Discovery-Provider konfigurieren**

   Richten Sie die GitHub-Integration ein und konfigurieren Sie den Discovery-Provider, damit GitHub-Organisationen automatisch nach Catalog-Entities gescannt   werden.

2. **Automatisierte Discovery verstehen**
   Erfahre, wie der GitHub-Discovery-Provider von Backstage Organisationen automatisch durchsucht, um Catalog-Entities zu finden und zu registrieren.
   
3. **Automatisierte Discovery mit echten Repositories testen**
   Erstelle Test-Repositories mit Catalog-Entities und beobachte, wie Backstage diese automatisch über das Scannen der GitHub-Organisation entdeckt.

---
















