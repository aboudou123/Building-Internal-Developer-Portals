
---

# Truck-Logistik-Datenpipeline

**Data Engineering**

**Autor:** Koffitse Aboudou



---

## Inhaltsverzeichnis

* [Problemstellung](#problemstellung)
* [Tools & Technologien](#tools--technologien)
* [Architekturdiagramm](#architekturdiagramm)
* [Datenquellen & Schema](#datenquellen--schema)
* [Mage-Orchestrierung](#mage-orchestrierung)
* [Dashboard & Visualisierung](#dashboard--visualisierung)
* [Weitere Ideen & nächste Schritte](#weitere-ideen--nächste-schritte)
* [Danksagungen & Quellen](#danksagungen--quellen)

---

## Problemstellung

Die Notwendigkeit zur Entwicklung einer Pipeline und eines Dashboards für ein Logistikunternehmen im Lkw-Bereich ergibt sich aus der Anforderung, kritische Informationen zu Transportaktivitäten effizient zu erfassen, zu verarbeiten und bereitzustellen – insbesondere Umsatzdaten pro Kunde. Durch den Aufbau einer umfassenden Pipeline, die die Phasen Datenerfassung, Vorverarbeitung, Analyse und Visualisierung umfasst, sowie eines benutzerfreundlichen Dashboard-Interfaces für Führungskräfte und das obere Management, kann das Unternehmen präzise Einblicke in Umsatzgenerierung, Kostenanalyse und die Gesamtleistung der operativen Prozesse gewinnen.

Dieses Projekt ist eine vollständige **ETL-Pipeline (Extract, Transform, Load)**, die darauf ausgelegt ist, die Datenverarbeitungsfähigkeiten eines Lkw-Logistikunternehmens zu optimieren und zu erweitern.

---

## Tools & Technologien

* **Containerisierung:** [Docker](https://www.docker.com/)
* **Workflow-Orchestrierung:** [Mage](https://www.mage.ai/)
* **Datentransformationen:** [dbt (Data Build Tool)](https://www.getdbt.com/)
* **Data Lake:** [Google Cloud Storage](https://cloud.google.com/storage?hl=en)
* **Data Warehouse:** [Google BigQuery](https://cloud.google.com/bigquery?hl=en)
* **Infrastructure as Code (IaC):** [Terraform](https://developer.hashicorp.com/terraform)
* **Visualisierung:** [Looker Studio](https://lookerstudio.google.com/navigation/reporting)

---

## Architekturdiagramm

<img width="949" height="664" alt="image" src="https://github.com/user-attachments/assets/8e7824d8-1f73-4fc6-b6a2-046c1705b927" />

---

## Datenquellen & Schema

Das Projekt wurde von einem Lkw-Flottenunternehmen inspiriert, bei dem Fahrer für unterschiedliche Unternehmen tätig sind. Um private Daten zu vermeiden, wurden die Daten simuliert. Die simulierten Daten befinden sich in den Verzeichnissen `data` und `seed`. Zur Generierung ähnlicher Daten kann das Python-Skript `datagenerator.py` ausgeführt werden.

Es gibt zwei zentrale CSV-Dateien:

1. Eine Datei im lokalen Ordner `data`, die wöchentlich aktualisiert wird.
2. Eine statische Tabelle mit Kundentarifen für verschiedene Fahrtarten.

### tripdata

Diese CSV-Datei enthält Fahrtdaten der Lkw-Logistik:

* **date:** Datum des Datensatzes
* **driver:** Name des Fahrers
* **customer:** Name des Kunden
* **hours:** Anzahl der Stunden für die Fahrt
* **km:** Zurückgelegte Strecke in Kilometern

### customerrates

Diese CSV-Datei enthält Kundentarife:

* **customer:** Name des Kunden
* **hourcity:** Stundensatz für Stadtfahrten
* **hourregular:** Stundensatz für reguläre Fahrten
* **hourhy:** Stundensatz für Autobahnfahrten
* **fsccity:** Kraftstoffzuschlagssatz für Stadtfahrten
* **fscregular:** Kraftstoffzuschlagssatz für reguläre Fahrten
* **fsch y:** Kraftstoffzuschlagssatz für Autobahnfahrten
* **hymileage:** Kilometerpreis für Autobahnfahrten

---

## Mage-Orchestrierung

<img src="static/assets/pipeline.png" alt="Pipeline" height="600" width="300">

Mage verwendet Python-, SQL- und dbt-Blöcke in folgender Reihenfolge:

1. CSV-Datei aus dem lokalen Verzeichnis laden
2. Transformationen durchführen und eine Parquet-Datei erstellen
3. Parquet-Datei in Google Cloud Storage exportieren
4. Externe Tabelle in BigQuery auf Basis der Parquet-Datei erstellen
5. dbt-Pakete mit `dbt deps` installieren
6. `customerrates` in das dbt-Projekt seeden
7. dbt-Modelle erstellen

   * Staging-Modelle mit zusätzlichen Spalten (`tripid`, `triptype`)
   * Core-Modell durch Join beider Staging-Modelle und Umsatzberechnung

---

## Dashboard & Visualisierung

Ein Dashboard wurde mit **Google Looker Studio** entwickelt, um Ereignisse in der Lkw-Logistik zu visualisieren. Durch die direkte Integration mit BigQuery profitiert das Dashboard von geringer Latenz und hoher Performance.

**Dashboard-Link:** [HIER](https://lookerstudio.google.com/reporting/8d39bf3b-80ee-4d58-898e-84173bb0af48/page/jcErD?s=pBt7UHZBg1k)

<img src="static/assets/dashboard.png" alt="Chart1" height="300" width="600">

---

## Weitere Ideen & nächste Schritte

* Größeren Datensatz verwenden
* `customer_rates`-Dateien nach Jahr trennen
* Mit mehr Daten Tools wie **dlt** einsetzen
* Partitionierung und Clustering implementieren

---


