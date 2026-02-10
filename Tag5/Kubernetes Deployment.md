
# Kubernetes-Grundlagen und Image-Building

Der Wechsel von Docker Compose zu Kubernetes bietet produktionsreife Skalierbarkeit, Zuverlässigkeit und Betriebsfähigkeiten für Ihre Backstage-Bereitstellung. Diese Lektion führt in die Vorteile von Kubernetes ein und lehrt Sie, wie Sie benutzerdefinierte Docker-Images gemäß dem offiziellen Backstage-Leitfaden erstellen.

## Warum Kubernetes für Backstage?

### Produktionsreife Vorteile
*   **Skalierbarkeit:** Horizontales Pod-Scaling basierend auf Nachfrage
*   **Hohe Verfügbarkeit:** Automatische Pod-Neustarts und Rescheduling
*   **Lastverteilung:** Eingebaute Service-Discovery und Lastverteilung
*   **Ressourcenmanagement:** Fein abgestufte CPU- und Speicherkontrollen
*   **Rolling Updates:** Zero-Downtime-Bereitstellungen und Rollbacks
*   **Selbstheilung:** Automatische Wiederherstellung bei Node-Ausfällen

## Kubernetes vs Docker Compose
Sowohl Kubernetes als auch Docker Compose verwenden deklarative Konfiguration, aber Kubernetes bietet produktionsreife Orchestrierungsfähigkeiten:

**Docker Compose (Entwicklung)**
```yaml
services:
  backstage:
    image: backstage:latest
    ports:
      - "7007:7007"
    environment:
      - POSTGRES_HOST=postgres
```

**Kubernetes Deployment (Produktion)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backstage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backstage
  template:
    spec:
      containers:
      - name: backstage
        image: backstage:1.0.0
        ports:
        - containerPort: 7007
        env:
        - name: POSTGRES_HOST
          value: postgres
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: POSTGRES_PASSWORD
```

## Backstage Docker Images erstellen

### Host-Build-Methode (Rückblick aus Abschnitt 3)
Diese Herangehensweise haben Sie bereits in Abschnitt 3 (Docker Deployment & PostgreSQL) gelernt. Hier eine kurze Wiederholung derselben Host-Build-Methode:

1. **Abhängigkeiten installieren:**
   ```bash
   yarn install --frozen-lockfile
   ```

2. **TypeScript-Definitionen erstellen:**
   ```bash
   yarn tsc
   ```

3. **Backend-Anwendung bauen:**
   ```bash
   yarn build:backend
   ```
   Dies erzeugt `skeleton.tar.gz` (Paketstruktur) und `bundle.tar.gz` (kompilierter Code), die im Dockerfile extrahiert werden.

4. **In Docker verpacken:**
   ```bash
   docker image build . -f packages/backend/Dockerfile --tag backstage:1.0.0
   ```

### Warum Host-Build? (Erinnerung)
*   **Schnellere Builds:** Besseres Dependency-Caching auf dem Host
*   **Bessere Developer Experience:** Direkte Sichtbarkeit des Build-Outputs
*   **Kleinere Images:** Finales Image enthält nur Runtime-Abhängigkeiten
*   **Offizielle Empfehlung:** Backstage-Maintainer empfehlen diesen Ansatz

### Images in Minikube laden (nur Lab-Umgebung)
In unserem Lab verwenden wir Minikube für lokale Kubernetes-Entwicklung. Laden Sie Ihr lokal erstelltes Image mit:
```bash
minikube image load backstage:1.0.0
```

**Wichtiger Produktionskontext:**
*   **Minikube:** Wird nur für lokale Entwicklung und Labs verwendet
*   **Produktions-Kubernetes:** Verwenden Sie managed Distributions (EKS, GKE, AKS) oder selbst gehostete Cluster (kubeadm, k3s)
*   **Container Registry:** Produktionsumgebungen benötigen eine ordentliche Container-Registry (Docker Hub, ECR, GCR, ACR, GHCR) statt Images direkt zu laden

## Kubernetes-Ressourcen verstehen

### Secrets vs Umgebungsvariablen
*   **Secrets:** Speichern sensible Daten base64-kodiert
*   **Umgebungsvariablen:** Referenzieren Secrets mit `valueFrom/secretKeyRef`
*   **Best Practice:** Niemals Credentials in Manifests hartkodieren

### Storage Classes
*   **PersistentVolume (PV):** Clusterweite Speicherressource
*   **PersistentVolumeClaim (PVC):** Speicheranfrage durch einen Pod
*   **Storage Class:** Definiert Speichertypen (manual, SSD, Cloud-Volumes)
*   **Reclaim Policy:** Daten behalten oder löschen, wenn PVC gelöscht wird

### Service Discovery
Kubernetes bietet automatisches DNS für Service-Discovery. Pods verbinden sich mit Service-Namen (z.B. `postgres`) innerhalb desselben Namespace oder mit vollqualifizierten DNS-Namen (z.B. `postgres.backstage.svc.cluster.local`).

## Nächste Schritte
Das Verständnis der Kubernetes-Grundlagen und des Image-Build-Prozesses bereitet Sie auf die Bereitstellung vor:

✅ Verstehen, warum Kubernetes für Backstage-Produktionsbereitstellungen essentiell ist  
✅ Kubernetes- und Docker-Compose-deklarative Konfigurationen vergleichen  
✅ Benutzerdefinierte Docker-Images mit der Host-Build-Methode erstellen  
✅ Images in Minikube für lokale Entwicklung laden  
✅ Kubernetes-Ressourcen verstehen (Secrets, PersistentVolumes, Service Discovery)  

In der nächsten Lektion lernen Sie, wie Sie PostgreSQL und Backstage auf Kubernetes mit Raw Manifests bereitstellen.

## Kernpunkte
*   Kubernetes bietet produktionsreife Skalierbarkeit, hohe Verfügbarkeit und Selbstheilungsfähigkeiten
*   Sowohl Kubernetes als auch Docker Compose verwenden deklarative Konfiguration, aber Kubernetes bietet erweiterte Orchestrierung
*   Die Host-Build-Methode (`yarn install`, `tsc`, `build:backend`) wird für das Erstellen von Backstage Docker Images empfohlen
*   Minikube wird für Lab-Umgebungen verwendet; die Produktion benötigt managed Kubernetes und Container-Registries
*   Kubernetes Secrets speichern sensible Daten base64-kodiert und injizieren Werte über Umgebungsvariablen
*   PersistentVolumes bieten Speicher, der Pod-Neustarts überlebt, wobei PersistentVolumeClaims Speicher anfordern
*   Service Discovery verwendet automatisches DNS, sodass sich Pods mit Service-Namen verbinden können

## Zusätzliche Ressourcen
*   [Backstage Kubernetes Deployment](https://backstage.io/docs/deployment/kubernetes/)
*   [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
*   [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)



===============================================

# Bereitstellung auf Kubernetes mit Manifests

===============================================


Diese Lektion folgt dem offiziellen Backstage Kubernetes-Bereitstellungsleitfaden unter Verwendung von Raw Manifests. Sie stellen PostgreSQL mit persistentem Speicher bereit, deployen Backstage mit korrekter Konfiguration und lernen, wie Sie auf Ihre Bereitstellung zugreifen und Probleme beheben.

## PostgreSQL mit Kubernetes Manifests bereitstellen

### Secrets für Datenbankzugangsdaten
Sichere Speicherung sensibler Datenbankzugangsdaten mit Kubernetes Secrets:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secrets
  namespace: backstage
type: Opaque
data:
  POSTGRES_USER: YmFja3N0YWdl      # base64: backstage
  POSTGRES_PASSWORD: aHVudGVyMg==  # base64: hunter2
```

### Persistenter Speicher für Daten
PostgreSQL benötigt persistenten Speicher, der Pod-Neustarts überlebt:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-storage
spec:
  storageClassName: manual
  capacity:
    storage: 2G
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: '/mnt/data'
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-storage-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2G
```

### PostgreSQL Deployment
Das Deployment zieht Zugangsdaten aus dem Secret und mountet persistenten Speicher:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: backstage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13.2-alpine
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 5432
        envFrom:
        - secretRef:
            name: postgres-secrets
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: postgresdb
      volumes:
      - name: postgresdb
        persistentVolumeClaim:
          claimName: postgres-storage-claim
```

### PostgreSQL Service
Erstellen Sie einen ClusterIP-Service für internen Datenbankzugriff:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: backstage
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
```

## Backstage mit Kubernetes Manifests bereitstellen

### Backstage Secrets
Speichern Sie API-Tokens und sensible Konfiguration:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: backstage-secrets
  namespace: backstage
type: Opaque
data:
  GITHUB_TOKEN: Z2hwX0VYQU1QTEU=  # base64 encoded token
```

### Backstage Deployment
Das Deployment referenziert das benutzerdefinierte Docker-Image und injiziert Umgebungsvariablen aus Secrets:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backstage
  namespace: backstage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backstage
  template:
    metadata:
      labels:
        app: backstage
    spec:
      containers:
      - name: backstage
        image: backstage:1.0.0
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 7007
        env:
        - name: POSTGRES_HOST
          value: postgres
        - name: POSTGRES_PORT
          value: '5432'
        - name: POSTGRES_USER
          value: backstage
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: POSTGRES_PASSWORD
        - name: GITHUB_TOKEN
          valueFrom:
            secretKeyRef:
              name: backstage-secrets
              key: GITHUB_TOKEN
```

### Backstage Service
Exponieren Sie Backstage auf Port 7007 für internen Cluster-Zugriff:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backstage
  namespace: backstage
spec:
  selector:
    app: backstage
  ports:
  - name: http
    port: 7007
    targetPort: http
```

## Auf Backstage mit Port Forwarding zugreifen

### Lab-Zugriffsmethode (Port Forwarding)
In unserer Lab-Umgebung verwenden wir `kubectl port-forward` für lokalen Zugriff:

```bash
kubectl port-forward --namespace=backstage --address=0.0.0.0 svc/backstage 7007:7007
```

Dies leitet den lokalen Port 7007 zum Backstage-Service weiter (http://localhost:7007) und läuft bis zum Stoppen mit Strg+C.

### Produktions-Zugriffsmethoden
Produktionsumgebungen benötigen ordentlichen externen Zugriff statt Port Forwarding:

*   **Ingress Controller:** Hostname-basiertes Routing mit TLS-Terminierung (empfohlen)
*   **LoadBalancer Service:** Cloud-Provider-Load-Balancer mit öffentlicher IP
*   **API Gateway:** Zentralisiertes API-Management mit Authentifizierung und Rate-Limiting

## Kubernetes-Bereitstellungen troubleshooten

### Pod-Status prüfen
Überwachen Sie den Pod-Lebenszyklus und Status:

```bash
kubectl get pods -n backstage
kubectl describe pod <pod-name> -n backstage
kubectl logs -n backstage deployment/backstage --tail=50
```

### Datenbankverbindung verifizieren
PostgreSQL-Konnektivität testen:

```bash
kubectl exec -it deployment/postgres -n backstage -- psql -U backstage -c "SELECT version();"
```

### Health Check Endpoints
Anwendungsgesundheit verifizieren:

```bash
curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" http://localhost:7007/healthcheck
```

Backend-API testen (erwartet 401 für nicht authentifizierte Anfragen):

```bash
curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" http://localhost:7007/api/catalog/entities
```

### Häufige Probleme
*   **ImagePullBackOff:** Image im Cluster nicht gefunden (`minikube image load` verwenden)
*   **CrashLoopBackOff:** Anwendung startet nicht (Logs prüfen)
*   **Pending Pods:** Ressourcenbeschränkungen oder Speicherprobleme (Pod beschreiben)
*   **Connection Refused:** Service nicht bereit oder falscher Port (Service-Endpunkte prüfen)

## Nächste Schritte
Das Verständnis der Kubernetes-Bereitstellung mit Manifests ermöglicht Ihnen:

✅ PostgreSQL mit Manifests unter Verwendung von Secrets und persistentem Speicher bereitzustellen  
✅ Backstage mit Umgebungsvariablen-Injektion aus Secrets zu deployen  
✅ Auf Anwendungen mit `kubectl port-forward` für die Entwicklung zuzugreifen  
✅ Bereitstellungsprobleme mit `kubectl`-Befehlen zu beheben  
✅ Health-Endpunkte und Datenbank-Konnektivität zu verifizieren  

Im kommenden Lab werden Sie:

*   PostgreSQL mit Secrets, persistentem Speicher, Deployment und Service bereitstellen
*   Ein benutzerdefiniertes Backstage Docker-Image mit der Host-Build-Methode erstellen
*   Das Image in Minikube laden und Backstage mit korrekter Konfiguration deployen
*   Auf Backstage mit Port-Forwarding zugreifen und die komplette Bereitstellung verifizieren

## Kernpunkte
*   Raw Kubernetes Manifests bieten vollständige Kontrolle über PostgreSQL- und Backstage-Bereitstellungskonfiguration
*   Kubernetes Secrets speichern Datenbankzugangsdaten und API-Tokens unter Verwendung von Base64-Kodierung
*   PersistentVolumes und PersistentVolumeClaims stellen sicher, dass PostgreSQL-Daten Pod-Neustarts überleben
*   Umgebungsvariablen injizieren Secret-Werte unter Verwendung von `valueFrom` und `secretKeyRef`-Patterns
*   ClusterIP-Services exponieren Anwendungen intern für Service-zu-Service-Kommunikation
*   `kubectl port-forward` wird in Labs verwendet; die Produktion verwendet Ingress-Controller, LoadBalancer oder API-Gateways
*   Troubleshooting verwendet `kubectl`-Befehle zum Prüfen von Pods, Logs und Verifizieren der Datenbank-Konnektivität

## Zusätzliche Ressourcen
*   [Backstage Kubernetes Deployment](https://backstage.io/docs/deployment/kubernetes/)
*   [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
*   [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)


===============================================

# Über das Lab
=============================================

Bringen Sie Backstage von der Docker-Entwicklung zur Produktions-Kubernetes-Bereitstellung, indem Sie der offiziellen Backstage-Dokumentation folgen. Stellen Sie PostgreSQL mit Kubernetes-Manifests bereit, erstellen Sie ein benutzerdefiniertes Backstage Docker-Image und deployen Sie es auf Kubernetes mit ordentlichem Secrets-Management und Service-Konfiguration.

**Sie werden lernen:**

*   Den Kubernetes-Cluster-Status und verfügbare Ressourcen zu verifizieren
*   PostgreSQL mit Kubernetes-Manifests bereitzustellen (Secrets, PV/PVC, Deployment, Service)
*   Benutzerdefinierte Backstage Docker-Images mit dem empfohlenen Host-Build-Ansatz zu erstellen
*   Backstage auf Kubernetes mit korrekter Umgebungsvariablen-Injektion bereitzustellen
*   Auf Backstage mit `kubectl port-forwarding` zuzugreifen
*   Die vollständige Integration zwischen Backstage und PostgreSQL zu verifizieren

Dieses Lab folgt dem offiziellen Backstage Kubernetes-Bereitstellungsleitfaden und lehrt Sie produktionsreife Bereitstellungspraktiken ohne Helm-Abstraktionen – und gibt Ihnen volle Kontrolle über jeden Aspekt der Bereitstellung.

## Wichtige Ressourcen

*   [Backstage Kubernetes Deployment](https://backstage.io/docs/deployment/kubernetes/)
*   [Backstage Docker Build](https://backstage.io/docs/deployment/docker/)
*   [Kubernetes Documentation](https://kubernetes.io/docs/home/)

## Was Sie lernen werden (4 Aufgaben)

### 1. PostgreSQL-Datenbank einrichten
Stellen Sie die PostgreSQL-Datenbank mit Kubernetes-Manifests bereit, inklusive korrekter Secrets, persistentem Speicher und Service-Konfiguration.

### 2. Backstage Docker Image erstellen
Erstellen Sie ein produktionsreifes Backstage Docker-Image mit dem empfohlenen Host-Build-Ansatz für optimale Performance.

### 3. Backstage auf Kubernetes deployen
Stellen Sie Backstage auf Kubernetes mit offiziellen Manifests, inklusive Secrets, Deployment und Service-Konfiguration, bereit.

### 4. Auf Backstage zugreifen und testen
Greifen Sie mit Port-Forwarding auf Backstage zu und verifizieren Sie, dass die komplette Bereitstellung korrekt funktioniert.


