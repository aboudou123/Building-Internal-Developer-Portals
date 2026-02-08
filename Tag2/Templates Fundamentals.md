## Backstage Software Templates: Fundamentals

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
## Additional Resources
Backstage Software Templates
[documentation](https://backstage.io/docs/features/software-templates/)
Template Writing Guide
[documentation](https://backstage.io/docs/features/software-templates/writing-templates/)


===========================================================
 # 2- Template Actions & Skeleton Content
========================================================


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


===========================================================
 # 3- Template Integration and Best Practices
========================================================


Here’s a structured summary of **Template Integration and Best Practices** for Backstage, capturing the key concepts and workflow:

---

## **1. Golden Path Templates**

Golden Path templates are your organization’s "blessed" way to create projects.

**Characteristics:**

* Complete setup for production
* Security by default (scanning, secrets management)
* Observability (metrics, logging, tracing)
* CI/CD integration
* Documentation (README, API docs, runbooks)

**Example (Node.js Golden Path):**

* TypeScript, ESLint, Prettier
* Jest testing
* Docker & containerization
* GitHub Actions CI/CD
* Prometheus metrics
* OpenAPI docs
* Security scanning

**Benefits:**

* Reduce developer decision fatigue
* Ensure standards and compliance
* Embed security and observability
* Accelerate onboarding

---

## **2. Template Organization**

Organize templates logically for maintainability:

**By Technology Stack**

```
templates/
├── nodejs-service/
├── python-service/
├── react-frontend/
└── golang-service/
```

**By Purpose**

```
templates/
├── microservice/
├── frontend-app/
├── library/
└── infrastructure/
```

**By Team**

```
templates/
├── backend-team/
├── frontend-team/
└── platform-team/
```

---

## **3. GitHub Integration**

Templates require GitHub access to create repositories.

**Configuration (`app-config.yaml`):**

```yaml
integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}
```

**Token Requirements:**

* Scopes: `repo`, `workflow`, `delete_repo`
* Set as environment variable: `GITHUB_TOKEN`

**Template Registration**

```yaml
catalog:
  locations:
    - type: file
      target: ../../templates/nodejs-service-template/template.yaml
```

---

## **4. Template Discovery**

**Create Component UI**:

* Browse templates by category, tech, or team
* Search by name/description
* View details (description, parameters, owner)

**Metadata Example:**

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

**Automated Discovery (GitHub Provider):**

```yaml
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
```

**Benefits:**

* Zero manual work
* Always up-to-date
* Scalable to thousands of repos

---

## **5. Backend Module Registration**

```ts
backend.add(import('@backstage/plugin-catalog-backend'));
backend.add(import('@backstage/plugin-catalog-backend-module-github'));
```

* Needed for GitHub provider to work
* Modern Backstage uses modular backend system

---

## **6. Template Best Practices**

**Design Principles:**

* Start simple, then add features
* Include everything for production
* Keep templates up to date
* Document everything

**Parameter Design:**

* Sensible defaults
* Clear validation
* Helpful descriptions
* Smart UI components

**Content Organization:**

* Modular structure
* Template comments
* Example data
* Cleanup instructions

**Testing Templates:**

* Run after every change
* Validate output
* Test edge cases
* User testing

**Version Control:**

* Track changes with Git
* Document updates
* Communicate breaking changes
* Consider template versioning

---

## **7. Common Template Patterns**

* **Microservice:** REST API, database integration, auth, health checks, CI/CD, monitoring
* **Frontend App:** React/Vue/Angular, state management, routing, testing, deployment
* **Library:** Language-specific structure, tests, docs, package publishing

---

## **8. Enterprise Template Strategies**

* Gradual adoption: start small, gather feedback, iterate
* Template governance: assign owners, review process, deprecation policy
* Measure success: adoption rate, time savings, developer satisfaction, compliance rate

---

### **Key Takeaways**

* Golden Path templates ensure consistent, secure, and observable project setups.
* GitHub integration + automated discovery eliminates manual catalog updates.
* Templates benefit developers (speed, consistency), organizations (standards), and platform teams (automation).
* Organize templates thoughtfully, test frequently, and follow governance to scale across an enterprise.

---
# Additional Resources

GitHub Discovery Provider
[documentation](https://backstage.io/docs/integrations/github/discovery/)

Backstage Integrations
[documentation](https://backstage.io/docs/integrations/)

Template Best Practices
[documentation](https://backstage.io/docs/features/software-templates/writing-templates/)

Catalog Configuration
[documentation](https://backstage.io/docs/features/software-catalog/configuration/)


======================================================================================



 # 4- **“Build Your First Template”** 

---

## **1. Understanding Backstage Software Templates**

* **Goal:** Learn how templates automate project creation.
* Explore:

  * YAML template definitions (`template.yaml`)
  * Skeleton directories (`skeleton/`)
  * Parameters and forms
  * Built-in actions (`fetch:template`, `publish:github`, `catalog:register`)
* **Tip:** Compare with a pre-existing template in your catalog to see all pieces in context.

---

## **2. Defining Template Metadata**

* **Goal:** Make the template discoverable and understandable.
* Key fields:

  * `metadata.name` – Unique identifier
  * `metadata.title` – Human-readable title
  * `metadata.description` – What the template does
  * `metadata.tags` – Categories (e.g., `nodejs`, `microservice`)
* **Spec:**

  * `spec.owner` – Team responsible for template
  * `spec.type` – e.g., `service` or `library`
* **Tip:** Proper metadata ensures the template appears in the Create Component UI.

---

## **3. Creating the User Input Form**

* **Goal:** Define what information developers provide when generating a project.
* Create `parameters` section:

  * Required fields (e.g., `name`, `description`, `owner`)
  * Validation: min/max length, regex patterns, enums
  * UI hints: autofocus, textarea, pickers
* **Tip:** Think about defaults and validation to reduce errors.

---

## **4. Creating the Template Skeleton**

* **Goal:** Provide project files that will be copied and customized.
* Skeleton folder structure example:

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
* Use **Nunjucks** templating for variables and conditionals:

  * `{{ values.name }}` → project name
  * `{% if parameters.include_swagger %}` → conditional content

---

## **5. Defining the Scaffolding Steps**

* **Goal:** Orchestrate actions that generate projects.
* Typical steps:

  1. `fetch:template` → copy skeleton files
  2. `fs:rename` → rename files if needed
  3. `publish:github` → create repository
  4. `catalog:register` → register entity in catalog
* Steps can use outputs from previous steps:
  `${{ steps.publish.output.repoContentsUrl }}`

---

## **6. Configuring the Template Output**

* **Goal:** Provide developers with feedback and links after generation.
* Example output section:

  ```yaml
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
          - Name: ${{ parameters.name }}
          - Owner: ${{ parameters.owner }}
          Next Steps:
          1. Clone repo
          2. Install dependencies
          3. Start development
  ```

---

## **7. Testing and Publishing Templates**

* **Goal:** Ensure your template works end-to-end.
* Steps:

  1. Configure GitHub integration in `app-config.yaml`:

     ```yaml
     integrations:
       github:
         - host: github.com
           token: ${GITHUB_TOKEN}
     ```
  2. Validate your template locally:

     ```bash
     npx @backstage/cli validate-entity template.yaml
     ```
  3. Execute the template via the **Create Component** UI.
  4. Confirm:

     * Repo is created on GitHub
     * Catalog entry appears in Backstage
     * Skeleton files generated with correct variables
* **Tip:** Test edge cases like optional parameters and invalid input.

---

## **Lab Tips**

* Start simple: Basic Node.js project first, then add features.
* Use conditional Nunjucks templates for optional functionality.
* Keep skeleton and parameters organized for readability.
* Document steps in README.md in skeleton to guide developers.

---




# 5 **Erweiterte Scaffolder-Muster**

Du hast die grundlegenden Aktionen gelernt – jetzt erkunden wir erweiterte Muster, die Templates leistungsfähig und flexibel machen. Diese Lektion behandelt die bedingte Ausführung, die Verkettung von Schritt-Ausgaben und den Aufbau vollständiger mehrstufiger Workflows.

---

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
documentation

**Writing Custom Actions**
documentation

**Template Writing Guide**
documentation

---

## Kursnotizen (Privat)

Mache dir private Notizen, während du lernst…

## Lerngruppe

Zu diesem Kurs ist noch keine Lerngruppe verknüpft.

Bitte deinen Kursleiter, eine Lerngruppe zu verknüpfen, um Diskussionen zu ermöglichen.














