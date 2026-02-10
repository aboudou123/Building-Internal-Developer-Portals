# Lektion: Docker-Bereitstellung & PostgreSQL

Erlerne die Grundlagen von Docker, PostgreSQL-Migration und die Entwicklungsbereitstellung mit Docker Compose.

## Kernpunkte

*   **Docker** bietet konsistente Umgebungen, Isolierung und Portabilität für die Backstage-Bereitstellung in allen Phasen.
*   **PostgreSQL** ersetzt SQLite für den Produktionseinsatz und ermöglicht gleichzeitigen Zugriff, ACID-Transaktionen und effizientes Connection-Pooling.
*   Der **Host-Build-Ansatz** erstellt vorab gebaute Artefakte (`skeleton.tar.gz` und `bundle.tar.gz`) für schnellere und zuverlässigere Docker-Builds.
*   Die **Dockerfile** verwendet ein Basis-Image mit Non-Root-User für mehr Sicherheit und extrahiert das vorab gebaute Bundle.
*   **Connection-Pooling** verwaltet 5-20 Verbindungen mit konfigurierbaren Timeout-Einstellungen für optimale Leistung.
*   **Docker Compose** orchestriert Multi-Container-Anwendungen mit Health-Checks und Service-Abhängigkeitsmanagement.
*   Die **Entwicklungskonfiguration** verwendet Umgebungsvariablen und exponiert Datenbank-Ports für Debugging-Zugriff.
*   **Benannte Volumes** persistieren PostgreSQL-Daten über Container-Neustarts hinweg, und die Bereitstellung von Konfigurationsdateien ermöglicht eine einfache Anpassung.


```markdown
# Docker-Sicherheit & Produktionsbereitstellung

Umgebungsvariablen eignen sich für die Entwicklung, Produktionsbereitstellungen erfordern jedoch eine sichere Verwaltung von Zugangsdaten. Diese Lektion behandelt Docker Secrets und Best Practices für Produktionssicherheit.

## Warum Umgebungsvariablen unsicher sind

Umgebungsvariablen haben für den Produktionseinsatz kritische Sicherheitseinschränkungen:

**Sichtbarkeitsprobleme:**
*   Sichtbar in `ps aux` und der Ausgabe von `docker inspect`
*   In Anwendungslogs und Fehlermeldungen offengelegt
*   Im Klartext in Docker-Images und Compose-Dateien gespeichert
*   Können leicht versehentlich in Git-Repositories eingecheckt werden
*   Jeder Prozess im Container kann sie lesen

Für die Produktion benötigen wir ein besseres Geheimnis-Management.

## Docker Secrets: Ein besserer Ansatz

Docker Secrets bieten eine verbesserte Sicherheit durch:
*   Verschlüsselte Speicherung im Docker-Daemon
*   Zugriffskontrolle – nur deklarierte Container erhalten Zugriff
*   Bereitstellung als Dateien unter `/run/secrets/` (nicht als Umgebung)
*   Nicht in Images – Secrets werden zur Laufzeit hinzugefügt, nicht zur Build-Zeit
*   Getrennt vom Code – außerhalb der Anwendungsdateien gespeichert

**Wichtig:** Secrets werden als schreibgeschützte Dateien unter `/run/secrets/<secret_name>` bereitgestellt und sind nur vom Benutzer `root` lesbar.

## Custom Entrypoint für Docker Secrets

### Die Herausforderung
Da die Dateien unter `/run/secrets/` nur vom Benutzer `root` lesbar sind, Anwendungen aber als Nicht-Root-Benutzer laufen sollten, benötigen wir einen benutzerdefinierten Entrypoint, der:
1.  Die Secret-Datei liest (als `root`)
2.  Sie der Anwendung verfügbar macht
3.  Zum Nicht-Root-Benutzer wechselt

### Die Lösung
```bash
#!/bin/sh
set -e

# Docker Secret lesen und als Umgebungsvariable exportieren
if [ -n "$POSTGRES_PASSWORD_FILE" ] && [ -f "$POSTGRES_PASSWORD_FILE" ]; then
    export POSTGRES_PASSWORD=$(cat "$POSTGRES_PASSWORD_FILE")
    echo "✓ PostgreSQL-Passwort aus Secret-Datei geladen"
fi

# Zu Nicht-Root-Benutzer mit gosu wechseln
if [ "$(id -u)" = "0" ]; then
    exec gosu node "$@"
else
    exec "$@"
fi
```
**Funktionsweise:**
1.  Prüft, ob `POSTGRES_PASSWORD_FILE` gesetzt ist und die Datei existiert
2.  Liest die Secret-Datei als `root` (nur `root` kann auf `/run/secrets/` zugreifen)
3.  Exportiert sie als `POSTGRES_PASSWORD` Umgebungsvariable (für Backstage)
4.  Wechselt vom Benutzer `root` zum Benutzer `node` mit `gosu`
5.  Führt die Anwendung als Nicht-Root-Benutzer aus

**Warum gosu?** Besser als `su`/`sudo` für Container – ordnungsgemäße Signalbehandlung, PID-1-Sicherheit und bewahrt Exit-Codes.

### Kritische Sicherheitseinschränkung
Dieser Ansatz ist besser als fest codierte Passwörter, aber nicht perfekt.

Sobald als Umgebungsvariable exportiert, bleibt das Passwort sichtbar in:
❌ `docker inspect`-Ausgabe
❌ Prozessumgebung (`ps aux`)
❌ Für jeden Prozess im Container

Was wir gewinnen:
✅ Secrets nicht in Docker-Images
✅ Secrets nicht in Compose-Dateien oder Git
✅ Getrennte Secret-Dateien mit `chmod 600`

Für echte Produktionssicherheit im Unternehmensmaßstab:
*   **Externe Secret-Manager:** Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager
*   **Lesen auf Anwendungsebene:** App so modifizieren, dass sie direkt von `/run/secrets/` liest
*   **Service-Mesh:** Istio/Linkerd mit mTLS (keine Passwörter erforderlich)

Dies ist ein pragmatischer Mittelweg – funktioniert mit dem Design von Backstage (erwartet Env-Vars) und ist sicherer als fest codierte Passwörter. Geeignet für kleine bis mittlere Bereitstellungen.

*Was ist mit K8s Secrets? Auch diese können wir nutzen, wenn wir später in K8s deployen, wie wir sehen werden.*

## Produktions-Compsoe-Konfiguration

### Secrets erstellen
```bash
mkdir -p secrets
echo "my-secure-production-password-2024!" > secrets/postgres_password.txt
chmod 600 secrets/postgres_password.txt
```

### Produktions-Docker-Compose
```yaml
services:
  postgres:
    image: postgres:13
    restart: unless-stopped
    environment:
      POSTGRES_USER: backstage
      POSTGRES_DB: backstage
      POSTGRES_PASSWORD_FILE: /run/secrets/postgres_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U backstage"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s
    secrets:
      - postgres_password

  backstage:
    build: .
    restart: unless-stopped
    ports:
      - "7007:7007"
    environment:
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_USER: backstage
      POSTGRES_DB: backstage
      POSTGRES_PASSWORD_FILE: /run/secrets/postgres_password
      NODE_ENV: production
      LOG_LEVEL: warn
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./app-config.yaml:/app/app-config.yaml:ro
      - ./app-config.production.yaml:/app/app-config.production.yaml:ro
      - ./examples:/app/examples:ro
    secrets:
      - postgres_password

secrets:
  postgres_password:
    file: ./secrets/postgres_password.txt

volumes:
  postgres_data:
    driver: local
```

### Hauptunterschiede zur Entwicklung
| Merkmal | Entwicklung | Produktion |
| :--- | :--- | :--- |
| Passwort | `POSTGRES_PASSWORD` (fest codiert) | `POSTGRES_PASSWORD_FILE` (Secret-Datei) |
| Datenbank-Port | Exposed (5432) | Nicht exponiert |
| Log-Level | `info` | `warn` |
| Secret-Quelle | In Compose-Datei | Separate Datei + `.gitignore` |

### Produktions-Workflow
```bash
# Bereitstellen
docker compose -f docker-compose.production.yml up -d

# Verifizieren, dass Secrets gemountet sind
docker compose -f docker-compose.production.yml exec backstage ls -la /run/secrets/

# Logs auf Secret-Loading prüfen
docker compose -f docker-compose.production.yml logs backstage | grep "Loaded PostgreSQL"

# Prüfen, dass Datenbank nicht exponiert ist
docker compose -f docker-compose.production.yml ps postgres
```

## Sicherheits-Best Practices

**Secret-Management:**
*   `secrets/` zu `.gitignore` hinzufügen
*   `chmod 600` für Secret-Dateien verwenden
*   Starke Passwörter generieren: `openssl rand -base64 32`
*   Secrets regelmäßig rotieren (alle 90 Tage)
*   Für Unternehmen: Vault, AWS Secrets Manager oder Service-Mesh verwenden

**Container-Sicherheit:**
*   Als Nicht-Root-Benutzer ausführen (Entrypoint wechselt zu `node`)
*   Keine unnötigen Ports exponieren (Datenbank nur intern)
*   Schreibgeschützte Volume-Mounts verwenden (`:ro`)
*   `LOG_LEVEL: warn` in der Produktion setzen

**Bereitstellungssicherheit:**
*   Separate Dev/Prod Compose-Dateien
*   Nie Entwicklungs-Zugangsdaten in der Produktion verwenden
*   Health-Checks implementieren
*   Docker und Host-Betriebssystem aktuell halten

## Häufige Probleme

**Passwort immer noch in der Umgebung sichtbar**
Realitätscheck: Nachdem der Entrypoint läuft, ist `POSTGRES_PASSWORD` gesetzt und in der Prozessumgebung des Containers sichtbar.

Was das bedeutet:
*   `docker inspect backstage` zeigt das Passwort
*   Prozesse innerhalb des Containers können es lesen
*   Trotzdem besser als feste Codierung in der Compose-Datei

Für vollständige Sicherheit: Externe Secret-Manager (Vault) verwenden oder Backstage so modifizieren, dass es direkt von `/run/secrets/` liest.

**Secrets in Image-Layern**
**Falsch:**
```dockerfile
ENV POSTGRES_PASSWORD=my-secret  # Ins Image eingebacken!
```
**Richtig:**
```dockerfile
ENTRYPOINT ["/usr/local/bin/custom-entrypoint.sh"]  # Secrets zur Laufzeit
```

## Nächste Schritte

Sie verstehen jetzt:
*   Warum Umgebungsvariablen unsicher sind
*   Wie Docker Secrets bessere Sicherheit bieten
*   Das Custom-Entrypoint-Pattern mit `gosu`
*   Die Einschränkungen dieses Ansatzes
*   Wann externe Secret-Manager verwendet werden sollten

Im Lab implementieren Sie eine Docker-Secrets-Bereitstellung und sehen die Sicherheitsverbesserungen aus erster Hand.

## Kernpunkte
*   Umgebungsvariablen sind in `docker inspect`, Prozesslisten und Logs sichtbar – ungeeignet für Produktionssecrets.
*   Docker Secrets stellen verschlüsselte Dateien unter `/run/secrets/` bereit, die nur vom Benutzer `root` lesbar sind.
*   Custom Entrypoint liest Secrets als `root`, exportiert sie in Umgebungsvariablen und wechselt dann mit `gosu` zu einem Nicht-Root-Benutzer.
*   Kritische Einschränkung: Exportierte Umgebungsvariablen bleiben in der Container-Prozessumgebung sichtbar.
*   Dieser Ansatz ist ein pragmatischer Mittelweg – Secrets nicht in Images oder Git, geeignet für kleine bis mittlere Bereitstellungen.
*   Produktionskonfiguration verwendet `POSTGRES_PASSWORD_FILE` mit separaten Secret-Dateien und `chmod 600`-Berechtigungen.
*   Für Unternehmenssicherheit: Vault, AWS Secrets Manager, Lesen auf Anwendungsebene oder Service-Mesh mit mTLS verwenden.
*   Sicherheits-Best Practices: Separate Dev/Prod-Konfigurationen, keine exponierten Datenbank-Ports, schreibgeschützte Mounts, regelmäßige Rotation.

## Zusätzliche Ressourcen
*   [Docker Secrets Dokumentation](https://docs.docker.com/engine/swarm/secrets/)
*   [Docker Security Best Practices](https://docs.docker.com/engine/security/)
*   [gosu - Simple Go-based setuid+setgid+setgroups+exec](https://github.com/tianon/gosu)

## Lektionsnotizen (Privat)
Nehmen Sie private Notizen während Sie lernen...

## Studiengruppe
Diesem Kurs ist noch keine Studiengruppe zugeordnet.

Bitten Sie Ihren Instruktor, eine Studiengruppe zu verlinken, um Diskussionen zu ermöglichen.
```