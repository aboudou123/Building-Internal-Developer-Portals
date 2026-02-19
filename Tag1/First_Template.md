
# Projektbeschreibung

**Projektname:** Internal Developer Platform (Platform Engineering Initiative)

**Beschreibung:**
Dieses Projekt implementiert eine vollständige **Internal Developer Platform (IDP)** auf Basis von **Backstage**, um Entwicklerteams Self-Service-Funktionen, Transparenz und standardisierte Entwicklungsprozesse bereitzustellen.

Ziel des Projekts ist es, eine zentrale Plattform zu schaffen, die Services, Dokumentation, CI/CD, Kubernetes-Workloads und GitOps-Deployments an einem Ort vereint. Entwickler können neue Projekte über sogenannte *Golden Path Templates* erstellen, Deployments verfolgen und ihre Services inklusive Dokumentation eigenständig verwalten.

Die Motivation hinter diesem Projekt ist die Reduzierung von kognitiver Last, die Standardisierung von Entwicklungsprozessen sowie die Beschleunigung von Onboarding und Delivery-Zyklen.

**Was Benutzer erwarten können:**

* Zentrales Developer Portal mit Software-Katalog
* Self-Service-Projekterstellung mit CI/CD
* Kubernetes-Workload-Transparenz
* GitOps-Integration
* Dokumentation-as-Code
* Single Sign-On mit rollenbasierter Zugriffskontrolle
* Produktionsnahe Setup-Struktur (lokal, Docker, Kubernetes)

---

## Technologien und Werkzeuge

Die folgenden Technologien und Tools wurden im Projekt eingesetzt:

### Core Platform

* **Backstage** – Developer Portal Framework
* **TypeScript** – Plugin- und Backend-Entwicklung
* **Node.js** – Backend Runtime
* **React** – Frontend & Plugin-Entwicklung

### Container & Deployment

* **Docker** – Containerisierung (Multi-Stage Builds)
* **Kubernetes** – Orchestrierung & Workload-Management
* **GitHub Container Registry** – Image Storage
* **Argo CD** – GitOps Deployment Visibility

### CI/CD & SCM

* **GitHub** – Source Control & Repository Discovery
* **GitHub Actions** – CI/CD Pipelines

### Authentifizierung & Sicherheit

* **Keycloak** – OIDC Authentication & SSO

### Dokumentation

* **TechDocs** – Documentation-as-Code
* **MkDocs** – Dokumentationsgenerierung

---

## Hauptmerkmale

### 1️⃣ Vollständiger Software Catalog

* Verwaltung von Services, APIs, Teams und Systemen
* Automatische GitHub Repository Discovery
* Katalog-Validierung & Troubleshooting

### 2️⃣ Golden Path Software Templates

* Vordefinierte Produktions-Templates
* Automatische Erstellung von Repositories
* CI/CD Pipeline inklusive Container-Build & Deployment
* GitHub Actions Workflows mit sicheren Registry-Permissions

### 3️⃣ Kubernetes & Workload Visibility

* Kubernetes Plugin Konfiguration
* Mapping von Catalog-Entities zu Kubernetes Ressourcen via Annotation
* Namespace-Isolation für Multi-Tenant-Cluster

### 4️⃣ GitOps Integration

* ArgoCD Integration für Deployment-Transparenz
* Git als Single Source of Truth
* End-to-End Self-Service Workflow

### 5️⃣ Dokumentation-as-Code

* TechDocs Integration
* Automatische Generierung mit MkDocs
* Versionskontrollierte Entwicklerdokumentation

### 6️⃣ Authentication & Authorization

* OIDC Login via Keycloak
* Automatische Benutzer- und Gruppensynchronisation
* Grundlegende Permission-Modelle

### 7️⃣ Erweiterbarkeit durch Plugins

* Entwicklung eigener React-Plugins
* Material-UI Theming & UI Customization
* Anpassung an Unternehmensanforderungen

### 8️⃣ Produktionsreife Architektur

* Multi-Stage Docker Builds für Sicherheit
* Deployment in dedizierte Kubernetes Namespaces
* Replizierbare Produktionsumgebung (lokal, VM, Cloud)

---

## Business Value

* Reduzierung der Developer Cognitive Load
* Schnellere Projektinitialisierung
* Standardisierte CI/CD & Deployment-Prozesse
* Transparenz über Service Ownership
* Verbesserte Developer Experience (DX)

---

---

### Schritt 1: Überprüfen, ob Backstage läuft

Überprüfen Sie den Status des Dienstes:

```bash
systemctl status backstage
```

<img width="838" height="680" alt="image" src="https://github.com/user-attachments/assets/420bdbc8-ea5f-4e0f-b6df-4bd0c89c5f17" />

# Ergebnis
<img width="1095" height="970" alt="image" src="https://github.com/user-attachments/assets/9004d71a-bdee-4fb0-b9fe-c1c8e2dc74d4" />


Drücken Sie **q**, um die Statusansicht zu verlassen. Klicken Sie im Backstage-App-Tab in der Seitenleiste auf **Create**, um den Bereich **Software Templates** anzuzeigen.


<img width="1072" height="731" alt="image" src="https://github.com/user-attachments/assets/4a29d274-db4f-4b2b-ae95-ad40b856731f" />



### Schritt 2: Template-Struktur untersuchen

Untersuchen Sie die Standard-Templates, um deren Struktur zu verstehen. Suchen Sie nach `template.yaml`-Dateien:

```bash
find /root/labs/developer-portal -name "template.yaml" -o -name "*.yaml" | grep -i template
```
<img width="1308" height="328" alt="image" src="https://github.com/user-attachments/assets/6bdebbc8-12fa-42e7-bab9-7154dcd9aa3d" />



### Schritt 3: Template-Komponenten analysieren

Erstellen Sie eine Verzeichnisstruktur zum Studium der Templates:

```bash
mkdir -p /root/labs/templates-study
```

Navigieren Sie in das Studienverzeichnis:

```bash
cd /root/labs/templates-study
```

Laden Sie ein Beispiel-Template herunter, um die Struktur zu verstehen:

```bash
curl -o sample-template.yaml https://raw.githubusercontent.com/backstage/software-templates/main/scaffolder-templates/react-ssr-template/template.yaml
```

<img width="1463" height="306" alt="image" src="https://github.com/user-attachments/assets/5d9efa88-7fb6-4042-b1f0-c896cf95e516" />


Untersuchen Sie die Template-Struktur:

```bash
cat sample-template.yaml
```
<img width="828" height="492" alt="image" src="https://github.com/user-attachments/assets/a33b97b9-6010-42c4-b5d5-b3dd9dfea918" />

Hier die ganze yaml file

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: react-ssr-template
  title: React SSR Template
  description: Create a website powered with Next.js
  tags:
    - recommended
    - react
spec:
  owner: web@example.com
  type: website
  parameters:
    - title: Provide some simple information
      required:
        - component_id
        - owner
      properties:
        component_id:
          title: Name
          type: string
          description: Unique name of the component
          ui:field: EntityNamePicker
        description:
          title: Description
          type: string
          description: Help others understand what this website is for.
        owner:
          title: Owner
          type: string
          description: Owner of the component
          ui:field: OwnerPicker
          ui:options:
            allowedKinds:
              - Group
    - title: Choose a location
      required:
        - repoUrl
      properties:
        repoUrl:
          title: Repository Location
          type: string
          ui:field: RepoUrlPicker
          ui:options:
            allowedHosts:
              - github.com
  steps:
    - id: template
      name: Fetch Skeleton + Template
      action: fetch:template
      input:
        url: ./skeleton
        copyWithoutRender:
          - .github/workflows/*
        values:
          component_id: parameters.component_id
          description: "{{ parameters.description }}"
          destination: parameters.repoUrl
          owner: "{{ parameters.owner }}"
root@patrickaboudou-backstage-dev-wrf:~/labs/templates-study# 

### Schritt 4: Template-Komponenten identifizieren

Templates bestehen aus mehreren zentralen Bestandteilen:

* **Metadata**: Name, Beschreibung, Tags
* **Spec.parameters**: Formularfelder für Benutzereingaben
* **Spec.steps**: Aktionen, die während des Scaffoldings ausgeführt werden
* **Spec.output**: Links und Informationen, die nach der Erstellung angezeigt werden

## Was sind Software-Templates in Backstage?

Software-Templates (auch **Scaffolder** genannt) helfen Entwicklerinnen und Entwicklern, neue Projekte schnell zu erstellen und dabei organisatorische Standards einzuhalten. Sie lösen mehrere Herausforderungen:

- **Konsistenz**: Jeder neue Service folgt denselben Mustern und enthält die erforderlichen Dateien  
- **Geschwindigkeit**: Entwickler beginnen nicht bei null, sondern erhalten eine funktionierende Grundlage  
- **Einhaltung von Standards**: Templates binden Best Practices, Sicherheitsanforderungen und Tooling ein  
- **Reduzierte kognitive Last**: Entwickler konzentrieren sich auf die Business-Logik statt auf Boilerplate-Setup  

Man kann sich Templates als „Baupläne“ oder „Cookiecutter“-Werkzeuge vorstellen, die mit nur wenigen Formulareingaben vollständige, sofort einsatzbereite Projekte generieren.

==========================================
---

## Schritt 1: Verzeichnisstruktur für das Template erstellen

Erstellen Sie zunächst die grundlegende Struktur für Ihr Template. Im Terminal-Tab:

```bash
cd /root/labs/developer-portal
```
<img width="1591" height="503" alt="image" src="https://github.com/user-attachments/assets/af28e554-6025-4d65-ac56-9f5cbf72b8f4" />


## Erstellen Sie ein Templates-Verzeichnis:

```bash
mkdir -p templates/nodejs-service-template
```

Navigieren Sie in Ihr Template-Verzeichnis:

```bash
cd templates/nodejs-service-template
```

<img width="912" height="753" alt="image" src="https://github.com/user-attachments/assets/47107061-fa25-4feb-8077-f6e2b6f80a29" />


Erstellen Sie Verzeichnisse für die Template-Komponenten:

```bash
mkdir -p skeleton docs
```


### Erklärung der Verzeichnisstruktur:

* `templates/`: Hier befinden sich alle Templates der Organisation
* `nodejs-service-template/`: Der Ordner für dieses spezifische Template
* `skeleton/`: Template-Dateien, die kopiert und verarbeitet werden
* `docs/`: Dokumentation zur Verwendung dieses Templates

---

## Schritt 2: Template-Metadaten definieren

Erstellen Sie die Template-Definitionsdatei. Erstellen Sie im Code-Editor-Tab eine Datei mit dem Namen `template.yaml` im Verzeichnis `nodejs-service-template`:

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: nodejs-service-template
  title: Node.js Microservice Template
  description: Create a new Node.js microservice with Express, testing, and Docker support
  tags:
    - nodejs
    - microservice
    - express
    - recommended
  links:
    - url: https://docs.company.com/templates/nodejs
      title: Node.js Service Standards
      icon: docs
    - url: https://github.com/company/nodejs-template-examples
      title: Example Services
      icon: github
spec:
  owner: platform-team
  type: service
  # Parameters, steps, and output will be added in subsequent tasks
```

Speichern Sie diese Datei als

`/root/labs/developer-portal/templates/nodejs-service-template/template.yaml`.

Wie mache ich das?

```bash
yamllint template.yaml
```

Dann

```bash
Ctrl + O
```

Enter
```bash
Ctrl + X
```

## Prüfen, ob die Datei gespeichert wurde


```bash
ls
```
oder
```bash
cat template.yaml
```

<img width="1011" height="196" alt="image" src="https://github.com/user-attachments/assets/fe4e2cf5-03ce-4178-9ff4-399fa36d8c8f" />


### Erklärung der wichtigsten Metadatenfelder:

* **apiVersion**: Verwendet die Scaffolder-API-Version von Backstage (immer `scaffolder.backstage.io/v1beta3`)
* **kind: Template**: Kennzeichnet dies als Software-Template-Entität
* **metadata.name**: Eindeutiger Bezeichner für dieses Template (wird in URLs und Referenzen verwendet)
* **metadata.title**: Für Menschen lesbarer Name, der im Template-Katalog angezeigt wird
* **metadata.description**: Kurze Beschreibung dessen, was dieses Template erstellt
* **metadata.tags**: Labels für Filterung und Auffindbarkeit im Template-Katalog
* **metadata.links**: Hilfreiche Ressourcen zu diesem Template
* **spec.owner**: Team oder Person, die für die Pflege dieses Templates verantwortlich ist
* **spec.type**: Kategorie dessen, was dieses Template erstellt (Service, Website, Bibliothek usw.)

---

## Schritt 3: Template-Tags und Auffindbarkeit verstehen

Tags helfen Entwicklerinnen und Entwicklern, das richtige Template für ihre Anforderungen zu finden. Wählen Sie Tags, die Folgendes widerspiegeln:

* **Technologie**: nodejs, react, python, java
* **Architektur**: microservice, monolith, serverless, api
* **Zweck**: frontend, backend, library, tool
* **Reifegrad**: recommended, experimental, deprecated

Aktualisieren Sie Ihr Template, um spezifischere Tags einzubinden:

```yaml
metadata:
  name: nodejs-service-template
  title: Node.js Microservice Template
  description: Create a new Node.js microservice with Express, testing, and Docker support
  tags:
    - nodejs
    - microservice
    - express
    - rest-api
    - docker
    - typescript
    - recommended
    - backend
```

Wie das  **Update** in meiner `template.yaml` (ersetzen/erweitern) gemacht wird.


<img width="877" height="1071" alt="image" src="https://github.com/user-attachments/assets/1837bb47-4a31-48f6-83ec-006db2cf2f56" />



### 1) Datei öffnen

```bash
nano template.yaml
```

### 2) In `metadata:` den `tags:` Block so ändern

Ersetze das hier:

```yaml
tags:
  - nodejs
  - microservice
  - express
  - recommended
```

durch das hier:

```yaml
tags:
  - nodejs
  - microservice
  - express
  - rest-api
  - docker
  - typescript
  - recommended
  - backend
```

### 3) Änder/Update -Speichern und prüfen


```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: nodejs-service-template
  title: Node.js Microservice Template
  description: Create a new Node.js microservice with Express, testing, and Docker support
  tags:
    - nodejs
    - microservice
    - express
    - rest-api
    - docker
    - typescript
    - recommended
    - backend
  links:
    - url: https://docs.company.com/templates/nodejs
      title: Node.js Service Standards
      icon: docs
    - url: https://github.com/company/nodejs-template-examples
      title: Example Services
      icon: github
spec:
  owner: platform-team
  type: service
  # Parameters, steps, and output will be added in subsequent tasks
```

Nano: **CTRL+O**, Enter, **CTRL+X**

Dann prüfen:

```bash
cat template.yaml
```

Fertig.


So machst du **Step 4: Template Documentation** sauber und ohne Fehler – genau in deinem Ordner-Setup (`docs/` existiert schon).

## Ziel

Du sollst eine Datei erstellen:

`/root/labs/developer-portal/templates/nodejs-service-template/docs/README.md`

mit dem Inhalt, den dein Lehrer gegeben hat.

---

## 1) Prüfen, dass du im richtigen Ordner bist

```bash
pwd
ls
```

Du solltest sowas sehen:

* `docs`
* `skeleton`
* `template.yaml`

Wenn du im falschen Ordner bist, geh in deinen Template-Ordner:

```bash
cd /root/labs/developer-portal/templates/nodejs-service-template
```

---

## 2) README im docs-Verzeichnis erstellen/bearbeiten

Öffne die Datei mit nano (sie wird erstellt, falls sie noch nicht existiert):

```bash
nano docs/README.md
```

---

## 3) Inhalt einfügen (genau so)

Kopiere **alles** hier rein:

```md
# Node.js Microservice Template

This template creates a production-ready Node.js microservice with industry best practices built-in.

## What's Included

- **Express.js** web framework with TypeScript
- **Jest** testing framework with coverage reports
- **Docker** containerization with multi-stage builds
- **ESLint** and **Prettier** for code quality
- **GitHub Actions** CI/CD pipeline
- **Swagger/OpenAPI** documentation
- **Health check** endpoints for monitoring

## When to Use This Template

Use this template when you need to create:
- REST APIs and microservices
- Backend services that integrate with other systems
- Services that need to be containerized and deployed to Kubernetes
- APIs that require formal documentation and testing

## Don't Use This Template For

- Frontend applications (use the React template instead)
- Serverless functions (use the Lambda template)
- Data processing jobs (use the Data Pipeline template)
- Static websites or documentation sites

## After Generation

1. Review the generated `README.md` for service-specific setup
2. Update the OpenAPI specification in `docs/api.yaml`
3. Add your business logic to `src/routes/`
4. Run tests with `npm test` 
5. Start development with `npm run dev`

## Support

- **Template owner:** Platform Team
- **Documentation:** https://docs.company.com/templates/nodejs
- **Issues:** Contact platform-team@company.com
```

---

## 4) Speichern und schließen (Nano)

* Speichern: **CTRL + O**
* Enter drücken
* Schließen: **CTRL + X**

---

## 5) Kontrollieren, dass es wirklich gespeichert wurde

```bash
ls -la docs
cat docs/README.md
```

Wenn `cat` den Text ausgibt, ist alles korrekt.

---

## Alternative (schneller, ohne Editor)

Wenn du lieber **ohne nano** direkt erstellen willst, nutze das hier:

```bash
cat > docs/README.md <<'EOF'
# Node.js Microservice Template

This template creates a production-ready Node.js microservice with industry best practices built-in.

## What's Included

- **Express.js** web framework with TypeScript
- **Jest** testing framework with coverage reports
- **Docker** containerization with multi-stage builds
- **ESLint** and **Prettier** for code quality
- **GitHub Actions** CI/CD pipeline
- **Swagger/OpenAPI** documentation
- **Health check** endpoints for monitoring

## When to Use This Template

Use this template when you need to create:
- REST APIs and microservices
- Backend services that integrate with other systems
- Services that need to be containerized and deployed to Kubernetes
- APIs that require formal documentation and testing

## Don't Use This Template For

- Frontend applications (use the React template instead)
- Serverless functions (use the Lambda template)
- Data processing jobs (use the Lambda template)
- Static websites or documentation sites

## After Generation

1. Review the generated `README.md` for service-specific setup
2. Update the OpenAPI specification in `docs/api.yaml`
3. Add your business logic to `src/routes/`
4. Run tests with `npm test` 
5. Start development with `npm run dev`

## Support

- **Template owner:** Platform Team
- **Documentation:** https://docs.company.com/templates/nodejs
- **Issues:** Contact platform-team@company.com
EOF
```

<img width="887" height="1075" alt="image" src="https://github.com/user-attachments/assets/698b95ca-70b5-42a8-91ef-e6a0ef4f3119" />


Danach prüfen:

```bash
cat docs/README.md
```

---

<img width="909" height="966" alt="image" src="https://github.com/user-attachments/assets/5dfd30ee-b023-4ae7-9d27-825732b34ebc" />


### Tag-Strategie:

* **nodejs, express, typescript**: Technologie-Stack
* **microservice, rest-api, backend**: Architektur und Zweck
* **docker**: Enthaltene Deployment-Methode
* **recommended**: Bevorzugtes und empfohlenes Template

---

## Schritt 4: Template-Dokumentation hinzufügen

Erstellen Sie eine Dokumentation für die Template-Nutzer. Erstellen Sie im Code-Editor-Tab eine README-Datei im `docs`-Verzeichnis:

```md
# Node.js Microservice Template

This template creates a production-ready Node.js microservice with industry best practices built-in.

## What's Included

- **Express.js** web framework with TypeScript
- **Jest** testing framework with coverage reports
- **Docker** containerization with multi-stage builds
- **ESLint** and **Prettier** for code quality
- **GitHub Actions** CI/CD pipeline
- **Swagger/OpenAPI** documentation
- **Health check** endpoints for monitoring

## When to Use This Template

Use this template when you need to create:
- REST APIs and microservices
- Backend services that integrate with other systems
- Services that need to be containerized and deployed to Kubernetes
- APIs that require formal documentation and testing

## Don't Use This Template For

- Frontend applications (use the React template instead)
- Serverless functions (use the Lambda template)
- Data processing jobs (use the Data Pipeline template)
- Static websites or documentation sites

## After Generation

1. Review the generated `README.md` for service-specific setup
2. Update the OpenAPI specification in `docs/api.yaml`
3. Add your business logic to `src/routes/`
4. Run tests with `npm test` 
5. Start development with `npm run dev`

## Support

- **Template owner:** Platform Team
- **Documentation:** https://docs.company.com/templates/nodejs
- **Issues:** Contact platform-team@company.com
```

Speichern Sie diese Datei als
`/root/labs/developer-portal/templates/nodejs-service-template/docs/README.md`.

---
## Ergebnis

<img width="909" height="966" alt="image" src="https://github.com/user-attachments/assets/5dfd30ee-b023-4ae7-9d27-825732b34ebc" />



## Template-Eigentümerschaft und Lebenszyklus verstehen

Die Eigentümerschaft von Templates ist entscheidend für die Wartung:

* **Owner-Team**: Verantwortlich für die Aktualisierung des Templates
* **Versionierung**: Templates sollten sich mit den organisatorischen Standards weiterentwickeln
* **Stilllegung**: Alte Templates benötigen klare Migrationspfade
* **Dokumentation**: Nutzer benötigen klare Hinweise, wann und wie Templates zu verwenden sind

### Template-Lebenszyklus:

* **Erstellung**: Das Plattform-Team erstellt Templates basierend auf den Anforderungen der Organisation
* **Testen**: Das Template wird mit realen Projekten validiert
* **Freigabe**: Das Template wird nach erfolgreicher Nutzung als „recommended“ eingestuft
* **Wartung**: Regelmäßige Updates für Sicherheit, Abhängigkeiten und Standards
* **Weiterentwicklung**: Neue Funktionen basierend auf Nutzerfeedback
* **Ablösung**: Wird schließlich durch bessere Ansätze ersetzt

Im nächsten Schritt erstellen Sie das Benutzereingabeformular (Parameter-Sektion), mit dem Informationen von Entwicklern gesammelt werden, um das generierte Projekt zu individualisieren.

Sie haben erfolgreich Template-Metadaten definiert, die Ihr Template auffindbar machen und seinen Zweck klar kommunizieren. Diese Grundlage stellt sicher, dass Entwickler Ihr Template finden und verstehen, bevor sie es verwenden.

---

# **Erstellen des Benutzer-Eingabeformulars**

**Erstellen des Benutzer-Eingabeformulars**
In dieser Aufgabe wird der Abschnitt „Parameter“ des Software-Templates erstellt, der das Formular definiert, das Entwickler ausfüllen, wenn sie das Template verwenden. Dieses Formular sammelt die Informationen, die benötigt werden, um das generierte Projekt an die spezifischen Bedürfnisse der Entwickler anzupassen.

### Warum benötigen Templates Benutzer-Eingabeformulare?

---

### Verständnis der Parameterstruktur

#### Schritt 1: Erstellen von Feldern für grundlegende Projektinformationen

#### Schritt 2: Hinzufügen von Repository- und Integrations-Einstellungen

#### Schritt 3: Hinzufügen von technischen Konfigurationsoptionen


---

### Erstellen des Benutzer-Eingabeformulars

In dieser Aufgabe erstellst du den Parameterbereich deines Software-Templates. Dieser definiert das Formular, das Entwickler ausfüllen, wenn sie dein Template verwenden. Das Formular sammelt die Informationen, die benötigt werden, um das generierte Projekt an die jeweiligen Anforderungen anzupassen.

---

### Warum benötigen Templates Benutzer-Eingabeformulare?

Benutzer-Eingabeformulare machen Templates flexibel und wiederverwendbar, da sie eine individuelle Anpassung ermöglichen:

* **Projektbenennung:** Jedes Projekt benötigt eindeutige Namen und Kennungen
* **Konfigurationsoptionen:** Unterschiedliche Teams benötigen möglicherweise unterschiedliche aktivierte Funktionen
* **Repository-Einrichtung:** Projekte benötigen unterschiedliche Git-Repositories und Zuständigkeiten
* **Umgebungseinstellungen:** Entwicklungs- und Produktionskonfigurationen können variieren
* **Integrationsentscheidungen:** Teams können unterschiedliche Datenbanken, CI/CD-Werkzeuge oder Cloud-Anbieter verwenden

Das Formular übersetzt die Benutzerauswahl in Variablen, die den generierten Code anpassen, sodass ein einziges Template für viele verschiedene Anwendungsfälle genutzt werden kann.

---

### Verständnis der Parameterstruktur

Backstage-Template-Parameter verwenden JSON Schema zur Definition der Formularfelder. Dies bietet:

* **Typvalidierung:** Stellt sicher, dass Benutzer geeignete Datentypen eingeben
* **Benutzererlebnis:** Umfangreiche Formularsteuerelemente wie Dropdown-Menüs, Kontrollkästchen und Textbereiche
* **Dokumentation:** Integrierte Hilfetexte und Feldbeschreibungen
* **Bedingte Logik:** Ein- oder Ausblenden von Feldern basierend auf anderen Auswahlen

---

### Schritt 1: Erstellen grundlegender Felder für Projektinformationen

Aktualisiere dein Template, um Benutzer-Eingabeparameter hinzuzufügen. Öffne im Reiter „Code Editor“ die Datei `nodejs-service-template/template.yaml` und füge den Parameterbereich nach den Metadaten hinzu.

---

Aktualisiere dein Template, um Benutzer-Eingabeparameter hinzuzufügen. Öffne im Reiter „Code Editor“ die Datei `nodejs-service-template/template.yaml` und füge den Parameterbereich nach den Metadaten hinzu:

```
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: nodejs-service-template
  title: Node.js Microservice Template
  description: |
    Creates a production-ready Node.js microservice with Express, testing, and Docker support.
    
    ## What this template creates
    
    - **Express.js API** with TypeScript for type safety
    - **Testing setup** with Jest and Supertest
    - **Docker configuration** for containerized deployment  
    - **GitHub Actions** CI/CD pipeline
    - **OpenAPI documentation** with Swagger UI
    - **Health check endpoints** for monitoring
    - **ESLint and Prettier** for code quality
  
  tags:
    - nodejs
    - microservice
    - express
    - rest-api
    - docker
    - typescript
    - recommended
    - backend
    - api
  
  links:
    - url: https://expressjs.com/
      title: Express.js Documentation
      icon: web
    - url: https://github.com/your-org/nodejs-examples
      title: Example Services
      icon: github

spec:
  owner: platform-team
  type: service
  system: developer-tools
  
  # User input form definition
  parameters:
    - title: Project Information
      required:
        - name
        - description
        - owner
      properties:
        name:
          title: Service Name
          type: string
          description: Name of your new microservice (will be used for repository and package names)
          pattern: '^[a-z0-9-]+$'
          minLength: 3
          maxLength: 50
          ui:autofocus: true
          ui:help: 'Use lowercase letters, numbers, and hyphens only. Example: user-auth-service'
          ui:placeholder: 'my-awesome-service'
        
        description:
          title: Service Description
          type: string
          description: Brief description of what this service does
          minLength: 10
          maxLength: 200
          ui:widget: textarea
          ui:options:
            rows: 3
          ui:help: 'This will appear in README files and service documentation'
          ui:placeholder: 'This service handles user authentication and session management'
        
        owner:
          title: Team Owner
          type: string
          description: Which team will own and maintain this service
          enum:
            - platform-team
            - frontend-team
            - backend-team
            - data-team
            - devops-team
          enumNames:
            - Platform Team
            - Frontend Team  
            - Backend Team
            - Data Team
            - DevOps Team
          ui:help: 'This determines who gets notified about service issues and changes'
```

<img width="1066" height="498" alt="image" src="https://github.com/user-attachments/assets/09ad34dd-ac18-420a-b324-8674379d0b56" />


<img width="1072" height="898" alt="image" src="https://github.com/user-attachments/assets/92f2749a-a61c-4ce2-a970-e8f78fe39988" />


<img width="970" height="1028" alt="image" src="https://github.com/user-attachments/assets/4632d54c-c230-466a-88ab-d08089bc8d4b" />


---

### Erklärung der Formularfeldtypen:

**string mit pattern:** Texteingabe mit Validierung mittels regulärem Ausdruck (der Servicename muss URL-sicher sein)

**string mit minLength/maxLength:** Texteingabe mit Längenbeschränkungen

**Textarea-Widget:** Mehrzeilige Texteingabe für längere Beschreibungen

**Enum-Dropdown:** Vordefinierte Auswahlmöglichkeiten aus einer Liste (Teamzuordnung, Datenbanktyp usw.)

**Boolean-Kontrollkästchen:** Ja/Nein-Umschalter für Funktionsoptionen (CI/CD aktivieren, Swagger einbinden usw.)

**minLength/maxLength:** Stellt angemessene Eingabelängen sicher

**Integer-Eingabe:** Numerische Werte mit Minimal- und Maximalgrenzen (z. B. Portnummern)

**Standardwerte:** Felder werden mit sinnvollen Standardwerten vorausgefüllt

**Template-Ausdrücke:** `{{ parameters.fieldName }}` zur Referenzierung anderer Formularwerte

**Pflichtfelder:** Diese Felder müssen ausgefüllt sein, bevor das Template verwendet werden kann

**ui:help:** Hilfreicher Anleitungstext für Benutzer

**ui:placeholder:** Beispieltext, der die erwartete Eingabe zeigt

**ui:autofocus:** Setzt den Cursor beim Laden des Formulars automatisch auf dieses Feld

**ui:widget:** Anpassung der Darstellung eines Feldes (z. B. Textarea für Zeichenketten)

---
 **Step 2** hinzufügen: eine **zweite Parameter-Sektion** („Repository and Integration“) in `spec.parameters`, ohne Einrückungsfehler.

## Schritt 1: Datei öffnen

```bash
cd /root/labs/developer-portal/templates/nodejs-service-template
nano template.yaml
```

## Schritt 2: `parameters:` komplett ersetzen (Step 1 + Step 2 zusammen)

Suche in `spec:` den Block `parameters:` und ersetze ihn **komplett** durch diesen korrekten Block:

```yaml
  parameters:
    - title: Project Information
      required:
        - name
        - description
        - owner
      properties:
        name:
          title: Service Name
          type: string
          description: Name of your new microservice (will be used for repository and package names)
          pattern: '^[a-z0-9-]+$'
          minLength: 3
          maxLength: 50
          ui:autofocus: true
          ui:help: 'Use lowercase letters, numbers, and hyphens only. Example: user-auth-service'
          ui:placeholder: 'my-awesome-service'

        description:
          title: Service Description
          type: string
          description: Brief description of what this service does
          minLength: 10
          maxLength: 200
          ui:widget: textarea
          ui:options:
            rows: 3
          ui:help: 'This will appear in README files and service documentation'
          ui:placeholder: 'This service handles user authentication and session management'

        owner:
          title: Team Owner
          type: string
          description: Which team will own and maintain this service
          enum:
            - platform-team
            - frontend-team
            - backend-team
            - data-team
            - devops-team
          enumNames:
            - Platform Team
            - Frontend Team
            - Backend Team
            - Data Team
            - DevOps Team
          ui:help: 'This determines who gets notified about service issues and changes'

    - title: Repository and Integration
      required:
        - github_owner
        - repository_name
      properties:
        github_owner:
          title: GitHub Owner
          type: string
          description: Your GitHub username or organization name where the repository will be created
          pattern: '^[a-z0-9-]+$'
          minLength: 1
          maxLength: 39
          ui:help: 'Example: my-github-username or my-company-org. The repository will be created at github.com/[owner]/[repository-name]'
          ui:placeholder: 'your-github-username'

        repository_name:
          title: Repository Name
          type: string
          description: GitHub repository name (usually same as service name)
          pattern: '^[a-z0-9-]+$'
          ui:help: 'Combined with GitHub Owner above to create: github.com/[owner]/[repository-name]'
          ui:placeholder: 'my-awesome-service'

        enable_cicd:
          title: Enable CI/CD Pipeline
          type: boolean
          description: Include GitHub Actions workflow for automated testing and deployment
          default: true
          ui:help: 'Recommended for all services. Creates automated testing on every commit.'

        database_type:
          title: Database Integration
          type: string
          description: Which database will this service use
          enum:
            - none
            - postgresql
            - mysql
            - mongodb
          enumNames:
            - No Database
            - PostgreSQL
            - MySQL
            - MongoDB
          default: postgresql
          ui:help: 'PostgreSQL is recommended for most applications. Choose "none" for stateless services.'
```

**Wichtig zur Einrückung:**

* `parameters:` hat **2 Leerzeichen** voran (weil es unter `spec:` steht).
* Jede Sektion beginnt mit `- title: ...` (auch 4 Leerzeichen vor `-`).

## Schritt 3: Speichern

Nano:

* **CTRL + O**, Enter
* **CTRL + X**

<img width="1110" height="1067" alt="image" src="https://github.com/user-attachments/assets/d9b3cff9-34ed-42ce-8aeb-5641b1b92d3b" />


<img width="1116" height="1057" alt="image" src="https://github.com/user-attachments/assets/bb8b8830-90e9-4019-bbdd-08404097e3cf" />

<img width="1143" height="1029" alt="image" src="https://github.com/user-attachments/assets/2c3e37c1-0ec8-4b24-a6a9-7f995c20bb98" />

<img width="1127" height="1020" alt="image" src="https://github.com/user-attachments/assets/48042cde-e4d0-46b3-afdb-f55528b40885" />


## Schritt 4: YAML prüfen (muss OK sein)

```bash
python3 - <<'PY'
import yaml
yaml.safe_load(open("template.yaml","r",encoding="utf-8"))
print("OK: template.yaml ist gültig")
PY
```



## Ergebniss

<img width="1219" height="949" alt="image" src="https://github.com/user-attachments/assets/d330c691-2134-49a7-a447-e59c8fa1e72e" />



<img width="1067" height="854" alt="image" src="https://github.com/user-attachments/assets/d510a683-f73e-4379-bb4b-e6925d52f7fb" />




<img width="1234" height="970" alt="image" src="https://github.com/user-attachments/assets/0905dbe4-6a92-42b5-8145-e3da57120175" />




<img width="1337" height="998" alt="image" src="https://github.com/user-attachments/assets/5fc35bae-714c-4504-9908-acd1061b309c" />




---

**Datei aussehen**

---


```yaml
# host: 127.0.0.1
```
###  ändern zu

```yaml
host: 0.0.0.0
```

👉 `0.0.0.0` = akzeptiert Verbindungen von außen
👉 `127.0.0.1` = nur lokal

---



```yaml
origin: http://localhost:3000
```

###  ändern zu

```yaml
origin: http://95.217.214.89:3000
```
---

**Finale korrekte Version**

So sollte es am Ende aussehen:

```yaml
backend:
  baseUrl: http://95.217.214.89:7007
  listen:
    port: 7007
    host: 0.0.0.0

  cors:
    origin: http://95.217.214.89:3000
    methods: [GET, HEAD, PATCH, POST, PUT, DELETE]
    credentials: true
```

---

#  **Danach neu starten**

Im Projektordner:

```bash
CTRL + C   # falls läuft stoppen
yarn dev
```

oder

```bash
yarn start
```

---

#  **im Browser öffnen**

```
http://95.217.214.89:3000
```

---

#  Kurz erklärt warum

| Setting               | Warum                            |
| --------------------- | -------------------------------- |
| host: 0.0.0.0         | Server von außen erreichbar      |
| VM-IP statt localhost | Browser kann verbinden           |
| CORS origin           | Frontend darf Backend ansprechen |

---


---

# **Anleitung: Rundgang durch die Projektstruktur**

Bei laufendem **Backstage** erkunden wir die Projektstruktur, um zu verstehen, wie eine **Backstage**-Anwendung organisiert ist. Dieses Wissen ist für die Anpassung und Erweiterung Ihres Portals unerlässlich.

## **Schritt 1: Root-Projektstruktur erkunden**
Beginnen Sie mit der Untersuchung der Hauptprojektverzeichnisse:

```bash
ls -la /root/labs/developer-portal/
```

<img width="1051" height="867" alt="image" src="https://github.com/user-attachments/assets/f79100ba-324b-45c9-9066-e66a097fadaf" />

Die wichtigsten Komponenten sind:

- **`app-config.yaml`** - Zentrale Konfigurationsdatei
- **`packages/`** - Enthält den Anwendungscode (Frontend und Backend)
- **`catalog-info.yaml`** - Metadaten über diese **Backstage**-Instanz selbst
- **`package.json`** - **Node.js**-Abhängigkeiten und Skripte

Dies folgt dem standardmäßigen **Backstage**-Projektaufbau, was es für andere **Backstage**-Entwickler vertraut macht.

## **Schritt 2: Die Monorepo-Architektur entdecken**
Listen Sie das `packages`-Verzeichnis auf, um die Code-Organisation zu verstehen:

```bash
ls -la /root/labs/developer-portal/packages/
```

<img width="1039" height="803" alt="image" src="https://github.com/user-attachments/assets/d0856b86-05a3-4edc-b9eb-604ff500f902" />



<img width="1039" height="803" alt="image" src="https://github.com/user-attachments/assets/5119d2d9-7e75-4b2c-9e94-09feffb6357c" />


<img width="1058" height="828" alt="image" src="https://github.com/user-attachments/assets/1efde67e-4171-40ad-929c-f61c70a6d621" />




<img width="1005" height="878" alt="image" src="https://github.com/user-attachments/assets/d65939a7-661d-409e-81ea-2fb6ad790c73" />


<img width="981" height="468" alt="image" src="https://github.com/user-attachments/assets/2a084b98-0a7a-47f7-b1af-13bca0126943" />



# Das Verzeichnis `packages` enthält:

- **`app`**: Die React-Frontend-Anwendung, mit der Benutzer interagieren
- **`backend`**: Der **Node.js**-**API**-Dienst, der das Portal betreibt

Diese Monorepo-Struktur ermöglicht es Teams, Frontend und Backend unabhängig voneinander zu entwickeln und dennoch synchron zu halten.

## **Schritt 3: Struktur der Frontend-Anwendung untersuchen**
Erkunden Sie, aus welchen Teilen die Benutzeroberfläche besteht:

```bash
ls -la /root/labs/developer-portal/packages/app/src/
```

<img width="1005" height="878" alt="image" src="https://github.com/user-attachments/assets/e398d683-e87b-4363-986b-f0a16b51dab1" />

Das Frontend umfasst:

- **`App.tsx`**: Haupt-React-Komponente, die das gesamte Portal rendert
- **`components/`**: Benutzerdefinierte **UI**-Komponenten für Ihre spezifischen Anforderungen
- **`App.test.tsx`**: Automatisierte Tests zur Qualitätssicherung

Das Verständnis dieser Struktur hilft Ihnen zu wissen, wo Sie benutzerdefinierte Seiten und Komponenten hinzufügen können.

## **Schritt 4: Aufbau des Backend-Dienstes untersuchen**
Sehen Sie, wie der **API**-Dienst strukturiert ist:

```bash
ls -la /root/labs/developer-portal/packages/backend/src/
```

<img width="981" height="468" alt="image" src="https://github.com/user-attachments/assets/f90399c5-1ad8-4565-bb70-62147de2dcf6" />

Das Backend enthält:

- **`index.ts`**: Einstiegspunkt, der den Server startet und Plugins lädt
- **`plugins/`**: Konfiguration für verschiedene **Backstage**-Plugins
- **`types.ts`**: **TypeScript**-Definitionen für benutzerdefinierte Datenstrukturen

Diese Organisation macht deutlich, wo neue **API**-Endpunkte und Integrationen hinzugefügt werden können.

## **Schritt 5: Kernkonfigurationsdateien prüfen**
Betrachten Sie die Hauptkonfigurationsdatei, um zu verstehen, wie **Backstage** konfiguriert ist:

```bash
cat /root/labs/developer-portal/app-config.yaml
```

Diese Datei steuert alle Aspekte Ihrer **Backstage**-Instanz, einschließlich Datenbank, Authentifizierung und Integrationen.

```yaml
app:
  title: Scaffolded Backstage App
  baseUrl: http://95.217.214.89:3000

organization:
  name: My Company

backend:
  # Wird für die Aktivierung der Authentifizierung verwendet, das Geheimnis wird von allen Backend-Plugins geteilt
  # Siehe https://backstage.io/docs/auth/service-to-service-auth für
  # Informationen zum Format
  # auth:
  #   keys:
  #     - secret: ${BACKEND_SECRET}
  baseUrl: http://95.217.214.89:7007
  listen:
    port: 7007
    # Entfernen Sie den Kommentar der folgenden host-Direktive, um an bestimmten Schnittstellen zu binden
    # host: 127.0.0.1
  csp:
    connect-src: ["'self'", 'http:', 'https:']
    # Die Content-Security-Policy-Direktiven folgen dem Helmet-Format: https://helmetjs.github.io/#reference
    # Standardmäßige Helmet-Content-Security-Policy-Werte können entfernt werden, indem der Schlüssel auf false gesetzt wird
  cors:
    origin: http://95.217.214.89:3000
    methods: [GET, HEAD, PATCH, POST, PUT, DELETE]
    credentials: true
  # Dies ist nur für die lokale Entwicklung gedacht, es wird nicht empfohlen, dies in der Produktion zu verwenden
  # Die Produktionsdatenbankkonfiguration wird in app-config.production.yaml gespeichert
  database:
    client: better-sqlite3
    connection: ':memory:'
  # workingDirectory: /tmp # Verwenden Sie dies, um ein Arbeitsverzeichnis für den Scaffolder zu konfigurieren, standardmäßig wird das temporäre Verzeichnis des Betriebssystems verwendet

integrations:
  github:
    - host: github.com
      # Dies ist ein Personal Access Token (PAT) von GitHub. Sie können herausfinden, wie Sie dieses Token generieren, und weitere Informationen
      # zur Einrichtung der GitHub-Integration hier: https://backstage.io/docs/integrations/github/locations#configuration
      token: ${GITHUB_TOKEN}
    ### Beispiel zum Hinzufügen Ihrer GitHub Enterprise-Instanz über die API:
    # - host: ghe.example.net
    #   apiBaseUrl: https://ghe.example.net/api/v3
    #   token: ${GHE_TOKEN}

proxy:
  ### Beispiel zum Hinzufügen eines Proxy-Endpunkts für das Frontend.
  ### Ein typischer Grund dafür ist die Handhabung von HTTPS und CORS für interne Dienste.
  # endpoints:
  #   '/test':
  #     target: 'https://example.com'
  #     changeOrigin: true

# Referenzdokumentation http://backstage.io/docs/features/techdocs/configuration
# Hinweis: Verwenden Sie nach dem Experimentieren mit der Basiskonfiguration CI/CD, um Dokumente zu generieren,
# und einen externen Cloud-Speicher, wenn Sie TechDocs für den Produktionseinsatz bereitstellen.
# https://backstage.io/docs/features/techdocs/how-to-guides#how-to-migrate-from-techdocs-basic-to-recommended-deployment-approach
techdocs:
  builder: 'local' # Alternativen - 'external'
  generator:
    runIn: 'docker' # Alternativen - 'local'
  publisher:
    type: 'local' # Alternativen - 'googleGcs' oder 'awsS3'. Lesen Sie die Dokumentation für die Verwendung von Alternativen.

auth:
  # siehe https://backstage.io/docs/auth/ , um mehr über Authentifizierungsanbieter zu erfahren
  providers:
    # Siehe https://backstage.io/docs/auth/guest/provider
    guest: {}

scaffolder:
  # siehe https://backstage.io/docs/features/software-templates/configuration für Optionen von Softwarevorlagen

catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
    - allow: [Component, System, API, Resource, Location]
  locations:
    # Lokale Beispieldaten, Dateipfade sind relativ zum Backend-Prozess, typischerweise `packages/backend`
    - type: file
      target: ../../examples/entities.yaml

    # Lokale Beispielvorlage
    - type: file
      target: ../../examples/template/template.yaml
      rules:
        - allow: [Template]

    # Lokale Beispieldaten für die Organisation
    - type: file
      target: ../../examples/org.yaml
      rules:
        - allow: [User, Group]

    ## Entfernen Sie die Kommentare für diese Zeilen, um weitere Beispieldaten hinzuzufügen
    # - type: url
    #   target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/all.yaml

    ## Entfernen Sie die Kommentare für diese Zeilen, um eine Beispielorganisation hinzuzufügen
    # - type: url
    #   target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/acme-corp.yaml
    #   rules:
    #     - allow: [User, Group]

kubernetes:
  # siehe https://backstage.io/docs/features/kubernetes/configuration für Kubernetes-Konfigurationsoptionen

# siehe https://backstage.io/docs/permissions/getting-started für mehr Informationen über das Berechtigungsframework
permission:
  # wenn dies auf `false` gesetzt wird, werden Berechtigungen deaktiviert
  enabled: true
```

Untersuchen Sie auch die Katalog-Metadaten für die **Backstage**-App selbst:

```bash
cat /root/labs/developer-portal/catalog-info.yaml
```

Diese Datei beschreibt Ihre **Backstage**-Instanz als eine Entität in ihrem eigenen Katalog.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: developer-portal
  description: Ein Beispiel für eine Backstage-Anwendung.
  # Beispiel für optionale Annotationen
  # annotations:
  #   github.com/project-slug: backstage/backstage
  #   backstage.io/techdocs-ref: dir:.
spec:
  type: website
  owner: john@example.com
  lifecycle: experimental
```

---


<img width="1797" height="866" alt="image" src="https://github.com/user-attachments/assets/9701e99f-b968-4e2b-9739-be6cb281e139" />




<img width="1337" height="998" alt="image" src="https://github.com/user-attachments/assets/c500ed15-0f80-4e4c-8d7d-e69cb84e3057" />




### **© ALLE RECHTE VORBEHALTEN**

Dieses Projekt wurde von **Koffitse Aboudou** im Rahmen des Masterarbeit an der **Technischen Hochschule Deggendorf (THD)** im Auftrag der **KUKA GmbH**  realisiert.

**Nutzungshinweise:**

* Jegliche Vervielfältigung, Verbreitung oder kommerzielle Nutzung des Inhalts, der Konzepte oder der Implementierungscodes – auch auszugsweise – ist ohne ausdrückliche schriftliche Genehmigung des Urhebers und der beteiligten Institutionen untersagt.
* Die Inhalte dienen ausschließlich Dokumentations- und Prüfungszwecken im akademischen Kontext. 


**Hinweis**: Dieser Abschnitt der Arbeit stellt nur einen Teil des Gesamtprojekts dar. Das vollständige Projekt ist Eigentum des Unternehmens und daher nicht öffentlich zugänglich. Es handelt sich um ein Projekt, bei dem lediglich ein Teil veröffentlicht wird.







