
# Kubernetes-Plugin-Übersicht

Das Backstage Kubernetes-Plugin bietet Entwicklern Workload-Transparenz direkt im Developer Portal und eliminiert die Notwendigkeit, zwischen Tools wie `kubectl` und Dashboards zu wechseln.

## Das Developer-Experience-Problem

Ohne das Plugin müssen Entwickler zwischen Backstage für Service-Informationen und `kubectl` oder Dashboards für Kubernetes-Status hin- und herwechseln. Dies fragmentiert die Developer-Experience und verlangsamt die Fehlerbehebung.

Mit dem Plugin sind Workload-Status, Pod-Health und Logs direkt in Backstage sichtbar. Entwickler erhalten Self-Service-Debugging-Fähigkeiten ohne tiefgreifende Kubernetes-Kenntnisse.

## Entwickler-zentriertes Design

Das Plugin ist für Service-Eigentümer, nicht für Cluster-Administratoren, gebaut:

**Entwickler-Fragen:**
*   Läuft mein Service?
*   Gibt es Fehler in meinen Pods?
*   Was zeigen die Logs an?

**Nicht dafür entwickelt:**
*   Cluster-Kapazitätsplanung
*   Node-Management
*   Netzwerkrichtlinien-Konfiguration

## Zwei-Komponenten-Architektur

Das Plugin benötigt sowohl Frontend- als auch Backend-Komponenten:

**Frontend (@backstage/plugin-kubernetes):**
*   React-Komponenten, die Workload-Informationen anzeigen
*   In `EntityPage.tsx` als Tab integriert
*   Konfigurierbare Aktualisierungsintervalle für Live-Updates
*   Pod-Log-Viewer für die Fehlerbehebung

**Backend (@backstage/plugin-kubernetes-backend):**
*   Kommuniziert mit Kubernetes-API-Servern
*   Authentifiziert sich mit ServiceAccount-Tokens
*   Aggregiert Ressourcen über Cluster hinweg
*   Stellt REST-API für das Frontend bereit

**Wie das Resource-Matching funktioniert:**
1.  Catalog-Entity hat Annotation: `backstage.io/kubernetes-id: my-service`
2.  Kubernetes-Ressourcen haben passendes Label: `backstage.io/kubernetes-id=my-service`
3.  Backend sucht in Clustern nach Ressourcen mit diesem Label
4.  Alle passenden Ressourcen erscheinen in der Backstage-UI
5.  Frontend aktualisiert Daten automatisch in konfigurierten Intervallen

## Was das Plugin anzeigt

Das Plugin zeigt standardmäßige Kubernetes-Ressourcen:
*   **Workloads:** Deployments, ReplicaSets, Pods, StatefulSets, DaemonSets, Jobs, CronJobs
*   **Networking:** Services, Ingresses
*   **Configuration:** ConfigMaps, Secrets (nur Metadaten)
*   **Storage:** PersistentVolumeClaims

Das Plugin kann auch konfiguriert werden, um Custom Resources (CRDs) anzuzeigen, wie z.B. ArgoCD Applications, Crossplane-Ressourcen oder Cert-manager Certificates.

## UI-Funktionen

Der Kubernetes-Tab zeigt:

**Fehlerberichterstattung:**
*   Fehlgeschlagene Pods oben hervorgehoben
*   Fehlermeldungen und Neustartzähler
*   Visuelle Indikatoren für Pod-Gesundheit

**Cluster-Übersicht:**
*   Cluster-Name und Pod-Zählungen
*   Deployments nach Cluster gruppiert
*   Health-Status-Indikatoren

**Pod-Details-Tabelle:**
*   **NAME:** Pod-Identifikator
*   **PHASE:** Running, Pending oder Failed
*   **STATUS:** Health-Status (grünes Häkchen für gesund)
*   **CONTAINERS READY:** Readiness-Zähler (z.B. 1/1)
*   **TOTAL RESTARTS:** Neustart-Zähler
*   **CPU/MEMORY USAGE:** Ressourcenverbrauch (benötigt metrics-server)

**Pod-Details-Drawer:**
Klicken Sie auf einen Pod, um zu sehen:
*   Pod-IP-Adresse und YAML-Spezifikation
*   Container-Health-Checks (Gestartet, Ready, Neustart-Zähler)
*   **LOGS**-Button für Live-Log-Streaming

**Log-Viewer:**
*   Zeilennummerierte, durchsuchbare Logs
*   Echtzeit-Streaming
*   Kein `kubectl`-Zugriff erforderlich

## Warum Entwickler dies lieben

**Schnelle Fehlerbehebung:**
Anstatt zu `kubectl` zu wechseln, sehen Entwickler Pod-Status und Logs direkt in Backstage, was die Fehlerbehebungszeit von Minuten auf Sekunden reduziert.

**Self-Service-Debugging:**
Junior-Entwickler können Probleme ohne `kubectl`-Kenntnisse oder Cluster-Zugriff untersuchen. Die visuelle Oberfläche führt sie durch Pod-Health, Events und Logs.

**Multi-Umgebung-Transparenz:**
Wenn Services über Dev-, Staging- und Produktions-Cluster hinweg laufen, sehen Entwickler alle Umgebungen in einer Ansicht, was es einfach macht, Versionsunterschiede oder umgebungsspezifische Probleme zu erkennen.

## Was das Plugin NICHT ist

**Kein Deployment-Tool:**
Das Plugin ist standardmäßig read-only. Es zeigt Workloads an, löst aber keine Deployments aus oder ändert Ressourcen.

**Nicht für Cluster-Admins:**
Das Plugin konzentriert sich auf die Bedürfnisse von Service-Eigentümern (Pod-Status, Logs, Fehler) statt auf Cluster-Operationen (Node-Management, Kapazitätsplanung, Netzwerkrichtlinien).

**Sicherheitsmodell:**
Die meisten Bereitstellungen verwenden read-only ServiceAccounts nach dem Prinzip der geringsten Rechte. Dies verhindert versehentliche Änderungen bei gleichzeitiger Ermöglichung von Transparenz.

## Nächste Schritte

In der nächsten Lektion lernen Sie über Konfigurationskonzepte: Wie Backstage Cluster entdeckt, sich bei Kubernetes-APIs authentifiziert und Catalog-Entities mittels Annotations und Labels auf Ressourcen abbildet.

## Kernpunkte
*   Plugin eliminiert Kontextwechsel durch Anzeige von Kubernetes-Workloads direkt in Backstage
*   Zwei-Komponenten-Architektur: Frontend für UI-Anzeige, Backend für Kubernetes-API-Kommunikation
*   Resource-Matching verwendet Annotations auf Catalog-Entities und Labels auf Kubernetes-Ressourcen
*   UI zeigt Pod-Health, Deployment-Status, Container-Logs und Fehlerindikatoren
*   Pod-Logs mit einem Klick zugänglich, keine `kubectl`-Kenntnisse erforderlich
*   Standardmäßig read-only, folgt dem Prinzip der geringsten Rechte
*   Für Service-Eigentümer, nicht für Cluster-Administratoren, entwickelt
*   Ermöglicht Self-Service-Debugging für Entwickler aller Kenntnisstufen

## Zusätzliche Ressourcen
*   [Kubernetes Plugin Documentation](https://backstage.io/docs/features/kubernetes/overview/)
*   [Kubernetes Backend Plugin](https://github.com/backstage/backstage/tree/master/plugins/kubernetes-backend)
*   [Kubernetes Configuration](https://backstage.io/docs/features/kubernetes/configuration/)

## Lektionsnotizen (Privat)
Nehmen Sie private Notizen während Sie lernen...

## Studiengruppe
Diesem Kurs ist noch keine Studiengruppe zugeordnet.

Bitten Sie Ihren Instruktor, eine Studiengruppe zu verlinken, um Diskussionen zu ermöglichen.
```


```markdown

# Kubernetes-Plugin-Setup

Konfigurieren Sie das Backstage Kubernetes-Plugin, um Entwicklern Transparenz in ihre Kubernetes-Workloads direkt aus dem Developer Portal zu bieten.

**Intermediate | 135 min | Developer Experience / backstage**

**Tags:** backstage, kubernetes, kubernetes-plugin, developer-experience, workload-visibility, self-monitoring

## Voraussetzungen:
- backstage-kubernetes-deployment

## Über dieses Lab

Aktivieren Sie Self-Monitoring in Backstage, indem Sie das Kubernetes-Plugin konfigurieren, um Workloads anzuzeigen, die in Ihrem Cluster laufen. Dieses Lab lehrt Sie, wie Sie das Plugin einrichten, RBAC-Berechtigungen konfigurieren und Catalog-Entities auf Kubernetes-Ressourcen abbilden.

**Sie werden lernen:**

*   Die Kubernetes-Frontend- und Backend-Plugins zu installieren
*   Einen ServiceAccount mit Read-Only-RBAC-Berechtigungen zu erstellen
*   Cluster-Locators und Authentifizierungsstrategien zu konfigurieren
*   Entity-Annotations hinzuzufügen, um Catalog-Entities auf Kubernetes-Ressourcen abzubilden
*   Passende Labels zu Kubernetes-Deployments hinzuzufügen
*   Pods, Deployments und Services durch die Backstage-UI zu erkunden

Dieses Lab demonstriert die entwicklerzentrierte Transparenz, die das Kubernetes-Plugin bietet, und ermöglicht es Service-Eigentümern, ihre Workloads zu troubleshooten und zu überwachen, ohne Backstage zu verlassen.

## Wichtige Ressourcen

*   [Kubernetes Plugin Documentation](https://backstage.io/docs/features/kubernetes/overview/)
*   [Kubernetes Plugin Configuration](https://backstage.io/docs/features/kubernetes/configuration/)
*   [Kubernetes Authentication](https://backstage.io/docs/features/kubernetes/authentication/)

## Was Sie lernen werden (4 Aufgaben)

### 1. Kubernetes-Plugin installieren
Installieren Sie die Backstage Kubernetes-Frontend- und Backend-Plugins, um Workload-Transparenz zu ermöglichen.

### 2. Cluster-Verbindung und RBAC konfigurieren
Erstellen Sie einen ServiceAccount mit Read-Only-Berechtigungen und konfigurieren Sie Backstage für die Verbindung zum Kubernetes-Cluster.

### 3. Entity-Annotations und Labels hinzufügen
Aktualisieren Sie Catalog-Entities mit Kubernetes-Annotations und fügen Sie passende Labels zu Kubernetes-Deployments hinzu.

### 4. Kubernetes-Workloads in Backstage erkunden
Erkunden Sie Ihre Kubernetes-Workloads durch die Backstage-UI und verifizieren Sie, dass das Plugin korrekt funktioniert.
```