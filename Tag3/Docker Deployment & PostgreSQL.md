
# Docker Deployment & PostgreSQL



---

## Warum Docker für Backstage?

### Vorteile der Containerisierung

- **Konsistente Umgebung**: Gleiche Laufzeitumgebung in Entwicklung, Staging und Produktion  
- **Einfache Skalierung**: Horizontale Skalierung mit Container-Orchestrierung  
- **Isolation**: Abhängigkeiten und Konfiguration innerhalb der Container  
- **Portabilität**: Läuft überall dort, wo Docker unterstützt wird  
- **Produktionsbereit**: Basis für Kubernetes- und Cloud-Deployments  

---

## Docker Compose für Multi-Container-Anwendungen

Docker Compose ermöglicht die Orchestrierung mehrerer Container lokal:

- **Backstage Backend**: Hauptanwendungsserver  
- **PostgreSQL-Datenbank**: Produktionsfähige Datenspeicherung  
- **Networking**: Automatische Service-Discovery zwischen Containern  
- **Volume-Management**: Persistenter Speicher für Datenbanken  

---

## Datenbankmigration: SQLite zu PostgreSQL

### Warum PostgreSQL in der Produktion?

SQLite eignet sich für die Entwicklung, hat aber Einschränkungen für den Produktionsbetrieb:

- **Nur Einzelbenutzer**: Datei-basierte DB kann keine parallelen Verbindungen handhaben  
- **Kein Netzwerkzugriff**: Kann nicht von mehreren Backend-Instanzen geteilt werden  
- **Begrenzte Skalierbarkeit**: Leistung sinkt bei größeren Datenmengen  

PostgreSQL bietet produktionsfähige Features:

- **Paralleler Zugriff**: Mehrere Backend-Instanzen können dieselbe DB sicher nutzen  
- **Datenintegrität**: ACID-Transaktionen gewährleisten Konsistenz unter Last  
- **Skalierbarkeit**: Effiziente Verarbeitung großer Datenmengen und komplexer Queries  
- **Backup und Recovery**: Robuste Tools für DB-Wartung und Disaster Recovery  
- **Connection Pooling**: Effizientes Management von DB-Verbindungen  

---

## Datenbankkonfiguration

Installiere zuerst das erforderliche PostgreSQL-Client-Paket:

```bash
yarn add pg@8.16.3
yarn add --dev @types/pg
````

Dann aktualisiere die Backend-Datenbankkonfiguration in der `app-config.yaml` von SQLite auf PostgreSQL:

```yaml
backend:
  database:
    client: pg
    connection:
      host: ${POSTGRES_HOST}
      port: ${POSTGRES_PORT}
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}
      database: ${POSTGRES_DB}
    pool:
      min: 5
      max: 20
      acquireTimeoutMillis: 60000
      idleTimeoutMillis: 600000
```

**Erläuterung der Pool-Einstellungen**:

* **min/max**: 5–20 Verbindungen für bessere Reaktionsfähigkeit
* **acquireTimeoutMillis**: bis zu 60 Sekunden auf verfügbare Verbindung warten
* **idleTimeoutMillis**: Leerlauf-Verbindungen 10 Minuten offen halten

---

## Backstage Build-Ansätze

### Host Build Approach (offizielle Empfehlung)

Backstage empfiehlt den Host-Build-Ansatz, bei dem auf dem Host gebaut wird und vorgebaute Artefakte ins Docker-Image gepackt werden.

**Vorteile**:

* Schnellere Builds (Host schneller als Docker)
* Bessere Caching-Mechanismen für Abhängigkeiten
* Zuverlässige Dependency-Resolution
* Offizieller Support von Backstage-Maintainern

**Prozess**:

```bash
# Abhängigkeiten installieren
yarn install

# TypeScript-Typen generieren
yarn tsc

# Backend-Bundle bauen
yarn build:backend
```

Dies erzeugt `skeleton.tar.gz` (Struktur) und `bundle.tar.gz` (kompilierter Code), die das Dockerfile extrahiert.

---

## Produktions-Dockerfile-Struktur

```dockerfile
FROM node:22-bookworm-slim

# Systemabhängigkeiten installieren (inkl. gosu)
RUN apt-get update && \
    apt-get install -y python3 g++ build-essential gosu

# Custom entrypoint für Docker-Secrets
COPY docker-entrypoint.sh /usr/local/bin/custom-entrypoint.sh
RUN chmod +x /usr/local/bin/custom-entrypoint.sh

# Nicht-root User
USER node
WORKDIR /app

# Yarn-Konfiguration kopieren
COPY --chown=node:node .yarn ./.yarn
COPY --chown=node:node .yarnrc.yml backstage.json ./

# Vorbuilt Skeleton extrahieren und Prod-Abhängigkeiten installieren
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz
RUN yarn workspaces focus --all --production

# Vorbuilt Bundle extrahieren
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

# Zurück zu root für entrypoint
USER root
ENTRYPOINT ["/usr/local/bin/custom-entrypoint.sh"]
CMD ["node", "packages/backend", "--config", "app-config.yaml"]
```

**Wichtige Komponenten**:

* Base Image: `node:22-bookworm-slim` (aktuelle LTS)
* `gosu`: sicheres User-Switching
* Custom EntryPoint: flexible Credential-Handling
* Non-root User: Sicherheitsrichtlinie
* Vorgebaute Artefakte: `skeleton.tar.gz` & `bundle.tar.gz`

---

## Docker Compose Konfiguration

### Entwicklungsumgebung

```yaml
services:
  postgres:
    image: postgres:13
    restart: unless-stopped
    environment:
      POSTGRES_USER: backstage
      POSTGRES_PASSWORD: backstage_secret
      POSTGRES_DB: backstage
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U backstage"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s

  backstage:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    ports:
      - "7007:7007"
    environment:
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_USER: backstage
      POSTGRES_PASSWORD: backstage_secret
      POSTGRES_DB: backstage
      NODE_ENV: production
      LOG_LEVEL: info
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./app-config.yaml:/app/app-config.yaml:ro
      - ./app-config.production.yaml:/app/app-config.production.yaml:ro
      - ./examples:/app/examples:ro

volumes:
  postgres_data:
    driver: local
```

**Wichtige Features**:

* Healthchecks: DB-Ready Status
* `depends_on` mit Bedingung: Backstage wartet auf PostgreSQL
* Restart-Policy: automatische Neustarts
* Named Volumes: persistente Daten
* Config-Mounts: Read-Only Konfigurationsdateien
* Environment Variables für Entwicklung

> Hinweis: Für Produktion sollten **Docker-Secrets** verwendet werden, nicht Environment-Variablen.

---

## Entwicklungs-Workflow

### Docker Compose Befehle

```bash
# Alle Services starten
docker compose up -d

# Logs eines Services anzeigen
docker compose logs -f backstage

# Status prüfen
docker compose ps

# Service neu starten
docker compose restart backstage

# Alles stoppen und Volumes entfernen
docker compose down -v
```

### Debugging

```bash
# Befehle im Container ausführen
docker compose exec backstage bash

# Logs anzeigen
docker compose logs backstage

# Health prüfen
docker compose exec postgres pg_isready -U backstage

# Konfiguration prüfen
docker compose exec backstage env | grep POSTGRES
```

---

## Häufige Probleme & Lösungen

### Datenbank-Verbindungsfehler

* Healthcheck prüfen: `docker compose ps`
* DB-Ready prüfen: `docker compose exec postgres pg_isready -U backstage`
* Netzwerk prüfen: `docker compose exec backstage ping postgres`
* Logs prüfen: `docker compose logs postgres`

### Port-Konflikte

```yaml
ports:
  - "7008:7007"  # Host-Port ändern
  - "5433:5432"  # Datenbank-Port ändern
```

### Volume-Berechtigungen

* Eigentümer prüfen: `docker compose exec postgres ls -la /var/lib/postgresql/data`
* Volumes neu erstellen: `docker compose down -v && docker compose up -d`

---

## Nächste Schritte

* Produktionstaugliches Dockerfile schreiben
* Backstage auf PostgreSQL konfigurieren
* Docker Compose Produktionskonfiguration erstellen
* Sicherheit mit Docker-Secrets implementieren

---

## Kernpunkte

* Docker sorgt für konsistente Umgebung, Isolation und Portabilität
* PostgreSQL ersetzt SQLite in Produktion (ACID, paralleler Zugriff, Connection Pooling)
* Host Build Approach erzeugt vorgebaute Artefakte für schnellere, zuverlässige Docker-Builds
* Dockerfile: non-root User, prebuilt Bundle
* Connection Pool: 5–20 Verbindungen, Timeout konfigurierbar
* Docker Compose orchestriert Multi-Container-Anwendungen mit Healthchecks
* Entwicklungsumgebung: Environment Variables, offene Ports für Debugging
* Named Volumes für persistente Daten und Config-Mounts für Updates

---

## Zusätzliche Ressourcen

* [Backstage Docker Documentation](https://backstage.io/docs)
* [Docker Compose Documentation](https://docs.docker.com/compose/)
* [PostgreSQL Docker Image](https://hub.docker.com/_/postgres)

```

---


```

