**Einrichtung der Umgebung und Voraussetzungen**
In dieser Aufgabe überprüfen Sie, ob Ihre Entwicklungsumgebung für Backstage bereit ist, und installieren fehlende Abhängigkeiten. Backstage benötigt Node.js, npm/yarn und Git, um korrekt zu funktionieren.

---

**Schritt 1: Node.js-Installation überprüfen**
Stellen Sie sicher, dass Node.js installiert ist. Führen Sie dazu im Terminal-Tab folgenden Befehl aus:

```bash
node --version
```

Sie sollten `v22.x.x` sehen (Node.js 22 wird für die Kompatibilität der nativen Module von Backstage benötigt).

---
<img width="1050" height="137" alt="image" src="https://github.com/user-attachments/assets/82f3cbb8-58a0-4940-a057-73e40a7d8949" />



**Schritt 2: npm und Yarn überprüfen**
**Schritt 2: npm und Yarn überprüfen**
Überprüfen Sie, ob npm ordnungsgemäß funktioniert:
it 
```bash
npm --version
```
<img width="1038" height="273" alt="image" src="https://github.com/user-attachments/assets/3383efd9-4990-41a5-a3e2-afc551b2ad1c" />

Stellen Sie sicher, dass Yarn installiert ist (Backstage funktioniert besser mit Yarn als mit npm):

```bash
yarn --version
```

Sie sollten Version `1.22.x` sehen. Yarn wurde im Rahmen der Lab-Umgebung bereits vorinstalliert.


**Erstellen Ihrer ersten Backstage-Anwendung**
Jetzt erstellen Sie eine neue Backstage-Anwendung mithilfe des offiziellen CLI. Dies erzeugt eine vollständige Projektstruktur mit allen erforderlichen Dateien und Abhängigkeiten.

---

**Schritt 1: Neue Backstage-App erstellen**
Stellen Sie zunächst sicher, dass Sie sich in Ihrem Arbeitsverzeichnis befinden:

```bash
cd /root/labs
```

Erstellen Sie nun eine neue Backstage-Anwendung unter Verwendung der festgelegten Version für die Kompatibilität:

```bash
echo 'developer-portal' | npx @backstage/create-app@0.7.7
```

Dieser Befehl leitet den App-Namen „developer-portal“ an das create-app CLI weiter. Der Vorgang dauert einige Minuten, da Abhängigkeiten heruntergeladen und die Projektstruktur eingerichtet werden.








**Schritt 3: Git-Installation überprüfen**
Stellen Sie sicher, dass Git für die Versionskontrolle verfügbar ist:

```bash
git --version
```

Konfigurieren Sie Git mit Ihrem Namen (ersetzen Sie dies durch Ihren tatsächlichen Namen):

```bash
git config --global user.name "Your Name"
```

Konfigurieren Sie Git mit Ihrer E-Mail-Adresse:

```bash
git config --global user.email "your.email@example.com"
```

---

**Schritt 4: Arbeitsverzeichnis erstellen**
Erstellen Sie eine dedizierte Verzeichnisstruktur für Ihre Backstage-Entwicklung:

```bash
mkdir -p /root/labs
```

Wechseln Sie in Ihr neues Arbeitsverzeichnis:

```bash
cd /root/labs
```

**Schritt 2: In das Projektverzeichnis wechseln**
Wechseln Sie in Ihr neues Backstage-Projekt:

```bash
cd developer-portal
```

---

**Schritt 3: Die erzeugten Dateien erkunden**
Listen Sie die Hauptverzeichnisse und Dateien auf:

```bash
ls -la
```

Sie sollten Verzeichnisse und Dateien wie diese sehen:

* `packages/` – enthält Frontend- und Backend-Code
* `app-config.yaml` – Hauptkonfigurationsdatei
* `package.json` – Projektabhängigkeiten
* `catalog-info.yaml` – Katalog-Entity für die App selbst

---

**Schritt 4: Netzwerkzugang konfigurieren**
Standardmäßig hört Backstage nur auf `localhost`. Um den Zugriff aus dem Netzwerk zu ermöglichen, müssen Sie die Konfiguration anpassen.

Ermitteln Sie die IP-Adresse Ihrer VM:

```bash
hostname -I | awk '{print $1}'
```

Öffnen Sie den Code-Editor und navigieren Sie zu `/root/labs/developer-portal/app-config.yaml`. Aktualisieren Sie folgende Schlüsselbereiche:

* Ändern Sie `app.baseUrl` von `http://localhost:3000` auf `http://IHRE_VM_IP:3000`
* Ändern Sie `backend.baseUrl` von `http://localhost:7007` auf `http://IHRE_VM_IP:7007`
* Unter `backend.listen` heben Sie die Auskommentierung auf und setzen `host: 0.0.0.0`
* Ändern Sie `backend.cors.origin` von `http://localhost:3000` auf `http://IHRE_VM_IP:3000`

Ersetzen Sie `IHRE_VM_IP` durch die zuvor ermittelte IP-Adresse.

Ihre Backstage-Anwendung ist nun erstellt und für den Remote-Zugriff konfiguriert! Als Nächstes werden wir sie starten und die Benutzeroberfläche erkunden.

app:
  title: Scaffolded Backstage App
  baseUrl: http://172.29.84.114:3000

organization:
  name: My Company

backend:
  # Used for enabling authentication, secret is shared by all backend plugins
  baseUrl: http://172.29.84.114:7007
  listen:
    port: 7007
    host: 0.0.0.0
  csp:
    connect-src: ["'self'", 'http:', 'https:']
  cors:
    origin: http://172.29.84.114:3000
    methods: [GET, HEAD, PATCH, POST, PUT, DELETE]
    credentials: true
  database:
    client: better-sqlite3
    connection: ':memory:'

integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}
    # Example for GitHub Enterprise:
    # - host: ghe.example.net
    #   apiBaseUrl: https://ghe.example.net/api/v3
    #   token: ${GHE_TOKEN}

proxy:
  # Example proxy endpoint:
  # endpoints:
  #   '/test':
  #     target: 'https://example.com'
  #     changeOrigin: true

techdocs:
  builder: 'local'
  generator:
    runIn: 'docker'
  publisher:
    type: 'local'

auth:
  providers:
    guest: {}

scaffolder:
  # Configuration for software templates goes here

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

    # Additional example data (optional)
    # - type: url
    #   target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/all.yaml
    # - type: url
    #   target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/acme-corp.yaml
    #   rules:
    #     - allow: [User, Group]

kubernetes:
  # Configuration for Kubernetes integration goes here

permission:
  enabled: true



===========================================




app:
  title: Scaffolded Backstage App
  baseUrl: http://172.29.84.114:3000

organization:
  name: My Company

backend:
  # Used for enabling authentication, secret is shared by all backend plugins
  baseUrl: http://172.29.84.114:7007
  listen:
    port: 7007
    host: 0.0.0.0
  csp:
    connect-src: ["'self'", 'http:', 'https:']
  cors:
    origin: http://172.29.84.114:3000
    methods: [GET, HEAD, PATCH, POST, PUT, DELETE]
    credentials: true
  database:
    client: better-sqlite3
    connection: ':memory:'

integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}
    # Example for GitHub Enterprise:
    # - host: ghe.example.net
    #   apiBaseUrl: https://ghe.example.net/api/v3
    #   token: ${GHE_TOKEN}

proxy:
  # Example proxy endpoint:
  # endpoints:
  #   '/test':
  #     target: 'https://example.com'
  #     changeOrigin: true

techdocs:
  builder: 'local'
  generator:
    runIn: 'docker'
  publisher:
    type: 'local'

auth:
  providers:
    guest: {}

scaffolder:
  # Configuration for software templates goes here

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

    # Additional example data (optional)
    # - type: url
    #   target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/all.yaml
    # - type: url
    #   target: https://github.com/backstage/backstage/blob/master/packages/catalog-model/examples/acme-corp.yaml
    #   rules:
    #     - allow: [User, Group]

kubernetes:
  # Configuration for Kubernetes integration goes here

permission:
  enabled: true




<img width="1150" height="807" alt="image" src="https://github.com/user-attachments/assets/5ae249e2-e647-4df2-91d4-669456acbb87" />












