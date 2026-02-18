
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

============================================================================================

<img width="700" height="84" alt="image" src="https://github.com/user-attachments/assets/f7b06665-2b46-4315-9844-3cc5fc0bd444" />


=========================================================================================

---

## Zielbild (was wir bauen)

* **Backstage Backend** als Docker-Container
* **PostgreSQL** als eigener Container
* **Persistente Daten** (DB-Volume)
* **Healthchecks**, saubere **Env-Variablen**
* **Secrets** nicht im Repo
* Optional: Reverse Proxy (Nginx/Caddy/Traefik) + TLS, wenn du Richtung “Production” willst

---

## 1) Verzeichnisstruktur (empfohlen)

Im Projekt (z. B. `/root/labs/developer-portal`):

```
developer-portal/
  docker/
    docker-compose.yml
    .env
  app-config.production.yaml
```

> Wichtig: `.env` NICHT einchecken (in `.gitignore`).

---

## 2) PostgreSQL + Backstage via Docker Compose

Erstelle `docker/docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: backstage-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 10
    networks:
      - backstage_net

  backstage:
    build:
      context: ..
      dockerfile: packages/backend/Dockerfile
    container_name: backstage
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      NODE_ENV: production
      APP_CONFIG_backend_listen_port: "7007"
      # Backstage liest config aus app-config.* via APP_CONFIG
      APP_CONFIG: >
        [
          {
            "path": "app-config.production.yaml"
          }
        ]

      # DB Connection (Backstage config greift per env darauf zu, siehe app-config.production.yaml)
      POSTGRES_HOST: postgres
      POSTGRES_PORT: "5432"
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}

      # Secrets
      BACKEND_SECRET: ${BACKEND_SECRET}

    ports:
      - "7007:7007"
    networks:
      - backstage_net
    healthcheck:
      test: ["CMD-SHELL", "wget -qO- http://localhost:7007/healthcheck || exit 1"]
      interval: 15s
      timeout: 5s
      retries: 10
    # optional: härtere Container-Sicherheit (siehe unten)

volumes:
  postgres_data:

networks:
  backstage_net:
    driver: bridge
```

### Und dazu `docker/.env`:

```env
POSTGRES_USER=backstage
POSTGRES_PASSWORD=CHANGE_ME_STRONG
POSTGRES_DB=backstage

# Backend-Secret: mindestens 32+ Zeichen, random
BACKEND_SECRET=CHANGE_ME_SUPER_RANDOM_LONG
```

---

## 3) Production-Config für Backstage (Postgres + Security)

Lege im Projekt-Root `app-config.production.yaml` an:

```yaml
app:
  title: Backstage
  baseUrl: http://localhost:7007

backend:
  baseUrl: http://localhost:7007
  listen:
    port: 7007
    host: 0.0.0.0

  # wichtig für Sessions/Signaturen/Plugins
  auth:
    keys:
      - secret: ${BACKEND_SECRET}

database:
  client: pg
  connection:
    host: ${POSTGRES_HOST}
    port: ${POSTGRES_PORT}
    user: ${POSTGRES_USER}
    password: ${POSTGRES_PASSWORD}
    database: ${POSTGRES_DB}
    ssl: false

# Beispiel: Catalog basic - bei dir anpassen
catalog:
  rules:
    - allow: [Component, System, API, Resource, Location]
```

> Wenn du später via Reverse Proxy + TLS arbeitest, werden wir `baseUrl` sauber auf deine Domain setzen (und `ssl: true` für DB nur, wenn du eine externe DB mit TLS nutzt).

---

## 4) Build & Start

Im Ordner `docker/`:

```bash
cd /root/labs/developer-portal/docker
docker compose up -d --build
docker compose logs -f backstage
```

Checks:

```bash
docker compose ps
curl -sSf http://localhost:7007/healthcheck && echo OK
```

Wenn du extern erreichbar sein willst (wie zuvor), dann über die Server-IP:

* `http://95.217.220.177:7007`

---

## 5) Production-Sicherheit: praktische Must-haves (ohne Overkill)

### A) Secrets & Config

* **Nie** Passwörter/Secrets in Git
* `.env` nur lokal/Server
* In “echter” Produktion: Docker Secrets / Vault / SSM

### B) Container Hardening (sinnvolle Defaults)

In `docker-compose.yml` bei `backstage:` ergänzen:

```yaml
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    tmpfs:
      - /tmp
```

> Falls Backstage irgendwo schreiben muss (z. B. temp files), sind `/tmp` als tmpfs + ggf. ein gezieltes Volume besser als “alles writable”.

### C) Network Exposure minimieren

* Postgres **nicht** nach außen publishen (wir tun’s oben nicht).
* Nur Backstage-Port veröffentlichen – besser: nur Reverse Proxy Port veröffentlichen (80/443).

### D) Reverse Proxy + TLS (Production-Standard)

Wenn du willst, machen wir als nächsten Schritt:

* `caddy` oder `nginx` Container davor
* TLS (Let’s Encrypt)
* `app.baseUrl` & `backend.baseUrl` auf `https://deine-domain.tld`

### E) Backups

Minimum für Postgres:

```bash
docker exec -t backstage-postgres pg_dump -U backstage backstage > backup.sql
```

Für Produktion: tägliche Dumps + Rotation + Offsite.

### F) Updates & SBOM

* Pinne Images (z. B. `postgres:16.2-alpine` statt “latest”)
* Scans: `docker scout`, `trivy`, Dependabot

---

## 6) Mini-Dokumentation (Template für dein Projekt)

Du kannst das so in `README-deployment.md` packen:

**Abschnitte**

1. Voraussetzungen (Docker, Compose)
2. Start/Stop/Logs
3. Konfiguration (app-config.production.yaml, .env)
4. Datenbank (Volume, Backup/Restore)
5. Security Checklist (Secrets, caps, read-only, proxy, TLS)
6. Troubleshooting (DB down, migrations, healthcheck, ports)

---

## Nächster sinnvoller Schritt (damit’s “production-like” wird)

Wenn du mir kurz sagst, was du bevorzugst:

* **(A)** nur Compose + Postgres + Port 7007 direkt offen lassen (Lab-Style), oder
* **(B)** “richtiger” Production-Flow mit **Reverse Proxy + TLS + Domain**

…dann gebe ich dir **die passende Proxy-Konfiguration** (Caddy ist super easy) und wir setzen `baseUrl` korrekt, damit Backstage später keine kaputten Links / Mixed-Content / Callback-Probleme hat.

Wenn du direkt weitermachen willst, poste bitte einmal:

```bash
ls -la /root/labs/developer-portal/packages/backend | head
cat /root/labs/developer-portal/packages/backend/Dockerfile | sed -n '1,200p'
```
===================================================================


# Writing the Dockerfile
In this task, you'll create a Dockerfile that containerizes your Backstage application. A Dockerfile is a text file that contains instructions for building a Docker image, which can then be run as a container in any environment.
<img width="1017" height="630" alt="image" src="https://github.com/user-attachments/assets/2faf5658-8c96-4507-8e27-871267b00b57" />


Why containerize Backstage with Docker?

Containerizing Backstage provides several critical benefits:

Consistent environments: Your application runs the same way in development, testing, and production
Simplified deployment: Package your entire application with its dependencies
Isolation: Each container runs independently without conflicting with other applications
Scalability: Easy to scale horizontally by running multiple container instances
Portability: Run anywhere Docker is supported - local machines, cloud platforms, Kubernetes
Think of a Docker container as a lightweight, portable "box" that contains your entire application and everything it needs to run.

Step 1: Understand Backstage's Structure

Before writing the Dockerfile, understand what needs to be containerized. In the Terminal tab, explore your Backstage project structure:


cd /root/labs/developer-portal
Look at the key files and directories:


ls -la
<img width="782" height="428" alt="image" src="https://github.com/user-attachments/assets/d0f3da9a-0bb6-4012-8235-f7731b5ab10a" />


<img width="1013" height="601" alt="image" src="https://github.com/user-attachments/assets/43bae65c-862c-4869-a2aa-b339e5a599d1" />



Step 2: Create a Custom Entrypoint for Docker Secrets


Before creating the Dockerfile, you need to create a custom entrypoint script that handles Docker secrets. This script will read passwords from secure files (when using production secrets) and make them available as environment variables.

In the Code Editor tab, create a new file named docker-entrypoint.sh in your project root directory (/root/labs/developer-portal/):


#!/bin/sh
set -e

# If POSTGRES_PASSWORD_FILE is set, read the password from the file
if [ -n "$POSTGRES_PASSWORD_FILE" ] && [ -f "$POSTGRES_PASSWORD_FILE" ]; then
    export POSTGRES_PASSWORD=$(cat "$POSTGRES_PASSWORD_FILE")
    echo "✓ Loaded PostgreSQL password from secrets file"
fi

# If running as root, switch to node user using gosu
if [ "$(id -u)" = "0" ]; then
    exec gosu node "$@"
else
    exec "$@"
fi

<img width="772" height="332" alt="image" src="https://github.com/user-attachments/assets/5485923b-62f3-42d5-b98e-36cc3a60f723" />

What this entrypoint does:

Checks for secrets file: If POSTGRES_PASSWORD_FILE environment variable is set and the file exists
Reads password: Extracts the password from the Docker secrets file
Exports variable: Makes it available as POSTGRES_PASSWORD for Backstage to use
Switches user: Uses gosu to switch from root to node user (required because Docker secrets are only readable by root)
Executes command: Runs the actual Node.js backend with all original arguments as the node user
This pattern allows the same Docker image to work in both development (with environment variables) and production (with Docker secrets).

Make the entrypoint script executable in the Terminal tab:

ls -la docker


<img width="749" height="124" alt="image" src="https://github.com/user-attachments/assets/1c92208f-eedf-419d-b0be-9cadfc7d0d8b" />
<img width="749" height="1017" alt="image" src="https://github.com/user-attachments/assets/56743088-5aaf-4f7b-8ba3-cc71e3b56683" />


Step 3: Create the Production Dockerfile

<img width="749" height="1017" alt="image" src="https://github.com/user-attachments/assets/c7dd18e4-b1e9-4a81-950d-3627f6586a73" />

Step 3: Create the Production Dockerfile
Now create a Dockerfile that uses the official Backstage host build approach and the custom entrypoint. In the Code Editor tab, create a new file named Dockerfile in your project root directory (/root/labs/developer-portal/):


# Production-ready Backstage Docker image using pre-built bundles (following official Backstage docs)
FROM node:22-bookworm-slim

# Set Python interpreter for `node-gyp` to use
ENV PYTHON=/usr/bin/python3

# Install isolate-vm dependencies and gosu for user switching
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential gosu && \
    rm -rf /var/lib/apt/lists/*

# Copy custom entrypoint script for Docker secrets support
COPY --chown=root:root docker-entrypoint.sh /usr/local/bin/custom-entrypoint.sh
RUN chmod +x /usr/local/bin/custom-entrypoint.sh

# Create app directory as node user
USER node

# This should create the app dir as `node`.
# If it is instead created as `root` then the `tar` command below will fail: `can't create directory 'packages/': Permission denied`.
# If this occurs, then ensure BuildKit is enabled (`DOCKER_BUILDKIT=1`)
# so the app dir is correctly created as `node`.
WORKDIR /app

# Copy files needed by Yarn
COPY --chown=node:node .yarn ./.yarn
COPY --chown=node:node .yarnrc.yml ./
COPY --chown=node:node backstage.json ./

# This switches many Node.js dependencies to production mode.
ENV NODE_ENV=production

# This disables node snapshot for Node 20+ to work with the Scaffolder
ENV NODE_OPTIONS="--no-node-snapshot"

# Copy repo skeleton first, to avoid unnecessary docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked,uid=1000,gid=1000 \
    yarn workspaces focus --all --production && rm -rf "$(yarn cache clean)"

# Then copy the rest of the backend bundle, along with any other files we might want.
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

# Switch back to root for entrypoint (will switch to node user after reading secrets)
USER root

# Use custom entrypoint that handles Docker secrets
ENTRYPOINT ["/usr/local/bin/custom-entrypoint.sh"]
CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]
Key parts of this production Dockerfile explained (following official Backstage docs):

Base image: node:22-bookworm-slim - Debian-based Node.js 22 LTS image (latest recommended by Backstage documentation)
System dependencies: Installs Python3, g++, build-essential for native modules, and gosu for secure user switching
Custom entrypoint: Copies and enables the Docker secrets handler script
User security: Switches to non-root node user for running the application
Yarn configuration: Copies .yarn, .yarnrc.yml, and backstage.json for proper Yarn workspace setup
Pre-built bundles: Uses skeleton.tar.gz and bundle.tar.gz created by the Backstage build process
Production mode: Sets NODE_ENV=production to optimize for production runtime
Workspaces focus: Uses yarn workspaces focus --all --production to install only production dependencies efficiently (official Backstage method)
ENTRYPOINT: Uses custom entrypoint to handle Docker secrets before starting the application

drwxr-xr-x 3 root root 4096 Feb 18 11:46 .
drwx------ 8 root root 4096 Feb 18 11:46 ..
drwxr-xr-x 2 root root 4096 Feb 18 11:46 postgres-data
root@patrickaboudou-backstage-docker-twj:~/labs/developer-portal# 

Alles klar — du **folgst exakt diesen Schritten**, ohne am Dockerfile-Inhalt etwas zu ändern. Ich sage dir nur **was du wo anlegst** und **welche Commands** du ausführst, damit es sauber läuft (inkl. dem Punkt, warum du vorher `gosu not found` gesehen hast).

---

## Was du in Step 3 konkret machst

### 1) Du legst im Projekt-Root eine neue Datei `Dockerfile` an

Pfad muss genau sein:
`/root/labs/developer-portal/Dockerfile`

Du kopierst **den Dockerfile-Block exakt so wie im Step** hinein (keine Änderungen).

**Command (copy/paste):**

```bash
cd /root/labs/developer-portal

cat > Dockerfile <<'EOF'
# Production-ready Backstage Docker image using pre-built bundles (following official Backstage docs)
FROM node:22-bookworm-slim

# Set Python interpreter for `node-gyp` to use
ENV PYTHON=/usr/bin/python3

# Install isolate-vm dependencies and gosu for user switching
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential gosu && \
    rm -rf /var/lib/apt/lists/*

# Copy custom entrypoint script for Docker secrets support
COPY --chown=root:root docker-entrypoint.sh /usr/local/bin/custom-entrypoint.sh
RUN chmod +x /usr/local/bin/custom-entrypoint.sh

# Create app directory as node user
USER node

# This should create the app dir as `node`.
# If it is instead created as `root` then the `tar` command below will fail: `can't create directory 'packages/': Permission denied`.
# If this occurs, then ensure BuildKit is enabled (`DOCKER_BUILDKIT=1`)
# so the app dir is correctly created as `node`.
WORKDIR /app

# Copy files needed by Yarn
COPY --chown=node:node .yarn ./.yarn
COPY --chown=node:node .yarnrc.yml ./
COPY --chown=node:node backstage.json ./

# This switches many Node.js dependencies to production mode.
ENV NODE_ENV=production

# This disables node snapshot for Node 20+ to work with the Scaffolder
ENV NODE_OPTIONS="--no-node-snapshot"

# Copy repo skeleton first, to avoid unnecessary docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked,uid=1000,gid=1000 \
    yarn workspaces focus --all --production && rm -rf "$(yarn cache clean)"

# Then copy the rest of the backend bundle, along with any other files we might want.
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

# Switch back to root for entrypoint (will switch to node user after reading secrets)
USER root

# Use custom entrypoint that handles Docker secrets
ENTRYPOINT ["/usr/local/bin/custom-entrypoint.sh"]
CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]
EOF
```

✅ Damit ist Step 3 (Dockerfile erstellen) erledigt.

---

### 2) Du brauchst zusätzlich die Datei `docker-entrypoint.sh` im Projekt-Root

Dein Dockerfile macht:
`COPY docker-entrypoint.sh ...`

Das heißt: die Datei muss hier liegen:
`/root/labs/developer-portal/docker-entrypoint.sh`

Wenn du sie noch nicht hast, legst du sie genau dort an.

**Wichtig:** Du hattest vorhin `gosu: not found`.
Dieses Dockerfile **installiert gosu** (`apt-get install ... gosu`). Damit ist das Problem später im Container weg – aber nur, wenn du wirklich dieses Image baust und nutzt.

Falls dein Kurs/Lab schon ein EntryPoint-Script vorgibt, nimm exakt das. Wenn du schon Text davon hast (du hattest etwas in deiner Konsole), muss es als Datei gespeichert werden.

So prüfst du, ob sie existiert:

```bash
ls -la /root/labs/developer-portal/docker-entrypoint.sh
```

---

## Vor dem Docker Build: Stelle sicher, dass die Bundles existieren

Dieser Dockerfile erwartet zwingend:

* `packages/backend/dist/skeleton.tar.gz`
* `packages/backend/dist/bundle.tar.gz`

Prüfen:

```bash
ls -la /root/labs/developer-portal/packages/backend/dist || true
```

Wenn die beiden `.tar.gz` Dateien **nicht** da sind, musst du vorher den Backend-Bundle Build ausführen (das ist typischerweise ein Kurs-Step **vor** Step 3 oder direkt danach). Häufig ist das:

```bash
cd /root/labs/developer-portal
yarn install
yarn build:backend
```

Danach nochmal prüfen:

```bash
ls -la packages/backend/dist
```

---

## Docker Image bauen (mit BuildKit, wichtig!)

Im Dockerfile steht schon der Hinweis: **BuildKit muss an**, sonst kann es Permission-Probleme geben.

So baust du:

```bash
cd /root/labs/developer-portal
DOCKER_BUILDKIT=1 docker build -t backstage:prod .
```

Wenn das durchläuft, ist Step 3 technisch sauber umgesetzt.

---

## Warum das “ohne Änderung” genau so passt

* Du änderst nichts am Codeblock.
* Du sorgst nur dafür, dass **die richtigen Dateien am richtigen Ort** liegen.
* Du stellst sicher, dass die **dist Bundles** da sind (sonst kann Docker die COPY Zeilen nicht erfüllen).
* Du baust mit **BuildKit**, damit `WORKDIR /app` wirklich als `node` angelegt wird.

---

## Mini-Checkliste, damit du nicht hängenbleibst

1. `Dockerfile` liegt im Root ✅
2. `docker-entrypoint.sh` liegt im Root ✅
3. `packages/backend/dist/{skeleton.tar.gz,bundle.tar.gz}` existieren ✅
4. Build mit `DOCKER_BUILDKIT=1` ✅

---

Wenn du magst, paste mir einfach die Ausgabe von:

```bash
ls -la /root/labs/developer-portal/docker-entrypoint.sh || true
ls -la /root/labs/developer-portal/packages/backend/dist || true
```

Dann sage ich dir sofort, ob dir noch **genau ein Step** fehlt (meistens nur das `yarn build:backend` bzw. der Bundle-Output).

Fehler

Danke — ich sehe genau, was passiert ist, und wir bekommen das jetzt **sauber** hin, ohne den Dockerfile-Inhalt “inhaltlich” zu verändern.

Du hast zwei Probleme:

1. **`docker-entrypoint.sh` fehlt** (Dockerfile will sie kopieren → Build würde scheitern).
2. Dein `Dockerfile` wurde beim Copy/Paste **am Ende kaputt**:
   Statt
   `ENTRYPOINT ...` + `CMD ...`
   steht jetzt bei dir:
   `EOF ["node", ...]`
   → das ist **kein gültiger Dockerfile**.

Wir reparieren das jetzt sauber.

---

## A) Dockerfile exakt korrekt neu schreiben (ohne Änderungen am Inhalt)

Bitte führe **genau** das aus. Das überschreibt die Dockerfile korrekt:

```bash
cd /root/labs/developer-portal

cat > Dockerfile <<'EOF'
# Production-ready Backstage Docker image using pre-built bundles (following official Backstage docs)
FROM node:22-bookworm-slim

# Set Python interpreter for `node-gyp` to use
ENV PYTHON=/usr/bin/python3

# Install isolate-vm dependencies and gosu for user switching
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential gosu && \
    rm -rf /var/lib/apt/lists/*

# Copy custom entrypoint script for Docker secrets support
COPY --chown=root:root docker-entrypoint.sh /usr/local/bin/custom-entrypoint.sh
RUN chmod +x /usr/local/bin/custom-entrypoint.sh

# Create app directory as node user
USER node

# This should create the app dir as `node`.
# If it is instead created as `root` then the `tar` command below will fail: `can't create directory 'packages/': Permission denied`.
# If this occurs, then ensure BuildKit is enabled (`DOCKER_BUILDKIT=1`)
# so the app dir is correctly created as `node`.
WORKDIR /app

# Copy files needed by Yarn
COPY --chown=node:node .yarn ./.yarn
COPY --chown=node:node .yarnrc.yml ./
COPY --chown=node:node backstage.json ./

# This switches many Node.js dependencies to production mode.
ENV NODE_ENV=production

# This disables node snapshot for Node 20+ to work with the Scaffolder
ENV NODE_OPTIONS="--no-node-snapshot"

# Copy repo skeleton first, to avoid unnecessary docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked,uid=1000,gid=1000 \
    yarn workspaces focus --all --production && rm -rf "$(yarn cache clean)"

# Then copy the rest of the backend bundle, along with any other files we might want.
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

# Switch back to root for entrypoint (will switch to node user after reading secrets)
USER root

# Use custom entrypoint that handles Docker secrets
ENTRYPOINT ["/usr/local/bin/custom-entrypoint.sh"]
CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]
EOF
```

Danach kurz prüfen, dass das Ende stimmt:

```bash
tail -n 8 Dockerfile
```

Du solltest dort `ENTRYPOINT ...` und `CMD ...` sehen.

<img width="974" height="97" alt="image" src="https://github.com/user-attachments/assets/37f2b664-78ea-4b9d-8680-43e83306796f" />



---

## B) `docker-entrypoint.sh` erstellen (weil sie fehlt)

Dein Step verlangt diese Datei im Projekt-Root. Du hattest bereits den Inhalt in deiner Konsole (mit `POSTGRES_PASSWORD_FILE` und `gosu`). Wir legen sie jetzt als Datei an.

```bash
cd /root/labs/developer-portal

cat > docker-entrypoint.sh <<'EOF'
#!/bin/sh
set -e

# If POSTGRES_PASSWORD_FILE is set, read the password from the file
if [ -n "$POSTGRES_PASSWORD_FILE" ] && [ -f "$POSTGRES_PASSWORD_FILE" ]; then
    export POSTGRES_PASSWORD=$(cat "$POSTGRES_PASSWORD_FILE")
    echo "✓ Loaded PostgreSQL password from secrets file"
fi

# If running as root, switch to node user using gosu
if [ "$(id -u)" = "0" ]; then
    exec gosu node "$@"
else
    exec "$@"
fi
EOF

chmod +x docker-entrypoint.sh
ls -la docker-entrypoint.sh
```

✅ Jetzt existiert die Datei und ist ausführbar.


<img width="889" height="652" alt="image" src="https://github.com/user-attachments/assets/6b4bed24-ad28-4373-9190-ca2825f45a5c" />



**Warum das `gosu not found` vorher kam:**
Du hast das Script mal direkt auf dem Host ausgeführt → Host hat kein `gosu`.
Im Container wird `gosu` aber durch den Dockerfile installiert. Also ist das dann ok.

---

## C) Prüfen, ob die dist-Bundles existieren (wichtig für diesen Dockerfile!)

Der Dockerfile erwartet:

* `packages/backend/dist/skeleton.tar.gz`
* `packages/backend/dist/bundle.tar.gz`

Check:

```bash
ls -la /root/labs/developer-portal/packages/backend/dist || true
```

<img width="1144" height="505" alt="image" src="https://github.com/user-attachments/assets/56f09f05-d09a-46d5-8f0f-085c82364091" />


Wenn die beiden Dateien fehlen, musst du vorher den Backstage Build machen (typisch im Kurs):

```bash
cd /root/labs/developer-portal
yarn install
yarn build:backend
```

<img width="1235" height="954" alt="image" src="https://github.com/user-attachments/assets/e778748d-738e-493b-b608-0da4739a3dd9" />

<img width="1018" height="1066" alt="image" src="https://github.com/user-attachments/assets/f5c2a8af-47eb-4f84-a291-db4d4808119a" />




Dann erneut:

```bash
ls -la packages/backend/dist
```

<img width="870" height="197" alt="image" src="https://github.com/user-attachments/assets/7c4385cf-6e9b-457f-9bc0-83a776950dee" />


---

## D) Image bauen (BuildKit an!)

```bash
cd /root/labs/developer-portal
DOCKER_BUILDKIT=1 docker build -t backstage:prod .
```

<img width="1236" height="952" alt="image" src="https://github.com/user-attachments/assets/099f666a-f1ea-403f-9971-a4747a6699bb" />




---

### Wenn du mir *eine* Ausgabe gibst, kann ich dich ohne Umwege weiterführen

Poste danach bitte:

```bash
tail -n 12 Dockerfile
ls -la packages/backend/dist || true
```

Dann sage ich dir direkt, ob dir noch nur der `yarn build:backend` Schritt fehlt oder ob du schon ready bist für den nächsten Step (Compose + Postgres + Secrets).
















Step 4: Verify Your Dockerfile

Check that your Dockerfile was created correctly:
<img width="795" height="537" alt="image" src="https://github.com/user-attachments/assets/8c7c21ee-9f34-41b9-8e1f-f8804732c649" />


cd /root/labs/developer-portal
View the Dockerfile:


head -20 Dockerfile
Verify the .dockerignore (created by setup script) exists:


cat .dockerignore


<img width="1052" height="690" alt="image" src="https://github.com/user-attachments/assets/6b44284f-dd1e-49bc-9576-75d1a7251016" />





Understanding the Host Build Approach

Understanding the Host Build Approach
This Dockerfile follows the official Backstage host build approach recommended in the documentation. This approach separates the build process from the Docker image creation:

Why Host Build?
The host build approach is recommended because:

Faster: Build steps run faster on your host machine than in Docker
Better caching: Dependencies are cached on the host
More reliable: Dependency resolution happens in a controlled environment
Official support: This is the method recommended by Backstage maintainers
The Process (which you'll complete in upcoming tasks):
Host Build (Task 2 - after configuring PostgreSQL):

Install dependencies on the host machine
Generate TypeScript definitions with yarn tsc
Build backend bundle with yarn build:backend
Creates skeleton.tar.gz and bundle.tar.gz files
Docker Packaging Process:
Pull base image: Downloads node:22-bookworm-slim with Node.js 22 LTS
Extract skeleton: Unpacks pre-built package structure from skeleton.tar.gz
Install production dependencies: Uses yarn workspaces focus --all --production
Extract bundle: Unpacks the compiled backend from bundle.tar.gz
Configure security: Sets up non-root user and production environment
Docker Packaging (Task 3 - when deploying):

Pull base image with Node.js 22 LTS
Extract pre-built skeleton and install production dependencies
Extract pre-built bundle with your application code
Start Backstage backend
You've successfully created a production-ready Dockerfile following the official Backstage host build approach! In the next task, you'll configure Backstage to use PostgreSQL instead of SQLite, and build the application bundles that this Dockerfile expects.

=================================================================


# Configuring for PostgreSQL

<img width="959" height="673" alt="image" src="https://github.com/user-attachments/assets/f89ee58f-e8be-4ac6-b0b8-9ac57b2af34c" />

In this task, you'll configure Backstage to use PostgreSQL instead of the default SQLite database. This involves updating configuration files and adding the necessary database dependencies for production deployment.


Why switch from SQLite to PostgreSQL?

While SQLite works well for development, PostgreSQL provides production-grade capabilities:

Concurrent access: Multiple backend instances can safely share the same database
Data integrity: ACID transactions ensure data consistency under load
Scalability: Handles larger datasets and more complex queries efficiently
Backup & recovery: Robust tools for database maintenance and disaster recovery
Performance: Better performance with complex queries and multiple connections
Production readiness: Industry-standard database used by major applications
SQLite is a file-based database that works well for single-user scenarios, while PostgreSQL is a full database server designed for production applications.

Step 1: Install PostgreSQL Database Dependencies

Step 1: Install PostgreSQL Database Dependencies
Backstage needs specific Node.js packages to connect to PostgreSQL. In the Terminal tab, install the required dependencies:


cd /root/labs/developer-portal

# Install the PostgreSQL driver and related packages:

yarn add pg@8.16.3
yarn add --dev @types/pg

What these packages provide:

pg: The PostgreSQL client library for Node.js
@types/pg: TypeScript type definitions for the pg library (development only)


Step 2: Update Backend Database Configuration

Configure the Backstage backend to use PostgreSQL. In modern Backstage applications, database configuration is handled through the app-config.yaml file.

In the Code Editor tab, open the existing app-config.yaml file in your project root directory (/root/labs/developer-portal/) and find the existing database configuration under the backend: section. You'll see something like:
<img width="1238" height="939" alt="image" src="https://github.com/user-attachments/assets/1e88e00f-3d46-4db1-8261-16b1a33fe09d" />


backend:
  database:
    client: better-sqlite3
    connection: ':memory:'
Replace this SQLite configuration with PostgreSQL configuration:

<img width="784" height="191" alt="image" src="https://github.com/user-attachments/assets/7da14480-111c-4f68-9358-fe719bac7f54" />

backend:
  # Database configuration for PostgreSQL
  database:
    client: pg
    connection:
      host: ${POSTGRES_HOST}
      port: ${POSTGRES_PORT}
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}
      database: ${POSTGRES_DB}
    # Optional: Connection pool settings for better performance
    pool:
      min: 5
      max: 20
      acquireTimeoutMillis: 60000
      idleTimeoutMillis: 600000
Keep all your existing backend configuration (listen, csp, cors, etc.) - just replace the database section.

Configuration details explained:

client: pg: Tells Backstage to use PostgreSQL instead of SQLite
connection: Database connection parameters using environment variables
pool: Connection pool settings for better performance:
min/max: Keep 10-20 connections open for better responsiveness
acquireTimeoutMillis: How long to wait for a connection (60 seconds)
idleTimeoutMillis: How long to keep idle connections (10 minutes)
Important: Backstage will automatically create all necessary database tables when it first connects to PostgreSQL. You don't need to manually create schemas or run migration scripts.

Alles klar — du bist an **Step 1: Install PostgreSQL Dependencies**. Du willst nur wissen, **welche Commands du genau eingibst**. Mach genau das hier:

```bash
cd /root/labs/developer-portal

yarn add pg@8.16.3
yarn add --dev @types/pg
```

Danach direkt prüfen, ob es sauber drin ist:

```bash
yarn why pg | head -n 40
grep -n '"pg"' package.json
```
Alles klar — du sollst **nur die DB-Sektion in `app-config.yaml` ersetzen**, sonst nichts. So machst du das **Schritt für Schritt** im Terminal (entspricht dem “Code Editor tab” Vorgehen).

## 1) Datei öffnen

```bash
cd /root/labs/developer-portal
nano app-config.yaml
```


```yaml
app:
  title: Scaffolded Backstage App
  baseUrl: https://backstage-app-patrickaboudou-yahoo-fr-backstage-docker.academy.tekanaid.com
organization:
  name: My Company
backend:
  baseUrl: https://backstage-app-patrickaboudou-yahoo-fr-backstage-docker.academy.tekanaid.com
  listen:
    port: 7007
    host: 0.0.0.0
  csp:
    connect-src:
    - '''self'''
    - 'http:'
    - 'https:'
  cors:
    origin:
    - https://backstage-app-patrickaboudou-yahoo-fr-backstage-docker.academy.tekanaid.com
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
  httpServer:
    trustProxy: true
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
    guest:
      dangerouslyAllowOutsideDevelopment: true
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
    target: /app/examples/entities.yaml
  - type: file
    target: /app/examples/template/template.yaml
    rules:
    - allow:
      - Template
  - type: file
    target: /app/examples/org.yaml
    rules:
    - allow:
      - User
      - Group
kubernetes: null
permission:
  enabled: true

```





## 2) Zur Database-Stelle springen

In nano:

* Drücke `Ctrl + W`
* tippe: `database:`
* Enter

Du solltest dann ungefähr das hier sehen:

```yaml
backend:
  database:
    client: better-sqlite3
    connection: ':memory:'
```

## 3) Genau diesen Block ersetzen (nur diesen!)

Lösche **nur** diese 3 Zeilen unter `backend: database:` und ersetze sie durch:

```yaml
backend:
  # Database configuration for PostgreSQL
  database:
    client: pg
    connection:
      host: ${POSTGRES_HOST}
      port: ${POSTGRES_PORT}
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}
      database: ${POSTGRES_DB}
    # Optional: Connection pool settings for better performance
    pool:
      min: 5
      max: 20
      acquireTimeoutMillis: 60000
      idleTimeoutMillis: 600000
```

⚠️ Wichtig:

* **Einrückung muss exakt stimmen** (2 Spaces pro Ebene).
* Lass alle anderen `backend:` Einstellungen (listen/cors/csp/…) **unangetastet**.

## 4) Speichern und schließen

* Speichern: `Ctrl + O` → Enter
* Schließen: `Ctrl + X`


Klar — hier ist deine **bereinigte, korrekte Version** von `app-config.yaml`, bei der **nur** der `backend.database`-Block von SQLite auf PostgreSQL ersetzt wurde und der Rest so bleibt wie bei dir (inkl. `listen`, `csp`, `cors`, `trustProxy`, etc.).

Hinweis: Ich habe **nichts inhaltlich geändert außer `backend.database`** und dabei die YAML-Einrückungen sauber gemacht (dein `catalog.rules` war in deinem Snippet optisch “aus dem Block gerutscht”; unten ist es korrekt eingerückt, damit Backstage es sicher lesen kann).

```yaml
app:
  title: Scaffolded Backstage App
  baseUrl: https://backstage-app-patrickaboudou-yahoo-fr-backstage-docker.academy.tekanaid.com

organization:
  name: My Company

backend:
  baseUrl: https://backstage-app-patrickaboudou-yahoo-fr-backstage-docker.academy.tekanaid.com
  listen:
    port: 7007
    host: 0.0.0.0
  csp:
    connect-src:
      - '''self'''
      - 'http:'
      - 'https:'
  cors:
    origin:
      - https://backstage-app-patrickaboudou-yahoo-fr-backstage-docker.academy.tekanaid.com
    methods:
      - GET
      - HEAD
      - PATCH
      - POST
      - PUT
      - DELETE
    credentials: true

  # Database configuration for PostgreSQL
  database:
    client: pg
    connection:
      host: ${POSTGRES_HOST}
      port: ${POSTGRES_PORT}
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}
      database: ${POSTGRES_DB}
    # Optional: Connection pool settings for better performance
    pool:
      min: 5
      max: 20
      acquireTimeoutMillis: 60000
      idleTimeoutMillis: 600000

  httpServer:
    trustProxy: true

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
    guest:
      dangerouslyAllowOutsideDevelopment: true

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
      target: /app/examples/entities.yaml
    - type: file
      target: /app/examples/template/template.yaml
      rules:
        - allow:
            - Template
    - type: file
      target: /app/examples/org.yaml
      rules:
        - allow:
            - User
            - Group

kubernetes: null

permission:
  enabled: true
```

Wenn du das jetzt **1:1** in die Datei schreiben willst, nimm diesen Command (überschreibt die Datei exakt mit der korrekten Version):

```bash
cat > /root/labs/developer-portal/app-config.yaml <<'EOF'
app:
  title: Scaffolded Backstage App
  baseUrl: https://backstage-app-patrickaboudou-yahoo-fr-backstage-docker.academy.tekanaid.com

organization:
  name: My Company

backend:
  baseUrl: https://backstage-app-patrickaboudou-yahoo-fr-backstage-docker.academy.tekanaid.com
  listen:
    port: 7007
    host: 0.0.0.0
  csp:
    connect-src:
      - '''self'''
      - 'http:'
      - 'https:'
  cors:
    origin:
      - https://backstage-app-patrickaboudou-yahoo-fr-backstage-docker.academy.tekanaid.com
    methods:
      - GET
      - HEAD
      - PATCH
      - POST
      - PUT
      - DELETE
    credentials: true

  # Database configuration for PostgreSQL
  database:
    client: pg
    connection:
      host: ${POSTGRES_HOST}
      port: ${POSTGRES_PORT}
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}
      database: ${POSTGRES_DB}
    # Optional: Connection pool settings for better performance
    pool:
      min: 5
      max: 20
      acquireTimeoutMillis: 60000
      idleTimeoutMillis: 600000

  httpServer:
    trustProxy: true

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
    guest:
      dangerouslyAllowOutsideDevelopment: true

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
      target: /app/examples/entities.yaml
    - type: file
      target: /app/examples/template/template.yaml
      rules:
        - allow:
            - Template
    - type: file
      target: /app/examples/org.yaml
      rules:
        - allow:
            - User
            - Group

kubernetes: null

permission:
  enabled: true
EOF
```

Danach kurz validieren:

```bash
grep -n "client: pg" /root/labs/developer-portal/app-config.yaml
```


<img width="1131" height="1065" alt="image" src="https://github.com/user-attachments/assets/79defcc4-5706-4597-8c1d-fd1f2825ba16" />



Perfekt ✅ `client: pg` ist drin – damit ist die Config-Änderung richtig.

Jetzt geht’s **genau so weiter**, damit Backstage wirklich mit PostgreSQL startet (statt später wegen fehlender ENV/DB zu scheitern):

---

## 1) PostgreSQL Dependencies installieren (falls noch nicht passiert)

```bash
cd /root/labs/developer-portal
yarn add pg@8.16.3
yarn add --dev @types/pg
```



Kurz prüfen:

```bash
yarn why pg | head -n 30
```


<img width="914" height="264" alt="image" src="https://github.com/user-attachments/assets/ce31974a-9db0-4cf6-b8b2-075f4b0fa604" />

---

## 2) PostgreSQL per Docker starten (DB + Persistenz)

Du hast bereits `docker/postgres-data/`, also starten wir Postgres als Container und nutzen dieses Verzeichnis.

### 2.1 Postgres Container starten

```bash
docker rm -f backstage-postgres 2>/dev/null || true

docker run -d \
  --name backstage-postgres \
  --restart unless-stopped \
  -e POSTGRES_USER=backstage \
  -e POSTGRES_PASSWORD='ChangeMe_Strong_Password' \
  -e POSTGRES_DB=backstage \
  -v /root/labs/developer-portal/docker/postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16-alpine
```

<img width="1060" height="581" alt="image" src="https://github.com/user-attachments/assets/3a7db325-b6ea-4679-b434-22f38e763531" />


### 2.2 Test: DB erreichbar?

```bash
docker exec -it backstage-postgres pg_isready -U backstage -d backstage
```

<img width="1210" height="417" alt="image" src="https://github.com/user-attachments/assets/e9ef7af6-b1b4-483a-af49-7bbf7e8f102c" />

Wenn du `accepting connections` siehst → DB läuft.

> Hinweis: Für echte Production würdest du **5432 nicht nach außen publishen**, aber fürs Lab ist es ok. Später können wir das wieder schließen.

---

## 3) ENV Variablen setzen (damit Backstage die DB findet)

Dein `app-config.yaml` nutzt `${POSTGRES_HOST}` usw. Die müssen beim Start gesetzt sein.

Im Terminal (für deinen aktuellen Shell-Start):

```bash
export POSTGRES_HOST=127.0.0.1
export POSTGRES_PORT=5432
export POSTGRES_USER=backstage
export POSTGRES_PASSWORD='ChangeMe_Strong_Password'
export POSTGRES_DB=backstage
```

---

## 4) Backstage starten (und prüfen)

### 4.1 Start (Dev-Mode)

```bash
cd /root/labs/developer-portal
yarn start
```

Oder wenn du backend separat startest:

```bash
yarn start-backend
```

### 4.2 Prüfen in Logs

Achte auf etwas wie “Connected to PostgreSQL” / keine SQLite-Meldungen.

---

## 5) Wenn du als systemd Service startest

Dann müssen die ENV Vars **im Service** stehen, sonst sieht Backstage sie nicht.

Sag Bescheid, ob du Backstage über `systemctl start backstage` laufen lässt — dann gebe ich dir die **exakten Commands**, um die Variablen sauber in `/etc/systemd/system/backstage.service` (oder drop-in) zu setzen.

---

Wenn du jetzt den schnellsten “Beweis” willst, dass es klappt:
Starte erst Postgres (Schritt 2), setze ENV (Schritt 3), dann `yarn start-backend` und poste mir die letzten ~50 Logzeilen, falls es irgendwo hakt.














## 5) Sofort prüfen, ob es drin ist

```bash
cd /root/labs/developer-portal
grep -n "backend:" -n app-config.yaml | head -n 20
grep -n "client: pg" app-config.yaml
```

Wenn `client: pg` gefunden wird, ist die Änderung korrekt.

## 6) Achtung: Das sind KEINE Terminal-Befehle

Diese YAML-Zeilen darfst du **nicht** direkt in die Shell einfügen (sonst kommt wieder `command not found`). Sie gehören **nur** in `app-config.yaml`.

Wenn du willst, poste kurz den relevanten Ausschnitt (z. B.):

```bash
sed -n '1,120p' /root/labs/developer-portal/app-config.yaml
```

Dann sag ich dir sofort, ob die Einrückung/Position genau passt.





Step 3: Rebuild Backstage with PostgreSQL Support


Since you've added the PostgreSQL dependencies, you need to rebuild the Backstage backend to include them in the bundle:


cd /root/labs/developer-portal
Rebuild the backend with the new dependencies:


yarn build:backend
Verify the build artifacts were updated:


ls -lh packages/backend/dist/
You should see updated timestamps on skeleton.tar.gz and bundle.tar.gz.

<img width="869" height="106" alt="image" src="https://github.com/user-attachments/assets/064de5aa-5b9f-43a8-9723-57aabf47ec78" />


```bash
cd /root/labs/developer-portal
yarn build:backend
```


<img width="1119" height="1031" alt="image" src="https://github.com/user-attachments/assets/6e7877ad-cd49-4fc7-83fb-6b09e71acb14" />

Danach prüfen:

```bash
ls -lh packages/backend/dist/
```

Wenn du dort mindestens diese Dateien siehst, bist du perfekt:

* `skeleton.tar.gz`
* `bundle.tar.gz`

<img width="1009" height="1040" alt="image" src="https://github.com/user-attachments/assets/f48cb1a8-a673-4eba-9b98-5cf2d02446f8" />



# Wenn das fertig ist: Nächster Start-Test (mit PostgreSQL)

Damit Backstage die DB-Variablen sieht, setze sie (für diesen Terminal-Run):

```bash
export POSTGRES_HOST=127.0.0.1
export POSTGRES_PORT=5432
export POSTGRES_USER=backstage
export POSTGRES_PASSWORD='ChangeMe_Strong_Password'
export POSTGRES_DB=backstage
```



<img width="929" height="132" alt="image" src="https://github.com/user-attachments/assets/a6fe16e7-4ff8-41a0-9b7d-38ab06c0929e" />


# - Backend mit PostgreSQL starten

```bash
cd /root/labs/developer-portal
yarn start-backend
```
<img width="923" height="124" alt="image" src="https://github.com/user-attachments/assets/9cb7925b-815d-4315-8c42-92ff68501c23" />

root@patrickaboudou-backstage-docker-twj:~/labs/developer-portal# cd /root/labs/developer-portal
cat package.json | sed -n '1,120p'
{
  "name": "root",
  "version": "1.0.0",
  "private": true,
  "engines": {
    "node": "20 || 22"
  },
  "scripts": {
    "start": "backstage-cli repo start",
    "build:backend": "yarn workspace backend build",
    "build:all": "backstage-cli repo build --all",
    "build-image": "yarn workspace backend build-image",
    "tsc": "tsc",
    "tsc:full": "tsc --skipLibCheck false --incremental false",
    "clean": "backstage-cli repo clean",
    "test": "backstage-cli repo test",
    "test:all": "backstage-cli repo test --coverage",
    "test:e2e": "playwright test",
    "fix": "backstage-cli repo fix",
    "lint": "backstage-cli repo lint --since origin/master",
    "lint:all": "backstage-cli repo lint",
    "prettier:check": "prettier --check .",
    "new": "backstage-cli new"
  },
  "workspaces": {
    "packages": [
      "packages/*",
      "plugins/*"
    ]
  },
  "devDependencies": {
    "@backstage/cli": "^0.34.5",
    "@backstage/e2e-test-utils": "^0.1.1",
    "@playwright/test": "^1.32.3",
    "@types/pg": "^8.16.0",
    "node-gyp": "^10.0.0",
    "prettier": "^2.3.2",
    "typescript": "~5.8.0"
  },
  "resolutions": {
    "@types/react": "^18",
    "@types/react-dom": "^18"
  },
  "prettier": "@backstage/cli/config/prettier",
  "lint-staged": {
    "*.{js,jsx,ts,tsx,mjs,cjs}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  },
  "packageManager": "yarn@4.4.1",
  "dependencies": {
    "pg": "8.16.3"
  }
}
root@patrickaboudou-backstage-docker-twj:~/labs/developer-portal# 


Understanding What You've Configured



You've prepared Backstage to connect to PostgreSQL, but haven't actually connected to a database yet. Here's what you configured:

1. Database Client Change:

Changed from client: better-sqlite3 (file-based database) to client: pg (PostgreSQL client)
Installed the pg Node.js driver to enable PostgreSQL connections
2. Connection Configuration:
Your app-config.yaml now expects these environment variables:

POSTGRES_HOST: Database server hostname
POSTGRES_PORT: Database port (typically 5432)
POSTGRES_USER: Database username
POSTGRES_PASSWORD: Database password
POSTGRES_DB: Database name
3. Connection Pooling:
Configured with:

min: 10: Keep at least 10 connections open
max: 20: Allow up to 20 simultaneous connections
This improves performance by reusing connections instead of creating new ones each time
4. Rebuilt Backend:
The yarn build:backend step packaged the pg dependency into your Docker image bundle, so the containerized app can connect to PostgreSQL.

What Happens Next:
In the next task, you'll create a docker-compose.yml file that:

Starts a PostgreSQL container
Sets those environment variables (POSTGRES_HOST, etc.)
Connects your Backstage container to the PostgreSQL container
Backstage will automatically create all the database tables it needs (for catalog entities, users, etc.) when it first connects to the empty PostgreSQL database.



=========================================================




# Creating the Docker Compose File


<img width="962" height="672" alt="image" src="https://github.com/user-attachments/assets/2717d42a-901a-4ec2-94e9-76ac721d4316" />

In this task, you'll create a Docker Compose file that orchestrates multiple containers: your Backstage application (using the Dockerfile from Task 1) and a PostgreSQL database (which you configured in Task 2). Docker Compose allows you to define and run multi-container applications using a simple YAML configuration.


What is Docker Compose?
Docker Compose is a tool for defining and managing multi-container applications. It solves several challenges:

Service orchestration: Manages multiple containers that work together
Networking: Creates networks so containers can communicate
Dependencies: Ensures services start in the correct order
Volume management: Handles persistent data storage
Environment configuration: Manages environment-specific settings
Think of Docker Compose as a "conductor" that orchestrates your application's containers, ensuring they work together harmoniously.


Step 1: Understand the Architecture

Before creating the compose file, understand what you're orchestrating:

Services you'll define:

postgres: Database service using official PostgreSQL image
backstage: Your Backstage application using the Dockerfile from Task 1
Configuration: In Task 2, you configured Backstage to connect to PostgreSQL using environment variables. Now you'll create the PostgreSQL container and provide those environment variables.

Networking: Docker Compose automatically creates a network so containers can reach each other by service name (e.g., postgres becomes the hostname).

Data persistence: PostgreSQL data needs to survive container restarts, so you'll use Docker volumes.




Step 2: Create the Docker Compose Configuration
<img width="782" height="978" alt="image" src="https://github.com/user-attachments/assets/656ecd4a-022b-4cc8-acdb-d8e99922e67b" />



Alles klar — ab hier machst du **genau** diese Schritte (1:1 wie im Kurs). Ich gebe dir nur die Commands, damit du es sauber anlegst und startest.

---

## 1) `docker-compose.yml` im Projekt-Root erstellen

Führe das aus (legt die Datei exakt mit dem Kurs-Inhalt an):

```bash
cd /root/labs/developer-portal

cat > docker-compose.yml <<'YAML'
services:
  postgres:
    image: postgres:13
    restart: unless-stopped
    environment:
      POSTGRES_USER: backstage
      POSTGRES_PASSWORD: backstage_secret
      POSTGRES_DB: backstage
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U backstage"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s

  backstage:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    ports:
      - "7007:7007"
    environment:
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_USER: backstage
      POSTGRES_PASSWORD: backstage_secret
      POSTGRES_DB: backstage
      NODE_ENV: production
      LOG_LEVEL: info
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./app-config.yaml:/app/app-config.yaml:ro
      - ./app-config.production.yaml:/app/app-config.production.yaml:ro
      - ./examples:/app/examples:ro

volumes:
  postgres_data:
    driver: local
YAML
```

Kurz prüfen:

```bash
ls -la docker-compose.yml
```

---

## 2) Alte Container/Ports bereinigen (wichtig, weil du schon einen Postgres laufen hast)

Dein `backstage-postgres` Container läuft aktuell und blockiert sonst `5432`.

Stoppe/entferne ihn:

```bash
docker rm -f backstage-postgres 2>/dev/null || true
```

Falls es schon Compose-Container gibt:

```bash
docker compose down --remove-orphans 2>/dev/null || true
```

-
<img width="1057" height="1067" alt="image" src="https://github.com/user-attachments/assets/679e338f-bab1-4a86-a78b-b4584a49b077" />




--

## 3) Compose Build + Start

Jetzt startest du alles:

```bash
cd /root/labs/developer-portal
DOCKER_BUILDKIT=1 docker compose up -d --build
```

---

## 4) Status & Logs prüfen

```bash
docker compose ps
docker compose logs -f postgres
```

Wenn Postgres healthy ist (oder nach ~30–60 Sekunden), Backstage Logs:

```bash
docker compose logs -f backstage
```

---

## 5) Test: Backstage Healthcheck

```bash
curl -sSf http://localhost:7007/healthcheck && echo OK
```

Extern dann:

* `http://95.217.220.177:7007`

---

### Falls Docker Build scheitert: häufigster Grund

Dein Dockerfile erwartet die Bundles:

* `packages/backend/dist/skeleton.tar.gz`
* `packages/backend/dist/bundle.tar.gz`

Du hast die schon ✅ (bei dir waren sie da). Wenn’s trotzdem meckert, poste einfach die letzten 30 Zeilen vom Build-Log.

---

Mach jetzt Schritt **1–3** (Compose-Datei schreiben, alten Postgres stoppen, `docker compose up -d --build`). Danach sag mir kurz die Ausgabe von:

```bash
docker compose ps
```

Dann weiß ich sofort, ob alles “healthy/up” ist oder ob ein kleiner Fix nötig ist.



Step 3: Understand Service Communication


With this Docker Compose setup, services communicate as follows:

Internal Network:

backstage container → postgres container (via hostname "postgres" on port 5432)
External Access:

Your browser → localhost:7007 → backstage container:7007 (frontend and backend)
Database tools → localhost:5432 → postgres container:5432 (development only)
Data Flow:

Browser requests go to Backstage on port 7007 (serves both frontend and backend)
Backstage backend connects to PostgreSQL for data storage
PostgreSQL data persists in Docker volume between restarts


Step 4: Verify Your Compose Configuration

In the Terminal tab, navigate to your project directory:


cd /root/labs/developer-portal
List the Docker Compose file:


ls -la docker-compose.yml
Validate the Docker Compose syntax:


docker compose config


root@patrickaboudou-backstage-docker-twj:~/labs/developer-portal# cd /root/labs/developer-portal
docker compose down --remove-orphans 2>/dev/null || true
root@patrickaboudou-backstage-docker-twj:~/labs/developer-portal# cd /root/labs/developer-portal
root@patrickaboudou-backstage-docker-twj:~/labs/developer-portal# ls -la docker-compose.yml
-rw-r--r-- 1 root root 1070 Feb 18 13:17 docker-compose.yml
root@patrickaboudou-backstage-docker-twj:~/labs/developer-portal# docker compose config
name: developer-portal
services:
  backstage:
    build:
      context: /root/labs/developer-portal
      dockerfile: Dockerfile
    depends_on:
      postgres:
        condition: service_healthy
        required: true
    environment:
      LOG_LEVEL: info
      NODE_ENV: production
      POSTGRES_DB: backstage
      POSTGRES_HOST: postgres
      POSTGRES_PASSWORD: backstage_secret
      POSTGRES_PORT: "5432"
      POSTGRES_USER: backstage
    networks:
      default: null
    ports:
      - mode: ingress
        target: 7007
        published: "7007"
        protocol: tcp
    restart: unless-stopped
    volumes:
      - type: bind
        source: /root/labs/developer-portal/app-config.yaml
        target: /app/app-config.yaml
        read_only: true
        bind: {}
      - type: bind
        source: /root/labs/developer-portal/app-config.production.yaml
        target: /app/app-config.production.yaml
        read_only: true
        bind: {}
      - type: bind
        source: /root/labs/developer-portal/examples
        target: /app/examples
        read_only: true
        bind: {}
  postgres:
    environment:
      POSTGRES_DB: backstage
      POSTGRES_PASSWORD: backstage_secret
      POSTGRES_USER: backstage
    healthcheck:
      test:
        - CMD-SHELL
        - pg_isready -U backstage
      timeout: 10s
      interval: 30s
      retries: 5
      start_period: 30s
    image: postgres:13
    networks:
      default: null
    ports:
      - mode: ingress
        target: 5432
        published: "5432"
        protocol: tcp
    restart: unless-stopped
    volumes:
      - type: volume
        source: postgres_data
        target: /var/lib/postgresql/data
        volume: {}
networks:
  default:
    name: developer-portal_default
volumes:
  postgres_data:
    name: developer-portal_postgres_data
    driver: local
root@patrickaboudou-backstage-docker-twj:~/labs/developer-portal# 



# Step 5: Start the Services
Start PostgreSQL first to ensure the database is ready:


docker compose up -d postgres
Wait a few seconds, then check PostgreSQL status:


docker compose exec postgres pg_isready -U backstage
You should see output like:


/var/run/postgresql:5432 - accepting connections
This indicates that PostgreSQL is running and ready to accept connections.

Now start the full stack:


docker compose up -d
Check all services are running:


docker compose ps
You should see output like:


NAME                           IMAGE                        SERVICE     STATUS                    PORTS
developer-portal-backstage-1   developer-portal-backstage   backstage   Up 2 minutes              0.0.0.0:7007->7007/tcp
developer-portal-postgres-1    postgres:13                  postgres    Up 3 minutes (healthy)    0.0.0.0:5432->5432/tcp
This shows:

Both services running: backstage and postgres containers are up
Health status: PostgreSQL shows (healthy) indicating the health check is passing
Port mappings: Port 7007 for Backstage; 5432 for PostgreSQL



Alles klar — dann machst du **genau Step 5** so, wie es da steht. Hier sind die exakten Commands in Reihenfolge:

## 1) Nur PostgreSQL starten

```bash
cd /root/labs/developer-portal
docker compose up -d postgres
```
<img width="888" height="158" alt="image" src="https://github.com/user-attachments/assets/cb7c41c3-3130-44fe-a227-7b706964d856" />

## 2) Kurz warten + Status prüfen

```bash
docker compose exec postgres pg_isready -U backstage
```
<img width="1029" height="176" alt="image" src="https://github.com/user-attachments/assets/2bd654cb-4b1b-4334-af80-fe754e642367" />

# 2) Backstage Container starten

<img width="942" height="123" alt="image" src="https://github.com/user-attachments/assets/fabb0471-0f67-4e3d-80c5-0074ac6821c7" />



Wenn du `accepting connections` siehst ✅, weiter.

## 3) Jetzt den ganzen Stack starten

```bash
docker compose up -d
```
<img width="1221" height="1002" alt="image" src="https://github.com/user-attachments/assets/df5abb59-7d24-498b-b168-76e94192c8cf" />

## 4) Prüfen, ob alles läuft

```bash
docker compose ps
```
<img width="1627" height="192" alt="image" src="https://github.com/user-attachments/assets/e627170b-605b-46c7-b830-81641672ad93" />

---

### Wenn `docker compose exec postgres ...` fehlschlägt

Nimm diesen Status-Check (zeigt dir warum):

```bash
curl -sSf http://localhost:7007/healthcheck && echo OK

```
<img width="1062" height="795" alt="image" src="https://github.com/user-attachments/assets/e32efe0c-f524-4061-bb2f-41813aadf8b9" />


<img width="865" height="1076" alt="image" src="https://github.com/user-attachments/assets/e9049fe0-9bbc-4a7e-8049-ede35049913f" />




Step 6: Verify Backstage is Running


<img width="1533" height="323" alt="image" src="https://github.com/user-attachments/assets/3bce1e86-e4e6-4963-b420-21dbd844e0f8" />



Understanding the Orchestration


When you run docker compose up, Docker will:

Create network: Sets up internal network for service communication
Create volumes: Sets up persistent storage for PostgreSQL
Start PostgreSQL: Runs database container and waits for health check
Build Backstage: Uses your Dockerfile to build the application image
Start Backstage: Runs application container after database is healthy
Monitor services: Restarts containers if they crash (unless manually stopped)
The depends_on configuration ensures Backstage waits for PostgreSQL to be fully ready, preventing connection errors during startup.

You've created a complete Docker Compose orchestration for development! In the next task, you'll learn how to deploy securely for production using Docker secrets instead of plain-text passwords.

Sehr stark ✅ Damit hast du auch den **Persistence-/Restart-Test** sauber bestanden:

* `postgres` kommt wieder hoch und wird **(healthy)** ✅
* `backstage` startet wieder **Up** ✅
* Netzwerk/Container werden korrekt neu erstellt ✅

Wenn du jetzt den nächsten Schritt im Kurs („Understanding the Orchestration“ / „Docker Security & Production Deployment“) machen willst, hier sind die **praktischen Commands**, die man dafür typischerweise direkt dokumentiert:

## 1) Verstehen, was Compose gerade orchestriert (kurz & konkret)

### Welche Services laufen?

```bash
docker compose ps
```

### Welche Netzwerke/Volumes nutzt das Stack?

```bash
docker compose config
docker volume ls | grep developer-portal
docker network ls | grep developer-portal
```

### Welche Container-Details (Ports, Mounts, Env)?

```bash
docker compose exec backstage env | sort | grep -E 'POSTGRES|NODE_ENV|LOG_LEVEL'
docker inspect developer-portal-backstage-1 | head -n 40
```

## 2) Logs und Health prüfen (Production-Routine)

```bash
docker compose logs --tail=120 postgres
docker compose logs --tail=200 backstage
```

## 3) “Verify” per HTTP (ohne UI)

UI:

```bash
curl -I http://localhost:7007 | head
```

Backend-API antwortet (401 ist ok ohne Login):

```bash
curl -i "http://localhost:7007/api/catalog/entities?limit=1" | head -n 20
```

---

### Nächster Kurs-Step?  Production Deployment with Secrets


<img width="992" height="776" alt="image" src="https://github.com/user-attachments/assets/0726c374-7f96-494b-a3a1-743bbde21f6a" />






Production Deployment with Secrets
In this task, you'll learn production deployment practices by using Docker secrets for secure credential management. This demonstrates why environment variables are not suitable for production secrets and how to properly secure sensitive data in containerized applications.


Why Docker Secrets Matter

Security Risks of Environment Variables:

Environment variables are visible in process listings (ps aux)
They appear in container logs and debugging output
They're stored in plain text in Docker images and compose files
They can be accidentally committed to version control
Benefits of Docker Secrets:

Secrets are stored securely in the Docker daemon
They're only accessible to containers that explicitly need them
Secrets are never written to disk in plain text
They provide better audit trails and access control

Step 1: Stop Development Services and Clean Up


```bash
cd /root/labs/developer-portal
docker compose down -v
```

<img width="1081" height="840" alt="image" src="https://github.com/user-attachments/assets/0930213b-886a-46cf-8b4e-65deeda87e94" />

Danach kurz prüfen, dass wirklich alles weg ist:

```bash
docker compose ps
docker volume ls | grep developer-portal || true
```
<img width="882" height="251" alt="image" src="https://github.com/user-attachments/assets/47e20b4d-a08f-4d77-9625-6d44a9912da7" />


Wenn `docker compose ps` nichts mehr zeigt und das Volume weg ist, bist du ready für den nächsten Step (Docker Secrets anlegen und in Compose einbinden).



# Step 2: Create the Secrets Directory and Password File

```bash
cd /root/labs/developer-portal

mkdir -p secrets

echo "my-secure-production-password-2024!" > secrets/postgres_password.txt

chmod 600 secrets/postgres_password.txt

ls -la secrets/
```

<img width="868" height="284" alt="image" src="https://github.com/user-attachments/assets/9d02326a-b869-4a2d-9f7f-2d508c8b92f8" />




Wenn du die Ausgabe von `ls -la secrets/` siehst (mit `-rw-------`), ist Step 2 erledigt und du kannst zum nächsten Schritt (Compose auf Docker secrets umstellen) weitergehen.









Step 3: Create a Production Docker Compose File


Alles klar — jetzt machst du **Step 3**: die Datei `docker-compose.production.yml` anlegen (separat vom dev-compose).

Führe exakt das aus (legt die Datei 1:1 wie im Kurs an):

```bash
cd /root/labs/developer-portal

cat > docker-compose.production.yml <<'YAML'
services:
  postgres:
    image: postgres:13
    restart: unless-stopped
    environment:
      POSTGRES_USER: backstage
      POSTGRES_DB: backstage
      POSTGRES_PASSWORD_FILE: /run/secrets/postgres_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U backstage"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s
    secrets:
      - postgres_password

  backstage:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    ports:
      - "7007:7007"
    environment:
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_USER: backstage
      POSTGRES_DB: backstage
      POSTGRES_PASSWORD_FILE: /run/secrets/postgres_password
      NODE_ENV: production
      LOG_LEVEL: warn
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./app-config.yaml:/app/app-config.yaml:ro
      - ./app-config.production.yaml:/app/app-config.production.yaml:ro
      - ./examples:/app/examples:ro
    secrets:
      - postgres_password

secrets:
  postgres_password:
    file: ./secrets/postgres_password.txt

volumes:
  postgres_data:
    driver: local
YAML
```


<img width="741" height="1040" alt="image" src="https://github.com/user-attachments/assets/736460e7-08c4-4d35-b0f2-a6c47bc7dba1" />


Danach kurz prüfen, dass die Datei existiert:

```bash
ls -la docker-compose.production.yml
```

<img width="909" height="68" alt="image" src="https://github.com/user-attachments/assets/b387d3fd-c72e-4d35-8801-ef660bc9cf49" />



Step 4: Deploy with Production Configuration

Mach jetzt **Step 4** exakt so (copy/paste):

## 1) In den Projektordner

```bash
cd /root/labs/developer-portal
```

## 2) Production Deployment starten

```bash
docker compose -f docker-compose.production.yml up -d
```

<img width="1086" height="167" alt="image" src="https://github.com/user-attachments/assets/bf59a111-2c4a-42b5-af2e-9b6730eaf1cc" />


## 3) Status prüfen

```bash
docker compose -f docker-compose.production.yml ps
```
<img width="1087" height="166" alt="image" src="https://github.com/user-attachments/assets/c9c62249-484d-4807-8258-042a12c0c624" />

## 4) Logs prüfen (letzte 20 Zeilen)

```bash
docker compose -f docker-compose.production.yml logs backstage | tail -20
```

<img width="1204" height="651" alt="image" src="https://github.com/user-attachments/assets/938f1afd-fde3-442f-b9a8-634f0f90d969" />


<img width="1203" height="157" alt="image" src="https://github.com/user-attachments/assets/abe8f58b-f1e7-4f5d-9cc0-5b506ec730a4" />

<img width="1209" height="65" alt="image" src="https://github.com/user-attachments/assets/ef44aaf5-9b99-4af6-af9d-666372323782" />

<img width="1215" height="175" alt="image" src="https://github.com/user-attachments/assets/02c90aae-b631-48d7-87cc-2e8c4a7e2779" />


✅ In den Logs willst du sehen:

* `✓ Loaded PostgreSQL password from secrets file`

Wenn du mir die Ausgabe von **ps** und den **tail -20** Logs hier reinkopierst, sag ich dir sofort, ob alles korrekt ist oder ob noch ein kleiner Fix nötig ist.









Step 5: Verify Security Improvements

Alles klar — mach **Step 5** genau so, hier sind die Commands 1:1:

## 1) Secret-Mount prüfen

```bash
cd /root/labs/developer-portal
docker compose -f docker-compose.production.yml exec backstage ls -la /run/secrets/
```
<img width="1014" height="142" alt="image" src="https://github.com/user-attachments/assets/426f6f9f-be72-4dd4-8e16-1171b74fe4db" />


## 2) Prüfen, dass der Entrypoint das Secret geladen hat

```bash
docker compose -f docker-compose.production.yml logs backstage | grep "Loaded PostgreSQL password"
```

<img width="1335" height="242" alt="image" src="https://github.com/user-attachments/assets/0009521a-afc3-4ae7-8178-45a73d633cb5" />

## 3) Prüfen, dass Postgres nicht extern exposed ist

```bash
docker compose -f docker-compose.production.yml ps postgres
```

<img width="1331" height="462" alt="image" src="https://github.com/user-attachments/assets/389ed343-b495-464b-bd11-d1fa02d13938" />


<img width="1237" height="94" alt="image" src="https://github.com/user-attachments/assets/42e95483-73eb-4c5b-9e36-198c45844c35" />





✅ Erwartung bei `ps postgres`: In der PORTS-Spalte **kein** `0.0.0.0:5432->...` (nur `5432/tcp` oder leer).

Wenn du die drei Outputs hier reinkopierst, bestätige ich dir sofort, dass Step 5 sauber erfüllt ist.








Step 6: Access the Backstage UI

Click on the Backstage App tab to access the production deployment. You should see the Backstage home page with the example components loaded.

The production deployment is now running with:

Secure credential management via Docker secrets
Database not exposed to external access
Production-optimized logging
Proper health checks and restart policies

Understanding Production



Hier ist eine **saubere, professionelle und GitHub-/Markdown-fertige deutsche Übersetzung** (ohne Fehler, technisch korrekt):

---

# Warum das wichtig ist

## Compliance

Viele Sicherheitsstandards verlangen eine verschlüsselte Verwaltung von Geheimnissen (Secrets).

## Auditierung

Docker Secrets bieten eine bessere Nachverfolgbarkeit, wer auf welche Geheimnisse zugreift.

## Rotation

Secrets können rotiert (gewechselt) werden, ohne Container neu zu bauen.

## Zero-Trust-Prinzip

Anwendungen erhalten nur Zugriff auf Secrets, die sie explizit deklarieren.

---

# Wichtige Sicherheitsüberlegung

Obwohl dieser Ansatz deutlich besser ist als Klartext-Passwörter in der `docker-compose.yml`, ist es wichtig, seine Einschränkungen zu verstehen.

Das benutzerdefinierte Entrypoint-Skript liest das Docker-Secret und exportiert es als Umgebungsvariable (`POSTGRES_PASSWORD`).
Das bedeutet, das Passwort ist weiterhin sichtbar in:

* der Ausgabe von `docker inspect backstage`
* der Prozessumgebung innerhalb des Containers
* jedem Prozess im Container, der Umgebungsvariablen lesen kann

---

# Was wir erreicht haben

* ✅ Secrets befinden sich **nicht in Docker-Image-Layern**
* ✅ Secrets stehen **nicht in der `docker-compose.yml`**
* ✅ Secrets werden **nicht in Versionskontrolle (Git) eingecheckt**
* ✅ Separate Secret-Dateien mit eingeschränkten Berechtigungen (`chmod 600`)
* ✅ Bessere Auditierbarkeit als einfache Umgebungsvariablen

---

# Für Enterprise-Produktionsdeployments in Betracht ziehen

## Externe Secret-Manager

* **HashiCorp Vault** – Dynamische Secrets mit automatischer Rotation
* **AWS Secrets Manager** – Cloud-nativer Secret-Speicher mit IAM-Integration
* **Azure Key Vault** oder **GCP Secret Manager** – Enterprise-taugliche Lösungen

---

## Secrets direkt auf Anwendungsebene lesen

* Backstage so modifizieren, dass Secrets direkt aus `/run/secrets/` gelesen werden statt aus Umgebungsvariablen
* Secrets bleiben vollständig außerhalb der Prozessumgebung
* Erfordert Codeänderungen in der Datenbank-Verbindungslogik

---

## Service-Mesh-Authentifizierung

* Verwendung von **Istio** oder **Linkerd** mit gegenseitigem TLS (mTLS)
* Passwörter werden komplett überflüssig
* Services authentifizieren sich über Zertifikate

---

# Fazit

Dieser Docker-Secrets-Ansatz ist ein pragmatischer Mittelweg, der sich für kleine bis mittlere Deployments eignet.
Er funktioniert mit dem aktuellen Backstage-Design (das Datenbank-Credentials als Umgebungsvariablen erwartet) und ist deutlich sicherer als fest codierte Passwörter.

---

# Best Practices für Produktion

* Docker Secrets oder externe Secret-Management-Systeme verwenden
* Secrets niemals in Umgebungsvariablen, Code oder Versionskontrolle speichern
* Secrets und Credentials regelmäßig rotieren
* Prinzip der geringsten Berechtigung (Least Privilege) anwenden
* Zugriffsmuster auf Secrets überwachen und auditieren
* Für Enterprise-Skalierung in dedizierte Secret-Management-Infrastruktur investieren

---

# Zusammenfassung

Du hast gelernt, wie man Backstage mit Docker Secrets deployt.
Du verstehst jetzt sowohl die Vorteile als auch die Einschränkungen dieses Ansatzes und weißt, wann du fortgeschrittene Secret-Management-Lösungen für Enterprise-Deployments in Betracht ziehen solltest.

---

---
















