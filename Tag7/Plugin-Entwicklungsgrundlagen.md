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

```


```markdown
# Material-UI-Anpassung in Backstage

Diese Lektion behandelt Material-UI-Patterns und Theme-Anpassungen, die Sie beim Erstellen von Backstage-Plugins anwenden werden. Das Verständnis dieser Patterns vor dem praxisorientierten Lab hilft Ihnen, polierte, konsistente Schnittstellen zu erstellen.

## Material-UI im Backstage-Kontext

### Versionen-Berücksichtigung
Backstage verwendet Material-UI v4 (jetzt MUI v4 genannt). Wenn Sie Dokumentation oder Beispiele online suchen, stellen Sie sicher, dass Sie v4-Patterns ansehen, nicht v5/MUI-Patterns, die eine andere Syntax haben.

**Wichtige Unterschiede zu beachten:**
*   **Import-Pfade:** v4 verwendet `@material-ui/core`, v5 verwendet `@mui/material`
*   **Styling-Ansatz:** v4 verwendet `makeStyles`, v5 verwendet `styled` oder `sx` prop
*   **Theme-API:** Ähnliche Konzepte, aber unterschiedliche Implementierungsdetails

### Import-Patterns
Backstage-Plugins verwenden zwei Hauptimportquellen für UI-Komponenten:

**Material-UI-Komponenten** (generische UI-Bausteine):
```typescript
import {
  Grid,
  Card,
  CardContent,
  Typography,
  Chip,
  Table,
  TableHead,
  TableBody,
  TableRow,
  TableCell,
  CircularProgress,
  Button
} from '@material-ui/core';
```

**Backstage Core Components** (Backstage-spezifische Wrapper und Patterns):
```typescript
import {
  Page,
  Header,
  Content,
  InfoCard,
  StatusOK,
  StatusWarning,
  StatusError,
  Progress,
  ResponseErrorPanel
} from '@backstage/core-components';
```

**Best Practice:** Verwenden Sie Backstage-Komponenten, wenn verfügbar (sie handhaben Theming und Barrierefreiheit), greifen Sie für Lücken auf Material-UI zurück.

## Wichtige Backstage-Komponenten

### Page-Layout-Struktur
Jede Backstage-Seite folgt einer konsistenten Struktur:

```typescript
import { Page, Header, Content } from '@backstage/core-components';

export const MyPluginPage = () => (
  <Page themeId="tool">
    <Header title="Team Dashboard" subtitle="View team metrics and services" />
    <Content>
      {/* Ihr Inhalt kommt hierhin */}
    </Content>
  </Page>
);
```

**Komponenten-Rollen:**
*   **Page** – Container, der Theme-Kontext und Navigation-Integration handhabt
*   **Header** – Konsistenter Header mit Titel, Untertitel und optionalen Aktionsbuttons
*   **Content** – Hauptinhaltsbereich mit korrektem Padding und responsivem Verhalten

Die `themeId`-Prop akzeptiert Werte wie `tool`, `service`, `website`, `library` oder `documentation`, um entsprechendes Styling anzuwenden.

### InfoCard für Content-Container
InfoCard ist der primäre Container für gruppierte Informationen:

```typescript
import { InfoCard } from '@backstage/core-components';
import { Typography } from '@material-ui/core';

<InfoCard title="Service Health" subheader="Last 24 hours">
  <Typography variant="body1">
    All 12 services operational
  </Typography>
</InfoCard>
```

**InfoCard-Props:**
*   `title` – Erforderlicher Header-Text
*   `subheader` – Optionaler Untertitel
*   `action` – Optionaler Aktionsbutton im Header
*   `deepLink` – Optionaler "View All"-Style-Link
*   `variant` – Card-Styling-Variante (`gridItem`, `fullHeight`)

### Status-Indikatoren
Backstage bietet semantische Status-Komponenten:

```typescript
import { StatusOK, StatusWarning, StatusError, StatusPending } from '@backstage/core-components';

// Verwenden für Service-Health, Deployment-Status, etc.
<StatusOK>Healthy</StatusOK>
<StatusWarning>Degraded</StatusWarning>
<StatusError>Down</StatusError>
<StatusPending>Deploying</StatusPending>
```

Diese wenden automatisch konsistente Farben und Icons über Ihr Portal hinweg an.

## Responsive Grid-Layouts

### Das 12-Spalten-System
Material-UI Grid verwendet ein 12-Spalten-responsives Layout. Komponenten definieren, wie viele Spalten sie bei verschiedenen Breakpoints überspannen:

```typescript
import { Grid } from '@material-ui/core';
import { InfoCard } from '@backstage/core-components';

<Grid container spacing={3}>
  {/* Volle Breite auf kleinen Screens, halb auf medium, ein Drittel auf large */}
  <Grid item xs={12} md={6} lg={4}>
    <InfoCard title="Team Members">
      <Typography>8 engineers</Typography>
    </InfoCard>
  </Grid>

  <Grid item xs={12} md={6} lg={4}>
    <InfoCard title="Active Services">
      <Typography>12 services</Typography>
    </InfoCard>
  </Grid>

  <Grid item xs={12} md={6} lg={4}>
    <InfoCard title="Recent Deployments">
      <Typography>23 this week</Typography>
    </InfoCard>
  </Grid>
</Grid>
```

**Breakpoints-Referenz:**

| Breakpoint | Bildschirmgröße | Verwendung |
| :--- | :--- | :--- |
| `xs` | 0px+ | Mobiltelefone |
| `sm` | 600px+ | Tablets im Hochformat |
| `md` | 960px+ | Tablets im Querformat, kleine Laptops |
| `lg` | 1280px+ | Desktop-Rechner |
| `xl` | 1920px+ | Große Displays |

**Grid-Props:**
*   `container` – Definiert einen Flex-Container
*   `item` – Definiert ein Flex-Item
*   `spacing` – Abstand zwischen Items (1 = 8px, 2 = 16px, 3 = 24px)
*   `xs`/`sm`/`md`/`lg`/`xl` – Spalten-Span bei jedem Breakpoint

### Häufige Layout-Patterns
**Zwei-Spalten-Dashboard:**
```typescript
<Grid container spacing={3}>
  <Grid item xs={12} md={8}>
    {/* Hauptinhalt - 2/3 Breite auf Desktop */}
    <InfoCard title="Activity Feed">...</InfoCard>
  </Grid>
  <Grid item xs={12} md={4}>
    {/* Sidebar - 1/3 Breite auf Desktop */}
    <InfoCard title="Quick Stats">...</InfoCard>
  </Grid>
</Grid>
```

**Equal Cards Row:**
```typescript
<Grid container spacing={3}>
  {cards.map(card => (
    <Grid item xs={12} sm={6} md={4} lg={3} key={card.id}>
      <InfoCard title={card.title}>
        <Typography>{card.value}</Typography>
      </InfoCard>
    </Grid>
  ))}
</Grid>
```

## Mit Chips und Tags arbeiten
Chips zeigen kompakte Informationen wie Status, Labels oder Tags an:

```typescript
import { Chip, Box } from '@material-ui/core';

// Basic chip
<Chip label="Production" />

// Colored chips for status
<Chip label="Running" style={{ backgroundColor: '#4caf50', color: 'white' }} />
<Chip label="Stopped" style={{ backgroundColor: '#f44336', color: 'white' }} />

// Chips with actions
<Chip label="Filter: Backend" onDelete={() => removeFilter('backend')} />

// Group of tags
<Box display="flex" gap={1} flexWrap="wrap">
  <Chip label="kubernetes" size="small" />
  <Chip label="node.js" size="small" />
  <Chip label="postgres" size="small" />
</Box>
```

**Chip-Props:**
*   `label` – Text-Inhalt
*   `color` – `default`, `primary`, `secondary`
*   `size` – `small`, `medium`
*   `variant` – `default`, `outlined`
*   `onDelete` – Macht Chip mit X-Button löschbar

## Daten-Tabellen

### Grundlegende Tabellen-Struktur
```typescript
import {
  Table,
  TableHead,
  TableBody,
  TableRow,
  TableCell
} from '@material-ui/core';

<Table>
  <TableHead>
    <TableRow>
      <TableCell>Service</TableCell>
      <TableCell>Status</TableCell>
      <TableCell align="right">Uptime</TableCell>
    </TableRow>
  </TableHead>
  <TableBody>
    {services.map(service => (
      <TableRow key={service.name}>
        <TableCell>{service.name}</TableCell>
        <TableCell>
          <StatusOK>{service.status}</StatusOK>
        </TableCell>
        <TableCell align="right">{service.uptime}%</TableCell>
      </TableRow>
    ))}
  </TableBody>
</Table>
```

### Backstage Table-Komponente
Für mehr Features (Sortieren, Filtern, Pagination) verwenden Sie Backstage's Table:

```typescript
import { Table, TableColumn } from '@backstage/core-components';

const columns: TableColumn[] = [
  { title: 'Service', field: 'name' },
  { title: 'Status', field: 'status', render: row => <StatusOK>{row.status}</StatusOK> },
  { title: 'Uptime', field: 'uptime', type: 'numeric' }
];

<Table
  title="Service Overview"
  columns={columns}
  data={services}
  options={{
    search: true,
    paging: true,
    pageSize: 10
  }}
/>
```

## Theme-Anpassung

### Backstage-Themes verstehen
Backstage unterstützt mehrere Themes (Light, Dark, Custom). Themes kontrollieren:
*   Farbpalette (Primary, Secondary, Background, Text)
*   Typografie (Fonts, Größen, Gewichte)
*   Navigation-Styling
*   Komponenten-Erscheinung

### Benutzerdefiniertes Theme erstellen
Erstellen Sie eine Theme-Datei in Ihrem App-Package:

```typescript
// packages/app/src/themes/companyTheme.ts
import { createTheme, lightTheme, darkTheme } from '@backstage/theme';

export const companyLightTheme = createTheme({
  palette: {
    ...lightTheme.palette,
    primary: {
      main: '#0052CC',  // Firmenblau
    },
    secondary: {
      main: '#FF5630',  // Akzent-Orange
    },
    navigation: {
      background: '#172B4D',  // Dunkler Nav-Hintergrund
      indicator: '#0052CC',   // Indikator für aktives Item
      color: '#FFFFFF',       // Nav-Textfarbe
      selectedColor: '#FFFFFF',
    },
  },
  defaultPageTheme: 'home',
  fontFamily: 'Inter, Roboto, sans-serif',
  pageTheme: {
    home: {
      colors: ['#0052CC', '#172B4D'],
      shape: 'wave',
      backgroundImage: '/static/background.svg',
    },
  },
});

export const companyDarkTheme = createTheme({
  palette: {
    ...darkTheme.palette,
    primary: {
      main: '#4C9AFF',  // Helleres Blau für Dark Mode
    },
    navigation: {
      background: '#0D1117',
      indicator: '#4C9AFF',
      color: '#C9D1D9',
      selectedColor: '#FFFFFF',
    },
  },
  defaultPageTheme: 'home',
  fontFamily: 'Inter, Roboto, sans-serif',
});
```

### Benutzerdefinierte Themes registrieren
Aktualisieren Sie Ihre `App.tsx`, um die benutzerdefinierten Themes zu verwenden:

```typescript
// packages/app/src/App.tsx
import { companyLightTheme, companyDarkTheme } from './themes/companyTheme';
import { UnifiedThemeProvider } from '@backstage/theme';

const app = createApp({
  // ... andere Konfiguration
  themes: [
    {
      id: 'company-light',
      title: 'Company Light',
      variant: 'light',
      Provider: ({ children }) => (
        <UnifiedThemeProvider theme={companyLightTheme} children={children} />
      ),
    },
    {
      id: 'company-dark',
      title: 'Company Dark',
      variant: 'dark',
      Provider: ({ children }) => (
        <UnifiedThemeProvider theme={companyDarkTheme} children={children} />
      ),
    },
  ],
});
```

Benutzer können dann zwischen Themes mit Backstage's eingebautem Theme-Selector wechseln.

## Häufige UI-Patterns

### Loading-States
Immer Loading-Feedback während asynchroner Operationen anzeigen:

```typescript
import { Progress } from '@backstage/core-components';
import { CircularProgress, Box } from '@material-ui/core';

// Full page loading (während initialem Data-Fetch)
if (loading) {
  return <Progress />;
}

// Inline loading (für partielle Updates)
<Box display="flex" alignItems="center" gap={1}>
  <CircularProgress size={20} />
  <Typography>Refreshing data...</Typography>
</Box>
```

### Fehlerbehandlung
Fehler klar mit Wiederherstellungsoptionen anzeigen:

```typescript
import { ResponseErrorPanel } from '@backstage/core-components';

if (error) {
  return (
    <ResponseErrorPanel
      error={error}
      title="Failed to load team data"
    />
  );
}

// Oder custom error display
<InfoCard title="Error">
  <Typography color="error">{error.message}</Typography>
  <Button onClick={retry} color="primary">
    Retry
  </Button>
</InfoCard>
```

### Empty States
Fälle behandeln, wenn keine Daten existieren:

```typescript
import { EmptyState } from '@backstage/core-components';

if (services.length === 0) {
  return (
    <EmptyState
      missing="data"
      title="No Services Found"
      description="This team doesn't have any registered services yet."
      action={
        <Button variant="contained" color="primary">
          Register a Service
        </Button>
      }
    />
  );
}
```

### Status-Cards Pattern
Ein häufiges Pattern für die Anzeige von Service- oder Deployment-Status:

```typescript
const StatusCard = ({ title, status, details }) => {
  const StatusComponent = {
    healthy: StatusOK,
    warning: StatusWarning,
    error: StatusError,
  }[status] || StatusPending;

  return (
    <InfoCard title={title}>
      <Box display="flex" alignItems="center" justifyContent="space-between">
        <StatusComponent>{status}</StatusComponent>
        <Typography variant="caption" color="textSecondary">
          {details}
        </Typography>
      </Box>
    </InfoCard>
  );
};

// Verwendung
<Grid container spacing={2}>
  <Grid item xs={12} sm={6} md={3}>
    <StatusCard title="API Gateway" status="healthy" details="99.9% uptime" />
  </Grid>
  <Grid item xs={12} sm={6} md={3}>
    <StatusCard title="Database" status="warning" details="High load" />
  </Grid>
</Grid>
```

## Styling-Ansätze

### makeStyles verwenden (Material-UI v4)
Für komponentenspezifische Styles:

```typescript
import { makeStyles } from '@material-ui/core/styles';

const useStyles = makeStyles(theme => ({
  card: {
    marginBottom: theme.spacing(2),
    '&:hover': {
      boxShadow: theme.shadows[4],
    },
  },
  statusHealthy: {
    color: theme.palette.success.main,
  },
  statusError: {
    color: theme.palette.error.main,
  },
}));

export const MyComponent = () => {
  const classes = useStyles();

  return (
    <Card className={classes.card}>
      <Typography className={classes.statusHealthy}>
        All systems operational
      </Typography>
    </Card>
  );
};
```

### Theme-Spacing
Theme-Spacing für konsistente Abstände verwenden:

```typescript
// theme.spacing(1) = 8px
// theme.spacing(2) = 16px
// theme.spacing(3) = 24px

const useStyles = makeStyles(theme => ({
  container: {
    padding: theme.spacing(3),      // 24px
    marginBottom: theme.spacing(2), // 16px
    gap: theme.spacing(1),          // 8px
  },
}));
```

## Diese Patterns anwenden
Im kommenden Lab werden Sie diese Patterns verwenden, um ein Team-Dashboard-Plugin zu erstellen:

*   Page-Struktur mit Header und Content
*   Grid-Layout für responsive Card-Anordnung
*   InfoCard-Container für Metriken
*   Status-Indikatoren für Service-Health
*   Loading- und Error-States für API-Interaktionen
*   Chips für Tags und Status-Labels

Das Verständnis dieser Patterns jetzt hilft Ihnen, sich während der praktischen Arbeit auf die Implementierungslogik statt auf die UI-Struktur zu konzentrieren.

## Kernpunkte

*   Backstage verwendet Material-UI v4 mit Imports von `@material-ui/core` für generische Komponenten und `@backstage/core-components` für Backstage-spezifische Wrapper
*   Jede Plugin-Seite folgt der Page > Header > Content-Struktur, mit InfoCard als primärem Container für gruppierte Informationen
*   Material-UI Grid verwendet ein 12-Spalten-responsives System mit Breakpoints (xs, sm, md, lg, xl) für adaptive Layouts
*   Backstage bietet semantische Status-Komponenten (StatusOK, StatusWarning, StatusError) für konsistentes Styling
*   Benutzerdefinierte Themes werden mit `createTheme()` erstellt, das lightTheme oder darkTheme-Palette erweitert, dann in `App.tsx` mit `UnifiedThemeProvider` registriert
*   Implementieren Sie immer Loading-States mit Progress-Komponente, Fehlerbehandlung mit ResponseErrorPanel und Empty-States mit EmptyState für professionelle UX

## Zusätzliche Ressourcen
*   [Backstage Core Components Storybook](https://backstage.io/storybook/)
*   [Material-UI v4 Documentation](https://v4.mui.com/)
*   [Backstage Theming Guide](https://backstage.io/docs/theming/overview/)
*   [Material-UI Grid System](https://v4.mui.com/components/grid/)

## Lektionsnotizen (Privat)
Nehmen Sie private Notizen während Sie lernen...

## Studiengruppe
Diesem Kurs ist noch keine Studiengruppe zugeordnet.

Bitten Sie Ihren Instruktor, eine Studiengruppe zu verlinken, um Diskussionen zu ermöglichen.
```
















