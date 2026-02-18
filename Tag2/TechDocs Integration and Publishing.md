
# TechDocs-Setup in Backstage.


========================================

---

## **1️⃣ Plugins installieren**

### Frontend Plugin

```bash
yarn --cwd packages/app add @backstage/plugin-techdocs
```

* Zeigt Dokumentation im UI an
* Fügt **TechDocs Index** und **Reader** Seiten hinzu
* Integriert Dokumentations-Tabs in Entity-Seiten

### Backend Plugin

```bash
yarn --cwd packages/backend add @backstage/plugin-techdocs-backend
```

* Generiert Dokumentation aus MkDocs-Quellen
* Dient sie dem Frontend
* Speichert Dokumentation lokal oder in der Cloud
* Führt Builds on-demand oder pre-built aus

#### Backend Registrierung

`packages/backend/src/index.ts`:

```ts
backend.add(import('@backstage/plugin-techdocs-backend'));
```

---

## **2️⃣ App-Config Setup (`app-config.yaml`)**

### Beispiel für lokale Speicherung:

```yaml
techdocs:
  builder: 'local'
  generator:
    runIn: 'local'
    mkdocsCommand: 'mkdocs' # oder '/opt/mkdocs-venv/bin/mkdocs'
  publisher:
    type: 'local'
    local:
      storageDir: '/root/labs/developer-portal/techdocs-storage'
```

**Erklärung:**

* `builder: 'local'` → Build lokal, nicht extern
* `generator.runIn: 'local'` → MkDocs lokal ausführen
* `publisher.local.storageDir` → Speicherort der generierten HTML-Dateien

Für Produktion: AWS S3 / GCS / Azure Blob Storage als Publisher nutzen.

---

## **3️⃣ Frontend Routes (`App.tsx`)**

```tsx
import { TechDocsIndexPage, TechDocsReaderPage } from '@backstage/plugin-techdocs';

<FlatRoutes>
  {/* ... andere Routen ... */}
  <Route path="/docs" element={<TechDocsIndexPage />} />
  <Route path="/docs/:namespace/:kind/:name/*" element={<TechDocsReaderPage />} />
</FlatRoutes>
```

**Erklärung:**

* `/docs` → Index aller Dokumentationen
* `/docs/:namespace/:kind/:name/*` → Dokumentation einer einzelnen Entity

---

## **4️⃣ Catalog Entity Annotation**

Jede Komponente, die Dokumentation haben soll, benötigt die Annotation:

```yaml
# catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: sample-service
  title: Sample Microservice
  annotations:
    backstage.io/techdocs-ref: dir:.
spec:
  type: service
  owner: platform-team
  lifecycle: experimental
```

**Optionen:**

* `dir:.` → MkDocs im Root
* `dir:./docs` → MkDocs in `docs/` Unterordner
* `url: https://github.com/org/repo` → Externes Repo

---

## **5️⃣ Registrierung im Catalog**

```yaml
catalog:
  locations:
    - type: file
      target: /root/sample-repos/sample-service/catalog-info.yaml
```

**Effekt:**

* Entity erscheint im Catalog
* TechDocs-Tab verfügbar
* Docs erreichbar unter `/docs/default/component/sample-service`

---

## **6️⃣ Lokaler Build**

```bash
cd /root/sample-repos/sample-service
mkdocs build
# Output in site/ Verzeichnis
ls -la site/
```

### Manuelle Veröffentlichung

```bash
cp -r site/* /root/labs/developer-portal/techdocs-storage/default/component/sample-service/
```

---

## **7️⃣ Automatisierte Veröffentlichung (CI/CD)**

Beispiel GitHub Actions Workflow:

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

**Vorteile:**

* Dokumentation immer aktuell
* Keine manuellen Schritte
* Versionierung durch Git

---
# Zusätzliche Ressourcen für TechDocs

## 1️⃣ MkDocs Benutzerhandbuch
- **Link:** [MkDocs User Guide](https://www.mkdocs.org/user-guide/)
- **Beschreibung:** Offizielles Handbuch für MkDocs.
  - Installation & Einrichtung
  - Projektstruktur (`mkdocs.yml`, `docs/` Ordner)
  - Themes & Plugins
  - Befehle: `mkdocs serve`, `mkdocs build`, `mkdocs gh-deploy`
- **Tipp:** Verwenden, um Dokumentation lokal zu testen und Builds zu überprüfen.

## 2️⃣ Material für MkDocs
- **Link:** [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- **Beschreibung:** Material Design Theme für MkDocs.
  - Modernes Styling für Backstage Docs
  - Navigation, Sidebar, Tabellen, Buttons
  - Erweiterungen: Suche, Tabs, Admonitions, Code-Highlighting
- **Tipp:** Fast alle TechDocs-Beispiele nutzen dieses Theme. Sieht in Backstage am besten aus.

## 3️⃣ Markdown Anleitung
- **Link:** [Markdown Guide](https://www.markdownguide.org/basic-syntax/)
- **Beschreibung:** Grundlagen von Markdown.
  - Überschriften, Listen, Links, Tabellen, Code-Blöcke
  - Erweiterungen wie Fußnoten oder Aufgabenlisten
- **Tipp:** Markdown ist die Basis jeder MkDocs-Dokumentation. Saubere Formatierung sorgt für bessere TechDocs-Darstellung.


