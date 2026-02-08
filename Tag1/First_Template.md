
## Anweisungen

### Verständnis von Backstage-Software-Templates

Bevor Sie Ihr eigenes Template erstellen, sollten Sie vorhandene Templates untersuchen, um zu verstehen, wie der Backstage Scaffolder funktioniert und aus welchen Komponenten ein Software-Template besteht.

---

### Schritt 1: Überprüfen, ob Backstage läuft

Überprüfen Sie den Status des Dienstes:

```bash
systemctl status backstage
```

<img width="838" height="680" alt="image" src="https://github.com/user-attachments/assets/420bdbc8-ea5f-4e0f-b6df-4bd0c89c5f17" />

# Ergebnis
<img width="1095" height="970" alt="image" src="https://github.com/user-attachments/assets/9004d71a-bdee-4fb0-b9fe-c1c8e2dc74d4" />


Drücken Sie **q**, um die Statusansicht zu verlassen. Klicken Sie im Backstage-App-Tab in der Seitenleiste auf **Create**, um den Bereich **Software Templates** anzuzeigen.


<img width="1072" height="731" alt="image" src="https://github.com/user-attachments/assets/4a29d274-db4f-4b2b-ae95-ad40b856731f" />

---

### Schritt 2: Template-Struktur untersuchen

Untersuchen Sie die Standard-Templates, um deren Struktur zu verstehen. Suchen Sie nach `template.yaml`-Dateien:

```bash
find /root/labs/developer-portal -name "template.yaml" -o -name "*.yaml" | grep -i template
```
<img width="1308" height="328" alt="image" src="https://github.com/user-attachments/assets/6bdebbc8-12fa-42e7-bab9-7154dcd9aa3d" />

---

### Schritt 3: Template-Komponenten analysieren

Erstellen Sie eine Verzeichnisstruktur zum Studium der Templates:

```bash
mkdir -p /root/labs/templates-study
```

Navigieren Sie in das Studienverzeichnis:

```bash
cd /root/labs/templates-study
```

Laden Sie ein Beispiel-Template herunter, um die Struktur zu verstehen:

```bash
curl -o sample-template.yaml https://raw.githubusercontent.com/backstage/software-templates/main/scaffolder-templates/react-ssr-template/template.yaml
```

<img width="1463" height="306" alt="image" src="https://github.com/user-attachments/assets/5d9efa88-7fb6-4042-b1f0-c896cf95e516" />


Untersuchen Sie die Template-Struktur:

```bash
cat sample-template.yaml
```
<img width="828" height="492" alt="image" src="https://github.com/user-attachments/assets/a33b97b9-6010-42c4-b5d5-b3dd9dfea918" />

Hier die ganze yaml file

```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: react-ssr-template
  title: React SSR Template
  description: Create a website powered with Next.js
  tags:
    - recommended
    - react
spec:
  owner: web@example.com
  type: website
  parameters:
    - title: Provide some simple information
      required:
        - component_id
        - owner
      properties:
        component_id:
          title: Name
          type: string
          description: Unique name of the component
          ui:field: EntityNamePicker
        description:
          title: Description
          type: string
          description: Help others understand what this website is for.
        owner:
          title: Owner
          type: string
          description: Owner of the component
          ui:field: OwnerPicker
          ui:options:
            allowedKinds:
              - Group
    - title: Choose a location
      required:
        - repoUrl
      properties:
        repoUrl:
          title: Repository Location
          type: string
          ui:field: RepoUrlPicker
          ui:options:
            allowedHosts:
              - github.com
  steps:
    - id: template
      name: Fetch Skeleton + Template
      action: fetch:template
      input:
        url: ./skeleton
        copyWithoutRender:
          - .github/workflows/*
        values:
          component_id: parameters.component_id
          description: "{{ parameters.description }}"
          destination: parameters.repoUrl
          owner: "{{ parameters.owner }}"
root@patrickaboudou-backstage-dev-wrf:~/labs/templates-study# 








---

### Schritt 4: Template-Komponenten identifizieren

Templates bestehen aus mehreren zentralen Bestandteilen:

* **Metadata**: Name, Beschreibung, Tags
* **Spec.parameters**: Formularfelder für Benutzereingaben
* **Spec.steps**: Aktionen, die während des Scaffoldings ausgeführt werden
* **Spec.output**: Links und Informationen, die nach der Erstellung angezeigt werden

---





