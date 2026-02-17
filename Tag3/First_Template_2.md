
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

Speichern Datei als:

`/root/labs/developer-portal/templates/nodejs-service-template/template.yaml`.

*yamllint template.yaml ist ein Kommando, das du im Terminal ausführst, um die Datei template.yaml mit yamllint zu überprüfen*

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


**Step 4: 

# Template Documentation** sauber und ohne Fehler – genau in Meinem Ordner-Setup (`docs/` existiert schon).

## Ziel

Ich sollte eine Datei erstellen:

`/root/labs/developer-portal/templates/nodejs-service-template/docs/README.md`

mit dem Inhalt, den ich ausfühlen muss.

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


Ich überprüffe ob, ich in richtigen oder falschen Ordner bin

Wenn ich im falschen Ordner bin, gehe ich in meinen Template-Ordner:

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
**ändern zu**

```yaml
host: 0.0.0.0
```

👉 `0.0.0.0` = akzeptiert Verbindungen von außen
👉 `127.0.0.1` = nur lokal

---

```yaml
origin: http://localhost:3000
```

**ändern zu**

```yaml
origin: http://95.217.214.89:3000
```
---

**Finale korrekte Version**

aussehen am Ende :

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
CTRL + C  
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

Bei laufendem **Backstage** erkunden wir die Projektstruktur, um zu verstehen, wie eine **Backstage**-Anwendung organisiert ist

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


## Das Verzeichnis `packages` enthält:

- **`app`**: Die React-Frontend-Anwendung, mit der Benutzer interagieren
- **`backend`**: Der **Node.js**-**API**-Dienst, der das Portal betreibt

Diese **Monorepo-Struktur** ermöglicht es Teams, Frontend und Backend unabhängig voneinander zu entwickeln und dennoch synchron zu halten.

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
Um zu sehen , wie der **API**-Dienst strukturiert ist:

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
Betrachtung der Hauptkonfigurationsdatei, um zu verstehen, wie **Backstage** konfiguriert ist:

```bash
cat /root/labs/developer-portal/app-config.yaml
```

Diese Datei steuert alle Aspekte Ihrer **Backstage**-Instanz, einschließlich Datenbank, Authentifizierung und Integrationen.

```bash
root@patrickaboudou-backstage-setup-cet:~# cat /root/labs/developer-portal/app-config.yaml
```

---

Die Ausgabe des *`cat`*-Befehls wurde hier bewusst ausgelassen, da sie bereits in früheren Nachrichten vollständig enthalten war und in einer echten `.md`-Datei an dieser Stelle folgen würde.

```
app:
  title: Scaffolded Backstage App
  baseUrl: http://95.217.214.89:3000

organization:
# Hier name: wäre KUKA Ausburg
  name: My Company

backend:
  # Used for enabling authentication, secret is shared by all backend plugins
  # See https://backstage.io/docs/auth/service-to-service-auth for
  # information on the format
  # auth:
  #   keys:
  #     - secret: ${BACKEND_SECRET}
  baseUrl: http://95.217.214.89:7007
  listen:
    port: 7007
    # Uncomment the following host directive to bind to specific interfaces
    # host: 127.0.0.1
  csp:
    connect-src: ["'self'", 'http:', 'https:']
    # Content-Security-Policy directives follow the Helmet format: https://helmetjs.github.io/#reference
    # Default Helmet Content-Security-Policy values can be removed by setting the key to false
  cors:
    origin: http://95.217.214.89:3000
    methods: [GET, HEAD, PATCH, POST, PUT, DELETE]
    credentials: true
  # This is for local development only, it is not recommended to use this in production
  # The production database configuration is stored in app-config.production.yaml
  database:
    client: better-sqlite3
    connection: ':memory:'
  # workingDirectory: /tmp # Use this to configure a working directory for the scaffolder, defaults to the OS temp-dir

integrations:
  github:
    - host: github.com
      # This is a Personal Access Token or PAT from GitHub. You can find out how to generate this token, and more information
      # about setting up the GitHub integration here: https://backstage.io/docs/integrations/github/locations#configuration
      token: ${GITHUB_TOKEN}
    ### Example for how to add your GitHub Enterprise instance using the API:
    # - host: ghe.example.net
    #   apiBaseUrl: https://ghe.example.net/api/v3
    #   token: ${GHE_TOKEN}

proxy:
  ### Example for how to add a proxy endpoint for the frontend.
  ### A typical reason to do this is to handle HTTPS and CORS for internal services.
  # endpoints:
  #   '/test':
  #     target: 'https://example.com'
  #     changeOrigin: true

# Reference documentation http://backstage.io/docs/features/techdocs/configuration
# Note: After experimenting with basic setup, use CI/CD to generate docs
# and an external cloud storage when deploying TechDocs for production use-case.
# https://backstage.io/docs/features/techdocs/how-to-guides#how-to-migrate-from-techdocs-basic-to-recommended-deployment-approach
techdocs:
  builder: 'local' # Alternatives - 'external'
  generator:
    runIn: 'docker' # Alternatives - 'local'
  publisher:
    type: 'local' # Alternatives - 'googleGcs' or 'awsS3'. Read documentation for using alternatives.

auth:
  # see https://backstage.io/docs/auth/ to learn about auth providers
  providers:
    # See https://backstage.io/docs/auth/guest/provider
    guest: {}

scaffolder:
  # see https://backstage.io/docs/features/software-templates/configuration for software template options

catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
    - allow: [Component, System, API, Resource, Location]
  locations:
    # Local example data, file locations are relative to the backend process, typically `packages/backend`
    - type: file
      target: ../../examples/entities.yaml

    # Local example template
    - type: file
      target: ../../examples/template/template.yaml
      rules:
        - allow: [Template]

    # Local example organizational data
    - type: file
      target: ../../examples/org.yaml
      rules:
        - allow: [User, Group]

    ## Uncomment these lines to add more example data
    # - type: url
    #   target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/all.yaml

    ## Uncomment these lines to add an example org
    # - type: url
    #   target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/acme-corp.yaml
    #   rules:
    #     - allow: [User, Group]

kubernetes:
  # see https://backstage.io/docs/features/kubernetes/configuration for kubernetes configuration options

# see https://backstage.io/docs/permissions/getting-started for more on the permission framework
permission:
  # setting this to `false` will disable permissions
  enabled: true
root@patrickaboudou-backstage-setup-cet:~# 

```



Examinieren wir auch die Katalog-Metadaten für die Backstage-App selbst:

cat /root/labs/developer-portal/catalog-info.yaml
This file describes your Backstage instance as an entity in its own catalog.

```
-rw-r--r-- 1 root root 2213 Feb 16 19:01 index.ts
```


```yaml
root@patrickaboudou-backstage-setup-cet:~# cat /root/labs/developer-portal/app-config.yaml
app:
  title: Scaffolded Backstage App
  baseUrl: http://95.217.214.89:3000

organization:
  name: My Company

backend:
  # Used for enabling authentication, secret is shared by all backend plugins
  # See https://backstage.io/docs/auth/service-to-service-auth for
  # information on the format
  # auth:
  #   keys:
  #     - secret: ${BACKEND_SECRET}
  baseUrl: http://95.217.214.89:7007
  listen:
    port: 7007
    # Uncomment the following host directive to bind to specific interfaces
    # host: 127.0.0.1
  csp:
    connect-src: ["'self'", 'http:', 'https:']
    # Content-Security-Policy directives follow the Helmet format: https://helmetjs.github.io/#reference
    # Default Helmet Content-Security-Policy values can be removed by setting the key to false
  cors:
    origin: http://95.217.214.89:3000
    methods: [GET, HEAD, PATCH, POST, PUT, DELETE]
    credentials: true
  # This is for local development only, it is not recommended to use this in production
  # The production database configuration is stored in app-config.production.yaml
  database:
    client: better-sqlite3
    connection: ':memory:'
  # workingDirectory: /tmp # Use this to configure a working directory for the scaffolder, defaults to the OS temp-dir

integrations:
  github:
    - host: github.com
      # This is a Personal Access Token or PAT from GitHub. You can find out how to generate this token, and more information
      # about setting up the GitHub integration here: https://backstage.io/docs/integrations/github/locations#configuration
      token: ${GITHUB_TOKEN}
    ### Example for how to add your GitHub Enterprise instance using the API:
    # - host: ghe.example.net
    #   apiBaseUrl: https://ghe.example.net/api/v3
    #   token: ${GHE_TOKEN}

proxy:
  ### Example for how to add a proxy endpoint for the frontend.
  ### A typical reason to do this is to handle HTTPS and CORS for internal services.
  # endpoints:
  #   '/test':
  #     target: 'https://example.com'
  #     changeOrigin: true

# Reference documentation http://backstage.io/docs/features/techdocs/configuration
# Note: After experimenting with basic setup, use CI/CD to generate docs
# and an external cloud storage when deploying TechDocs for production use-case.
# https://backstage.io/docs/features/techdocs/how-to-guides#how-to-migrate-from-techdocs-basic-to-recommended-deployment-approach
techdocs:
  builder: 'local' # Alternatives - 'external'
  generator:
    runIn: 'docker' # Alternatives - 'local'
  publisher:
    type: 'local' # Alternatives - 'googleGcs' or 'awsS3'. Read documentation for using alternatives.

auth:
  # see https://backstage.io/docs/auth/ to learn about auth providers
  providers:
    # See https://backstage.io/docs/auth/guest/provider
    guest: {}

scaffolder:
  # see https://backstage.io/docs/features/software-templates/configuration for software template options

catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
    - allow: [Component, System, API, Resource, Location]
  locations:
    # Local example data, file locations are relative to the backend process, typically `packages/backend`
    - type: file
      target: ../../examples/entities.yaml

    # Local example template
    - type: file
      target: ../../examples/template/template.yaml
      rules:
        - allow: [Template]

    # Local example organizational data
    - type: file
      target: ../../examples/org.yaml
      rules:
        - allow: [User, Group]

    ## Uncomment these lines to add more example data
    # - type: url
    #   target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/all.yaml

    ## Uncomment these lines to add an example org
    # - type: url
    #   target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/acme-corp.yaml
    #   rules:
    #     - allow: [User, Group]

kubernetes:
  # see https://backstage.io/docs/features/kubernetes/configuration for kubernetes configuration options

# see https://backstage.io/docs/permissions/getting-started for more on the permission framework
permission:
  # setting this to `false` will disable permissions
  enabled: true
root@patrickaboudou-backstage-setup-cet:~# cat /root/labs/developer-portal/catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: developer-portal
  description: An example of a Backstage application.
  # Example for optional annotations
  # annotations:
  #   github.com/project-slug: backstage/backstage
  #   backstage.io/techdocs-ref: dir:.
spec:
  type: website
  owner: john@example.com
  lifecycle: experimental
root@patrickaboudou-backstage-setup-cet:~# 

```

Nun wissen wir, wie Backstage-Projekte organisiert sind und warum diese Struktur sowohl die Anpassung als auch die Wartbarkeit unterstützt!


### **© ALLE RECHTE VORBEHALTEN**

Dieses Projekt wurde von **Koffitse Aboudou** im Rahmen des Studiums an der **Technischen Hochschule Deggendorf (THD)** im Auftrag der **KUKA GmbH**  realisiert.

**Nutzungshinweise:**

* Jegliche Vervielfältigung, Verbreitung oder kommerzielle Nutzung des Inhalts, der Konzepte oder der Implementierungscodes – auch auszugsweise – ist ohne ausdrückliche schriftliche Genehmigung des Urhebers und der beteiligten Institutionen untersagt.
* Die Inhalte dienen ausschließlich Dokumentations- und Prüfungszwecken im akademischen Kontext. 


**Hinweis**: Dieser Abschnitt der Arbeit stellt nur einen Teil des Gesamtprojekts dar. Das vollständige Projekt ist Eigentum des Unternehmens und daher nicht öffentlich zugänglich. Es handelt sich um ein Projekt, bei dem lediglich ein Teil veröffentlicht wird.


========================================

============================================


---

```text
SSH-Verbindung hergestellt.
Willkommen zu Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-90-generic x86_64)

 * Dokumentation:  https://help.ubuntu.com
 * Verwaltung:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 Systeminformationen Stand Montag, 16. Februar 2026, 19:52:36 UTC

  Systemlast:  0.0               Prozesse:             168
  Belegung von /:   6.8% von 74.79GB   Angemeldete Benutzer:       1
  Arbeitsspeichernutzung: 49%           IPv4-Adresse für eth0: 95.217.214.89
  Swap-Nutzung:   0%                    IPv6-Adresse für eth0: 2a01:4f9:c014:4b29::1


Erweiterte Sicherheitswartung (ESM) für Anwendungen ist nicht aktiviert.

51 Updates können sofort angewendet werden.
32 dieser Updates sind normale Sicherheitsupdates.
Um diese zusätzlichen Updates zu sehen, führen Sie aus: apt list --upgradable

Aktivieren Sie ESM Apps, um zukünftige Sicherheitsupdates zu erhalten.
Siehe https://ubuntu.com/esm oder führen Sie aus: sudo pro status


*** System-Neustart erforderlich ***

```
Letzte Anmeldung: Montag, 16. Februar 2026, 19:32:55 von 46.62.185.214
Hinweis: Ihr Bash-Verlauf ist so konfiguriert, dass er nach jedem Befehl automatisch gespeichert wird, um die Aufgabenüberprüfung zu erleichtern.
Hinweis: Ihr Bash-Verlauf ist so konfiguriert, dass er nach jedem Befehl automatisch gespeichert wird, um die Aufgabenüberprüfung zu erleichtern.
root@patrickaboudou-backstage-setup-cet:~# ls -la /root/labs/developer-portal/
insgesamt 1376
drwx------    9 root root    4096 16. Feb 19:14 .
drwxr-xr-x    3 root root    4096 16. Feb 19:01 ..
-rw-r--r--    1 root root      74 16. Feb 19:01 app-config.local.yaml
-rw-r--r--    1 root root    2232 16. Feb 19:01 app-config.production.yaml
-rw-r--r--    1 root root    4228 16. Feb 19:14 app-config.yaml
-rw-r--r--    1 root root      26 16. Feb 19:01 backstage.json
-rw-r--r--    1 root root     357 16. Feb 19:01 catalog-info.yaml
drwxr-xr-x    3 root root    4096 16. Feb 19:02 dist-types
-rw-r--r--    1 root root     113 16. Feb 19:01 .dockerignore
-rw-r--r--    1 root root      21 16. Feb 19:01 .eslintignore
-rw-r--r--    1 root root      36 16. Feb 19:01 .eslintrc.js
drwxr-xr-x    3 root root    4096 16. Feb 19:01 examples
drwxr-xr-x    8 root root    4096 16. Feb 19:01 .git
-rw-r--r--    1 root root     685 16. Feb 19:01 .gitignore
drwxr-xr-x 1705 root root   69632 16. Feb 19:02 node_modules
-rw-r--r--    1 root root    1552 16. Feb 19:02 package.json
drwxr-xr-x    4 root root    4096 16. Feb 19:01 packages
-rw-r--r--    1 root root    1789 16. Feb 19:01 playwright.config.ts
drwxr-xr-x    2 root root    4096 16. Feb 19:01 plugins
-rw-r--r--    1 root root      33 16. Feb 19:01 .prettierignore
-rw-r--r--    1 root root     152 16. Feb 19:01 README.md
-rw-r--r--    1 root root     355 16. Feb 19:01 tsconfig.json
drwxr-xr-x    3 root root    4096 16. Feb 19:02 .yarn
-rw-r--r--    1 root root 1233415 16. Feb 19:38 yarn.lock
-rw-r--r--    1 root root      66 16. Feb 19:01 .yarnrc.yml
root@patrickaboudou-backstage-setup-cet:~# ls -la /root/labs/developer-portal/packages/
insgesamt 20
drwxr-xr-x 4 root root 4096 16. Feb 19:01 .
drwx------ 9 root root 4096 16. Feb 19:14 ..
drwxr-xr-x 5 root root 4096 16. Feb 19:01 app
drwxr-xr-x 4 root root 4096 16. Feb 19:02 backend
-rw-r--r-- 1 root root  405 16. Feb 19:01 README.md
root@patrickaboudou-backstage-setup-cet:~# ls -la /root/labs/developer-portal/packages/app/src/
insgesamt 32
drwxr-xr-x 3 root root 4096 16. Feb 19:01 .
drwxr-xr-x 5 root root 4096 16. Feb 19:01 ..
-rw-r--r-- 1 root root  457 16. Feb 19:01 apis.ts
-rw-r--r-- 1 root root  660 16. Feb 19:01 App.test.tsx
-rw-r--r-- 1 root root 3696 16. Feb 19:01 App.tsx
drwxr-xr-x 5 root root 4096 16. Feb 19:01 components
-rw-r--r-- 1 root root  214 16. Feb 19:01 index.tsx
-rw-r--r-- 1 root root   36 16. Feb 19:01 setupTests.ts
root@patrickaboudou-backstage-setup-cet:~# ls -la /root/labs/developer-portal/packages/backend/src/
insgesamt 12
drwxr-xr-x 2 root root 4096 16. Feb 19:01 .
drwxr-xr-x 4 root root 4096 16. Feb 19:02 ..
-rw-r--r-- 1 root root 2213 16. Feb 19:01 index.ts

root@patrickaboudou-backstage-setup-cet:~# cat /root/labs/developer-portal/app-config.yaml
```

```
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

```
  # siehe https://backstage.io/docs/features/kubernetes/configuration für Kubernetes-Konfigurationsoptionen

# siehe https://backstage.io/docs/permissions/getting-started für mehr Informationen über das Berechtigungsframework
permission:
  # wenn dies auf `false` gesetzt wird, werden Berechtigungen deaktiviert
  enabled: true
```
  
root@patrickaboudou-backstage-setup-cet:~# cat /root/labs/developer-portal/catalog-info.yaml

```
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
root@patrickaboudou-backstage-setup-cet:~#

```



==========Komplete Code============


---

### **Anweisungen**

## **Umgebungseinrichtung und Voraussetzungen**
In dieser Aufgabe überprüfen Sie, ob Ihre Entwicklungsumgebung für **Backstage** bereit ist, und installieren gegebenenfalls fehlende Abhängigkeiten. **Backstage** benötigt **Node.js**, **npm**/**yarn** und **Git**, um ordnungsgemäß zu funktionieren.

### **Schritt 1: Überprüfen der Node.js-Installation**
Stellen Sie sicher, dass **Node.js** installiert ist. Führen Sie diesen Befehl im Tab **Terminal** aus:

```bash
node --version
```

Sie sollten `v22.x.x` sehen (Node.js 22 wird für die Kompatibilität nativer Module von **Backstage** benötigt).

### **Schritt 2: Überprüfen von npm und Yarn**
Prüfen Sie, ob **npm** ordnungsgemäß funktioniert:

```bash
npm --version
```

Stellen Sie sicher, dass **Yarn** installiert ist (**Backstage** funktioniert besser mit **Yarn** als mit **npm**):

```bash
yarn --version
```

Sie sollten Version `1.22.x` sehen. **Yarn** wurde als Teil der Lab-Umgebung vorinstalliert.

### **Schritt 3: Überprüfen der Git-Installation**
Stellen Sie sicher, dass **Git** für die Versionskontrolle verfügbar ist:

```bash
git --version
```

Konfigurieren Sie **Git** mit Ihrem Namen (ersetzen Sie `"Your Name"` durch Ihren tatsächlichen Namen):

```bash
git config --global user.name "Your Name"
```

Konfigurieren Sie **Git** mit Ihrer E-Mail-Adresse:

```bash
git config --global user.email "your.email@example.com"
```

### **Schritt 4: Arbeitsverzeichnis erstellen**
Erstellen Sie eine dedizierte Verzeichnisstruktur für Ihre **Backstage**-Entwicklung:

```bash
mkdir -p /root/labs
```

Wechseln Sie in Ihr neues Arbeitsverzeichnis:

```bash
cd /root/labs
```

Sie sind jetzt bereit, Ihre erste **Backstage**-Anwendung zu erstellen! Die Umgebung verfügt über alle notwendigen Werkzeuge.

---

## **Ihre erste Backstage-App erstellen**
Jetzt erstellen Sie eine neue **Backstage**-Anwendung mit der offiziellen **CLI**. Dies generiert eine vollständige Projektstruktur mit allen notwendigen Dateien und Abhängigkeiten.

### **Schritt 1: Neue Backstage-App erstellen**
Stellen Sie zunächst sicher, dass Sie sich in Ihrem Arbeitsverzeichnis befinden:

```bash
cd /root/labs
```

Erstellen Sie nun eine neue **Backstage**-Anwendung mit der festgelegten Version für Kompatibilität:

```bash
echo 'developer-portal' | npx @backstage/create-app@0.7.7
```

Dieser Befehl übergibt den App-Namen `developer-portal` an die `create-app`-**CLI**. Der Vorgang dauert einige Minuten.

### **Schritt 2: In das Projektverzeichnis wechseln**
Gehen Sie in Ihr neues **Backstage**-Projekt:

```bash
cd developer-portal
```

### **Schritt 3: Generierte Dateien erkunden**
```bash
ls -la
```

### **Schritt 4: Netzwerkzugriff konfigurieren**
Standardmäßig lauscht **Backstage** nur auf `localhost`. Um es remote erreichbar zu machen, muss die Konfiguration aktualisiert werden.

Ermitteln Sie die **IP**-Adresse Ihrer **VM**:

```bash
hostname -I | awk '{print $1}'
```

Öffnen Sie den Tab **Code Editor** und navigieren Sie zu `/root/labs/developer-portal/app-config.yaml`. Aktualisieren Sie die folgenden Abschnitte:
- Ändern Sie `app.baseUrl` von `http://localhost:3000` zu `http://<IHRE_VM_IP>:3000`
- Ändern Sie `backend.baseUrl` von `http://localhost:7007` zu `http://<IHRE_VM_IP>:7007`
- Entfernen Sie unter `backend.listen` den Kommentar und setzen Sie `host: 0.0.0.0`
- Ändern Sie `backend.cors.origin` von `http://localhost:3000` zu `http://<IHRE_VM_IP>:3000`

Ersetzen Sie `<IHRE_VM_IP>` mit der **IP**-Adresse aus dem obigen Befehl.

---

## **Backstage als Dienst ausführen**

### **Schritt 1: Abhängigkeiten installieren**
Navigieren Sie in Ihr Verzeichnis:

```bash
cd /root/labs/developer-portal
```

Installieren Sie alle erforderlichen Abhängigkeiten:

```bash
yarn install
```

Nach Abschluss der Installation aktualisieren Sie ein Kernpaket für die Kompatibilität mit **Node.js 22**:

```bash
yarn up @backstage/backend-defaults@0.14.0
```

### **Schritt 2: Eine systemd-Service-Datei erstellen**
Erstellen Sie eine Service-Datei, um **Backstage** als Systemdienst zu verwalten:

```bash
tee /etc/systemd/system/backstage.service << 'EOF'
[Unit]
Description=Backstage Developer Portal
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/labs/developer-portal
ExecStart=/usr/bin/yarn start
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=backstage

[Install]
WantedBy=multi-user.target
EOF
```

### **Schritt 3: Backstage-Dienst starten**
Laden Sie **systemd** neu, um den neuen Dienst zu erkennen:

```bash
systemctl daemon-reload
```

Aktivieren Sie den Dienst für den automatischen Start:

```bash
systemctl enable backstage
```

Starten Sie den Dienst:

```bash
systemctl start backstage
```

### **Schritt 4: Dienststatus überprüfen**
Status prüfen:

```bash
systemctl status backstage
```

Logs anzeigen:

```bash
journalctl -u backstage -f
```

### **Schritt 5: Auf die Anwendung zugreifen**
Ermitteln Sie die **IP**-Adresse Ihrer **VM**:

```bash
hostname -I | awk '{print $1}'
```

Öffnen Sie einen neuen Browser-Tab und navigieren Sie zu: `http://<ipv4-adresse>:3000`

### **Schritt 6: Navigation erkunden**
Klicken Sie sich durch die Hauptnavigationselemente in der Seitenleiste:
- **Startseite**
- **Katalog**
- **Erstellen**
- **Tech-Dokumente**
- **APIs**

### **Schritt 7: Software-Katalog untersuchen**
Klicken Sie im Seitenmenü auf **Katalog**. Sie sehen Beispielservices und -komponenten.

### **Nützliche systemd-Befehle**
- **Status:** `systemctl status backstage`
- **Logs:** `journalctl -u backstage -f`
- **Stoppen:** `systemctl stop backstage`
- **Neustart:** `systemctl restart backstage`
- **Autostart deaktivieren:** `systemctl disable backstage`

---

## **Rundgang durch die Projektstruktur**

### **Schritt 1: Root-Projektstruktur erkunden**
```bash
ls -la /root/labs/developer-portal/
```
Wichtige Komponenten:
- **`app-config.yaml`** - Zentrale Konfigurationsdatei
- **`packages/`** - Anwendungscode (Frontend und Backend)
- **`catalog-info.yaml`** - Metadaten über diese **Backstage**-Instanz
- **`package.json`** - **Node.js**-Abhängigkeiten und Skripte

### **Schritt 2: Monorepo-Architektur entdecken**
```bash
ls -la /root/labs/developer-portal/packages/
```
Das Verzeichnis `packages` enthält:
- **`app`**: Die React-Frontend-Anwendung
- **`backend`**: Der **Node.js**-**API**-Dienst

### **Schritt 3: Struktur der Frontend-Anwendung**
```bash
ls -la /root/labs/developer-portal/packages/app/src/
```
Die wichtigsten Dateien sind:
- **`App.tsx`**: Haupt-React-Komponente
- **`components/`**: Benutzerdefinierte **UI**-Komponenten

### **Schritt 4: Aufbau des Backend-Dienstes**
```bash
ls -la /root/labs/developer-portal/packages/backend/src/
```
Das Backend enthält:
- **`index.ts`**: Einstiegspunkt, der Server und Plugins lädt
- **`plugins/`**: Konfiguration für verschiedene **Backstage**-Plugins
- **`types.ts`**: **TypeScript**-Definitionen

### **Schritt 5: Kernkonfigurationsdateien prüfen**
```bash
cat /root/labs/developer-portal/app-config.yaml
cat /root/labs/developer-portal/catalog-info.yaml
```





SSH-Verbindung hergestellt.
Willkommen zu Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-90-generic x86_64)

 * Dokumentation:  https://help.ubuntu.com
 * Verwaltung:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 Systeminformationen Stand Montag, 16. Februar 2026, 18:57:19 UTC

  Systemlast:  0.0               Prozesse:             158
  Belegung von /:   2.7% von 74.79GB   Angemeldete Benutzer:       1
  Arbeitsspeichernutzung: 9%            IPv4-Adresse für eth0: 95.217.214.89
  Swap-Nutzung:   0%                    IPv6-Adresse für eth0: 2a01:4f9:c014:4b29::1


Erweiterte Sicherheitswartung (ESM) für Anwendungen ist nicht aktiviert.

51 Updates können sofort angewendet werden.
32 dieser Updates sind normale Sicherheitsupdates.
Um diese zusätzlichen Updates zu sehen, führen Sie aus: apt list --upgradable

Aktivieren Sie ESM Apps, um zukünftige Sicherheitsupdates zu erhalten.
Siehe https://ubuntu.com/esm oder führen Sie aus: sudo pro status


*** System-Neustart erforderlich ***
Letzte Anmeldung: Montag, 16. Februar 2026, 18:41:28 von 46.62.185.214
Hinweis: Ihr Bash-Verlauf ist so konfiguriert, dass er nach jedem Befehl automatisch gespeichert wird, um die Aufgabenüberprüfung zu erleichtern.
Hinweis: Ihr Bash-Verlauf ist so konfiguriert, dass er nach jedem Befehl automatisch gespeichert wird, um die Aufgabenüberprüfung zu erleichtern.
root@patrickaboudou-backstage-setup-cet:~# cd /root/labs
root@patrickaboudou-backstage-setup-cet:~/labs# echo 'developer-portal' | npx @backstage/create-app@0.7.7
npm warn exec Das folgende Paket wurde nicht gefunden und wird installiert: @backstage/create-app@0.7.7
npm warn deprecated boolean@3.2.0: Paket wird nicht mehr unterstützt. Kontaktieren Sie den Support unter https://www.npmjs.com/support für weitere Informationen.
? Geben Sie einen Namen für die App ein [erforderlich] developer-portal

Erstelle die App...

 Prüfe, ob das Verzeichnis verfügbar ist:
  prüfe      developer-portal ✔ 

 Erstelle ein temporäres App-Verzeichnis:

 Bereite Dateien vor:
  kopiere       .dockerignore ✔ 
  kopiere       .eslintignore ✔ 
  templating    .eslintrc.js.hbs ✔ 
  templating    .gitignore.hbs ✔ 
  kopiere       .prettierignore ✔ 
  templating    .yarnrc.yml.hbs ✔ 
  kopiere       README.md ✔ 
  kopiere       app-config.local.yaml ✔ 
  kopiere       app-config.production.yaml ✔ 
  templating    app-config.yaml.hbs ✔ 
  templating    backstage.json.hbs ✔ 
  templating    catalog-info.yaml.hbs ✔ 
  templating    package.json.hbs ✔ 
  kopiere       playwright.config.ts ✔ 
  kopiere       tsconfig.json ✔ 
  kopiere       yarn.lock ✔ 
  kopiere       README.md ✔ 
  kopiere       yarn-4.4.1.cjs ✔ 
  kopiere       entities.yaml ✔ 
  kopiere       org.yaml ✔ 
  kopiere       template.yaml ✔ 
  kopiere       catalog-info.yaml ✔ 
  kopiere       index.js ✔ 
  kopiere       package.json ✔ 
  kopiere       README.md ✔ 
  templating    .eslintrc.js.hbs ✔ 
  kopiere       Dockerfile ✔ 
  kopiere       README.md ✔ 
  templating    package.json.hbs ✔ 
  kopiere       index.ts ✔ 
  kopiere       .eslintignore ✔ 
  templating    .eslintrc.js.hbs ✔ 
  templating    package.json.hbs ✔ 
  kopiere       app.test.ts ✔ 
  kopiere       android-chrome-192x192.png ✔ 
  kopiere       apple-touch-icon.png ✔ 
  kopiere       favicon-16x16.png ✔ 
  kopiere       favicon-32x32.png ✔ 
  kopiere       favicon.ico ✔ 
  kopiere       index.html ✔ 
  kopiere       manifest.json ✔ 
  kopiere       robots.txt ✔ 
  kopiere       safari-pinned-tab.svg ✔ 
  kopiere       App.test.tsx ✔ 
  kopiere       App.tsx ✔ 
  kopiere       apis.ts ✔ 
  kopiere       index.tsx ✔ 
  kopiere       setupTests.ts ✔ 
  kopiere       LogoFull.tsx ✔ 
  kopiere       LogoIcon.tsx ✔ 
  kopiere       Root.tsx ✔ 
  kopiere       index.ts ✔ 
  kopiere       EntityPage.tsx ✔ 
  kopiere       SearchPage.tsx ✔ 

 Verschieben an den endgültigen Speicherort:
  verschieben   developer-portal ✔ 
  abrufen       yarn.lock seed ✔ 
  init          git repository ◜ 
 Installiere Abhängigkeiten:
  init          git repository ✔ 
  ausführen     yarn install ✔ 
  ausführen     yarn tsc ✔ 

🥇  developer-portal erfolgreich erstellt


 Alles erledigt! Nun können Sie:
  Die App ausführen: cd developer-portal && yarn start
  Den Softwarekatalog einrichten: https://backstage.io/docs/features/software-catalog/configuration
  Authentifizierung hinzufügen: https://backstage.io/docs/auth/

root@patrickaboudou-backstage-setup-cet:~/labs# cd developer-portal
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# ls -la
insgesamt 1376
drwx------    9 root root    4096 16. Feb 19:02 .
drwxr-xr-x    3 root root    4096 16. Feb 19:01 ..
-rw-r--r--    1 root root      74 16. Feb 19:01 app-config.local.yaml
-rw-r--r--    1 root root    2232 16. Feb 19:01 app-config.production.yaml
-rw-r--r--    1 root root    4216 16. Feb 19:01 app-config.yaml
-rw-r--r--    1 root root      26 16. Feb 19:01 backstage.json
-rw-r--r--    1 root root     357 16. Feb 19:01 catalog-info.yaml
drwxr-xr-x    3 root root    4096 16. Feb 19:02 dist-types
-rw-r--r--    1 root root     113 16. Feb 19:01 .dockerignore
-rw-r--r--    1 root root      21 16. Feb 19:01 .eslintignore
-rw-r--r--    1 root root      36 16. Feb 19:01 .eslintrc.js
drwxr-xr-x    3 root root    4096 16. Feb 19:01 examples
drwxr-xr-x    8 root root    4096 16. Feb 19:01 .git
-rw-r--r--    1 root root     685 16. Feb 19:01 .gitignore
drwxr-xr-x 1705 root root   69632 16. Feb 19:02 node_modules
-rw-r--r--    1 root root    1552 16. Feb 19:02 package.json
drwxr-xr-x    4 root root    4096 16. Feb 19:01 packages
-rw-r--r--    1 root root    1789 16. Feb 19:01 playwright.config.ts
drwxr-xr-x    2 root root    4096 16. Feb 19:01 plugins
-rw-r--r--    1 root root      33 16. Feb 19:01 .prettierignore
-rw-r--r--    1 root root     152 16. Feb 19:01 README.md
-rw-r--r--    1 root root     355 16. Feb 19:01 tsconfig.json
drwxr-xr-x    3 root root    4096 16. Feb 19:02 .yarn
-rw-r--r--    1 root root 1233417 16. Feb 19:02 yarn.lock
-rw-r--r--    1 root root      66 16. Feb 19:01 .yarnrc.yml
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# hostname -I | awk '{print $1}'
95.217.214.89
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# ^C
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# ^C
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# /root/labs/developer-portal/app-config.yaml
-bash: /root/labs/developer-portal/app-config.yaml: Keine Berechtigung
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# nano /root/labs/developer-portal/app-config.yaml
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# yarn start
Starte App, Backend
Geladene Konfiguration von app-config.yaml
<i> [webpack-dev-server] Projekt läuft unter:
<i> [webpack-dev-server] In Ihrem Netzwerk (IPv4): http://95.217.214.89:3000/
<i> [webpack-dev-server] Inhalte, die nicht von webpack stammen, werden aus dem Verzeichnis '/root/labs/developer-portal/packages/app/public' bereitgestellt
<i> [webpack-dev-server] 404s werden auf '/index.html' zurückgreifen
Rspack erfolgreich kompiliert
Lade Konfiguration von MergedConfigSource{FileConfigSource{Pfad="/root/labs/developer-portal/app-config.yaml"}, FileConfigSource{Pfad="/root/labs/developer-portal/app-config.local.yaml"}, EnvConfigSource{Anzahl=0}}
2026-02-16T19:14:39.276Z backstage info 1 neue Geheimnisse in der Konfiguration gefunden, die geschwärzt werden 
2026-02-16T19:14:39.284Z rootHttpRouter info Lauscht auf :7007 
2026-02-16T19:14:39.285Z backstage info Plugin-Initialisierung gestartet: 'app', 'auth', 'catalog', 'kubernetes', 'notifications', 'permission', 'proxy', 'scaffolder', 'search', 'signals', 'techdocs' typ="initialization"
2026-02-16T19:14:39.381Z search warn Postgres-Suchmaschine wird nicht unterstützt, überspringe Registrierung von search-backend-module-pg 
2026-02-16T19:14:39.812Z auth info Konfiguriere "database" als KeyStore-Anbieter 
2026-02-16T19:14:39.812Z kubernetes warn Initialisierung des Kubernetes-Backends fehlgeschlagen: gültige Kubernetes-Konfiguration fehlt 
2026-02-16T19:14:39.815Z signals info Signals-Manager abonniert Signals-Ereignisse subscriberId="signals-8f50c07f83a03491"
2026-02-16T19:14:39.818Z techdocs info Erstelle lokalen Publisher für TechDocs 
2026-02-16T19:14:39.832Z search info DefaultCatalogCollatorFactory Collator-Factory für Typ software-catalog hinzugefügt 
2026-02-16T19:14:39.833Z search info DefaultTechDocsCollatorFactory Collator-Factory für Typ techdocs hinzugefügt 
2026-02-16T19:14:39.999Z scaffolder info Starte Scaffolder mit den folgenden aktivierten Aktionen notification:send, github:actions:dispatch, github:autolinks:create, github:deployKey:create, github:environment:create, github:issues:label, github:issues:create, github:repo:create, github:repo:push, github:webhook, publish:github, publish:github:pull-request, github:pages:enable, github:branch-protection:create, fetch:plain, fetch:plain:file, fetch:template, fetch:templa...................





==========================================


SSH-Verbindung hergestellt.
Willkommen zu Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-90-generic x86_64)

 * Dokumentation:  https://help.ubuntu.com
 * Verwaltung:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 Systeminformationen Stand Montag, 16. Februar 2026, 19:32:49 UTC

  Systemlast:  0.0               Prozesse:             164
  Belegung von /:   6.8% von 74.79GB   Angemeldete Benutzer:       1
  Arbeitsspeichernutzung: 50%            IPv4-Adresse für eth0: 95.217.214.89
  Swap-Nutzung:   0%                    IPv6-Adresse für eth0: 2a01:4f9:c014:4b29::1


Erweiterte Sicherheitswartung (ESM) für Anwendungen ist nicht aktiviert.

51 Updates können sofort angewendet werden.
32 dieser Updates sind normale Sicherheitsupdates.
Um diese zusätzlichen Updates zu sehen, führen Sie aus: apt list --upgradable

Aktivieren Sie ESM Apps, um zukünftige Sicherheitsupdates zu erhalten.
Siehe https://ubuntu.com/esm oder führen Sie aus: sudo pro status


*** System-Neustart erforderlich ***
Letzte Anmeldung: Montag, 16. Februar 2026, 18:57:26 von 46.62.185.214
Hinweis: Ihr Bash-Verlauf ist so konfiguriert, dass er nach jedem Befehl automatisch gespeichert wird, um die Aufgabenüberprüfung zu erleichtern.
Hinweis: Ihr Bash-Verlauf ist so konfiguriert, dass er nach jedem Befehl automatisch gespeichert wird, um die Aufgabenüberprüfung zu erleichtern.
root@patrickaboudou-backstage-setup-cet:~# cd /root/labs/developer-portal
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# yarn install
➤ YN0000: · Yarn 4.4.1
➤ YN0000: ┌ Auflösungsschritt
➤ YN0000: └ Abgeschlossen in 0s 550ms
➤ YN0000: ┌ Überprüfung nach der Auflösung
➤ YN0060: │ @testing-library/react wird von Ihrem Projekt mit Version 14.3.1 (pc9eb9) aufgeführt, was nicht den Anforderungen von @backstage/test-utils (^16.0.0) entspricht.
➤ YN0060: │ react wird von Ihrem Projekt mit Version 18.3.1 (pd98da) aufgeführt, was nicht den Anforderungen von @material-ui/core und anderen Abhängigkeiten entspricht (aber sie haben sich nicht überschneidende Bereiche!).
➤ YN0060: │ react-dom wird von Ihrem Projekt mit Version 18.3.1 (pfa800) aufgeführt, was nicht den Anforderungen von @material-ui/core und anderen Abhängigkeiten entspricht (aber sie haben sich nicht überschneidende Bereiche!).
➤ YN0002: │ app@workspace:packages/app bietet @types/react nicht an (pceee1), angefordert von @backstage/app-defaults und anderen Abhängigkeiten.
➤ YN0002: │ app@workspace:packages/app bietet jest nicht an (p99cdc), angefordert von @backstage/cli.
➤ YN0002: │ app@workspace:packages/app bietet webpack nicht an (p299d9), angefordert von @backstage/cli.
➤ YN0002: │ backend@workspace:packages/backend bietet jest nicht an (p35ee3), angefordert von @backstage/cli.
➤ YN0002: │ backend@workspace:packages/backend bietet webpack nicht an (p00f29), angefordert von @backstage/cli.
➤ YN0002: │ root@workspace:. bietet webpack nicht an (p40c38), angefordert von @backstage/cli.
➤ YN0086: │ Einige Peer-Abhängigkeiten werden von Ihrem Projekt nicht korrekt erfüllt; führen Sie yarn explain peer-requirements <hash> für Details aus, wobei <hash> der sechsbuchstabige p-präfixierte Code ist.
➤ YN0086: │ Einige Peer-Abhängigkeiten werden von Abhängigkeiten nicht korrekt erfüllt; führen Sie yarn explain peer-requirements für Details aus.
➤ YN0000: └ Abgeschlossen
➤ YN0000: ┌ Abrufschritt
➤ YN0000: └ Abgeschlossen in 2s 208ms
➤ YN0000: ┌ Verknüpfungsschritt
➤ YN0000: └ Abgeschlossen in 0s 916ms
➤ YN0000: · Fertig mit Warnungen in 4s 96ms
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# yarn up @backstage/backend-defaults@0.14.0
➤ YN0000: · Yarn 4.4.1
➤ YN0000: ┌ Auflösungsschritt
➤ YN0085: │ + @backstage/backend-defaults@npm:0.14.0
➤ YN0085: │ - @backstage/backend-defaults@npm:0.14.1
➤ YN0000: └ Abgeschlossen in 1s 104ms
➤ YN0000: ┌ Überprüfung nach der Auflösung
➤ YN0060: │ @testing-library/react wird von Ihrem Projekt mit Version 14.3.1 (pc9eb9) aufgeführt, was nicht den Anforderungen von @backstage/test-utils (^16.0.0) entspricht.
➤ YN0060: │ react wird von Ihrem Projekt mit Version 18.3.1 (pd98da) aufgeführt, was nicht den Anforderungen von @material-ui/core und anderen Abhängigkeiten entspricht (aber sie haben sich nicht überschneidende Bereiche!).
➤ YN0060: │ react-dom wird von Ihrem Projekt mit Version 18.3.1 (pfa800) aufgeführt, was nicht den Anforderungen von @material-ui/core und anderen Abhängigkeiten entspricht (aber sie haben sich nicht überschneidende Bereiche!).
➤ YN0002: │ app@workspace:packages/app bietet @types/react nicht an (pceee1), angefordert von @backstage/app-defaults und anderen Abhängigkeiten.
➤ YN0002: │ app@workspace:packages/app bietet jest nicht an (p99cdc), angefordert von @backstage/cli.
➤ YN0002: │ app@workspace:packages/app bietet webpack nicht an (p299d9), angefordert von @backstage/cli.
➤ YN0002: │ backend@workspace:packages/backend bietet jest nicht an (p35ee3), angefordert von @backstage/cli.
➤ YN0002: │ backend@workspace:packages/backend bietet webpack nicht an (p00f29), angefordert von @backstage/cli.
➤ YN0002: │ root@workspace:. bietet webpack nicht an (p40c38), angefordert von @backstage/cli.
➤ YN0086: │ Einige Peer-Abhängigkeiten werden von Ihrem Projekt nicht korrekt erfüllt; führen Sie yarn explain peer-requirements <hash> für Details aus, wobei <hash> der sechsbuchstabige p-präfixierte Code ist.
➤ YN0086: │ Einige Peer-Abhängigkeiten werden von Abhängigkeiten nicht korrekt erfüllt; führen Sie yarn explain peer-requirements für Details aus.
➤ YN0000: └ Abgeschlossen
➤ YN0000: ┌ Abrufschritt
➤ YN0013: │ Ein Paket wurde zum Projekt hinzugefügt (+ 1.53 MiB).
➤ YN0000: └ Abgeschlossen in 2s 187ms
➤ YN0000: ┌ Verknüpfungsschritt
➤ YN0000: └ Abgeschlossen in 1s 59ms
➤ YN0000: · Fertig mit Warnungen in 4s 742ms
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# tee /etc/systemd/system/backstage.service << 'EOF'
[Unit]
Description=Backstage Developer Portal
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/labs/developer-portal
ExecStart=/usr/bin/yarn start
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=backstage

[Install]
WantedBy=multi-user.target
EOF
[Unit]
Description=Backstage Developer Portal
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/labs/developer-portal
ExecStart=/usr/bin/yarn start
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=backstage

[Install]
WantedBy=multi-user.target
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# systemctl daemon-reload
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# systemctl enable backstage
Erstellt symbolischen Link /etc/systemd/system/multi-user.target.wants/backstage.service → /etc/systemd/system/backstage.service.
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# systemctl start backstage
root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# systemctl status backstage
● backstage.service - Backstage Developer Portal
     Geladen: geladen (/etc/systemd/system/backstage.service; aktiviert; voreingestellt: aktiviert)
     Aktiv: aktiv (läuft) seit Montag 2026-02-16 19:40:01 UTC; vor 1min 17s
   Haupt-PID: 8691 (node)
      Aufgaben: 47 (Limit: 4531)
     Speicher: 1.2G (Spitze: 1.4G)
        CPU: 19.724s
     CGroup: /system.slice/backstage.service
             ├─8691 node /usr/bin/yarn start
             ├─8698 /usr/local/bin/node /root/labs/developer-portal/.yarn/releases/yarn-4.4.1.cjs start
             ├─8709 /usr/local/bin/node /root/labs/developer-portal/node_modules/@backstage/cli/bin/backstage->
             └─8722 /usr/local/bin/node --enable-source-maps --require /root/labs/developer-portal/node_module>

Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.593Z rootHttpRouter info>
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.625Z rootHttpRouter info>
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.627Z rootHttpRouter info>
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.628Z rootHttpRouter info>
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.648Z rootHttpRouter info>
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.649Z rootHttpRouter info>
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.651Z rootHttpRouter info>
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.656Z rootHttpRouter info>
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.657Z rootHttpRouter info>
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.658Z rootHttpRouter info>

root@patrickaboudou-backstage-setup-cet:~/labs/developer-portal# journalctl -u backstage -f
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.593Z rootHttpRouter info [2026-02-16T19:40:53.593Z] "GET /api/scaffolder/v2/templates/default/template/example-nodejs-template/parameter-schema HTTP/1.1" 304 0 "http://95.217.214.89:3000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36" typ="incomingRequest" datum="2026-02-16T19:40:53.593Z" methode="GET" url="/api/scaffolder/v2/templates/default/template/example-nodejs-template/parameter-schema" status=304 httpVersion="1.1" userAgent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36" referrer="http://95.217.214.89:3000/"
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.625Z rootHttpRouter info [2026-02-16T19:40:53.625Z] "GET /api/auth/v1/userinfo HTTP/1.1" 200 108 "-" "node" typ="incomingRequest" datum="2026-02-16T19:40:53.625Z" methode="GET" url="/api/auth/v1/userinfo" status=200 httpVersion="1.1" userAgent="node" contentLength=108
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.627Z rootHttpRouter info [2026-02-16T19:40:53.627Z] "POST /api/permission/authorize HTTP/1.1" 200 108 "-" "node-fetch/1.0 (+https://github.com/bitinn/node-fetch)" typ="incomingRequest" datum="2026-02-16T19:40:53.627Z" methode="POST" url="/api/permission/authorize" status=200 httpVersion="1.1" userAgent="node-fetch/1.0 (+https://github.com/bitinn/node-fetch)" contentLength=108
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.628Z rootHttpRouter info [2026-02-16T19:40:53.628Z] "GET /api/catalog/entities/by-name/template/default/example-nodejs-template HTTP/1.1" 200 0 "http://95.217.214.89:3000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36" typ="incomingRequest" datum="2026-02-16T19:40:53.628Z" methode="GET" url="/api/catalog/entities/by-name/template/default/example-nodejs-template" status=200 httpVersion="1.1" userAgent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36" referrer="http://95.217.214.89:3000/"
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.648Z rootHttpRouter info [2026-02-16T19:40:53.648Z] "GET /api/auth/v1/userinfo HTTP/1.1" 200 108 "-" "node" typ="incomingRequest" datum="2026-02-16T19:40:53.648Z" methode="GET" url="/api/auth/v1/userinfo" status=200 httpVersion="1.1" userAgent="node" contentLength=108
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.649Z rootHttpRouter info [2026-02-16T19:40:53.649Z] "POST /api/permission/authorize HTTP/1.1" 200 108 "-" "node-fetch/1.0 (+https://github.com/bitinn/node-fetch)" typ="incomingRequest" datum="2026-02-16T19:40:53.649Z" methode="POST" url="/api/permission/authorize" status=200 httpVersion="1.1" userAgent="node-fetch/1.0 (+https://github.com/bitinn/node-fetch)" contentLength=108
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.651Z rootHttpRouter info [2026-02-16T19:40:53.651Z] "GET /api/catalog/entities/by-name/template/default/example-nodejs-template HTTP/1.1" 200 0 "-" "node-fetch/1.0 (+https://github.com/bitinn/node-fetch)" typ="incomingRequest" datum="2026-02-16T19:40:53.651Z" methode="GET" url="/api/catalog/entities/by-name/template/default/example-nodejs-template" status=200 httpVersion="1.1" userAgent="node-fetch/1.0 (+https://github.com/bitinn/node-fetch)"
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.656Z rootHttpRouter info [2026-02-16T19:40:53.656Z] "GET /api/auth/v1/userinfo HTTP/1.1" 200 108 "-" "node" typ="incomingRequest" datum="2026-02-16T19:40:53.656Z" methode="GET" url="/api/auth/v1/userinfo" status=200 httpVersion="1.1" userAgent="node" contentLength=108
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.657Z rootHttpRouter info [2026-02-16T19:40:53.657Z] "POST /api/permission/authorize HTTP/1.1" 200 197 "-" "node-fetch/1.0 (+https://github.com/bitinn/node-fetch)" typ="incomingRequest" datum="2026-02-16T19:40:53.657Z" methode="POST" url="/api/permission/authorize" status=200 httpVersion="1.1" userAgent="node-fetch/1.0 (+https://github.com/bitinn/node-fetch)" contentLength=197
Feb 16 19:40:53 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:40:53.658Z rootHttpRouter info [2026-02-16T19:40:53.658Z] "GET /api/scaffolder/v2/templates/default/template/example-nodejs-template/parameter-schema HTTP/1.1" 304 0 "http://95.217.214.89:3000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36" typ="incomingRequest" datum="2026-02-16T19:40:53.658Z" methode="GET" url="/api/scaffolder/v2/templates/default/template/example-nodejs-template/parameter-schema" status=304 httpVersion="1.1" userAgent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36" referrer="http://95.217.214.89:3000/"
Feb 16 19:42:35 patrickaboudou-backstage-setup-cet backstage[8722]: 2026-02-16T19:42:35.505Z rootHttpRouter info [2026-02-16T19:42:35.505Z] "GET /api/catalog/entities/by-name/user/development/guest HTTP/1.1" 404 671 "-" "node-fetch/1.0 (+https://github.com/bitinn/node-fetch















