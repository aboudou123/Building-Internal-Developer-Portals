
## Anweisungen

### Verständnis von Backstage-Software-Templates

Bevor Sie Ihr eigenes Template erstellen, sollten Sie vorhandene Templates untersuchen, um zu verstehen, wie der Backstage Scaffolder funktioniert und aus welchen Komponenten ein Software-Template besteht.

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



=================================================================================

Alles klar ❤️ — das ist eine **sehr gute Entscheidung**. Wir machen jetzt einen **kompletten Reset** und gehen **sauber zurück zu Step 1**, so als wäre Backstage **nie gestartet worden**.
Ich gebe dir **klare, sichere Schritte**, nichts Gefährliches, nichts Unnötiges.

---

# ✅ ZIEL

* Backstage **komplett stoppen**
* **systemd-Service entfernen**
* **alle installierten Node/Yarn/Backstage-Artefakte löschen**
* Danach **neu bei Step 1 beginnen**, exakt nach deiner Anleitung

---

## 🔴 SCHRITT 0 – Backstage komplett stoppen

```bash
systemctl stop backstage || true
systemctl disable backstage || true
```

---

## 🔴 SCHRITT 1 – systemd-Service **vollständig löschen**

```bash
rm -f /etc/systemd/system/backstage.service
systemctl daemon-reload
```

Prüfen (soll **NOT FOUND** sein):

```bash
systemctl status backstage
```

---

## 🔴 SCHRITT 2 – Backstage-Projekt **komplett löschen**

⚠️ **Das löscht ALLES in `developer-portal`**

```bash
rm -rf /root/labs/developer-portal
```

Prüfen:

```bash
ls /root/labs/developer-portal
# → sollte "No such file or directory" melden
```

---

## 🔴 SCHRITT 3 – Yarn Cache & Node Artefakte aufräumen (empfohlen)

```bash
rm -rf ~/.yarn ~/.cache/yarn
```

Optional (nur wenn du ganz sauber willst):

```bash
rm -rf /root/.cache
```

---

## 🟢 JETZT IST ALLES SAUBER

Du bist **100 % zurück auf Anfang**
👉 **Kein Service, kein Config-Chaos, kein Cache**

---

# 🚀 NEU STARTEN – AB **STEP 1**

## ✅ Step 1: Projekt neu erstellen

```bash
cd /root/labs
npx @backstage/create-app@latest
```

* App-Name z. B.: `developer-portal`
* Default-Optionen wählen

Dann:

```bash
cd developer-portal
```

---

## ✅ Step 2: Dependencies installieren

```bash
yarn install
```

Danach:

```bash
yarn up @backstage/backend-defaults@0.14.0
```

---

## ⚠️ WICHTIGE REGEL (damit der alte Fehler NICHT wiederkommt)

❌ **KEINE Public-IP binden**
❌ **KEIN systemd-Service jetzt**

👉 **ERST lokal testen**

---

## ✅ Step 3: Lokal starten (ohne systemd)

```bash
yarn start
```

Backstage läuft dann auf:

```
http://localhost:3000
```

Oder per SSH-Port-Forward:

```bash
ssh -L 3000:localhost:3000 root@SERVER_IP
```

---

## 🧠 WARUM DAS DER RICHTIGE WEG IST

Dein vorheriges Problem kam von:

* systemd **zu früh**
* Public-IP **falsch gebunden**
* YAML + Dev-Server + rspack gleichzeitig

👉 **Backstage MUSS zuerst lokal stabil laufen**, **bevor** man:

* systemd
* Public IP
* Reverse Proxy
* Prod-Config

macht.

---

## 💬 Nächster Schritt

Sag mir einfach **„bereit für Step 2“** oder
**„Backstage läuft lokal“**, dann gehen wir **ruhig, sauber und ohne Stress** weiter.

Du hast alles richtig gemacht 👍




Super — das ist exakt der richtige Zwischenstand ✅
`Unit backstage.service could not be found.` heißt: **systemd-Service ist komplett weg**.

Jetzt machen wir den Reset fertig und starten wieder bei **Step 1**.

---

## 1) Prozess-Reste killen (falls noch was auf 3000/7007 läuft)

```bash
ss -lntp | egrep ':3000|:7007' || true
```

Wenn da noch was hängt (LISTEN), dann:

```bash
fuser -k 3000/tcp 7007/tcp || true
```

---

## 2) Projektordner komplett löschen

⚠️ löscht wirklich alles in `developer-portal`:

```bash
rm -rf /root/labs/developer-portal
```

Check:

```bash
ls -la /root/labs | grep developer-portal || true
```

---

## 3) Yarn/Backstage Artefakte aufräumen (empfohlen)

```bash
rm -rf /root/.yarn /root/.cache/yarn ~/.yarn ~/.cache/yarn
```

Optional (wenn du ganz sauber willst, kann aber groß sein):

```bash
rm -rf /root/labs/developer-portal.tgz 2>/dev/null || true
```

---

# Jetzt wieder bei **Step 1**: Neues Backstage App-Scaffold

## 4) Neu erstellen

```bash
cd /root/labs
npx @backstage/create-app@latest
```

Wenn er nach dem Namen fragt: **developer-portal**

Dann:

```bash
cd /root/labs/developer-portal
```

## 5) Step 1 aus deiner Anleitung: Dependencies

```bash
yarn install
yarn up @backstage/backend-defaults@0.14.0
```

## 6) Starten (erstmal ohne systemd)

```bash
yarn start
```

---

### Kurz zu deinem `^M : command not found`

Das ist nur ein “Enter/Carriage Return” Artefakt. Kannst du ignorieren.

Wenn du willst, poste nach Schritt (1) kurz die Ausgabe von:

```bash
ss -lntp | egrep ':3000|:7007' || true
```

Dann weiß ich sofort, ob noch ein Prozess im Hintergrund blockiert.



Alles okay — das war nur ein kleines Terminal-Missverständnis 😊

Du warst **noch im interaktiven `npx @backstage/create-app` Prompt** und hast dann offenbar wieder im normalen Shell-Prompt `kuka-idp` getippt, deshalb: `command not found`.

Wir machen’s jetzt sauber Schritt für Schritt.

---

## 1) Prüfen: läuft der create-app Prompt noch?

Wenn du **noch** diese Zeile siehst und der Cursor dahinter blinkt:

`? Enter a name for the app [required]`

➡️ Dann tippe **direkt dort** (nicht im Shell-Prompt) einfach:

`kuka-idp`

und drücke **Enter**.

---

## 2) Wenn du **nicht mehr** im Prompt bist (so wie jetzt, du siehst `root@...#`)

Dann starte den Wizard neu:

```bash
cd /root/labs
npx @backstage/create-app@latest
```

Wenn dann wieder gefragt wird:

`? Enter a name for the app [required]`

➡️ tippe:

`kuka-idp`
➡️ Enter

---

## 3) Wenn er meckert, dass der Ordner schon existiert

Dann vorher löschen:

```bash
rm -rf /root/labs/kuka-idp
```

und dann nochmal:

```bash
npx @backstage/create-app@latest
```

---

## 4) Wenn das Scaffold fertig ist

Dann:

```bash
cd /root/labs/kuka-idp
yarn install
yarn up @backstage/backend-defaults@0.14.0
yarn start
```

---

Wenn du willst: Poste einfach **die letzten 5–10 Zeilen**, nachdem du `npx @backstage/create-app@latest` erneut gestartet hast (inkl. der Fragezeile). Dann sag ich dir exakt, was du als Nächstes im Wizard auswählen sollst.


<img width="1080" height="701" alt="image" src="https://github.com/user-attachments/assets/aa317cc2-ddcf-48c8-a4ba-e93e324f1016" />



<img width="993" height="1008" alt="image" src="https://github.com/user-attachments/assets/3258d5ba-918f-4971-97d1-f69f816f0826" />



Mega — das ist jetzt **sauber durchgelaufen** ✅🎉
Du bist wieder genau bei **Step 1 / First Run** angekommen (und dependencies sind sogar schon installiert worden).

Jetzt machst du exakt das hier:

## 1) Ins Projekt wechseln

```bash
cd /root/labs/kuka-idp
```


<img width="984" height="412" alt="image" src="https://github.com/user-attachments/assets/5d33060e-2065-4052-9de9-dbd0d12bea0a" />




## 2) (Wie in deiner Anleitung) Upgrade für Node 22

```bash
yarn up @backstage/backend-defaults@0.14.0
```

## 3) App starten (erstmal ohne systemd)

```bash
yarn start
```




<img width="1508" height="877" alt="image" src="https://github.com/user-attachments/assets/abde9fa7-eeee-4723-8cd4-ead36d271cd7" />

---

### Wenn du von außen zugreifen willst (Browser)

Weil du vorher das Public-IP-Binding-Problem hattest: **nicht die Public-IP als listen-host setzen**.

Stattdessen: nachdem `yarn start` läuft, öffne die Ports (falls Firewall aktiv) und rufe im Browser auf:

```
http://95.217.214.89:3000
```
<img width="954" height="626" alt="image" src="https://github.com/user-attachments/assets/bfbb6c67-e87e-4334-8ddf-07ca4f6c2452" />


Wenn es **nicht erreichbar** ist, sag mir dann die Ausgabe von:

```bash
ss -lntp | egrep ':3000|:7007'
```

---

### Kurz: Dein Ziel „wieder von Step 1 gehen“

✅ Erledigt. Jetzt bist du wieder an Step 1/Run the app.

Mach jetzt Schritt 1–3 oben und paste mir die ersten ~20 Logzeilen aus dem laufenden `yarn start`, falls irgendwas meckert.






<img width="1236" height="698" alt="image" src="https://github.com/user-attachments/assets/23d97c9e-7b54-41ae-82ed-cde8e5437aec" />





Yesss ✅✅ genau so muss es aussehen!

* `0.0.0.0:3000` → Frontend ist **extern erreichbar**
* `0.0.0.0:7007` → Backend ist **extern erreichbar**

<img width="1178" height="724" alt="image" src="https://github.com/user-attachments/assets/417dbe88-021f-4a4b-b492-8d1c77b6d4ba" />


👉 **[http://95.217.220.177:3000](http://95.217.220.177:3000)**

<img width="1848" height="840" alt="image" src="https://github.com/user-attachments/assets/a4bbdced-9212-4487-9a19-1581d55c8834" />



## Wenn es im Browser trotzdem nicht lädt

Dann ist es zu 99% Firewall/Ports (UFW oder Cloud-Security-Group). Check bitte:

```bash
ufw status || true
```

Und ein schneller Selbsttest direkt auf der VM:

```bash
curl -I http://127.0.0.1:3000 | head -n 1
curl -I http://127.0.0.1:7007 | head -n 1
```

Wenn `curl` OK zeigt (200/302/etc.), aber Browser nicht: dann müssen wir **Port 3000** (und ggf. 7007) in der Firewall/Provider öffnen.

<img width="1271" height="823" alt="image" src="https://github.com/user-attachments/assets/f82720e5-2be4-4709-b5ae-c057ea6ad6ff" />






Sag mir kurz:

* Lädt `http://95.217.220.177:3000` im Browser?
* Output von `ufw status` (falls nicht lädt)

# Von Kuka zu developer-app



<img width="1020" height="319" alt="image" src="https://github.com/user-attachments/assets/e201b396-99c4-499a-b262-b22aad623991" />



Perfekt ✅ — der Symlink sitzt genau richtig:

`/root/labs/developer-portal -> /root/labs/kuka-idp`


<img width="939" height="204" alt="image" src="https://github.com/user-attachments/assets/912fdd83-263e-4a1a-9bdb-c005206f461f" />

Damit sollte der Task-Checker jetzt **den Ordner finden**.

Zwei kleine Sachen noch (weil du `disable` gemacht hast):

Alles gut — das ist ein **klassischer Symlink-Fall** 😊
Weil `/root/labs/developer-portal` gerade ein **Symlink auf `/root/labs/kuka-idp`** ist, versucht `mv` faktisch, `kuka-idp` **in sich selbst** zu verschieben → daher die Meldung.

Wenn du wirklich “sauber umbenennen” willst (statt Symlink), mach das so:

## Option A (empfohlen): Symlink entfernen, dann umbenennen

1. Sicherstellen, dass Backstage aus ist:

```bash
systemctl stop backstage || true
```

2. Symlink löschen:

```bash
rm -f /root/labs/developer-portal
```


3. Jetzt umbenennen:

```bash
cd /root/labs
mv kuka-idp developer-portal
```


<img width="1026" height="355" alt="image" src="https://github.com/user-attachments/assets/17668e26-1f14-4df7-9d3a-5585640fb2bb" />





4. Service auf den neuen Pfad setzen:

```bash
sed -i 's#^WorkingDirectory=.*#WorkingDirectory=/root/labs/developer-portal#' /etc/systemd/system/backstage.service
systemctl daemon-reload
systemctl enable backstage
systemctl restart backstage
```
<img width="1425" height="133" alt="image" src="https://github.com/user-attachments/assets/d991490e-ef66-4dd0-a05c-6756285c14bf" />

5. Check:

```bash
ls -la /root/labs | grep developer-portal
ss -lntp | egrep ':3000|:7007' || true
```
<img width="1054" height="147" alt="image" src="https://github.com/user-attachments/assets/5cfe827c-f2a9-4bdd-96f4-2f7c5341709f" />



---

## Option B: Symlink behalten (für den Checker reicht das!)

Wenn du nur willst, dass der Checker zufrieden ist, brauchst du **gar kein `mv`**. Dann mach einfach:

```bash
systemctl enable backstage
systemctl restart backstage
```

---
Das ist jetzt **genau** so, wie der Task-Checker es erwartet ✅

* ✅ Verzeichnis existiert: `/root/labs/developer-portal`
* ✅ systemd Service ist **enabled**
* ✅ Backstage lauscht extern:

  * `0.0.0.0:3000`
  * `0.0.0.0:7007`

Du kannst jetzt direkt:

1. **Task-Check nochmal ausführen** (in deinem Lab/Portal UI).
2. Im Browser öffnen:
   **[http://95.217.220.177:3000](http://95.217.220.177:3000)**

<img width="1868" height="777" alt="image" src="https://github.com/user-attachments/assets/fe124aa3-cb87-4689-a9a4-e4feea48c154" />


<img width="1875" height="859" alt="image" src="https://github.com/user-attachments/assets/c0f0772f-fab1-459d-a243-b35b043d815a" />

<img width="1875" height="859" alt="image" src="https://github.com/user-attachments/assets/f56f032c-b254-41f8-8b7f-7f8444125d00" />

<img width="1885" height="828" alt="image" src="https://github.com/user-attachments/assets/865d0bb7-f16d-4668-a71f-33eb422fa9c2" />



Wenn der Checker trotzdem noch meckert, ist es meistens eine Kleinigkeit, die er zusätzlich prüft. Dann poste bitte:

```bash
systemctl status backstage --no-pager
journalctl -u backstage -n 80 --no-pager
```


=================================================


Project Structure Tour
With Backstage running, explore the project structure to understand how a Backstage application is organized. This knowledge is essential for customizing and extending your portal.


Step 1: Explore the Root Project Structure
```bash
ls -la /root/labs/developer-portal/
```
<img width="925" height="625" alt="image" src="https://github.com/user-attachments/assets/4d59d5e7-8ad7-4e8a-b09a-f47714564a06" />


Step 2: Discover the Monorepo Architecture
```bash
ls -la /root/labs/developer-portal/packages/
```
<img width="759" height="177" alt="image" src="https://github.com/user-attachments/assets/95732e9b-7c9c-4f70-b1f9-b2be9a652129" />



Step 3: Examine Frontend Application Structure

```bash
ls -la /root/labs/developer-portal/packages/app/src/
```

<img width="933" height="209" alt="image" src="https://github.com/user-attachments/assets/7b6fd9ad-2006-499e-aa7d-7c75a9436741" />


Step 4: Investigate Backend Service Organization

```bash
ls -la /root/labs/developer-portal/packages/backend/src/
```

<img width="893" height="128" alt="image" src="https://github.com/user-attachments/assets/681d028a-4e25-4af9-9ea6-41fa36bf7ccd" />

root@patrickaboudou-backstage-setup-kbb:~# cat /root/labs/developer-portal/app-config.yaml

```bash
app:
  title: Scaffolded Backstage App
  baseUrl: http://95.217.220.177:3000
  listen:
    host: 0.0.0.0
    port: 3000
organization:
  name: My Company
backend:
  baseUrl: http://95.217.220.177:7007
  listen:
    port: 7007
    host: 0.0.0.0
  csp:
    connect-src:
    - '''self'''
    - 'http:'
    - 'https:'
  cors:
    origin: http://95.217.220.177:3000
    methods:
    - GET
    - HEAD
    - PATCH
    - POST
    - PUT
    - DELETE
    credentials: true
  database:
    client: better-sqlite3
    connection: ':memory:'
integrations:
  github:
  - host: github.com
    token: ${GITHUB_TOKEN}
proxy: null
techdocs:
  builder: local
  generator:
    runIn: docker
  publisher:
    type: local
auth:
  providers:
    guest: {}
scaffolder: null
catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
  - allow:
    - Component
    - System
    - API
    - Resource
    - Location
  locations:
  - type: file
    target: ../../examples/entities.yaml
  - type: file
    target: ../../examples/template/template.yaml
    rules:
    - allow:
      - Template
  - type: file
    target: ../../examples/org.yaml
    rules:
    - allow:
      - User
      - Group
kubernetes: null
permission:
  enabled: true

```

root@patrickaboudou-backstage-setup-kbb:~# 

<img width="860" height="1001" alt="image" src="https://github.com/user-attachments/assets/4e9bcd18-17e9-4a3f-994f-aff78ca00d1a" />

<img width="731" height="918" alt="image" src="https://github.com/user-attachments/assets/48b573d8-db27-4d52-8973-dc235cff7620" />

Step 5: Examine Core Configuration Files

```bash
cat /root/labs/developer-portal/catalog-info.yaml
```
root@patrickaboudou-backstage-setup-kbb:~# cat /root/labs/developer-portal/catalog-info.yaml

```bash
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: kuka-idp
  description: An example of a Backstage application.
  # Example for optional annotations
  # annotations:
  #   github.com/project-slug: backstage/backstage
  #   backstage.io/techdocs-ref: dir:.
spec:
  type: website
  owner: john@example.com
  lifecycle: experimental
root@patrickaboudou-backstage-setup-kbb:~# 
```bash

<img width="843" height="294" alt="image" src="https://github.com/user-attachments/assets/654e552f-68a4-4a16-b263-de91e0149ed5" />




========================================================


Understanding Backstage Configuration
Now that you've explored the project structure, let's understand how Backstage is configured and what each piece controls. This knowledge is essential for customizing your portal.

<img width="1018" height="710" alt="image" src="https://github.com/user-attachments/assets/4dbb66b7-faed-46b5-b963-d7db34f99d13" />

Step 1: Analyze the Main Configuration Structure
```bash
cat /root/labs/developer-portal/app-config.yaml
```

```bash
app:
  title: Scaffolded Backstage App
  baseUrl: http://5.161.90.197:3000

organization:
  name: My Company

backend:
  baseUrl: http://5.161.90.197:7007
  listen:
    port: 7007
    host: 0.0.0.0
  csp:
    connect-src: ["'self'", 'http:', 'https:']
  cors:
    origin: http://5.161.90.197:3000
    methods: [GET, HEAD, PATCH, POST, PUT, DELETE]
    credentials: true
  database:
    client: better-sqlite3
    connection: ':memory:'

integrations:
  github:
    - host: github.com
      token: 

proxy: {}

techdocs:
  builder: 'local'
  generator:
    runIn: 'docker'
  publisher:
    type: 'local'

auth:
  providers:
    guest: {}

scaffolder: {}

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

kubernetes: {}

permission:
  enabled: true
```

<img width="822" height="1027" alt="image" src="https://github.com/user-attachments/assets/673dd1f1-3b2f-4496-b261-74bc9d0a4d8f" />

<img width="620" height="955" alt="image" src="https://github.com/user-attachments/assets/60ddd42f-71a5-4615-aa60-e2fe1b4699ba" />

Step 2: Understand Application Metadata

ocus on the app and organization sections you just saw:

app.title is set to Scaffolded Backstage App, which is what appears in the browser tab and the portal header.
app.baseUrl points to http://5.161.90.197:3000, the URL you use to access the Backstage frontend.
organization.name is My Company, which is shown across the portal wherever the organization name is referenced.
This metadata shapes your portal's identity and branding.

Step 3: Examine Backend Configuration

Review the backend section to understand how the server is exposed:

baseUrl confirms the API endpoint at http://5.161.90.197:7007 that the frontend calls.
listen binds the backend to port 7007 on 0.0.0.0, making it reachable inside the lab VM.
csp and cors define security policies that allow the frontend at http://5.161.90.197:3000 to communicate safely with the backend.
database uses an in-memory better-sqlite3 store for this lab environment.
These settings determine how your frontend and backend communicate.

Step 4: Review Supporting Services

Several additional sections enable platform features:

integrations.github configures access to GitHub. The token is empty here, which is expected in the sandboxed lab.
techdocs runs the TechDocs generator in Docker and stores docs locally, so you can preview documentation without external services.
auth.providers.guest and an empty scaffolder section keep authentication and scaffolding simple for the exercise.
These choices keep the lab experience focused while mirroring production features.

Step 5: Explore Catalog Configuration

The catalog section is crucial for entity management:

import settings control branch naming (backstage-integration) and expected filenames (catalog-info.yaml).
rules allow core entity types such as Component, System, API, Resource, and Location.
locations list the local example files Backstage ingests, including entities, templates, and organization data.
This configuration determines what content appears in your software catalog and where it comes from.

Step 6: Confirm Cluster and Permission Settings


kubernetes is currently empty, signalling that Kubernetes plugins are not configured in this lab.
permission.enabled is set to true, showcasing how you can toggle the Backstage permission framework when you extend the portal.
With this context, you now understand how Backstage is configured in the lab environment and how each section maps to the actual YAML you inspected.

# resumer

```bash
SSH connection established.
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-90-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Feb 18 11:22:34 AM UTC 2026

  System load:  0.0               Processes:             176
  Usage of /:   6.9% of 74.79GB   Users logged in:       1
  Memory usage: 53%               IPv4 address for eth0: 95.217.220.177
  Swap usage:   0%                IPv6 address for eth0: 2a01:4f9:c013:6473::1
```

Expanded Security Maintenance for Applications is not enabled.

30 updates can be applied immediately.
14 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


*** System restart required ***
Last login: Wed Feb 18 11:16:28 2026 from 46.62.185.214
Note: Your bash history is configured to save automatically after each command for easy task verification.
Note: Your bash history is configured to save automatically after each command for easy task verification.

```bash
root@patrickaboudou-backstage-setup-kbb:~# cat /root/labs/developer-portal/app-config.yaml
app:
  title: Scaffolded Backstage App
  baseUrl: http://95.217.220.177:3000
  listen:
    host: 0.0.0.0
    port: 3000
organization:
  name: My Company
backend:
  baseUrl: http://95.217.220.177:7007
  listen:
    port: 7007
    host: 0.0.0.0
  csp:
    connect-src:
    - '''self'''
    - 'http:'
    - 'https:'
  cors:
    origin: http://95.217.220.177:3000
    methods:
    - GET
    - HEAD
    - PATCH
    - POST
    - PUT
    - DELETE
    credentials: true
  database:
    client: better-sqlite3
    connection: ':memory:'
integrations:
  github:
  - host: github.com
    token: ${GITHUB_TOKEN}
proxy: null
techdocs:
  builder: local
  generator:
    runIn: docker
  publisher:
    type: local
auth:
  providers:
    guest: {}
scaffolder: null
catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
  - allow:
    - Component
    - System
    - API
    - Resource
    - Location
  locations:
  - type: file
    target: ../../examples/entities.yaml
  - type: file
    target: ../../examples/template/template.yaml
    rules:
    - allow:
      - Template
  - type: file
    target: ../../examples/org.yaml
    rules:
    - allow:
      - User
      - Group
kubernetes: null
permission:
  enabled: true
root@patrickaboudou-backstage-setup-kbb:~# 
```

















