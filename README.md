# Test: Building Internal Developer Portals mit Backstage

## Inhaltsverzeichnis

- [Aufgabe 1 – Grundlagen: Ziel und Nutzen eines Internal Developer Portals](#aufgabe-1--grundlagen-ziel-und-nutzen-eines-internal-developer-portals)
- [Aufgabe 2 – Lokale Umgebung & erste Backstage-App](#aufgabe-2--lokale-umgebung--erste-backstage-app)
- [Aufgabe 3 – Projektstruktur und Konfiguration verstehen](#aufgabe-3--projektstruktur-und-konfiguration-verstehen)
- [Aufgabe 4 – Überblick: Software-Templates und Scaffolder](#aufgabe-4--überblick-software-templates-und-scaffolder)
- [Aufgabe 5 – Template-Metadaten & Auffindbarkeit](#aufgabe-5--template-metadaten--auffindbarkeit)
- [Aufgabe 6 – Benutzereingabeformulare und Parameter](#aufgabe-6--benutzereingabeformulare-und-parameter)
- [Aufgabe 7 – Template-Skelett, Scaffolding-Schritte & Ausgabe](#aufgabe-7--template-skelett-scaffolding-schritte--ausgabe)
- [Aufgabe 8 – Testen und Veröffentlichen von Templates](#aufgabe-8--testen-und-veröffentlichen-von-templates)
- [Aufgabe 9 – TechDocs & Documentation-as-Code](#aufgabe-9--techdocs--documentation-as-code)
- [Aufgabe 10 – Docker-Bereitstellung & Sicherheit](#aufgabe-10--docker-bereitstellung--sicherheit)
- [Aufgabe 11 – Keycloak-Integration (Authentifizierung)](#aufgabe-11--keycloak-integration-authentifizierung)
- [Aufgabe 12 – Role-Based Access Control mit Permissions](#aufgabe-12--role-based-access-control-mit-permissions)
- [Aufgabe 13 – Kubernetes-Bereitstellung von Backstage](#aufgabe-13--kubernetes-bereitstellung-von-backstage)
- [Aufgabe 14 – Kubernetes-Plugin-Konfiguration](#aufgabe-14--kubernetes-plugin-konfiguration)
- [Aufgabe 15 – Eigenes Backstage-Plugin entwickeln](#aufgabe-15--eigenes-backstage-plugin-entwickeln)
- [Aufgabe 16 – Backstage in Produktion mit GitOps](#aufgabe-16--backstage-in-produktion-mit-gitops)

---

## Aufgabe 1 – Grundlagen: Ziel und Nutzen eines Internal Developer Portals

**Aufgabe:**  
Erläutern Sie, was unter *Building Internal Developer Portals* mit Backstage zu verstehen ist.

Gehen Sie insbesondere darauf ein:

1. Welche Probleme und Reibungspunkte im Alltag von Entwicklungsteams ein internes Developer-Portal adressiert  
2. Welche Rolle Backstage dabei spielt (z. B. als zentrales Portal, Plugin-Plattform, „Single Pane of Glass“)  
3. Wie ein IDP zur Verbesserung der Developer Experience, Standardisierung und Effizienz beiträgt

**Antwortbereich:**

---

## Aufgabe 2 – Lokale Umgebung & erste Backstage-App

**Aufgabe:**  
Beschreiben Sie die grundlegenden Schritte, um Backstage lokal lauffähig zu machen und erste Erfahrungen mit der Oberfläche zu sammeln.

Gehen Sie dabei auf Folgendes ein:

1. **Umgebung einrichten**  
   - Welche Abhängigkeiten (z. B. Node.js, Yarn, Docker) geprüft und ggf. installiert werden müssen  
   - Wie Sie sicherstellen, dass die Entwicklungsumgebung korrekt konfiguriert ist

2. **Erste Backstage-App erstellen**  
   - Wie mit der Backstage-CLI eine neue Anwendung erstellt wird  
   - Welche Eingaben/Optionen im Scaffold-Prozess relevant sind

3. **App starten & Oberfläche erkunden**  
   - Wie die App gestartet wird  
   - Welche zentralen Bereiche der Oberfläche (Katalog, Templates, Doku, Admin) Sie zuerst erkunden sollten

**Antwortbereich:**

---

## Aufgabe 3 – Projektstruktur und Konfiguration verstehen

**Aufgabe:**  
Analysieren Sie die interne Struktur der neu erstellten Backstage-App.

1. **Projektstruktur verstehen**  
   - Welche Hauptbestandteile das Projekt hat (z. B. `packages/app`, `packages/backend`, Plugins)  
   - Wie Frontend, Backend und Plugins organisatorisch getrennt sind

2. **Konfiguration verstehen**  
   - Welche zentralen Konfigurationsdateien existieren (z. B. `app-config.yaml`, `app-config.local.yaml`)  
   - Welche typischen Anpassungen für erste Experimente sinnvoll sind (z. B. Integrationen, Feature-Flags)

3. **Bedeutung für spätere Erweiterungen**  
   - Warum ein gutes Verständnis der Struktur wichtig ist, um später Templates, TechDocs, Plugins, Auth usw. sinnvoll zu konfigurieren

**Antwortbereich:**

---

## Aufgabe 4 – Überblick: Software-Templates und Scaffolder

**Aufgabe:**  
Erklären Sie die Rolle von Backstage-Software-Templates im Kontext eines Internal Developer Portals.

Nutzen Sie die folgenden Leitfragen:

1. Was ist der Zweck von Software-Templates in Backstage für Entwicklungsteams?  
2. Wie funktioniert der Backstage Scaffolder auf hoher Ebene, um neue Projekte aus Templates zu erzeugen?  
3. Welche typischen Komponenten gehören zu einem Software-Template (z. B. Metadaten, Parameter, Skelett, Scaffolding-Schritte, Ausgabe)?

**Basistext aus dem Test:**  
> Bestehende Templates erkunden, um zu verstehen, wie der Backstage Scaffolder funktioniert und aus welchen Komponenten ein Software-Template besteht.

**Antwortbereich:**

---

## Aufgabe 5 – Template-Metadaten & Auffindbarkeit

**Aufgabe:**  
Fokussieren Sie sich auf den Metadaten-Bereich eines Software-Templates.

1. Welche Informationen werden in den Template-Metadaten typischerweise gepflegt (z. B. Name, Beschreibung, Tags, Owner, Kategorien)?  
2. Wie helfen diese Metadaten, ein Template im Portal leicht zu finden und den Anwendungszweck zu verstehen?  
3. Wie tragen gut gepflegte Metadaten zur Governance und zur Etablierung von „Golden Paths“ bei?

**Basistext aus dem Test:**  
> Den Metadaten-Bereich eines Software-Templates definieren, damit es für Entwicklungsteams auffindbar und nutzbar ist.

**Antwortbereich:**

---

## Aufgabe 6 – Benutzereingabeformulare und Parameter

**Aufgabe:**  
Analysieren Sie den Parameterbereich eines Templates, der das Benutzereingabeformular definiert.

1. Welche typischen Parameter/Felder würden Sie einem Template geben (z. B. Service-Name, Repository-URL, Programmiersprache, CI/CD-Optionen)?  
2. Wie wird daraus ein Formular erzeugt, das Entwickler beim Verwenden des Templates ausfüllen?  
3. Wie lässt sich über Parameter eine gute Balance zwischen Standardisierung (vorgegebene Strukturen) und Flexibilität (Optionen/Varianten) erreichen?

**Basistext aus dem Test:**  
> Den Parameterbereich erstellen, der das Formular definiert, das Entwickler beim Verwenden des Templates ausfüllen.

**Antwortbereich:**

---

## Aufgabe 7 – Template-Skelett, Scaffolding-Schritte & Ausgabe

**Aufgabe:**  
Beschreiben Sie, wie aus einem Template ein neues Projekt generiert wird.

1. **Template-Skelett**  
   - Was sind Skelettdateien?  
   - Wie werden Platzhaltervariablen dort hinterlegt und später ersetzt?

2. **Scaffolding-Schritte**  
   - Welche typischen Schritte steuern die Generierung eines Projekts (Dateien erzeugen, Git-Repo anlegen, CI/CD-Pipeline erstellen usw.)?  
   - Wie würden Sie diese Schritte in einer sinnvollen Reihenfolge anordnen?

3. **Template-Ausgabe**  
   - Welche Informationen sollten Nutzer:innen nach Abschluss des Scaffolding-Prozesses sehen (z. B. Repo-Links, nächste Schritte, lokale Startbefehle)?  
   - Wie kann die Ausgabe so gestaltet werden, dass Onboarding und Weiterarbeit möglichst einfach sind?

**Basistexte aus dem Test:**  
> Die Skelettdateien erstellen, die als Vorlage für die Generierung neuer Projekte mit Platzhaltervariablen dienen.  
> Die Scaffolding-Schritte definieren, die steuern, wie das Template Projekte generiert.  
> Den Ausgabebereich konfigurieren, der den Benutzern anzeigt, was erstellt wurde und wie sie nach der Generierung darauf zugreifen können.

**Antwortbereich:**

---

## Aufgabe 8 – Testen und Veröffentlichen von Templates

**Aufgabe:**  
Stellen Sie dar, wie ein neues Template qualitätsgesichert und für Teams verfügbar gemacht wird.

1. Welche Schritte sind nötig, um ein Template end-to-end zu testen (inkl. GitHub-Authentifizierung und Erzeugung eines echten Repositories)?  
2. Warum ist dieser Testprozess wichtig für Zuverlässigkeit und Vertrauen der Nutzer?  
3. Wie wird ein Template so veröffentlicht, dass es im Backstage-Katalog erscheint und von Teams verwendet werden kann?

**Basistext aus dem Test:**  
> Die GitHub-Authentifizierung einrichten und das vollständige Template testen, indem es end-to-end ausgeführt wird, um ein echtes Repository zu erstellen.

**Antwortbereich:**

---

## Aufgabe 9 – TechDocs & Documentation-as-Code

**Aufgabe:**  
Erklären Sie das Konzept von TechDocs und wie es Documentation-as-Code in Backstage ermöglicht.

Beantworten Sie:

1. Was bedeutet *Documentation-as-Code* im Kontext eines Developer-Portals?  
2. Welche Rolle spielen folgende Komponenten:
   - TechDocs in Backstage  
   - Speicher-Backend (z. B. lokaler Storage)  
   - MkDocs (inkl. Material-Theme)
3. Wie können Dokumentationen mit Catalog-Entitäten verknüpft werden, und welche Vorteile hat das für Entwicklerteams?

**Basistext aus dem Test (verkürzt):**  
> TechDocs konfigurieren, um Documentation-as-Code-Workflows in Backstage zu ermöglichen.  
> MkDocs integrieren, Speicher-Backends einrichten, Dokumentationen erstellen, mit Catalog-Entitäten integrieren und Workflows testen.

**Antwortbereich:**

---

## Aufgabe 10 – Docker-Bereitstellung & Sicherheit

**Aufgabe:**  
Überführen Sie Backstage von der lokalen Entwicklung zu einer produktionsnäheren Docker-Bereitstellung.

1. Welche Bestandteile gehören zu einer produktionsreifen Docker-Bereitstellung von Backstage (Dockerfile, Docker Compose, PostgreSQL, persistente Volumes)?  
2. Warum ist der Einsatz von Docker Secrets gegenüber einfachen Umgebungsvariablen für Produktionssicherheit „kritisch“?  
3. Nennen Sie zentrale Best Practices für Deployment und Sicherheit in diesem Setup (z. B. least privilege, Netzwerkgrenzen, Logging).

**Basistexte aus dem Test (verkürzt):**  
> Backstage mit Docker Compose und PostgreSQL bereitstellen.  
> Persistente Speicherung einrichten und Produktionssicherheit mit Docker Secrets umsetzen.  
> Containerisierte Bereitstellung testen und häufige Probleme beheben.

**Antwortbereich:**

---

## Aufgabe 11 – Keycloak-Integration (Authentifizierung)

**Aufgabe:**  
Beschreiben Sie, wie Keycloak als OIDC-Provider für Backstage integriert wird.

1. Welche Elemente müssen in Keycloak konfiguriert werden (Realms, Clients, Benutzer, Gruppen, OIDC-Einstellungen)?  
2. Wie wird der OIDC-Authentifizierungsprovider in Backstage eingerichtet, sodass sich Nutzer anmelden und Sitzungen erzeugt werden können?  
3. Welche Best Practices sollten Sie bei produktionsreifer Authentifizierung beachten (z. B. sichere Redirect-URIs, Token-Lebensdauer, TLS)?

**Basistext aus dem Test (verkürzt):**  
> Keycloak mit Docker bereitstellen, Realms, Clients und OIDC für Backstage konfigurieren, Authentifizierung integrieren und Workflows testen.

**Antwortbereich:**

---

## Aufgabe 12 – Role-Based Access Control mit Permissions

**Aufgabe:**  
Aufbauend auf der Authentifizierung soll nun Autorisierung umgesetzt werden.

1. Wie wird das Backstage-Berechtigungsframework grundsätzlich aktiviert und konfiguriert?  
2. Wie können Keycloak-Gruppen (z. B. `platform-admins`, `developers`, `guests`) genutzt werden, um Rollen und Zugriffslevel (Admin, Entwickler, Gast) in Backstage umzusetzen?  
3. Geben Sie Beispiele dafür,
   - wie Software-Templates und Katalogaktionen rollenbasiert gesperrt werden können  
   - und wie die Policy-Entscheidungen ALLOW, DENY und CONDITIONAL praktisch eingesetzt werden.

**Basistext aus dem Test (verkürzt):**  
> Das Backstage-Berechtigungsframework aktivieren, Policies schreiben, unterschiedliche Zugriffslevel implementieren und testen.

**Antwortbereich:**

---

## Aufgabe 13 – Kubernetes-Bereitstellung von Backstage

**Aufgabe:**  
Skizzieren Sie die Schritte, um Backstage produktionsreif auf Kubernetes zu deployen.

1. Wie stellen Sie PostgreSQL über Kubernetes-Manifeste bereit (Secrets, PersistentVolume/PersistentVolumeClaim, Deployment, Service)?  
2. Wie bauen Sie ein benutzerdefiniertes Backstage-Docker-Image (Host-Build-Ansatz) und deployen es mittels Deployments/Services in Kubernetes?  
3. Wie konfigurieren Sie Secrets und Umgebungsvariablen korrekt, und wie prüfen Sie die Integration zwischen Backstage und PostgreSQL (z. B. über `kubectl port-forward`)?

**Basistext aus dem Test (verkürzt):**  
> Backstage auf Kubernetes bereitstellen, PostgreSQL via Manifests deployen, eigenes Image bauen und Integration prüfen.

**Antwortbereich:**

---

## Aufgabe 14 – Kubernetes-Plugin-Konfiguration

**Aufgabe:**  
Konfigurieren Sie das Kubernetes-Plugin in Backstage, um Workloads sichtbar zu machen.

1. Welche Schritte sind nötig, um Kubernetes-Frontend- und Backend-Plugins in Backstage zu installieren und zu aktivieren?  
2. Wie richten Sie einen ServiceAccount mit passenden Lese-RBAC-Rechten ein und konfigurieren Cluster-Lokatoren und Authentifizierungsstrategien?  
3. Wie verknüpfen Sie Katalog-Entitäten mit Kubernetes-Ressourcen (Annotations, Labels), sodass Pods, Deployments und Services in der Backstage-Oberfläche sichtbar werden?

**Basistext aus dem Test (verkürzt):**  
> Kubernetes-Plugin installieren, RBAC konfigurieren, Entity-Anmerkungen und Labels setzen, Workloads im Portal erkunden.

**Antwortbereich:**

---

## Aufgabe 15 – Eigenes Backstage-Plugin entwickeln

**Aufgabe:**  
Betrachten Sie die Entwicklung eines eigenen Plugins mit Backend- und Frontend-Komponenten.

1. Welche technischen Voraussetzungen (JavaScript/TypeScript, React, Node.js) sollten idealerweise gegeben sein, bevor man ein eigenes Plugin entwickelt? Begründen Sie, warum.  
2. Skizzieren Sie die Schritte zum Erstellen eines einfachen Plugins, das in Ihr bestehendes Authentifizierungssystem integriert ist (z. B. neue Ansicht mit geschützten Daten).  
3. Nennen Sie praxisnahe Beispiele, wofür Organisationen eigene Plugins bauen (z. B. interne Tools, Compliance-Checks, Deployment-Dashboards).

**Basistexte aus dem Test (verkürzt):**  
> Ein einfaches Plugin mit Backend- und Frontend-Komponenten erstellen.  
> Fortgeschrittenes Lab für erfahrene JavaScript/TypeScript-, React- und Node.js-Entwickler.

**Antwortbereich:**

---

## Aufgabe 16 – Backstage in Produktion mit GitOps

**Aufgabe:**  
Zeichnen Sie ein Gesamtbild eines produktionsreifen Developer-Portals mit GitOps.

1. Wie wird ArgoCD in Backstage integriert, um eine Übersicht über Deployments zu geben und GitOps-Patterns zu ermöglichen?  
2. Welche Rolle spielt Keycloak SSO im produktiven Setup für Entwickler-Onboarding, Gruppenverwaltung und Single Sign-On?  
3. Beschreiben Sie einen vollständigen, idealtypischen Self-Service-Workflow:
   - vom ersten Login ins Portal  
   - über die Nutzung eines Golden-Path-Templates  
   - bis zur automatisierten Bereitstellung auf Kubernetes inklusive Sichtbarkeit der Workloads in Backstage.

**Basistext aus dem Test (verkürzt):**  
> Produktionsreifes Developer-Portal mit Keycloak SSO, ArgoCD-GitOps und Kubernetes-Integration aufbauen.  
> Vollständigen Self-Service-Workflow und Plattform-Engineering-Patterns erleben.

**Antwortbereich:**
