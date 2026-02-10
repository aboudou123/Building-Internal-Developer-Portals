
# Self-Hosted Backstage Setup-Übersicht

Willkommen zum Bonus-Abschnitt über selbst-gehostete Backstage-Bereitstellungen. In diesem Abschnitt lernen Sie, wie Sie die Produktions-GitOps-Lab-Umgebung auf Ihrem eigenen Laptop, Server oder Cloud-VM replizieren. Dieses Wissen ermöglicht es Ihnen, Ihre eigene produktionsreife Backstage-Bereitstellung außerhalb der TeKanAid-Lab-Plattform aufzubauen.

## Warum Backstage selbst hosten?
Während Backstage lokal während der Entwicklung laufen kann, erfordern Produktionsbereitstellungen:

- **Persistenten Speicher** für den Softwarekatalog und Benutzerdaten
- **SSO-Authentifizierung** für sicheren Team-Zugriff
- **GitOps-Deployments** für zuverlässige, nachvollziehbare Änderungen
- **Container-Orchestrierung** für Zuverlässigkeit und Skalierung

Self-Hosting gibt Ihnen vollständige Kontrolle über Ihr Developer Portal und ermöglicht Anpassungen, die den spezifischen Bedürfnissen Ihrer Organisation entsprechen.

## Produktionsarchitektur-Übersicht
Das selbst-gehostete Setup umfasst fünf Hauptkomponenten, die zusammenarbeiten:

### Kubernetes Cluster (Minikube)
Minikube bietet einen lokalen Kubernetes-Cluster, der als Orchestrierungsschicht dient. Alle Services laufen als Kubernetes-Deployments mit korrektem Ressourcen-Management, Health-Checks und Service-Discovery. Für Produktionsumgebungen würden Sie Minikube durch einen Managed-Kubernetes-Service wie EKS, GKE oder AKS ersetzen.

### PostgreSQL-Datenbank
PostgreSQL speichert alle persistenten Backstage-Daten, einschließlich:
- Softwarekatalog-Entities
- Benutzereinstellungen und Präferenzen
- Suchindex-Daten
- Scaffolder-Task-Historie

Die Datenbank verwendet Kubernetes PersistentVolumes, um sicherzustellen, dass Daten Pod-Neustarts überleben.

### Keycloak für SSO-Authentifizierung
Keycloak bietet Enterprise-fähiges Single Sign-On mit dem OIDC-Protokoll. Es handhabt:
- Benutzerauthentifizierung
- Gruppenmitgliedschafts-Management
- JWT-Token-Generierung mit benutzerdefinierten Claims
- Rollenbasierte Zugriffskontrolle-Integration

### ArgoCD für GitOps-Deployments
ArgoCD managed Deployments mit GitOps-Prinzipien. Anwendungskonfigurationen, die in Git gespeichert sind, werden zur Single Source of Truth, wobei ArgoCD automatisch den Cluster-Zustand mit Repository-Definitionen synchronisiert.

### Backstage mit Produktions-Plugins
Die Backstage-Instanz enthält vorinstallierte Plugins für:
- OIDC-Authentifizierung (Keycloak-Integration)
- Benutzer-/Gruppen-Synchronisation von Keycloak
- GitHub-Integration für Katalog-Discovery
- ArgoCD-Integration für Deployment-Transparenz

## Voraussetzungs-Checkliste
Bevor Sie mit dem selbst-gehosteten Setup beginnen, verifizieren Sie, dass Ihr System diese Anforderungen erfüllt:

| Anforderung | Minimum | Empfohlen |
| :--- | :--- | :--- |
| RAM | 6GB | 8GB+ |
| CPU-Kerne | 4 | 6+ |
| Festplattenspeicher | 30GB | 50GB |
| OS | macOS, Linux, WSL2 | Ubuntu 24.04+ |

## URL-Konfigurationsstrategie
Das Setup-Skript detektiert automatisch Ihre Umgebung und konfiguriert URLs entsprechend. Das Verständnis dieser Logik hilft bei der Anpassung für Ihre eigene Umgebung:

```bash
resolve_vm_ip() {
    local ip_from_file=""
    # Prüfe auf VM-IP-Datei (TeKanAid-Lab-Plattform)
    if [ -f /opt/tekanaid/vm_ips.txt ]; then
        ip_from_file=$(awk -F'=' '/backstage-production=/ {print $2}' /opt/tekanaid/vm_ips.txt | tr -d '[:space:]')
    fi

    if [ -n "$ip_from_file" ]; then
        echo "$ip_from_file"
    else
        # Fallback zur Hostname-Auflösung
        hostname -I | awk '{print $1}'
    fi
}
```

Für selbst-gehostete Setups verwenden Sie typischerweise `localhost` oder die IP-Adresse Ihres Computers. Die Proxy-URL-Logik ist spezifisch für die TeKanAid-Lab-Plattform und kann für persönliche Bereitstellungen ignoriert werden.

## Bereitstellungsansätze
Sie haben drei Optionen für die Bereitstellung der selbst-gehosteten Umgebung:

### Vollautomatisierung
Führen Sie das gesamte Setup-Skript von Anfang bis Ende aus. Dieser Ansatz dauert 30-40 Minuten und erfordert minimale Eingriffe. Am besten für das schnelle Einrichten einer kompletten Umgebung.

### Abschnittsweise
Führen Sie Skriptabschnitte manuell aus, während Sie jede Komponente studieren. Dieser Ansatz bietet die tiefste Lernerfahrung, während Sie die Auswirkung jedes Schritts auf das System beobachten.

### Hybrider Ansatz
Verwenden Sie das Skript für Infrastrukturkomponenten (Kubernetes, PostgreSQL, Keycloak, ArgoCD) und konfigurieren Sie Backstage manuell. Funktioniert gut, wenn Sie die Backstage-Konfiguration für Ihre spezifischen Bedürfnisse anpassen möchten.

## Umgebungsempfehlungen
Für die sauberste Erfahrung erwägen Sie, eine Cloud-VM statt Ihres lokalen Laptops zu verwenden:

- **AWS EC2:** t3.large oder größer mit Ubuntu 24.04
- **GCP Compute:** e2-standard-2 oder größer
- **Azure VM:** Standard_D2s_v3 oder größer
- **DigitalOcean Droplet:** 4GB oder größer

Cloud-VMs vermeiden Konflikte mit existierender Software auf Ihrem Laptop und bieten eine frische Umgebung für jeden Versuch.

## Pre-Flight-Check-Skript
Überprüfen Sie vor dem Start des vollständigen Setups, ob Ihre Voraussetzungen installiert sind:

```bash
#!/bin/bash
echo "Prüfe Voraussetzungen..."

command -v docker >/dev/null 2>&1 && echo "Docker installiert" || echo "FEHLEND: Docker"
command -v kubectl >/dev/null 2>&1 && echo "kubectl installiert" || echo "FEHLEND: kubectl"
command -v minikube >/dev/null 2>&1 && echo "Minikube installiert" || echo "FEHLEND: Minikube"
command -v node >/dev/null 2>&1 && echo "Node.js installiert ($(node --version))" || echo "FEHLEND: Node.js"
command -v yarn >/dev/null 2>&1 && echo "Yarn installiert" || echo "FEHLEND: Yarn"
command -v gh >/dev/null 2>&1 && echo "GitHub CLI installiert" || echo "FEHLEND: GitHub CLI"

# Prüfe Ressourcen
MEM=$(free -g | awk '/^Mem:/{print $2}')
echo "Verfügbarer RAM: ${MEM}GB (mindestens 6GB erforderlich)"
```

Dieses Skript identifiziert fehlende Tools, bevor Sie Zeit in die vollständige Bereitstellung investieren.

## Was Sie lernen werden
Dieser Abschnitt behandelt:

- **Voraussetzungen installieren** – Festgelegte Versionen aller erforderlichen Tools
- **Kubernetes und PostgreSQL** – Cluster-Setup und Datenbank-Bereitstellung
- **Keycloak-Konfiguration** – SSO-Setup via REST-API-Automatisierung
- **ArgoCD und Backstage** – GitOps-Bereitstellung des Developer Portals
- **Docker-Build-Prozess** – Verstehen der vorbereiteten Image-Struktur

Durch Abschluss dieses Abschnitts haben Sie das Wissen, eine produktionsreife Backstage-Instanz in jeder Umgebung, die Sie kontrollieren, bereitzustellen und zu warten.

## Kernpunkte
- Selbst-gehostetes Backstage erfordert fünf Hauptkomponenten: Kubernetes, PostgreSQL, Keycloak, ArgoCD und Backstage
- Mindestanforderungen umfassen 6GB RAM, 4 CPU-Kerne und 30GB Festplattenspeicher
- Drei Bereitstellungsansätze verfügbar: Vollautomatisierung, abschnittsweise oder hybrid
- Cloud-VMs bieten sauberere Umgebungen als lokale Laptops für Self-Hosting
- URL-Konfigurationslogik passt sich Lab-Plattform- oder persönlichen Bereitstellungsumgebungen an

## Zusätzliche Ressourcen
- [Backstage Deployment Documentation](https://backstage.io/docs/deployment/docker/)
- [Keycloak Getting Started Guide](https://www.keycloak.org/guides#getting-started)
- [ArgoCD Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)




==============================

# Voraussetzungen installieren

===============================


Diese Lektion behandelt die Installation aller Tools, die für das Self-Hosting von Backstage erforderlich sind. Das Verständnis, warum bestimmte Versionen festgelegt werden, hilft Ihnen dabei, eine reproduzierbare, stabile Umgebung zu erhalten.

## Warum Versionsfestlegung wichtig ist
Versionsfestlegung ist entscheidend für Reproduzierbarkeit. Ohne festgelegte Versionen:

- Verschiedene Installationen können sich unterschiedlich verhalten
- Breaking Changes in neuen Releases verursachen unerwartete Fehler
- Fehlerbehebung wird schwierig, wenn Versionen variieren

Das Setup-Skript legt jedes Tool auf eine getestete Version fest, um konsistentes Verhalten über alle Umgebungen hinweg sicherzustellen.

## Core-Tools Installation
Die folgenden Tools bilden das Fundament der Bereitstellung:

| Tool | Version | Zweck |
| :--- | :--- | :--- |
| Docker | 24.0+ | Container-Runtime |
| kubectl | v1.34.2 | Kubernetes CLI |
| Minikube | v1.37.0 | Lokaler K8s-Cluster |
| ArgoCD CLI | v2.13.2 | GitOps-Management |

### Docker Installation
Docker bietet die Container-Runtime für sowohl Minikube als auch das Erstellen von Images:

```bash
# Docker installation
curl -fsSL https://get.docker.com | sh
usermod -aG docker $USER
systemctl start docker
systemctl enable docker
```

Nach der Installation melden Sie sich ab und wieder an, damit die Gruppenmitgliedschaft wirksam wird, oder führen Sie `newgrp docker` aus.

### kubectl Installation
kubectl ist die primäre Schnittstelle für Kubernetes-Cluster-Management:

```bash
# kubectl installation with pinned version
KUBECTL_VERSION="v1.34.2"
curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/local/bin/
```

### Minikube Installation
Minikube erstellt einen lokalen Kubernetes-Cluster für Entwicklung und Tests:

```bash
# Minikube installation with pinned version
MINIKUBE_VERSION="v1.37.0"
curl -Lo minikube "https://storage.googleapis.com/minikube/releases/${MINIKUBE_VERSION}/minikube-linux-amd64"
chmod +x minikube
mv minikube /usr/local/bin/
```

### ArgoCD CLI Installation
Die ArgoCD CLI ermöglicht Kommandozeilen-Management von GitOps-Anwendungen:

```bash
# ArgoCD CLI installation
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

## Node.js-Umgebung
Node.js erfordert besondere Aufmerksamkeit aufgrund von Native-Modul-Kompatibilität.

### Warum speziell Version 22.15.0?
Backstage verwendet das `isolated-vm`-Paket für sichere Code-Ausführung im Scaffolder. Dieses Paket enthält vorgebaute Native-Binaries, die mit bestimmten Node.js-Versionen übereinstimmen müssen. Versionsinkonsistenzen verursachen Native-Kompilierungsfehler, die schwer zu diagnostizieren sind.

```bash
# Pinned Node.js installation
NODE_VERSION="22.15.0"

# Remove any existing Node.js installation
apt-get remove -y nodejs npm 2>/dev/null || true
rm -rf /usr/local/lib/node* /usr/local/include/node* /usr/local/bin/node* /usr/local/bin/npm*

# Clear node-gyp cache to prevent using old V8 headers
rm -rf /root/.cache/node-gyp /root/.node-gyp

# Download and install pinned version
cd /tmp
curl -fsSL "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz" -o node.tar.xz
tar -xf node.tar.xz
cp -r node-v${NODE_VERSION}-linux-x64/{bin,lib,share} /usr/local/
rm -rf node.tar.xz node-v${NODE_VERSION}-linux-x64

# Create symlinks
ln -sf /usr/local/bin/node /usr/bin/node
ln -sf /usr/local/bin/npm /usr/bin/npm
ln -sf /usr/local/bin/npx /usr/bin/npx
```

### Yarn 4.x via Corepack
Backstage erfordert Yarn 4.x für Workspace-Management. Corepack ist der offizielle Node.js-Mechanismus für die Verwaltung von Package-Managern:

```bash
# Enable Corepack for Yarn 4.x
corepack enable
corepack prepare yarn@4.4.1 --activate
```

Corepack stellt sicher, dass die korrekte Yarn-Version automatisch verwendet wird, wenn Sie ein Backstage-Projektverzeichnis betreten.

## Unterstützende Tools
Zusätzliche Tools verbessern die Produktivität während der Entwicklung und Fehlerbehebung:

| Tool | Version | Zweck |
| :--- | :--- | :--- |
| k9s | v0.50.16 | Interaktive K8s-UI |
| jq | Neueste | JSON-Verarbeitung |
| GitHub CLI | Neueste | GitHub-Operationen |
| kubectx/kubens | Neueste | Context-Switching |

### k9s Installation
k9s bietet eine Terminal-UI für Kubernetes, die allgemeine Operationen erheblich beschleunigt:

```bash
# k9s installation
K9S_VERSION="v0.50.16"
curl -LO "https://github.com/derailed/k9s/releases/download/${K9S_VERSION}/k9s_Linux_amd64.tar.gz"
tar -xzf k9s_Linux_amd64.tar.gz
mv k9s /usr/local/bin/
rm -f k9s_Linux_amd64.tar.gz LICENSE README.md
```

### GitHub CLI Installation
Die GitHub CLI vereinfacht Repository-Operationen und Authentifizierung:

```bash
# GitHub CLI installation
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null
apt-get update
apt-get install -y gh
```

### Build-Tools
Native-Modul-Kompilierung erfordert Build-Tools:

```bash
# Build tools for native module compilation
apt-get install -y build-essential python3 python3-pip
```

Diese Pakete sind für das Kompilieren nativer Node.js-Module wie `isolated-vm` erforderlich, wenn vorgebaute Binaries nicht für Ihre exakte Plattform verfügbar sind.

## Shell-Konfiguration
Das Setup-Skript konfiguriert nützliche Aliases, die den täglichen Workflow verbessern:

```bash
# Kubernetes aliases
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe'
alias kaf='kubectl apply -f'
alias kl='kubectl logs'

# Backstage-specific aliases
alias bs-restart='kubectl rollout restart deployment/backstage -n backstage'
alias bs-logs='kubectl logs -f deployment/backstage -n backstage'
alias bs-status='kubectl get all -n backstage'

# Minikube aliases
alias mk='minikube'
alias mkstart='minikube start'
alias mkstop='minikube stop'
alias mkstatus='minikube status'

# Docker aliases
alias d='docker'
alias dps='docker ps'
alias di='docker images'
```

Fügen Sie diese zu Ihrer `.bashrc` oder `.zshrc` Datei hinzu, um sie über Sessions hinweg zu erhalten.

### Bash-Completion
Aktivieren Sie Command-Completion für schnelleres Tippen:

```bash
# Generate completion scripts
kubectl completion bash > /etc/bash_completion.d/kubectl
minikube completion bash > /etc/bash_completion.d/minikube
gh completion -s bash > /etc/bash_completion.d/gh
argocd completion bash > /etc/bash_completion.d/argocd
```

Nach dem Hinzufügen von Completion-Skripts starten Sie eine neue Shell-Session oder sourcen Sie die Completion-Datei, um sie zu aktivieren.

## Verifizierung
Verifizieren Sie nach der Installation, dass alle Tools korrekt installiert sind:

```bash
# Verify installations
docker --version
kubectl version --client
minikube version
node --version
yarn --version
argocd version --client
k9s version
gh --version
```

Jeder Befehl sollte Versionsinformationen ohne Fehler zurückgeben. Wenn ein Tool fehlschlägt, überprüfen Sie die Installationsschritte für diese spezifische Komponente.

## Kernpunkte
- Versionsfestlegung sichert reproduzierbare, stabile Umgebungen über Installationen hinweg
- Node.js v22.15.0 ist speziell für `isolated-vm` vorgebaute Binary-Kompatibilität erforderlich
- Yarn 4.x wird via Corepack installiert, dem offiziellen Node.js-Package-Manager-Tool
- k9s bietet eine interaktive Terminal-UI, die Kubernetes-Operationen beschleunigt
- Shell-Aliases und Bash-Completion verbessern die tägliche Workflow-Effizienz erheblich

## Zusätzliche Ressourcen
- [Node.js Downloads](https://nodejs.org/en/download/)
- [Minikube Start Guide](https://minikube.sigs.k8s.io/docs/start/)
- [k9s Terminal UI](https://k9scli.io/)
- [GitHub CLI Manual](https://cli.github.com/manual/)





==================================
# Kubernetes und PostgreSQL Setup
===================================



Diese Lektion behandelt das Starten des Kubernetes-Clusters und die Bereitstellung von PostgreSQL als persistenten Speicher-Backend für Backstage. Das Verständnis dieser Komponenten hilft Ihnen, Bereitstellungen für verschiedene Umgebungen anzupassen.

## Minikube-Cluster-Konfiguration
Minikube erstellt einen Single-Node-Kubernetes-Cluster, der für Entwicklung und Tests geeignet ist. Die Konfiguration weist ausreichende Ressourcen für alle Komponenten zu:

```bash
# Start Minikube mit produktionsähnlichen Ressourcen
# Hinweis: --force hinzufügen, wenn als root-Benutzer ausgeführt (üblich auf VMs/Cloud-Instanzen)
minikube start \
    --driver=docker \
    --memory=6144 \
    --cpus=4 \
    --kubernetes-version=v1.30.0 \
    --force
```

### Konfigurationsoptionen erklärt
- **Driver-Option:** Spezifiziert Docker als Container-Runtime. Docker bietet bessere Kompatibilität über Plattformen hinweg im Vergleich zu VM-basierten Treibern.
- **Speicherzuweisung:** 6144MB (6GB) stellt sicher, dass PostgreSQL, Keycloak, ArgoCD und Backstage gleichzeitig ohne Speicherdruck laufen können.
- **CPU-Kerne:** Vier CPU-Kerne ermöglichen parallele Verarbeitung während Build-Operationen und unterstützen mehrere gleichzeitige Services.
- **Kubernetes-Version:** Auf v1.30.0 festgelegt für Konsistenz mit der getesteten Konfiguration.
- **--force Flag:** Erforderlich, wenn Minikube als root-Benutzer läuft, was auf Cloud-VMs und Lab-Umgebungen üblich ist. Ohne dieses Flag verweigert Minikube den Start mit dem Docker-Treiber.

## Namespaces erstellen
Kubernetes-Namespaces bieten logische Isolation zwischen Komponenten:

```bash
# Namespaces für Isolation erstellen
kubectl create namespace backstage
kubectl create namespace keycloak
kubectl create namespace argocd
```

**Trennung von Services in Namespaces:**
- Vereinfacht Ressourcenmanagement und Cleanup
- Ermöglicht unterschiedliche RBAC-Richtlinien pro Namespace
- Verhindert Namenskonflikte zwischen Komponenten
- Erleichtert Troubleshooting durch Scoping von Befehlen

## PostgreSQL Secrets
Kubernetes-Secrets speichern sensible Daten wie Datenbankzugangsdaten. Das Setup verwendet Opaque-Secrets mit base64-kodierten Werten:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secrets
  namespace: backstage
type: Opaque
data:
  POSTGRES_USER: YmFja3N0YWdl
  POSTGRES_PASSWORD: aHVudGVyMg==
```

Die base64-kodierten Werte dekodieren zu:
- `POSTGRES_USER`: backstage
- `POSTGRES_PASSWORD`: hunter2

Für Produktionsumgebungen verwenden Sie eine Secrets-Management-Lösung wie HashiCorp Vault oder external-secrets-operator, um Zugangsdaten zu injizieren, statt sie in Manifests zu speichern.

## Persistenter Speicher
Minikube verwendet hostPath-Volumes für persistenten Speicher. Dieser Ansatz funktioniert für lokale Entwicklung, erfordert aber verschiedene Storage-Klassen in der Produktion.

### hostPath-Verzeichnisse erstellen
Erstellen Sie zuerst die Speicherverzeichnisse auf dem Minikube-Node:

```bash
# hostPath-Verzeichnisse auf Minikube-Node erstellen
minikube ssh "sudo mkdir -p /mnt/data/postgres && sudo chmod 777 /mnt/data/postgres"
```

### PersistentVolume und PersistentVolumeClaim
Der PersistentVolume definiert den tatsächlichen Speicherort, während der PersistentVolumeClaim Speicher von verfügbaren Volumes anfordert:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-storage
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 2G
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: '/mnt/data/postgres'
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-storage-claim
  namespace: backstage
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2G
```

**Schlüsselkonfigurationspunkte:**
- `storageClassName: manual` – Verwendet eine manuell bereitgestellte Storage-Klasse statt dynamischer Bereitstellung.
- `accessModes: ReadWriteOnce` – Das Volume kann read-write von einem einzelnen Node gemountet werden, was für eine einzelne PostgreSQL-Instanz angemessen ist.
- `persistentVolumeReclaimPolicy: Retain` – Daten bleiben erhalten, wenn die PVC gelöscht wird, verhindert versehentlichen Datenverlust.

Ersetzen Sie für Cloud-Umgebungen hostPath durch Cloud-native Storage-Klassen wie gp3 (AWS EBS), pd-standard (GCP) oder managed-premium (Azure).

## PostgreSQL Deployment
Das Deployment managed den PostgreSQL-Pod-Lebenszyklus:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: backstage
  labels:
    backstage.io/kubernetes-id: backstage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
        backstage.io/kubernetes-id: backstage
    spec:
      containers:
        - name: postgres
          image: postgres:15-alpine
          imagePullPolicy: 'IfNotPresent'
          ports:
            - containerPort: 5432
          envFrom:
            - secretRef:
                name: postgres-secrets
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgresdb
              subPath: postgres
      volumes:
        - name: postgresdb
          persistentVolumeClaim:
            claimName: postgres-storage-claim
```

### Deployment-Konfigurationsdetails
- `image: postgres:15-alpine` – Verwendet PostgreSQL 15 mit Alpine-Linux-Basis für kleinere Image-Größe. PostgreSQL 15 bietet die Features, die Backstage benötigt.
- `envFrom` mit `secretRef` – Injiziert alle Keys aus dem postgres-secrets-Secret als Umgebungsvariablen. PostgreSQL verwendet POSTGRES_USER und POSTGRES_PASSWORD automatisch.
- `subPath: postgres` – Mountet in ein Unterverzeichnis des Volumes. Verhindert, dass PostgreSQL das lost+found-Verzeichnis sieht, das auf einigen Dateisystemen existiert.
- `backstage.io/kubernetes-id` Label – Ermöglicht dem Backstage-Kubernetes-Plugin, diese Ressourcen auf Entity-Seiten zu entdecken und anzuzeigen.

## PostgreSQL Service
Der Service exponiert PostgreSQL für andere Pods im Cluster:

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

Dieser ClusterIP-Service macht PostgreSQL unter `postgres.backstage.svc.cluster.local:5432` von jedem Pod im Cluster aus erreichbar.

## Bereitstellungsverifizierung
Verifizieren Sie nach dem Anwenden der Manifests, dass PostgreSQL korrekt läuft:

```bash
# Warten, bis PostgreSQL ready ist
kubectl wait --for=condition=available --timeout=120s deployment/postgres -n backstage

# Verifizieren, dass der Pod läuft
kubectl get pods -n backstage

# Prüfen, dass der Persistent Volume Claim gebunden ist
kubectl get pvc -n backstage
```

Der PVC-Status sollte `Bound` zeigen, was erfolgreichen Storage-Anschluss anzeigt.

### Datenbankkonnektivitätstest
Verifizieren Sie, dass die Datenbank Verbindungen akzeptiert:

```bash
# Datenbankkonnektivität testen
kubectl run psql-test --rm -it --image=postgres:15-alpine -n backstage -- \
    psql -h postgres -U backstage -d postgres -c "SELECT 1"
```

Wenn nach einem Passwort gefragt, geben Sie `hunter2` ein. Eine erfolgreiche Verbindung gibt eine einzelne Reihe mit dem Wert `1` zurück.

## Produktionsüberlegungen
Für Produktionsbereitstellungen erwägen Sie diese Verbesserungen:

- **Hohe Verfügbarkeit** – Verwenden Sie einen managed PostgreSQL-Service wie RDS, Cloud SQL oder Azure Database for PostgreSQL für automatisches Failover und Backups.
- **Connection-Pooling** – Fügen Sie PgBouncer vor PostgreSQL hinzu, um Verbindungslimits effizient zu verwalten.
- **Backup-Strategie** – Implementieren Sie regelmäßige Backups mit pg_dump oder kontinuierliches Archiving mit WAL-Shipping.
- **Ressourcenlimits** – Fügen Sie CPU- und Speicherlimits hinzu, um Ressourcenkonflikte zu verhindern:

  ```yaml
  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"
  ```

- **Network-Policies** – Beschränken Sie, welche Pods zu PostgreSQL verbinden können, mit Kubernetes NetworkPolicies.

## Kernpunkte
- Minikube benötigt 6GB Speicher und 4 CPUs, um alle Backstage-Komponenten gleichzeitig laufen zu lassen
- Kubernetes-Namespaces bieten logische Isolation und vereinfachen Ressourcenmanagement
- hostPath-PersistentVolumes funktionieren für Minikube, erfordern aber Cloud-Storage-Klassen in der Produktion
- Die `subPath`-Mount-Option verhindert Dateisystemkonflikte mit PostgreSQL-Datenverzeichnissen
- Labels wie `backstage.io/kubernetes-id` ermöglichen Backstage, Kubernetes-Ressourcen zu entdecken und anzuzeigen

## Zusätzliche Ressourcen
- [PostgreSQL Docker Image](https://hub.docker.com/_/postgres)
- [Kubernetes Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)



## 



==================================
# Keycloak SSO-Konfiguration
===================================


```markdown
# Keycloak SSO-Konfiguration

Diese Lektion behandelt die Bereitstellung von Keycloak und seine programmatische Konfiguration via REST-API. Das Verständnis dieser Automatisierung ermöglicht Ihnen, SSO-Konfigurationen zuverlässig zu reproduzieren und in CI/CD-Pipelines zu integrieren.

## Warum Keycloak für Backstage?
Keycloak bietet Enterprise-fähiges Identity-Management mit:
- OIDC- und SAML 2.0-Protokollunterstützung
- User-Federation mit LDAP und Active Directory
- Social-Login-Integration
- Fein abgestuften Autorisierungsrichtlinien
- Eingebauter Benutzerverwaltungskonsole

Für Backstage handhabt Keycloak Authentifizierung und stellt Gruppenmitgliedschafts-Claims bereit, die rollenbasierte Zugriffskontrolle antreiben.

## Keycloak-Bereitstellung
Die Keycloak-Bereitstellung enthält korrekte Health-Probes, um sicherzustellen, dass der Service bereit ist, bevor die Konfiguration beginnt:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
  namespace: keycloak
spec:
  replicas: 1
  selector:
    matchLabels:
      app: keycloak
  template:
    metadata:
      labels:
        app: keycloak
    spec:
      containers:
        - name: keycloak
          image: quay.io/keycloak/keycloak:24.0
          args:
            - start-dev
          env:
            - name: KEYCLOAK_ADMIN
              value: admin
            - name: KEYCLOAK_ADMIN_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: keycloak-secrets
                  key: KEYCLOAK_ADMIN_PASSWORD
            - name: KC_PROXY_HEADERS
              value: "xforwarded"
            - name: KC_HTTP_ENABLED
              value: "true"
            - name: KC_HOSTNAME_STRICT
              value: "false"
            - name: KC_HEALTH_ENABLED
              value: "true"
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 30
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 30
            timeoutSeconds: 10
            failureThreshold: 3
```

### Wichtige Konfigurationsoptionen
- **start-dev argument** – Führt Keycloak im Entwicklungsmodus ohne HTTPS-Anforderung aus. Für Produktion verwenden Sie `start` mit korrekter TLS-Konfiguration.
- **KC_PROXY_HEADERS: xforwarded** – Vertraut X-Forwarded-Headern von Reverse-Proxies, erforderlich wenn Keycloak hinter einem Load-Balancer oder Ingress-Controller sitzt.
- **KC_HOSTNAME_STRICT: false** – Erlaubt Zugriff von jedem Hostname, nützlich während der Entwicklung. Produktion sollte strikte Hostname-Validierung setzen.
- **KC_HEALTH_ENABLED: true** – Exponiert Health-Endpunkte unter `/health/ready` und `/health/live` für Kubernetes-Probes.

### Health Probes erklärt
Der `readinessProbe` prüft `/health/ready`, um zu bestimmen, wann Keycloak Traffic akzeptieren kann. Das hohe `failureThreshold` (30) berücksichtigt Keycloaks langsame Startzeit, besonders beim ersten Boot, wenn es die Datenbank initialisiert.

Der `livenessProbe` prüft `/health/live`, um zu erkennen, ob Keycloak nicht mehr reagiert und neu gestartet werden muss. Die längeren `initialDelaySeconds` (60) verhindern vorzeitige Neustarts während des Starts.

## REST-API-Authentifizierung
Die Keycloak-Konfiguration verwendet die Admin-REST-API. Holen Sie zuerst ein Admin-Access-Token:

```bash
# Port-forward für Zugriff auf Keycloak-API
kubectl port-forward svc/keycloak 8082:8080 -n keycloak &

# Warten, bis Port-forward etabliert ist
sleep 5

# Admin-Token von master-Realm holen
TOKEN=$(curl -s -X POST http://localhost:8082/realms/master/protocol/openid-connect/token \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d "client_id=admin-cli" \
    -d "username=admin" \
    -d "password=keycloak-admin-1234" \
    -d "grant_type=password" | jq -r '.access_token')
```

Der `admin-cli`-Client ist ein eingebauter Public-Client im master-Realm speziell für administrativen API-Zugriff. Tokens laufen nach 60 Sekunden standardmäßig ab, daher benötigen langlaufende Skripts möglicherweise Token-Aktualisierung.

## Backstage-Realm erstellen
Realms bieten vollständige Isolation für Benutzer, Clients und Einstellungen. Erstellen Sie einen dedizierten Realm für Backstage:

```bash
# Backstage-Realm erstellen
curl -s -X POST http://localhost:8082/admin/realms \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d '{
        "realm": "backstage",
        "enabled": true,
        "displayName": "Backstage Realm"
    }'
```

Alle nachfolgenden Konfigurationen geschehen innerhalb dieses Realms, halten Backstage getrennt vom master-Realm und anderen Anwendungen.

## OIDC-Client-Konfiguration
Der OIDC-Client definiert, wie Backstage Benutzer authentifiziert:

```bash
# OIDC-Client für Backstage erstellen
curl -s -X POST http://localhost:8082/admin/realms/backstage/clients \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d '{
        "clientId": "backstage",
        "enabled": true,
        "protocol": "openid-connect",
        "publicClient": false,
        "clientAuthenticatorType": "client-secret",
        "redirectUris": [
            "https://ihre-backstage-url/api/auth/oidc/handler/frame",
            "https://ihre-backstage-url/*",
            "http://localhost:7007/*",
            "http://localhost:3000/*"
        ],
        "webOrigins": [
            "https://ihre-backstage-url",
            "http://localhost:7007",
            "http://localhost:3000"
        ],
        "standardFlowEnabled": true,
        "directAccessGrantsEnabled": true,
        "serviceAccountsEnabled": true,
        "authorizationServicesEnabled": true
    }'
```

### Client-Konfigurationsdetails
- **publicClient: false** – Der Client benötigt ein Secret, bietet stärkere Sicherheit als Public-Clients.
- **redirectUris** – URLs, wohin Keycloak nach Authentifizierung umleiten kann. Muss sowohl Produktions- als auch Entwicklungs-URLs enthalten.
- **webOrigins** – Ursprünge erlaubt für CORS-Anfragen. Backstage macht API-Aufrufe zu Keycloak vom Browser aus.
- **standardFlowEnabled** – Aktiviert Authorization-Code-Flow, der von Webanwendungen verwendet wird.
- **serviceAccountsEnabled** – Erlaubt dem Client, sich als sich selbst (nicht als Benutzer) für Backend-Operationen wie Katalog-Synchronisation zu authentifizieren.

### Client-Secret abrufen
Nach Erstellung des Clients, holen Sie das automatisch generierte Secret:

```bash
# Client-UUID holen
CLIENT_UUID=$(curl -s http://localhost:8082/admin/realms/backstage/clients \
    -H "Authorization: Bearer $TOKEN" | jq -r '.[] | select(.clientId=="backstage") | .id')

# Client-Secret holen
CLIENT_SECRET=$(curl -s http://localhost:8082/admin/realms/backstage/clients/$CLIENT_UUID/client-secret \
    -H "Authorization: Bearer $TOKEN" | jq -r '.value')

echo "Client Secret: $CLIENT_SECRET"
```

Speichern Sie dieses Secret sicher – Backstage benötigt es für OIDC-Authentifizierung.

## Benutzer und Gruppen erstellen
Gruppieren Sie Benutzer und mappen Sie auf Backstage-Ownership und Berechtigungen.

### Gruppen erstellen
```bash
# Gruppen erstellen
for group in "platform-admins" "developers" "viewers"; do
    curl -s -X POST http://localhost:8082/admin/realms/backstage/groups \
        -H "Authorization: Bearer $TOKEN" \
        -H "Content-Type: application/json" \
        -d "{\"name\": \"$group\"}"
done
```

### Benutzer erstellen
```bash
# Benutzer mit Zugangsdaten erstellen
curl -s -X POST http://localhost:8082/admin/realms/backstage/users \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d '{
        "username": "alice.admin",
        "email": "alice.admin@techcorp.com",
        "firstName": "Alice",
        "lastName": "Admin",
        "enabled": true,
        "emailVerified": true,
        "credentials": [{"type": "password", "value": "password123", "temporary": false}]
    }'
```

Die `temporary: false`-Einstellung bedeutet, dass Benutzer nicht aufgefordert werden, ihr Passwort beim ersten Login zu ändern.

### Benutzer Gruppen zuweisen
```bash
# Benutzer-ID holen
USER_ID=$(curl -s http://localhost:8082/admin/realms/backstage/users?username=alice.admin \
    -H "Authorization: Bearer $TOKEN" | jq -r '.[0].id')

# Gruppen-ID holen
GROUP_ID=$(curl -s http://localhost:8082/admin/realms/backstage/groups \
    -H "Authorization: Bearer $TOKEN" | jq -r '.[] | select(.name=="platform-admins") | .id')

# Benutzer zu Gruppe hinzufügen
curl -s -X PUT http://localhost:8082/admin/realms/backstage/users/$USER_ID/groups/$GROUP_ID \
    -H "Authorization: Bearer $TOKEN"
```

## Group-Mapper-Konfiguration
Der Group-Mapper schließt Gruppenmitgliedschaft in JWT-Tokens ein, ermöglicht Backstage-RBAC:

```bash
# Group-Mapper hinzufügen, um Gruppen in JWT-Claims einzubinden
curl -s -X POST http://localhost:8082/admin/realms/backstage/clients/$CLIENT_UUID/protocol-mappers/models \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d '{
        "name": "groups",
        "protocol": "openid-connect",
        "protocolMapper": "oidc-group-membership-mapper",
        "config": {
            "full.path": "false",
            "id.token.claim": "true",
            "access.token.claim": "true",
            "claim.name": "groups",
            "userinfo.token.claim": "true"
        }
    }'
```

### Mapper-Konfigurationsoptionen
- **full.path: false** – Schließt nur Gruppennamen ein, nicht vollständige Pfade. Auf `true` setzen, wenn verschachtelte Gruppen verwendet werden.
- **id.token.claim: true** – Schließt Gruppen in das ID-Token ein, das vom Frontend verwendet wird.
- **access.token.claim: true** – Schließt Gruppen in das Access-Token ein, das von Backend-API-Aufrufen verwendet wird.
- **claim.name: groups** – Der JWT-Claim-Name, nach dem Backstage sucht, um Benutzergruppenmitgliedschaft zu bestimmen.

## Service-Account-Rollen
Damit Backstage Benutzer von Keycloak synchronisieren kann, benötigt der Service-Account `view-users`-Berechtigung:

```bash
# Service-Account-Benutzer holen
SERVICE_ACCOUNT_USER=$(curl -s http://localhost:8082/admin/realms/backstage/clients/$CLIENT_UUID/service-account-user \
    -H "Authorization: Bearer $TOKEN" | jq -r '.id')

# Realm-Management-Client-ID holen
REALM_MGMT_CLIENT=$(curl -s http://localhost:8082/admin/realms/backstage/clients \
    -H "Authorization: Bearer $TOKEN" | jq -r '.[] | select(.clientId=="realm-management") | .id')

# view-users-Rolle holen
VIEW_USERS_ROLE=$(curl -s http://localhost:8082/admin/realms/backstage/clients/$REALM_MGMT_CLIENT/roles \
    -H "Authorization: Bearer $TOKEN" | jq '.[] | select(.name=="view-users")')

# Rolle Service-Account zuweisen
curl -s -X POST http://localhost:8082/admin/realms/backstage/users/$SERVICE_ACCOUNT_USER/role-mappings/clients/$REALM_MGMT_CLIENT \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d "[$VIEW_USERS_ROLE]"


## Häufige Fehlerbehebung
| Problem | Ursache | Lösung |
| :--- | :--- | :--- |
| `Invalid redirect_uri` | URL-Mismatch | Exakte URL zu Client-`redirectUris` hinzufügen |
| `Invalid token` | Token abgelaufen | Neu authentifizieren, Tokens laufen in 60s ab |
| `Groups not in JWT` | Fehlender Mapper | Group-Membership-Mapper hinzufügen |
| `403 on admin API` | Falscher Realm | `/realms/master` für Admin-Token verwenden |
| `Connection refused` | Keycloak nicht bereit | Warten, bis readiness-Probe passt |

Beim Troubleshooting von OIDC-Flows verwenden Sie Browser-Entwicklertools, um die Redirect-URLs zu inspizieren und mit den konfigurierten `redirectUris` zu vergleichen. Selbst kleine Unterschiede wie Schrägstriche am Ende verursachen Fehler.

## Kernpunkte
- Keycloak-Health-Probes unter `/health/ready` und `/health/live` stellen sicher, dass der Service bereit ist, bevor die Konfiguration beginnt
- Der `admin-cli`-Client im master-Realm bietet API-Zugriff mit kurzlebigen Tokens
- OIDC-Client-Konfiguration muss alle Redirect-URIs für sowohl Produktion als auch Entwicklung enthalten
- Der Group-Mapper bettet Gruppenmitgliedschaft in JWT-Tokens für Backstage-RBAC ein
- Service-Account-Rollen ermöglichen Backstage, Benutzer von Keycloak zu synchronisieren

## Zusätzliche Ressourcen
- [Keycloak Admin REST API](https://www.keycloak.org/docs-api/24.0.3/rest-api/)
- [Keycloak OIDC Protocol](https://www.keycloak.org/docs/latest/securing_apps/#_oidc)
- [Backstage OIDC Authentication](https://backstage.io/docs/auth/oidc/provider/)




==================================

# Den Docker-Build-Prozess verstehen

===================================




Diese Lektion erklärt, wie das vorbereitete Backstage Docker-Image konstruiert wird. Das Verständnis des Build-Prozesses ermöglicht es Ihnen, das Image für die spezifischen Bedürfnisse Ihrer Organisation anzupassen.

## Warum Multi-Stage-Builds?
Backstage benötigt einen komplexen Build-Prozess, der beinhaltet:
- TypeScript-Kompilierung für Frontend und Backend
- Native-Modul-Kompilierung (isolated-vm)
- Plugin-Installation und Konfiguration
- Asset-Bundling und Optimierung

Multi-Stage-Builds trennen die Build-Umgebung von der Runtime-Umgebung, was zu kleineren, sichereren Produktions-Images führt.

## Build-Stage-Übersicht
Das Dockerfile verwendet zwei Stages:

1. **Builder Stage** – Vollständige Entwicklungsumgebung mit allen Build-Tools
2. **Production Stage** – Minimale Runtime mit nur notwendigen Dateien

## Builder Stage
Die Builder-Stage erstellt die Backstage-Anwendung und installiert alle Plugins:

```dockerfile
FROM node:22-bookworm AS builder
WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y \
    python3 python3-pip python-is-python3 \
    build-essential g++ make

# Create Backstage app from template (pinned version!)
RUN echo 'app' | npx @backstage/create-app@0.7.6 --skip-install

WORKDIR /app/app
```

### Wichtige Punkte
- **node:22-bookworm** – Verwendet Node.js 22 auf Debian Bookworm für breite Kompatibilität und Zugang zu vorgebauten Native-Modul-Binaries.
- **build-essential** – Erforderlich für das Kompilieren nativer Module wie `isolated-vm`, wenn vorgebaute Binaries nicht verfügbar sind.
- **@backstage/create-app@0.7.6** – Feste Version stellt reproduzierbare Builds sicher. Der `echo 'app'`-Pipe liefert den App-Namen non-interaktiv.
- **--skip-install** – Überspringt die initiale `yarn install`, da wir package.json vor der Installation modifizieren.

## Backend-Plugins hinzufügen
Plugins werden hinzugefügt, indem `package.json` vor dem Ausführen von `yarn install` modifiziert wird:

```dockerfile
# Add backend plugins using Python script
RUN python3 << 'EOF'
import json

# Read package.json
with open('packages/backend/package.json', 'r') as f:
    pkg = json.load(f)

# Add dependencies
pkg['dependencies']['@backstage/plugin-auth-backend-module-oidc-provider'] = '^0.4.9'
pkg['dependencies']['@backstage-community/plugin-catalog-backend-module-keycloak'] = '^3.1.1'
pkg['dependencies']['@backstage/plugin-catalog-backend-module-github'] = '^0.8.0'
pkg['dependencies']['@roadiehq/backstage-plugin-argo-cd-backend'] = '^4.0.0'

# Write back
with open('packages/backend/package.json', 'w') as f:
    json.dump(pkg, f, indent=2)
EOF
```

Dieser Ansatz:
- Modifiziert das generierte Template vor der Installation
- Verwendet Python für zuverlässige JSON-Manipulation
- Legt Plugin-Versionen fest für Reproduzierbarkeit

## Backend-Konfiguration
Die Backend-`index.ts`-Datei wird modifiziert, um alle Plugins zu registrieren:

```typescript
// packages/backend/src/index.ts
import { createBackend } from '@backstage/backend-defaults';
import { createBackendModule } from '@backstage/backend-plugin-api';
import { authProvidersExtensionPoint, createOAuthProviderFactory } from '@backstage/plugin-auth-node';
import { oidcAuthenticator } from '@backstage/plugin-auth-backend-module-oidc-provider';

const backend = createBackend();

// Register default plugins
backend.add(import('@backstage/plugin-app-backend'));
backend.add(import('@backstage/plugin-catalog-backend'));
backend.add(import('@backstage/plugin-scaffolder-backend'));
backend.add(import('@backstage/plugin-techdocs-backend'));
backend.add(import('@backstage/plugin-search-backend'));
backend.add(import('@backstage/plugin-kubernetes-backend'));

// Register GitHub catalog integration
backend.add(import('@backstage/plugin-catalog-backend-module-github'));

// Register Keycloak user/group sync
backend.add(import('@backstage-community/plugin-catalog-backend-module-keycloak'));

// Register ArgoCD backend
backend.add(import('@roadiehq/backstage-plugin-argo-cd-backend'));
```

## Benutzerdefiniertes OIDC-Authentifizierungsmodul
Die OIDC-Authentifizierung benötigt ein benutzerdefiniertes Modul, um Keycloak's Group-Claims zu handhaben:

```typescript
// Custom Keycloak auth module
const customOidcModule = createBackendModule({
  pluginId: 'auth',
  moduleId: 'custom-oidc',
  register(reg) {
    reg.registerInit({
      deps: { providers: authProvidersExtensionPoint },
      async init({ providers }) {
        providers.registerProvider({
          providerId: 'oidc',
          factory: createOAuthProviderFactory({
            authenticator: oidcAuthenticator,
            signInResolver: async ({ result }, ctx) => {
              // Extract groups from OIDC claims
              const groups = result.fullProfile.userinfo.groups || [];
              return ctx.signInWithCatalogUser({
                entityRef: { name: result.fullProfile.userinfo.preferred_username },
                annotations: { 'backstage.io/groups': groups.join(',') }
              });
            }
          })
        });
      }
    });
  }
});

// Register the custom module
backend.add(customOidcModule);
```

### Authentifizierungs-Flow
1. Benutzer klickt "Mit Keycloak anmelden"
2. Browser leitet zur Keycloak-Login-Seite weiter
3. Nach erfolgreichem Login leitet Keycloak mit einem Authorization-Code zurück
4. Backstage tauscht den Code gegen Tokens
5. Der `signInResolver` extrahiert Gruppen aus dem JWT und erstellt eine Backstage-Benutzer-Session

## Frontend-Konfiguration
Das Frontend benötigt OIDC-API-Konfiguration:

```typescript
// packages/app/src/apis.ts
import { OAuth2 } from '@backstage/core-app-api';
import { AnyApiFactory, createApiFactory, discoveryApiRef, oauthRequestApiRef } from '@backstage/core-plugin-api';

export const apis: AnyApiFactory[] = [
  createApiFactory({
    api: oidcAuthApiRef,
    deps: { discoveryApi: discoveryApiRef, oauthRequestApi: oauthRequestApiRef },
    factory: ({ discoveryApi, oauthRequestApi }) =>
      OAuth2.create({
        discoveryApi,
        oauthRequestApi,
        provider: { id: 'oidc', title: 'Keycloak', icon: () => null },
        defaultScopes: ['openid', 'profile', 'email', 'groups'],
      }),
  }),
];
```

Das `defaultScopes`-Array muss `groups` enthalten, um Gruppenmitgliedschaft in Tokens zu erhalten.

## Production Stage
Die Production-Stage erstellt ein minimales Runtime-Image:

```dockerfile
FROM node:22-bookworm-slim

WORKDIR /app

# Copy built app from builder
COPY --from=builder /app/app/packages/backend/dist ./packages/backend/dist
COPY --from=builder /app/app/packages/app/dist ./packages/app/dist
COPY --from=builder /app/app/node_modules ./node_modules
COPY --from=builder /app/app/package.json ./

# Install MkDocs for TechDocs
RUN apt-get update && apt-get install -y python3 python3-pip && \
    pip3 install mkdocs mkdocs-material mkdocs-techdocs-core

EXPOSE 7007
CMD ["yarn", "workspace", "backend", "start", "--config", "/app/app-config.yaml"]
```

### Produktions-Image-Optimierung
- **node:22-bookworm-slim** – Minimales Debian-Image ohne Entwicklungstools, reduziert Angriffsfläche und Image-Größe.
- **Selektive COPY** – Kopiert nur die gebauten `dist`-Verzeichnisse und `node_modules`, schließt Quelldateien und Build-Artefakte aus.
- **TechDocs-Abhängigkeiten** – MkDocs ist erforderlich für das Generieren von Dokumentation aus Markdown-Dateien.
- **Externe Konfiguration** – Der CMD referenziert `/app/app-config.yaml`, der zur Laufzeit von einem ConfigMap gemountet wird.

## Eigenes Image bauen
Um das Image für Ihre Organisation anzupassen:

```bash
# Repository mit dem Dockerfile klonen
git clone https://github.com/tekanaid/backstage-production-gitops
cd backstage-production-gitops/docker

# Dockerfile modifizieren, um Ihre Plugins hinzuzufügen
# Python-Skript-Abschnitt bearbeiten, um Ihre Abhängigkeiten hinzuzufügen

# Lokal bauen
docker build -t my-backstage:1.0.0 .

# Lokal testen
docker run -p 7007:7007 my-backstage:1.0.0

# Zu Ihrer Registry pushen
docker tag my-backstage:1.0.0 ghcr.io/myorg/backstage:1.0.0
docker push ghcr.io/myorg/backstage:1.0.0
```

### Benutzerdefinierte Plugins hinzufügen
Um ein neues Plugin zum Image hinzuzufügen:

1. **Backend-Abhängigkeit hinzufügen** – Python-Skript modifizieren, um das Paket zu `packages/backend/package.json` hinzuzufügen
2. **Backend-Modul registrieren** – Import und `backend.add()`-Aufruf zu `packages/backend/src/index.ts` hinzufügen
3. **Frontend-Plugin hinzufügen** (falls anwendbar) – `packages/app/src/App.tsx` modifizieren, um Routen und Komponenten des Plugins einzubinden
4. **app-config.yaml konfigurieren** – Erforderliche Konfiguration für das Plugin hinzufügen

**Beispiel für Hinzufügen des SonarQube-Plugins:**

```python
# Im Python-Skript
pkg['dependencies']['@backstage/plugin-sonarqube-backend'] = '^0.2.0'
```

```typescript
// In backend/src/index.ts
backend.add(import('@backstage/plugin-sonarqube-backend'));
```

### Build-Optimierungs-Tipps
- **Layer-Caching** – Dockerfile-Befehle von am wenigsten bis am häufigsten geändert anordnen. `yarn install` vor dem Kopieren von Quelldateien setzen.
- **Build-Arguments** – `ARG` für Versionsnummern verwenden, um einfache Updates zu ermöglichen:

  ```dockerfile
  ARG BACKSTAGE_VERSION=0.7.6
  RUN echo 'app' | npx @backstage/create-app@${BACKSTAGE_VERSION} --skip-install
  ```

- **Multi-Architecture** – Für sowohl `amd64` als auch `arm64` bauen, um diverse Bereitstellungsumgebungen zu unterstützen:

  ```bash
  docker buildx build --platform linux/amd64,linux/arm64 -t backstage:1.0.0 .
  ```

- **Security-Scanning** – Images vor der Bereitstellung scannen:

  ```bash
  docker scout quickview backstage:1.0.0
  trivy image backstage:1.0.0
  ```

## Kernpunkte
- Multi-Stage-Builds trennen Build-Tools von der Produktions-Runtime, reduzieren Image-Größe und Angriffsfläche
- Plugin-Abhängigkeiten werden hinzugefügt, indem `package.json` modifiziert wird, bevor `yarn install` läuft
- Benutzerdefinierte OIDC-Module extrahieren Keycloak-Group-Claims für Backstage-RBAC-Integration
- Das Produktions-Image verwendet `node:22-bookworm-slim` für minimalen Runtime-Footprint
- `app-config.yaml` wird zur Laufzeit via ConfigMap gemountet statt ins Image eingebacken

## Zusätzliche Ressourcen
- [Backstage Docker Dokumentation](https://backstage.io/docs/deployment/docker/)
- [Multi-Stage-Builds](https://docs.docker.com/build/building/multi-stage/)
- [Backstage Plugin Development](https://backstage.io/docs/plugins/)



==================================

# ArgoCD und Backstage Bereitstellung

===================================



Diese Lektion behandelt die Installation von ArgoCD für GitOps-basierte Bereitstellungen und die Bereitstellung von Backstage auf Kubernetes. Das Verständnis dieser Komponenten ermöglicht es Ihnen, das Developer Portal mit deklarativen Konfigurationen zu verwalten, die in Git gespeichert sind.

## ArgoCD-Installation
ArgoCD bietet GitOps-basierte Continuous Delivery für Kubernetes. Installation mit den offiziellen Manifests:

```bash
# ArgoCD-Namespace erstellen
kubectl create namespace argocd

# ArgoCD mit fester Version installieren
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.13.2/manifests/install.yaml

# Warten, bis ArgoCD-Komponenten bereit sind
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd
kubectl wait --for=condition=available --timeout=300s deployment/argocd-repo-server -n argocd
kubectl wait --for=condition=available --timeout=300s deployment/argocd-applicationset-controller -n argocd
```

Die Installation erstellt mehrere Komponenten:

- **argocd-server** – API-Server und Web-UI
- **argocd-repo-server** – Repository-Operationen (klonen, diff)
- **argocd-application-controller** – Überwacht Anwendungen und vergleicht Status
- **argocd-applicationset-controller** – Managed ApplicationSets für Templating

## ArgoCD-Konfiguration
Konfigurieren Sie ArgoCD für die Lab-Umgebung:

```bash
# TLS für Entwicklung deaktivieren (TLS-Terminierung auf Proxy/Ingress-Ebene)
kubectl patch configmap argocd-cmd-params-cm -n argocd -p '{"data":{"server.insecure":"true"}}' || \
    kubectl create configmap argocd-cmd-params-cm -n argocd --from-literal=server.insecure=true

# API-Key-Fähigkeit für Admin-Konto aktivieren
kubectl patch configmap argocd-cm -n argocd --type merge -p '{"data":{"accounts.admin":"apiKey"}}'

# ArgoCD-Server neu starten, um Konfigurationsänderungen anzuwenden
kubectl rollout restart deployment/argocd-server -n argocd
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd
```

### Konfiguration erklärt
- **server.insecure: true** – Deaktiviert TLS auf dem ArgoCD-Server. Erforderlich, wenn TLS-Terminierung an einem Reverse-Proxy oder Ingress-Controller stattfindet, um Redirect-Loops zu verhindern.
- **accounts.admin: apiKey** – Erlaubt dem Admin-Konto, API-Tokens zu generieren, die Backstage für die ArgoCD-Plugin-Integration benötigt.

## ArgoCD-Zugangsdaten abrufen
Holen Sie das automatisch generierte Admin-Passwort:

```bash
# ArgoCD-Admin-Passwort holen
ARGOCD_PASSWORD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
echo "ArgoCD Admin Password: $ARGOCD_PASSWORD"
```

## ArgoCD-Token für Backstage generieren
Backstage benötigt einen API-Token zur Kommunikation mit ArgoCD:

```bash
# ArgoCD-Session-Token mit REST-API generieren
ARGOCD_TOKEN=$(kubectl run argocd-token-generator --rm -i --restart=Never --image=curlimages/curl:latest -- \
  curl -sk -X POST http://argocd-server.argocd.svc.cluster.local/api/v1/session \
  -H "Content-Type: application/json" \
  -d "{\"username\":\"admin\",\"password\":\"$ARGOCD_PASSWORD\"}" | grep -o '"token":"[^"]*"' | cut -d'"' -f4)

echo "$ARGOCD_TOKEN" > /root/backstage-k8s/argocd-backstage-token.txt
```

Dieser Ansatz generiert den Token innerhalb des Clusters, vermeidet die Notwendigkeit von externem Port-Forwarding während des Setups.

## Backstage Docker Image bauen
Backstage von Source zu bauen dauert 5-7 Minuten aufgrund von TypeScript-Kompilierung und Native-Modul-Bau. Diese Investition ermöglicht es Ihnen, das Image mit den spezifischen Plugins und Konfigurationen Ihrer Organisation anzupassen.

Erstellen Sie zuerst ein Arbeitsverzeichnis und das Dockerfile:

```bash
mkdir -p ~/backstage-build
cd ~/backstage-build
```

Erstellen Sie das Multi-Stage-Dockerfile (wie in der vorherigen Lektion erklärt):

```dockerfile
FROM node:22-bookworm AS builder
WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y \
    python3 python3-pip python-is-python3 \
    build-essential g++ make

# Create Backstage app from template (pinned version!)
RUN echo 'app' | npx @backstage/create-app@0.7.6 --skip-install

WORKDIR /app/app

# Add backend plugins using Python script
RUN python3 << 'EOF'
import json
with open('packages/backend/package.json', 'r') as f:
    pkg = json.load(f)
pkg['dependencies']['@backstage/plugin-auth-backend-module-oidc-provider'] = '^0.4.9'
pkg['dependencies']['@backstage-community/plugin-catalog-backend-module-keycloak'] = '^3.1.1'
pkg['dependencies']['@backstage/plugin-catalog-backend-module-github'] = '^0.8.0'
pkg['dependencies']['@roadiehq/backstage-plugin-argo-cd-backend'] = '^4.0.0'
with open('packages/backend/package.json', 'w') as f:
    json.dump(pkg, f, indent=2)
EOF

# Install dependencies and build
RUN corepack enable && yarn install && yarn build:backend

# Production stage
FROM node:22-bookworm-slim
WORKDIR /app
COPY --from=builder /app/app/packages/backend/dist ./packages/backend/dist
COPY --from=builder /app/app/packages/app/dist ./packages/app/dist
COPY --from=builder /app/app/node_modules ./node_modules
COPY --from=builder /app/app/package.json ./

# Install MkDocs for TechDocs
RUN apt-get update && apt-get install -y python3 python3-pip && \
    pip3 install mkdocs mkdocs-material mkdocs-techdocs-core --break-system-packages

EXPOSE 7007
CMD ["node", "packages/backend", "--config", "/app/app-config.yaml"]
```

Bauen und laden Sie das Image in Minikube:

```bash
# Image lokal bauen (dauert 5-7 Minuten)
docker build -t backstage:1.0.0 .

# Image in Minikube laden
minikube image load backstage:1.0.0

# Image in Minikube verifizieren
minikube image ls | grep backstage
```

Das gebaute Image enthält diese Plugins:
- OIDC-Authentifizierungsmodul
- Keycloak-Benutzer/Gruppen-Synchronisation
- GitHub-Integration
- ArgoCD-Plugin
- Kubernetes-Plugin
- TechDocs

## Backstage-Secrets
Erstellen Sie ein Secret mit allen Zugangsdaten, die Backstage benötigt:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: backstage-secrets
  namespace: backstage
type: Opaque
stringData:
  POSTGRES_HOST: postgres
  POSTGRES_PORT: "5432"
  POSTGRES_USER: backstage
  POSTGRES_PASSWORD: hunter2
  KEYCLOAK_CLIENT_SECRET: your-keycloak-client-secret
  GITHUB_TOKEN: your-github-pat-token
  GITHUB_USERNAME: your-github-username
  ARGOCD_AUTH_TOKEN: your-argocd-token
```

Die Verwendung von `stringData` erlaubt Klartext-Werte, die Kubernetes automatisch base64-kodiert, wenn das Secret erstellt wird.

## Backstage-RBAC-Konfiguration
Backstage benötigt Cluster-weiten Lesezugriff, um Kubernetes-Ressourcen anzuzeigen:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: backstage
  namespace: backstage
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: backstage-read
rules:
  - apiGroups: ["*"]
    resources:
      - pods
      - pods/log
      - configmaps
      - services
      - deployments
      - replicasets
      - horizontalpodautoscalers
      - ingresses
      - statefulsets
      - limitranges
      - resourcequotas
      - persistentvolumeclaims
      - daemonsets
    verbs: ["get", "list", "watch"]
  - apiGroups: ["batch"]
    resources: ["jobs", "cronjobs"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods"]
    verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: backstage-read
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: backstage-read
subjects:
  - kind: ServiceAccount
    name: backstage
    namespace: backstage
```

Diese ClusterRole bietet Lesezugriff auf gängige Kubernetes-Ressourcen. Das Kubernetes-Plugin verwendet diesen Zugriff, um Pods, Deployments und andere Ressourcen auf Entity-Seiten anzuzeigen.

## Backstage-Deployment
Das Deployment führt Backstage mit allen erforderlichen Konfigurationen aus:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backstage
  namespace: backstage
  labels:
    backstage.io/kubernetes-id: backstage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backstage
  template:
    metadata:
      labels:
        app: backstage
        backstage.io/kubernetes-id: backstage
    spec:
      serviceAccountName: backstage
      containers:
        - name: backstage
          image: backstage:1.0.0
          imagePullPolicy: Never
          ports:
            - containerPort: 7007
          env:
            - name: NODE_OPTIONS
              value: "--no-node-snapshot"
          envFrom:
            - secretRef:
                name: backstage-secrets
          volumeMounts:
            - name: techdocs
              mountPath: /app/techdocs
            - name: backstage-config
              mountPath: /app/app-config.yaml
              subPath: app-config.yaml
      volumes:
        - name: techdocs
          emptyDir: {}
        - name: backstage-config
          configMap:
            name: backstage-config
```

### Deployment-Konfigurationsdetails
- **imagePullPolicy: Never** – Verwendet das direkt in Minikube geladene Image statt es aus einer Registry zu ziehen.
- **NODE_OPTIONS: --no-node-snapshot** – Deaktiviert Node.js-Start-Snapshots, die Probleme mit Native-Modules in containerisierten Umgebungen verursachen können.
- **envFrom mit secretRef** – Injiziert alle Keys von `backstage-secrets` als Umgebungsvariablen, entspricht dem, was Backstage von `app-config.yaml` erwartet.
- **ConfigMap-Mount** – Die `app-config.yaml`-Datei wird von einem ConfigMap gemountet, ermöglicht Konfigurationsänderungen ohne Neubau des Images.

## Backstage-Service
Exponieren Sie Backstage innerhalb des Clusters:

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
      targetPort: 7007
```

## ConfigMap erstellen
Die Backstage-Konfiguration lebt in einem ConfigMap:

```bash
# ConfigMap aus app-config.yaml erstellen
kubectl create configmap backstage-config \
  --from-file=app-config.yaml=/pfad/zu/ihrer/app-config.yaml \
  -n backstage \
  --dry-run=client -o yaml | kubectl apply -f -
```

Um Backstage-Konfiguration zu aktualisieren:

```bash
# ConfigMap aktualisieren
kubectl create configmap backstage-config \
  --from-file=app-config.yaml=/pfad/zu/aktualisierter/app-config.yaml \
  -n backstage \
  --dry-run=client -o yaml | kubectl apply -f -

# Backstage neu starten, um Änderungen zu übernehmen
kubectl rollout restart deployment/backstage -n backstage
```

## Port-Forwarding für Zugriff
Greifen Sie auf Backstage durch Port-Forwarding zu:

```bash
# Backstage-Port-Forward
kubectl port-forward svc/backstage 7007:7007 -n backstage &

# Keycloak-Port-Forward
kubectl port-forward svc/keycloak 8082:8080 -n keycloak &

# ArgoCD-Port-Forward
kubectl port-forward svc/argocd-server 8080:443 -n argocd &
```

Für persistentes Port-Forwarding erstellen Sie systemd-Services:

```ini
[Unit]
Description=Backstage Port Forward
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/kubectl port-forward -n backstage svc/backstage 7007:7007 --address=0.0.0.0
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

## Bereitstellungsverifizierung
Verifizieren Sie, dass alle Komponenten laufen:

```bash
# Warten, bis Backstage bereit ist
kubectl wait --for=condition=available --timeout=300s deployment/backstage -n backstage

# Alle Ressourcen im Backstage-Namespace prüfen
kubectl get all -n backstage

# Backstage-Logs anzeigen
kubectl logs -n backstage deployment/backstage --tail=20
```

## Zugriff auf die Services
Nach Abschluss der Bereitstellung:

| Service | URL | Zugangsdaten |
| :--- | :--- | :--- |
| Backstage | http://localhost:7007 | Keycloak-SSO |
| Keycloak | http://localhost:8082 | admin / keycloak-admin-1234 |
| ArgoCD | https://localhost:8080 | admin / (siehe Passwort-Datei) |

Melden Sie sich bei Backstage mit einem der vorkonfigurierten Keycloak-Benutzer an:

- alice.admin / password123 (`platform-admins` Gruppe)
- bob.developer / password123 (`developers` Gruppe)
- carol.viewer / password123 (`viewers` Gruppe)

## Kernpunkte
- ArgoCD-Installation erfordert Aktivierung der API-Key-Fähigkeit für Backstage-Integration
- Backstage von Source zu bauen dauert 5-7 Minuten, ermöglicht aber vollständige Anpassung der Plugins
- `imagePullPolicy: Never` verwendet direkt in Minikube geladene Images
- Backstage-RBAC erfordert ClusterRole für Cross-Namespace-Kubernetes-Ressourcensichtbarkeit
- ConfigMaps ermöglichen Aktualisieren von `app-config.yaml` ohne Neubau des Docker-Images

## Zusätzliche Ressourcen
- [ArgoCD Installation Guide](https://argo-cd.readthedocs.io/en/stable/getting_started/)
- [Backstage Kubernetes Deployment](https://backstage.io/docs/deployment/kubernetes/)

