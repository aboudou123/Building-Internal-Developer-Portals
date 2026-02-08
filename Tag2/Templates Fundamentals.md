---

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
