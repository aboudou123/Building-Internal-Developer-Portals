# Plugin-Entwicklungsgrundlagen

Diese Lektion führt die Konzepte und Architekturmuster ein, die Sie im kommenden Lab verwenden werden, um benutzerdefinierte Backstage-Plugins zu erstellen.

## ⚠️ Voraussetzungen für die Plugin-Entwicklung

Die Plugin-Entwicklung erfordert solide JavaScript/TypeScript- und React-Kenntnisse. Sie sollten vertraut sein mit:

*   ES6+ JavaScript-Syntax (async/await, Destructuring, Arrow Functions)
*   TypeScript-Grundlagen (Typen, Interfaces, Generics)
*   React-Fundamentals (Komponenten, Hooks wie `useState` und `useEffect`, Props)
*   Node.js und Express.js für Backend-Entwicklung

Wenn Sie mit diesen Technologien neu sind, erwägen Sie, zuerst einführende Kurse zu absolvieren. Ohne dieses Grundlagenwissen könnte das Lab herausfordernd sein, aber es wird Sie dennoch durch den Prozess führen.

## Aufbauend auf bisherigem Wissen

Sie haben in diesem Kurs bereits viele Backstage-Plugins verwendet – Catalog, Templates, TechDocs, Kubernetes-Plugin. Jetzt lernen Sie, wie Sie Ihre eigenen erstellen, um Backstage mit organisationsspezifischer Funktionalität zu erweitern.

## Warum benutzerdefinierte Plugins wichtig sind

### Backstage für Ihre Organisation erweitern
Out-of-the-box Backstage ist leistungsstark, aber jede Organisation hat einzigartige Bedürfnisse. Benutzerdefinierte Plugins ermöglichen Ihnen:

*   **Proprietäre Systeme integrieren:** Verbindung zu internem Monitoring, Deployment-Tools oder Datenbanken herstellen
*   **Benutzerdefinierte Metriken anzeigen:** Team-Performance, Service-Health oder Business-KPIs zeigen
*   **Workflows automatisieren:** Organisationsspezifische Automatisierung erstellen, die zu Ihren Prozessen passt
*   **Branded Experiences kreieren:** An die Design-Sprache und Terminologie Ihres Unternehmens anpassen

**Das Platform-Engineering-Prinzip:** Generische Tools bringen Sie 80% voran. Benutzerdefinierte Plugins liefern die restlichen 20%, die Ihr Developer Portal wirklich zu Ihrem machen.

## Plugin-Architektur-Übersicht

### Alles ist ein Plugin
Wie Sie bereits erfahren haben, kommt Backstage-Funktionalität von Plugins:

*   **Core-Features:** Catalog, Templates, TechDocs (Abschnitt 1-2)
*   **Integrationen:** Kubernetes-Plugin (Abschnitt 6)
*   **Benutzerdefinierte Erweiterungen:** Was Sie in diesem Abschnitt erstellen werden

### Drei Plugin-Typen

**Frontend-Plugins:**

*   React-Komponenten und Seiten (Benutzeroberfläche)
*   Client-seitiges State-Management
*   Material-UI für konsistentes Design
*   Beispiele: Dashboards, Entity-Seiten, benutzerdefinierte Tools

**Backend-Plugins:**

*   REST-API-Endpunkte mit Express.js
*   Business-Logic und Datenverarbeitung
*   Externe Systemintegrationen
*   Beispiele: Datenaggregation, Service-Orchestrierung, API-Proxies

**Full-Stack-Plugins:**

*   Frontend und Backend arbeiten zusammen
*   Frontend konsumiert Backend-APIs
*   Komplette Features wie Team-Dashboards
*   Beispiele: Metriken-Anzeige, Admin-Panels, benutzerdefinierte Workflows

**Lab-Fokus:** Sie erstellen ein Full-Stack-Team-Dashboard-Plugin, das Frontend und Backend kombiniert.

## Entwicklungs-Workflow

### Scaffolding mit Backstage CLI
Die Backstage CLI (dasselbe Tool, das Sie für `yarn dev` verwendet haben) scaffolded Plugins:

```bash
# Frontend-Plugin erstellen
yarn backstage-cli new --select=frontend-plugin

# Backend-Plugin erstellen
yarn backstage-cli new --select=backend-plugin
```

Die CLI generiert eine korrekte Verzeichnisstruktur, Konfigurationsdateien und Boilerplate-Code gemäß Backstage-Konventionen.

### Plugin-Verzeichnisstruktur
Plugins leben im Monorepo, das Sie bereits kennen:

```
plugins/
├── team-dashboard/              # Frontend-Plugin
│   ├── src/components/          # React-Komponenten
│   ├── src/plugin.ts            # Plugin-Registrierung
│   └── package.json
└── team-dashboard-backend/      # Backend-Plugin
    ├── src/service/router.ts    # HTTP-Routen
    ├── src/plugin.ts            # Backend-Registrierung
    └── package.json
```

## Frontend-Plugin-Konzepte

### React-Komponenten-Patterns
Frontend-Plugins verwenden React mit dem Backstage-Design-System:

**State Management:**
*   `useState` – Verwalten von Komponenten-State (Loading, Data, Errors)
*   `useEffect` – Daten abrufen, wenn Komponente gemountet wird
*   `useApi` – Auf Backstage-APIs zugreifen (Config, Catalog, Discovery)

**Material-UI-Komponenten:**
*   `Page`, `Header`, `Content` – Layout-Struktur
*   `InfoCard` – Card-basierte Content-Container
*   `Grid` – Responsive Layouts
*   Standard Material-UI-Komponenten (`Typography`, `Chip`, `List`)

**API-Integration:**
Frontend-Komponenten rufen Backend-APIs auf mit:
*   `configApi.getString('backend.baseUrl')` – Backend-URL holen
*   `fetch()` oder typisierte API-Clients – HTTP-Anfragen stellen
*   Korrekte Fehlerbehandlung und Loading-States

**Beispiel-Pattern (Sie implementieren die vollständige Version im Lab):**

```typescript
const [data, setData] = useState(null);
const [loading, setLoading] = useState(true);
const configApi = useApi(configApiRef);

useEffect(() => {
  const fetchData = async () => {
    const backendUrl = configApi.getString('backend.baseUrl');
    const response = await fetch(`${backendUrl}/api/team-dashboard/metrics`);
    const data = await response.json();
    setData(data);
    setLoading(false);
  };
  fetchData();
}, [configApi]);
```

**Schlüsselkonzept:** Komponenten holen Daten von Backend-APIs, nicht direkt von externen Systemen. Diese Trennung ermöglicht Sicherheit, Caching und Business-Logic.

## Backend-Plugin-Konzepte

### Service-Layer-Architektur
Backend-Plugins trennen Anliegen in Schichten:

**Service Layer (Business-Logic):**
*   Datenaggregation und -transformation
*   Integration mit externen APIs
*   Business-Regeln und Berechnungen
*   Catalog-Queries und -Verarbeitung

**Router Layer (HTTP-Endpunkte):**
*   Express.js-Routendefinitionen
*   Request-Validierung
*   Fehlerbehandlung
*   Delegiert Logik an Service-Layer

**Beispiel-Pattern (Sie implementieren die vollständige Version im Lab):**

```typescript
// Service layer
class TeamDataService {
  async getTeamMetrics(teamName: string) {
    // Catalog abfragen, Metriken berechnen, Daten zurückgeben
    const memberCount = await this.getTeamMemberCount(teamName);
    const activeServices = await this.getActiveServiceCount(teamName);
    return { teamName, memberCount, activeServices };
  }
}

// Router layer
router.get('/teams/:teamName/metrics', async (req, res) => {
  const metrics = await teamDataService.getTeamMetrics(req.params.teamName);
  res.json(metrics);
});
```

**Schlüsselkonzept:** Router sind dünn (nur HTTP-Handling), Services sind dick (enthalten Business-Logic). Dies erleichtert Tests und macht den Code wartbarer.

### Catalog-Integration
Backend-Plugins fragen häufig den Backstage-Catalog ab:

```typescript
// Get group entity and member count
const groupRef = `group:default/${teamName}`;
const groupEntity = await catalogApi.getEntityByRef(groupRef);
const memberCount = groupEntity?.spec?.members?.length || 0;
```

Dieses Pattern ermöglicht es Ihnen, Features auf Basis von Catalog-Daten zu erstellen, ohne sie zu duplizieren.

## Plugin-Integration

### Frontend-Plugins registrieren
Nach Erstellung eines Plugins integrieren Sie es in Ihre Backstage-App, indem Sie eine Route hinzufügen:

1.  Importieren der Plugin-Page-Komponente
2.  Hinzufügen einer Route in `packages/app/src/App.tsx`
3.  Plugin erscheint in der Backstage-Navigation

### Backend-Plugins registrieren
Backend-Plugins registrieren sich mit dem neuen Backend-System:

1.  Importieren des Backend-Plugin-Moduls
2.  Hinzufügen zu `packages/backend/src/index.ts`
3.  Backend entdeckt und mountet automatisch die Routen des Plugins

Das Lab führt Sie durch beide Integrationsschritte im Detail.

## Entwicklungs-Best Practices

### Design-Prinzipien
**Single Responsibility:**
*   Jedes Plugin sollte eine Sache gut machen
*   Vermeiden von "Mega-Plugins", die alles versuchen

**Loose Coupling:**
*   Abhängigkeiten von anderen Plugins minimieren
*   Gut definierte API-Contracts verwenden (TypeScript-Interfaces)

**Fehlerbehandlung:**
*   Graceful Degradation, wenn Services nicht verfügbar sind
*   Benutzerfreundliche Fehlermeldungen anzeigen
*   Fehler ordnungsgemäß für Debugging loggen

### Test-Strategie
**Unit Tests:**
*   Service-Logik unabhängig testen
*   Externe Abhängigkeiten mocken (Catalog, APIs)
*   Fokus auf Korrektheit der Business-Logic

**Component Tests:**
*   React-Komponenten isoliert testen
*   API-Antworten mocken
*   Verifizieren, dass UI für verschiedene States korrekt rendert

**Integration Tests:**
*   Verifizieren, dass Plugin mit Backstage-APIs funktioniert
*   End-to-End-Workflows testen
*   Echte API-Contracts validieren

### Performance-Optimierung
**Lazy Loading:**
*   Plugin-Code nur bei Bedarf laden
*   Reduziert initiale Bundle-Größe

**Caching:**
*   Teure API-Aufrufe angemessen cachen
*   React Query oder ähnliches für Data-Fetching verwenden

**Pagination:**
*   Große Datensätze effizient handhaben
*   Nicht alles auf einmal abrufen

**Loading-States:**
*   Immer Feedback während asynchroner Operationen geben
*   Benutzer sollten nie im Unklaren sein, ob etwas funktioniert

## Was Sie im Lab erstellen werden
Das kommende Lab setzt diese Konzepte in die Praxis um:

1.  **Plugin-Foundation scaffolden** – Backstage CLI verwenden, um Frontend- und Backend-Plugins zu generieren
2.  **Backend-API erstellen** – Service-Layer erstellen, der den Catalog nach Team-Daten abfragt
3.  **React-Frontend erstellen** – Dashboard mit Material-UI bauen, das Team-Metriken anzeigt
4.  **Plugin integrieren** – Routen hinzufügen und Plugin in Backstage-App registrieren

Am Ende haben Sie ein funktionierendes Team-Dashboard, das alle hier behandelten Patterns demonstriert.

## Kernpunkte
*   Backstage hat drei Plugin-Typen: Frontend (React UI), Backend (Express.js APIs) und Full-Stack (beide zusammenarbeitend)
*   Frontend-Plugins verwenden React mit Material-UI-Komponenten, State-Management-Hooks (`useState`/`useEffect`) und `useApi` für Backstage-Integration
*   Backend-Plugins trennen Business-Logic (Service-Layer) von HTTP-Handling (Router-Layer) unter Verwendung von Dependency-Injection-Patterns
*   Plugins integrieren sich in Backstage-Core-APIs wie die Catalog-API, um Features auf existierenden Daten ohne Duplikation zu erstellen
*   Die Backstage CLI scaffolded Plugin-Boilerplate mit `yarn backstage-cli new`, erstellt korrekte Verzeichnisstruktur und Konfiguration
*   Plugin-Entwicklung folgt Best Practices: Single Responsibility, Loose Coupling, korrekte Fehlerbehandlung und umfassende Tests

## Zusätzliche Ressourcen
*   [Plugin Development Guide](https://backstage.io/docs/plugins/)
*   [New Backend System](https://backstage.io/docs/backend-system/)
*   [Frontend Plugin Tutorial](https://backstage.io/docs/plugins/frontend-plugin-tutorial/)


## Lektionsnotizen (Privat)

Nehmen Sie private Notizen während Sie lernen...

## Studiengruppe
Diesem Kurs ist noch keine Studiengruppe zugeordnet.

Bitten Sie Ihren Instruktor, eine Studiengruppe zu verlinken, um Diskussionen zu ermöglichen.
```