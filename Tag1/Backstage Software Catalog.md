# Backstage Software Catalog: Troubleshooting
---

## Backstage Software Catalog: Troubleshooting

Wenn du mit dem **Backstage Software Catalog** arbeitest, treten gelegentlich Fehler beim **Ingestieren von Entitäten** auf. 
Das Verständnis, wie man diese Fehler diagnostiziert und behebt, ist essenziell für Plattformingenieure.

---

### Häufige Katalog-Fehler

Backstage verarbeitet Entitäten in mehreren Schritten:
**Discovery → Validation → Processing → Indexing**. Fehler können in jedem Schritt auftreten.

#### 1. EntityRef Not Found

* **Beschreibung:** Eine Entität referenziert eine andere, die nicht existiert (z.B. ein Component verweist auf einen nicht registrierten Owner).

* **Beispiel:**

```yaml
spec:
  owner: team-alpha  # Fehler, wenn Group "team-alpha" nicht existiert
```

* **Lösungen:**

  * Fehlende Entität erstellen oder registrieren
  * Referenz korrigieren (schreibtipps, Groß-/Kleinschreibung beachten)
  * Namespace korrekt angeben (`kind:namespace/name`)

---

#### 2. Invalid Entity Descriptor

* **Beschreibung:** YAML entspricht nicht dem Entity-Schema.
* **Ursachen:**

  * Fehlende Pflichtfelder (`apiVersion`, `kind`, `metadata.name`, `spec`)
  * Falscher Feldtyp oder ungültige Enum-Werte
  * YAML-Einrückung oder Formatfehler
* **Lösungen:**

  * `@backstage/cli validate-entity` lokal ausführen
  * Schema und Dokumentation prüfen
  * Mit funktionierenden Entity-Dateien vergleichen

---

#### 3. Refresh Failed

* **Beschreibung:** Backstage kann die Entität nicht vom Quellort abrufen.
* **Ursachen:**

  * Abgelaufener oder fehlender GitHub-Token
  * Repository/Pfad existiert nicht
  * Netzwerkprobleme oder Rate-Limiting
* **Lösungen:**

  * GitHub-Token mit ausreichendem Scope prüfen
  * Datei-/Pfadzugriff testen
  * Integration in `app-config.yaml` prüfen

---

#### 4. Entity Conflict

* **Beschreibung:** Zwei Entitäten haben die gleiche Kombination aus `kind`, `namespace` und `name`.
* **Lösungen:**

  * Eindeutige Namen verwenden
  * Namespaces nutzen
  * Doppelte Registrierungen entfernen

---

### Werkzeug: Processing Errors Panel

* Zugriff über das Backstage UI auf der Entity-Seite (`Processing` oder `Errors` Tab)
* Zeigt:

  * Fehlerbeschreibung
  * Zeitstempel
  * Retry-Count
  * Stack Trace (falls verfügbar)

---

### Manueller Refresh

* Über **Settings → Locations → Refresh**
* Alternative: „Refresh“-Button auf der Entity-Seite
* Hinweis: Standardmäßig refreshen Entitäten alle 100–200 Sekunden automatisch

---

### CLI-Validierung

```bash
# Einzelne Datei validieren
npx @backstage/cli validate-entity catalog-info.yaml

# Mehrere Dateien
npx @backstage/cli validate-entity entities/*.yaml
```

* Prüft YAML-Syntax, Pflichtfelder, Enum-Werte und Entity-Referenzen
* Integration in CI/CD empfiehlt sich, z. B. mit GitHub Actions

---

### Discovery-Tracing

**Flow:** `Location → Entity Files → Catalog`

1. **Location Registration**

   * App-Config oder manuelle UI-Registrierung
   * Status prüfen: Settings → Locations

2. **Entity File Discovery**

   * Pfad existiert und Zugriff vorhanden?
   * GitHub-Token für private Repos korrekt?

3. **Processing & Indexing**

   * Parsing, Preprocessing, Schema Validation, Relation Processing, Indexing
   * Fehler hier verhindern das Erscheinen der Entität

---

### Typische Troubleshooting-Szenarien

| Szenario                                | Ursache                                                   | Lösung                                                                       |
| --------------------------------------- | --------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Entity im Source, aber nicht im Catalog | Location nicht registriert oder Refresh-Zyklus abgewartet | Location prüfen, manuell refreshen, CLI-Validierung                          |
| Orphan Warning                          | Quelle gelöscht, Entity existiert noch                    | Datei wiederherstellen, Location re-registrieren oder Orphan manuell löschen |
| Broken Relations                        | Referenzierte Entity fehlt oder Tippfehler                | Fehlende Entität registrieren, Referenz korrigieren (`kind:namespace/name`)  |
| TechDocs nicht generiert                | `techdocs-ref` fehlt oder Builder falsch konfiguriert     | Annotation prüfen, `mkdocs.yml` prüfen, Build-Logs ansehen                   |

---

### Best Practices für einen gesunden Catalog

* Immer **lokal validieren** vor Commit
* Validation in **CI/CD-Workflows** einbauen
* **Processing Errors** regelmäßig überwachen
* **Ownership dokumentieren**
* **Naming Conventions** einhalten
* Orphan-Entitäten regelmäßig überprüfen
* Neue Entitäten zunächst in **Staging** testen

---

### Zusammenfassung Key Points

* Vier typische Fehler: `EntityRef not found`, `Invalid entity descriptor`, `Refresh failed`, `Entity conflict`
* Processing Errors Panel zeigt Details
* CLI Validator erkennt Probleme vor Commit
* Discovery-Flow: Location → File → Processing → Catalog
* Häufige Szenarien: Entity fehlt, Orphan-Warnungen, Broken Relations, TechDocs-Probleme

---

### Additional Resources

Backstage Entity Model Documentation
[documentation](https://backstage.io/docs/features/software-catalog/descriptor-format/)

Backstage Catalog Configuration
[documentation](https://backstage.io/docs/features/software-catalog/configuration/)

Backstage CLI Reference
[documentation](https://backstage.io/docs/tooling/cli/templates/#installing-custom-templates)

GitHub Integration for Backstage
[documentation](https://backstage.io/docs/integrations/github/locations/)













