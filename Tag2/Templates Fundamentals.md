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






If you want, I can make a **visual diagram showing the full template workflow**—from **template creation → GitHub integration → automated discovery → catalog registration → developer usage**—that fits neatly into your lab materials.








