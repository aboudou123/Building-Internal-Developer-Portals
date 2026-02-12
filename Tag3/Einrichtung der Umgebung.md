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


**Erstellen der ersten Backstage-Anwendung**
Jetzt erstellen Sie eine neue Backstage-Anwendung mithilfe des offiziellen CLI. Dies erzeugt eine vollständige Projektstruktur mit allen erforderlichen Dateien und Abhängigkeiten.

---

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

```bash

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

=====================================

====================================

---

### Ziel:

* Standardmäßig hört Backstage nur auf `localhost` (127.0.0.1) → nur lokal erreichbar.
* Um Backstage aus dem Netzwerk deiner VM erreichbar zu machen, musst du die URL- und Netzwerk-Einstellungen auf die IP-Adresse deiner VM ändern und Backstage so konfigurieren, dass es auf allen Interfaces (`0.0.0.0`) lauscht.

---

### Schritt-für-Schritt-Anleitung für deine Datei `app-config.yaml`:

---

#### 1. **Finde die IP-Adresse deiner VM:**

Im Terminal:

```bash
hostname -I | awk '{print $1}'
```

Nehmen wir an, die Ausgabe ist:

```
192.168.1.100
```

Diese IP-Adresse ersetzt du dann in den nächsten Schritten.

---

#### 2. **Ändere `app.baseUrl`:**

Aktuell steht in deiner Datei:

```yaml
app:
  baseUrl: http://localhost:3000
```

Ersetze das mit deiner IP-Adresse, also:

```yaml
app:
  baseUrl: http://192.168.1.100:3000
```

---

#### 3. **Ändere `backend.baseUrl`:**

Aktuell:

```yaml
backend:
  baseUrl: http://localhost:7007
```

Ersetze das durch:

```yaml
backend:
  baseUrl: http://192.168.1.100:7007
```

---

#### 4. **Passe die `backend.listen`-Einstellungen an:**

Du hast aktuell:

```yaml
backend:
  listen:
    port: 7007
    # host: 127.0.0.1
```

**Entferne das Kommentarzeichen vor `host` und ändere es auf `0.0.0.0`:**

```yaml
backend:
  listen:
    port: 7007
    host: 0.0.0.0
```

> Dadurch lauscht der Backend-Server an **allen Netzwerk-Interfaces** und nicht nur lokal.

---

#### 5. **Ändere die CORS-Origin Einstellung:**

Aktuell steht:

```yaml
backend:
  cors:
    origin: http://localhost:3000
```

Ersetze es durch:

```yaml
backend:
  cors:
    origin: http://192.168.1.100:3000
```

---

### Vollständige relevante Sektion im `app-config.yaml` nach deiner Änderung:

```yaml
app:
  title: Scaffolded Backstage App
  baseUrl: http://192.168.1.100:3000

backend:
  baseUrl: http://192.168.1.100:7007
  listen:
    port: 7007
    host: 0.0.0.0
  csp:
    connect-src: ["'self'", 'http:', 'https:']
  cors:
    origin: http://192.168.1.100:3000
    methods: [GET, HEAD, PATCH, POST, PUT, DELETE]
    credentials: true
  database:
    client: better-sqlite3
    connection: ':memory:'
```

---







================================
Hier ist die fertige Datei:

```yaml
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
```

---

### ✅ Wichtige Punkte:

1. **Frontend:** `app.baseUrl` → `http://172.29.84.114:3000`
2. **Backend:** `backend.baseUrl` → `http://172.29.84.114:7007`
3. **Backend host:** `0.0.0.0` → damit externe Geräte (z.B. dein Browser) die App erreichen können
4. Alle Einrückungen sind **2 Leerzeichen pro Ebene**
5. Kommentare und auskommentierte Abschnitte sind korrekt eingerückt

---


### Nach den Änderungen:

* **Speichere** die Datei (`Ctrl + O` in nano), dann `Enter` und `Ctrl + X` zum Verlassen.
* Starte deinen Backstage-Server neu:

```bash
yarn start
```

---

### Zugriff von deinem Browser:

Gib jetzt in deinem Browser (z.B. auf deinem Host-PC oder einem anderen Gerät im gleichen Netzwerk) ein:

```
http://192.168.1.100:3000
```

---

Falls du deine öffentliche IP verwendest, musst du sicherstellen, dass keine Firewall den Zugriff blockiert.

---

### Zusammenfassung

* Ändere `localhost` URLs auf deine VM-IP.
* Backend soll auf `0.0.0.0` hören.
* Passe CORS-Origins an.
* Starte Backstage neu.

---


<img width="1150" height="807" alt="image" src="https://github.com/user-attachments/assets/5ae249e2-e647-4df2-91d4-669456acbb87" />












