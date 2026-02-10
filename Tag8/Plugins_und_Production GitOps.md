
# Grundlagen der Plugin-Entwicklung

Diese Lektion führt die Konzepte und Architekturmuster ein, die Sie im kommenden Lab zum Erstellen benutzerdefinierter Backstage-Plugins verwenden werden.

## ⚠️ Voraussetzungen für die Plugin-Entwicklung

Die Plugin-Entwicklung erfordert solide JavaScript/TypeScript- und React-Kenntnisse. Sie sollten vertraut sein mit:

- ES6+ JavaScript-Syntax (async/await, Destructuring, Arrow-Functions)
- TypeScript-Grundlagen (Typen, Interfaces, Generics)
- React-Fundamentals (Komponenten, Hooks wie useState und useEffect, Props)
- Node.js und Express.js für die Backend-Entwicklung

Wenn Sie mit diesen Technologien neu sind, sollten Sie zunächst Einführungskurse absolvieren. Das Lab könnte ohne diese Grundkenntnisse herausfordernd sein, aber es wird Sie dennoch durch den Prozess führen.

## Aufbauend auf bisherigem Wissen

Sie haben in diesem Kurs viele Backstage-Plugins verwendet – Catalog, Templates, TechDocs, Kubernetes-Plugin. Jetzt lernen Sie, wie Sie Ihre eigenen erstellen, um Backstage mit organisationsspezifischer Funktionalität zu erweitern.

## Warum benutzerdefinierte Plugins wichtig sind

### Erweiterung von Backstage für Ihre Organisation
Out-of-the-box Backstage ist leistungsstark, aber jede Organisation hat einzigartige Bedürfnisse. Benutzerdefinierte Plugins ermöglichen Ihnen:

- **Proprietäre Systeme integrieren** – Verbindung zu internem Monitoring, Deployment-Tools oder Datenbanken herstellen
- **Benutzerdefinierte Metriken anzeigen** – Team-Performance, Service-Health oder Business-KPIs zeigen
- **Workflows automatisieren** – Organisationsspezifische Automatisierung erstellen, die zu Ihren Prozessen passt
- **Branded Experiences kreieren** – An die Design-Sprache und Terminologie Ihres Unternehmens anpassen

**Das Platform-Engineering-Prinzip:** Generische Tools bringen Sie 80% voran. Benutzerdefinierte Plugins liefern die restlichen 20%, die Ihr Developer Portal wirklich zu Ihrem machen.

## Plugin-Architektur-Übersicht

### Alles ist ein Plugin
Wie Sie bereits erfahren haben, kommt Backstage-Funktionalität von Plugins:

- **Core-Features** – Catalog, Templates, TechDocs (Abschnitt 1-2)
- **Integrationen** – Kubernetes-Plugin (Abschnitt 6)
- **Benutzerdefinierte Erweiterungen** – Was Sie in diesem Abschnitt erstellen werden

### Drei Plugin-Typen
**Frontend-Plugins:**
- React-Komponenten und Seiten (Benutzeroberfläche)
- Client-seitiges State-Management
- Material-UI für konsistentes Design
- Beispiele: Dashboards, Entity-Seiten, benutzerdefinierte Tools

**Backend-Plugins:**
- REST-API-Endpunkte mit Express.js
- Business-Logic und Datenverarbeitung
- Externe Systemintegrationen
- Beispiele: Datenaggregation, Service-Orchestrierung, API-Proxies

**Full-Stack-Plugins:**
- Frontend und Backend arbeiten zusammen
- Frontend konsumiert Backend-APIs
- Komplette Features wie Team-Dashboards
- Beispiele: Metriken-Anzeige, Admin-Panels, benutzerdefinierte Workflows

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
- `useState` – Verwalten von Komponenten-State (Loading, Data, Errors)
- `useEffect` – Daten abrufen, wenn Komponente gemountet wird
- `useApi` – Auf Backstage-APIs zugreifen (Config, Catalog, Discovery)

**Material-UI-Komponenten:**
- `Page`, `Header`, `Content` – Layout-Struktur
- `InfoCard` – Card-basierte Content-Container
- `Grid` – Responsive Layouts
- Standard Material-UI-Komponenten (`Typography`, `Chip`, `List`)

**API-Integration:**
Frontend-Komponenten rufen Backend-APIs auf mit:
- `configApi.getString('backend.baseUrl')` – Backend-URL holen
- `fetch()` oder typisierte API-Clients – HTTP-Anfragen stellen
- Korrekte Fehlerbehandlung und Loading-States

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
- Datenaggregation und -transformation
- Integration mit externen APIs
- Business-Regeln und Berechnungen
- Catalog-Queries und -Verarbeitung

**Router Layer (HTTP-Endpunkte):**
- Express.js-Routendefinitionen
- Request-Validierung
- Fehlerbehandlung
- Delegiert Logik an Service-Layer

**Beispiel-Pattern (Sie implementieren die vollständige Version im Lab):**

```typescript
// Service layer
class TeamDataService {
  async getTeamMetrics(teamName: string) {
    // Query catalog, calculate metrics, return data
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

1. Importieren der Plugin-Page-Komponente
2. Hinzufügen einer Route in `packages/app/src/App.tsx`
3. Plugin erscheint in der Backstage-Navigation

### Backend-Plugins registrieren
Backend-Plugins registrieren sich mit dem neuen Backend-System:

1. Importieren des Backend-Plugin-Moduls
2. Hinzufügen zu `packages/backend/src/index.ts`
3. Backend entdeckt und mountet automatisch die Routen des Plugins

Das Lab führt Sie durch beide Integrationsschritte im Detail.

## Entwicklungs-Best Practices

### Design-Prinzipien
**Single Responsibility:**
- Jedes Plugin sollte eine


====================================

# Material-UI-Anpassung in Backstage
=====================================



Diese Lektion behandelt Material-UI-Muster und Theme-Anpassungen, die Sie beim Erstellen von Backstage-Plugins anwenden werden. Das Verständnis dieser Muster vor dem praktischen Lab hilft Ihnen, polierte, konsistente Schnittstellen zu erstellen.

## Material-UI im Backstage-Kontext

### Versionen
Backstage verwendet Material-UI v4 (jetzt MUI v4 genannt). Stellen Sie beim Durchsuchen von Dokumentationen oder Beispielen online sicher, dass Sie v4-Muster verwenden, nicht v5/MUI-Muster, die eine andere Syntax haben.

**Wichtige Unterschiede:**
- **Importpfade:** v4 verwendet `@material-ui/core`, v5 verwendet `@mui/material`
- **Styling-Ansatz:** v4 verwendet `makeStyles`, v5 verwendet `styled` oder `sx` prop
- **Theme-API:** Ähnliche Konzepte, aber unterschiedliche Implementierungsdetails

### Import-Muster
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

**Backstage Core Components** (Backstage-spezifische Wrapper und Muster):
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

**Best Practice:** Verwenden Sie Backstage-Komponenten, wenn verfügbar (sie behandeln Theming und Barrierefreiheit), und greifen Sie auf Material-UI für Lücken zurück.

## Wichtige Backstage-Komponenten

### Seitenlayout-Struktur
Jede Backstage-Seite folgt einer konsistenten Struktur:

```typescript
import { Page, Header, Content } from '@backstage/core-components';

export const MyPluginPage = () => (
  <Page themeId="tool">
    <Header title="Team-Dashboard" subtitle="Team-Metriken und Services anzeigen" />
    <Content>
      {/* Ihr Inhalt kommt hierher */}
    </Content>
  </Page>
);
```

**Komponentenrollen:**
- **Page** – Container, der Theme-Kontext und Navigation-Integration behandelt
- **Header** – Konsistenter Header mit Titel, Untertitel und optionalen Aktionsbuttons
- **Content** – Hauptinhaltbereich mit angemessenem Padding und responsivem Verhalten

Die `themeId`-Prop akzeptiert Werte wie `tool`, `service`, `website`, `library` oder `documentation`, um angemessenes Styling anzuwenden.

### InfoCard für Inhaltscontainer
InfoCard ist der primäre Container für gruppierte Informationen:

```typescript
import { InfoCard } from '@backstage/core-components';
import { Typography } from '@material-ui/core';

<InfoCard title="Service-Status" subheader="Letzte 24 Stunden">
  <Typography variant="body1">
    Alle 12 Services betriebsbereit
  </Typography>
</InfoCard>
```

**InfoCard-Props:**
- `title` – Erforderlicher Header-Text
- `subheader` – Optionaler Untertitel
- `action` – Optionaler Aktionsbutton im Header
- `deepLink` – Optionaler "Alle anzeigen"-Stil-Link
- `variant` – Karten-Styling-Variante (`gridItem`, `fullHeight`)

### Statusindikatoren
Backstage bietet semantische Statuskomponenten:

```typescript
import { StatusOK, StatusWarning, StatusError, StatusPending } from '@backstage/core-components';

// Verwenden für Service-Status, Deployment-Status, etc.
<StatusOK>Gesund</StatusOK>
<StatusWarning>Beeinträchtigt</StatusWarning>
<StatusError>Ausfall</StatusError>
<StatusPending>Wird bereitgestellt</StatusPending>
```

Diese wenden automatisch konsistente Farben und Symbole über Ihr Portal hinweg an.

## Responsive Grid-Layouts

### Das 12-Spalten-System
Material-UI Grid verwendet ein 12-Spalten-responsives Layout. Komponenten definieren, wie viele Spalten sie bei verschiedenen Breakpoints überspannen:

```typescript
import { Grid } from '@material-ui/core';
import { InfoCard } from '@backstage/core-components';

<Grid container spacing={3}>
  {/* Volle Breite auf kleinen Bildschirmen, halb auf mittleren, ein Drittel auf großen */}
  <Grid item xs={12} md={6} lg={4}>
    <InfoCard title="Teammitglieder">
      <Typography>8 Ingenieure</Typography>
    </InfoCard>
  </Grid>

  <Grid item xs={12} md={6} lg={4}>
    <InfoCard title="Aktive Services">
      <Typography>12 Services</Typography>
    </InfoCard>
  </Grid>

  <Grid item xs={12} md={6} lg={4}>
    <InfoCard title="Aktuelle Bereitstellungen">
      <Typography>23 diese Woche</Typography>
    </InfoCard>
  </Grid>
</Grid>
```

**Breakpoint-Referenz:**

| Breakpoint | Bildschirmgröße | Verwendung |
|------------|-----------------|------------|
| `xs` | 0px+ | Mobiltelefone |
| `sm` | 600px+ | Tablets im Hochformat |
| `md` | 960px+ | Tablets im Querformat, kleine Laptops |
| `lg` | 1280px+ | Desktop-Computer |
| `xl` | 1920px+ | Große Displays |

**Grid-Props:**
- `container` – Definiert einen Flex-Container
- `item` – Definiert ein Flex-Item
- `spacing` – Abstand zwischen Items (1 = 8px, 2 = 16px, 3 = 24px)
- `xs`/`sm`/`md`/`lg`/`xl` – Spaltenspanne bei jedem Breakpoint

### Häufige Layout-Muster
**Zweispaltiges Dashboard:**
```typescript
<Grid container spacing={3}>
  <Grid item xs={12} md={8}>
    {/* Hauptinhalt - 2/3 Breite auf Desktop */}
    <InfoCard title="Aktivitätsfeed">...</InfoCard>
  </Grid>
  <Grid item xs={12} md={4}>
    {/* Seitenleiste - 1/3 Breite auf Desktop */}
    <InfoCard title="Schnellstatistiken">...</InfoCard>
  </Grid>
</Grid>
```

**Gleichmäßige Kartenreihe:**
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

## Arbeiten mit Chips und Tags
Chips zeigen kompakte Informationen wie Status, Labels oder Tags an:

```typescript
import { Chip, Box } from '@material-ui/core';

// Einfacher Chip
<Chip label="Produktion" />

// Farbige Chips für Status
<Chip label="Läuft" style={{ backgroundColor: '#4caf50', color: 'white' }} />
<Chip label="Gestoppt" style={{ backgroundColor: '#f44336', color: 'white' }} />

// Chips mit Aktionen
<Chip label="Filter: Backend" onDelete={() => removeFilter('backend')} />

// Gruppe von Tags
<Box display="flex" gap={1} flexWrap="wrap">
  <Chip label="kubernetes" size="small" />
  <Chip label="node.js" size="small" />
  <Chip label="postgres" size="small" />
</Box>
```

**Chip-Props:**
- `label` – Textinhalt
- `color` – `default`, `primary`, `secondary`
- `size` – `small`, `medium`
- `variant` – `default`, `outlined`
- `onDelete` – Macht Chip mit X-Button löschbar

## Datentabellen

### Grundlegende Tabellenstruktur
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
      <TableCell align="right">Betriebszeit</TableCell>
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

### Backstage Tabellenkomponente
Für mehr Funktionen (Sortieren, Filtern, Paginierung) verwenden Sie Backstage's Table:

```typescript
import { Table, TableColumn } from '@backstage/core-components';

const columns: TableColumn[] = [
  { title: 'Service', field: 'name' },
  { title: 'Status', field: 'status', render: row => <StatusOK>{row.status}</StatusOK> },
  { title: 'Betriebszeit', field: 'uptime', type: 'numeric' }
];

<Table
  title="Service-Übersicht"
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
Backstage unterstützt mehrere Themes (hell, dunkel, benutzerdefiniert). Themes kontrollieren:
- Farbpalette (primär, sekundär, Hintergrund, Text)
- Typografie (Schriftarten, Größen, Gewichte)
- Navigation-Styling
- Komponenten-Erscheinung

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
      indicator: '#0052CC',   // Indikator für aktives Element
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
      main: '#4C9AFF',  // Helleres Blau für Dunkelmodus
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
Aktualisieren Sie Ihre App.tsx, um die benutzerdefinierten Themes zu verwenden:

```typescript
// packages/app/src/App.tsx
import { companyLightTheme, companyDarkTheme } from './themes/companyTheme';
import { UnifiedThemeProvider } from '@backstage/theme';

const app = createApp({
  // ... andere Konfiguration
  themes: [
    {
      id: 'company-light',
      title: 'Firma Hell',
      variant: 'light',
      Provider: ({ children }) => (
        <UnifiedThemeProvider theme={companyLightTheme} children={children} />
      ),
    },
    {
      id: 'company-dark',
      title: 'Firma Dunkel',
      variant: 'dark',
      Provider: ({ children }) => (
        <UnifiedThemeProvider theme={companyDarkTheme} children={children} />
      ),
    },
  ],
});
```

Benutzer können dann zwischen Themes mit Backstage's eingebautem Theme-Auswahl umschalten.

## Häufige UI-Muster

### Ladezustände
Zeigen Sie immer Lade-Feedback während asynchroner Operationen an:

```typescript
import { Progress } from '@backstage/core-components';
import { CircularProgress, Box } from '@material-ui/core';

// Vollständige Seitenladung (während des anfänglichen Datenabrufs)
if (loading) {
  return <Progress />;
}

// Inline-Ladung (für partielle Aktualisierungen)
<Box display="flex" alignItems="center" gap={1}>
  <CircularProgress size={20} />
  <Typography>Daten werden aktualisiert...</Typography>
</Box>
```

### Fehlerbehandlung
Zeigen Sie Fehler klar mit Wiederherstellungsoptionen an:

```typescript
import { ResponseErrorPanel } from '@backstage/core-components';

if (error) {
  return (
    <ResponseErrorPanel
      error={error}
      title="Fehler beim Laden der Teamdaten"
    />
  );
}

// Oder benutzerdefinierte Fehleranzeige
<InfoCard title="Fehler">
  <Typography color="error">{error.message}</Typography>
  <Button onClick={retry} color="primary">
    Erneut versuchen
  </Button>
</InfoCard>
```

### Leere Zustände
Behandeln Sie Fälle, wenn keine Daten vorhanden sind:

```typescript
import { EmptyState } from '@backstage/core-components';

if (services.length === 0) {
  return (
    <EmptyState
      missing="data"
      title="Keine Services gefunden"
      description="Dieses Team hat noch keine registrierten Services."
      action={
        <Button variant="contained" color="primary">
          Service registrieren
        </Button>
      }
    />
  );
}
```

### Statuskarten-Muster
Ein häufiges Muster zur Anzeige von Service- oder Bereitstellungsstatus:

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
    <StatusCard title="API-Gateway" status="healthy" details="99,9% Betriebszeit" />
  </Grid>
  <Grid item xs={12} sm={6} md={3}>
    <StatusCard title="Datenbank" status="warning" details="Hohe Auslastung" />
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
        Alle Systeme betriebsbereit
      </Typography>
    </Card>
  );
};
```

### Theme-Abstände
Verwenden Sie Theme-Abstände für konsistente Lücken:

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

## Anwendung dieser Muster
Im kommenden Lab verwenden Sie diese Muster zum Erstellen eines Team-Dashboard-Plugins:

- Seitenstruktur mit Header und Content
- Grid-Layout für responsive Kartenanordnung
- InfoCard-Container für Metriken
- Statusindikatoren für Service-Status
- Lade- und Fehlerzustände für API-Interaktionen
- Chips für Tags und Status-Labels

Das Verständnis dieser Muster jetzt hilft Ihnen, sich während der praktischen Arbeit auf die Implementierungslogik statt auf die UI-Struktur zu konzentrieren.

## Kernpunkte
- Backstage verwendet Material-UI v4 mit Imports von `@material-ui/core` für generische Komponenten und `@backstage/core-components` für Backstage-spezifische Wrapper
- Jede Plugin-Seite folgt der Page > Header > Content-Struktur, mit InfoCard als primärem Container für gruppierte Informationen
- Material-UI Grid verwendet ein 12-Spalten-responsives System mit Breakpoints (xs, sm, md, lg, xl) für adaptive Layouts
- Backstage bietet semantische Statuskomponenten (StatusOK, StatusWarning, StatusError), die konsistentes Styling über das Portal hinweg anwenden
- Benutzerdefinierte Themes werden mit `createTheme()` erstellt, das lightTheme oder darkTheme-Palette erweitert, dann in App.tsx mit `UnifiedThemeProvider` registriert
- Implementieren Sie immer Ladezustände mit Progress-Komponente, Fehlerbehandlung mit ResponseErrorPanel und leere Zustände mit EmptyState für professionelle UX

## Zusätzliche Ressourcen
- [Backstage Core Components Storybook](https://backstage.io/storybook/)
- [Material-UI v4 Dokumentation](https://v4.mui.com/)
- [Backstage Theming Guide](https://backstage.io/docs/theming/overview/)
- [Material-UI Grid System](https://v4.mui.com/components/grid/)




===================================================
# Produktions-GitOps und Platform-Engineering-Muster
====================================================


Diese Lektion stellt die Konzepte und Muster vor, die Sie im kommenden Produktions-Lab erleben werden: GitOps mit ArgoCD, automatisiertes Identitätsmanagement mit Keycloak und Golden-Path-Templates, die komplette Service-Ökosysteme erstellen.

## Aufbauend auf bisherigem Wissen
Sie haben Keycloak-Authentifizierung (Abschnitt 4), das Kubernetes-Plugin (Abschnitt 6) konfiguriert und grundlegende Templates erstellt (Abschnitt 2). Jetzt werden Sie sehen, wie diese sich in Produktions-Workflows integrieren.

## GitOps: Git als Single Source of Truth

### Kernprinzipien
GitOps verwendet Git als Single Source of Truth für deklarative Infrastruktur. Statt manueller Bereitstellungen oder imperativer Skripte committen Sie Konfigurationsänderungen in Git, und automatisierte Tools harmonisieren Ihren Cluster entsprechend.

**Drei Kernprinzipien:**
1. **Deklarative Konfiguration** – Beschreibe den gewünschten Zustand (3 Replikate, Port 8080), nicht wie man ihn erreicht
2. **Git als Single Source of Truth** – Alle Kubernetes-Manifests leben in Git-Repositories
3. **Automatisierte Reconciliation** – Tools vergleichen kontinuierlich Git (gewünscht) mit Cluster (aktuell) und beheben Abweichungen

**Platform-Engineering-Vorteile:**
- **Versionskontrolle** – Jede Bereitstellung mit Commit-Historie verfolgt
- **Nachvollziehbarkeit** – Kompletter Record, wer was wann deployed hat
- **Rollback** – Git-Commit rückgängig machen zum Rollback
- **Zusammenarbeit** – Pull-Request-Reviews für Infrastrukturänderungen
- **Disaster Recovery** – Kompletten Cluster von Git neu erstellen

## ArgoCD: Continuous Delivery für Kubernetes

### Wie ArgoCD funktioniert
ArgoCD läuft in Ihrem Kubernetes-Cluster und beobachtet Git-Repositories auf Kubernetes-Manifest-Änderungen. Wenn Sie Commits pushen, detektiert ArgoCD Änderungen und synchronisiert sie zum Cluster.

**Schlüsselkonzepte:**

**Application (CRD)** – Definiert, was zu deployen ist, woher es kommt, wohin es deployed wird:
- **Source** – Git-Repository, Branch und Verzeichnispfad
- **Destination** – Kubernetes-Cluster und Namespace
- **Sync Policy** – Manuell (Human Approval) oder automatisiert (sofortiges Deployment)
- **Sync Status** – Stimmt Cluster mit Git überein?
  - `Synced` – Cluster stimmt perfekt mit Git überein
  - `OutOfSync` – Git hat noch nicht angewendete Änderungen
- **Health Status** – Funktionieren deployede Ressourcen?
  - `Healthy` – Pods laufen, Services ready
  - `Progressing` – Deployment rollt aus
  - `Degraded` – Pods fehlgeschlagen, Fehler vorhanden

## Der GitOps-Workflow
Der komplette Ablauf, den Sie im Lab erleben werden:

1. Entwickler pusht Code zu GitHub Main-Branch
2. GitHub Actions führt Tests aus und erstellt Container-Image
3. GitHub Actions pusht Image zu GitHub Container Registry (GHCR)
4. GitHub Actions aktualisiert Kubernetes-Manifest mit neuem Image-Tag
5. ArgoCD detektiert Manifest-Änderung, markiert Application als `OutOfSync`
6. Manuelle Sync löst Deployment aus (oder automatisiert)
7. ArgoCD überwacht Health, bis Pods ready sind
8. Entwickler sieht Status in Backstage via ArgoCD-Plugin

Diese Trennung (GitHub Actions = CI, ArgoCD = CD) schafft klare Grenzen und ermöglicht GitOps-Muster.

## Produktions-Identitätsintegration

### Automatisiertes User- und Group-Management
Erinnerung aus Abschnitt 4: Sie haben Keycloak-Authentifizierung mit Sign-in-Resolver und Catalog-Synchronisation konfiguriert.

**Entwicklungs-Muster (was Sie getan haben):**
- Manuelles Erstellen von User- und Group-Entities in YAML
- Backstage-Konfiguration aktualisieren, wenn Personen hinzukommen/gehen

**Produktions-Muster (was das Lab demonstriert):**
1. IAM-Admin fügt Benutzer zu Keycloak hinzu (Corporate Identity Provider)
2. Keycloak-Catalog-Provider synchronisiert Benutzer/Gruppen automatisch zu Backstage
3. Entwickler loggen sich via OIDC ein (Keycloak handhabt Authentifizierung)
4. Sign-in-Resolver mappt Identität mit Gruppenmitgliedschaften
5. RBAC-Richtlinien verwenden Gruppen für Autorisierung
6. Templates weisen Ownership realen Teams zu

**Das Platform-Engineering-Prinzip:** Single source of truth für Organisationsstruktur. Backstage reflektiert die Realität in Echtzeit ohne manuelle Updates.

## Golden-Path-Templates in der Produktion

### Über grundlegende Templates hinaus
Erinnerung aus Abschnitt 2: Sie haben Templates mit `fetch:template`, `publish:github` und `catalog:register` Aktionen erstellt.

**Produktions-Templates erstellen komplette Ökosysteme, nicht nur Code:**

**Anwendungscode:**
- Komplette Service-Struktur (TypeScript/Python/Go)
- Tests und Qualitätschecks
- Dockerfile mit Multi-Stage-Builds

**Infrastruktur:**
- Kubernetes-Manifests (Deployment, Service, Namespace)
- ArgoCD-Application-Definition
- Resource-Limits und Health-Checks

**Automatisierung:**
- GitHub Actions CI/CD Workflows
- Container-Registry-Integration (GHCR)
- Automatisierte Manifest-Updates

**Integration:**
- `catalog-info.yaml` mit allen notwendigen Annotations
- TechDocs-Dokumentations-Skeleton
- Monitoring- und Observability-Hooks

**Developer-Experience:** Vom Ausfüllen eines Formulars zu produktions-deployedem Service in Minuten, mit allen Best-Practices eingebaut.

### Template-Annotations für Integration

Produktions-Templates erstellen `catalog-info.yaml` Dateien mit Annotations, die mehrere Systeme verbinden:
- `github.com/project-slug` – Verlinkt zu Source-Repository
- `argocd/app-name` – Verbindet mit ArgoCD-Deployment-Status
- `backstage.io/kubernetes-id` – Zeigt live Kubernetes-Workloads
- `backstage.io/kubernetes-namespace` – Beschränkt auf spezifischen Namespace
- `backstage.io/techdocs-ref` – Ermöglicht Dokumentation

Diese Annotations transformieren Backstage in eine vereinheitlichte Schnittstelle, die Code, Deployments und Runtime-Status zeigt.

## Backstage als Platform-Schnittstelle

### Vereinigte Developer-Experience
Wenn alle Integrationen zusammenarbeiten, erhalten Entwickler eine Single-Pane-of-Glass:

**Auf einer einzelnen Entity-Seite:**
- **Code** – Repository, Commits, Pull Requests (GitHub)
- **CI/CD** – Workflow-Runs und Build-Status (GitHub Actions)
- **Deployment** – Sync- und Health-Status (ArgoCD-Plugin)
- **Runtime** – Live-Pods, Logs, Events (Kubernetes-Plugin aus Abschnitt 6)
- **Dokumentation** – Inline-Docs (TechDocs aus Abschnitt 2)

**Platform-Engineering-Wert:**
- **Kein Kontext-Wechsel** – Alles von einem Ort aus zugänglich
- **Self-Service-Operationen** – Deployment-Status prüfen ohne Platform-Team
- **Reduzierte Cognitive Load** – Keine Notwendigkeit, mehrere Tool-UIs zu merken
- **Schnelleres Debugging** – Alle Informationen für Troubleshooting an einem Ort

## Enterprise-Architektur-Organisation

### Services im Maßstab strukturieren
Produktions-Portale organisieren Services hierarchisch:
- **Domain (Business-Bereich)** – E-Commerce, Payments, Platform
- **System (Sammlung von Services)** – Customer Portal, Payment Processing
- **Component (Microservice)** – `product-api`, `payment-api`, `frontend-app`
- **Resource (Infrastruktur)** – `product-database`, `cache`, `message-queue`

**Warum dies wichtig ist:**
- **Klare Ownership** – Jedes System owned von einem Team (synchronisiert von Keycloak-Gruppen)
- **Dependency-Tracking** – Verstehen, welche Services von welchen Datenbanken abhängen
- **Impact-Analyse** – Sehen, was kaputt geht, wenn eine Resource ausfällt
- **Team-Dashboards** – Catalog nach Domain oder System filtern

Diese Struktur ist nicht willkürlich – sie reflektiert, wie Ihre Organisation tatsächlich arbeitet.

## Produktionsbereitschafts-Prinzipien

### Sicherheit und Compliance
- **Authentifizierung** – OIDC mit Keycloak (Abschnitt 4)
- **Autorisierung** – RBAC via Gruppenmitgliedschaft (Abschnitt 4)
- **Secrets-Management** – Kubernetes-Secrets (Abschnitt 5)
- **Audit-Trails** – Git-Historie für Deployments, Keycloak für Benutzeraktionen

### Operational Excellence
- **Deklarative Infrastruktur** – Alles in Code definiert
- **Automatisierte Tests** – GitHub Actions validiert vor Deployment
- **Progressives Deployment** – ArgoCD managed Rollouts sicher
- **Disaster Recovery** – Git ermöglicht komplette Cluster-Wiederherstellung

### Developer Experience
- **Schnelles Onboarding** – Keycloak-SSO, Login am ersten Tag
- **Service-Erstellung** – Golden-Path-Templates, produktionsbereit in Minuten
- **Self-Service** – Deployen und überwachen ohne Platform-Team-Engpässe
- **Komplette Transparenz** – Backstage zeigt gesamten Service-Lifecycle

### Business-Impact messen
Platform-Engineering liefert messbaren Wert:
- **Developer Onboarding:** Wochen → Tage (80% Reduktion)
- **Service-Erstellung:** Tage → Minuten (95% Reduktion)
- **Dokumentations-Abdeckung:** 30% → 90%+ der Services
- **Service-Discovery:** 30 Minuten → Sofort
- **Platform-Support-Tickets:** 60% Reduktion

Diese Metriken rechtfertigen Platform-Engineering-Investition und demonstrieren kontinuierliche Verbesserung.

## Was Sie im Lab erleben werden
Das kommende Lab bringt alles zusammen:

1. **Produktions-Infrastruktur erkunden** – ArgoCD, Keycloak, Backstage vorkonfiguriert
2. **GitHub-Integration konfigurieren** – Template ermöglichen, Repositories zu erstellen
3. **Golden-Path-Template prüfen** – Komplette Produktions-Template-Struktur sehen
4. **Service via Template erstellen** – Developer-Self-Service erleben
5. **Kompletten Workflow überwachen** – CI-Build, ArgoCD-Deployment, Kubernetes-Run beobachten
6. **Backstage-Integrationen nutzen** – Deployment-Status, Live-Workloads, Logs ansehen

Sie werden sehen, wie Platform-Engineering-Muster nahtlose Developer-Experiences schaffen, während produktionsreife Zuverlässigkeit erhalten bleibt.

## Kernpunkte
- GitOps verwendet Git als Single Source of Truth mit deklarativer Konfiguration und automatisierter Reconciliation, ermöglicht Version Control, Auditability und Disaster Recovery
- ArgoCD Applications definieren Source (Git-Repo), Destination (Cluster/Namespace) und Sync Policy, reconciliert kontinuierlich gewünschten mit tatsächlichem Cluster-Zustand
- GitHub Actions handhabt CI (Testen, Bauen, Images pushen), während ArgoCD CD (Deployment auf Kubernetes) handhabt, schafft klare Trennung der Zuständigkeiten
- Produktionsumgebungen synchronisieren Benutzer und Gruppen automatisch von Keycloak, eliminieren manuelle YAML-Updates und schaffen Single Source of Truth
- Golden-Path-Templates erstellen komplette Ökosysteme: Anwendungscode, Kubernetes-Manifests, ArgoCD Applications, CI/CD Pipelines und Catalog-Integration
- Backstage-Integrationen (ArgoCD, Kubernetes, GitHub, TechDocs) bieten vereinigte Transparenz, eliminieren Kontext-Wechsel und ermöglichen Developer-Self-Service
- Enterprise-Architektur (Domains → Systems → Components → Resources) reflektiert Organisationsstruktur mit Team-Ownership von Keycloak-Gruppen

## Zusätzliche Ressourcen
- [ArgoCD Official Documentation](https://argo-cd.readthedocs.io/)
- [GitOps Principles - OpenGitOps](https://opengitops.dev/)
- [Backstage ArgoCD Plugin](https://github.com/backstage/backstage/tree/master/plugins/argocd)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

