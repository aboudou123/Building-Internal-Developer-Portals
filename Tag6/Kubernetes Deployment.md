```markdown
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

## Lektionsnotizen (Privat)
Nehmen Sie private Notizen während Sie lernen...

## Studiengruppe
Diesem Kurs ist noch keine Studiengruppe zugeordnet.

Bitten Sie Ihren Instruktor, eine Studiengruppe zu verlinken, um Diskussionen zu ermöglichen.
```