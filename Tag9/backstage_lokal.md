
Du hast aktuell **kein Node.js**, und `corepack` zeigt auf einen kaputten Pfad. Ohne sauberes Node/Yarn können wir Backstage nicht erstellen/starten.

## Schritt 1: Node 20 LTS + Yarn (über NVM + Corepack)

Führe diese Befehle nacheinander aus:

```bash
sudo apt update
sudo apt install -y curl ca-certificates
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 20
nvm use 20
nvm alias default 20
corepack enable
corepack prepare yarn@stable --activate
```

## Schritt 2: Prüfen, ob alles passt

Danach:

```bash
node -v
npm -v
corepack --version
yarn -v
```

<img width="1272" height="701" alt="image" src="https://github.com/user-attachments/assets/3ff355ad-8471-4d36-a0f0-bf530dd023fb" />

 Du bist in einer Umgebung, die **Windows-npm Pfade** im `PATH` hat (`C:\Users\patri\...`). Deshalb versucht `npm` auf Windows-Verzeichnisse zuzugreifen und knallt.

Bevor wir Node installieren, müssen wir **nvm in dieser Shell aktivieren** und sicherstellen, dass wir **Linux-Node/npm** benutzen (nicht Windows).

<img width="1495" height="726" alt="image" src="https://github.com/user-attachments/assets/772e756e-fe32-4401-9280-7ddb530113fa" />
## Schritt 1: nvm in der aktuellen Shell aktivieren

Führe aus:

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
```

## Schritt 2: Kurz prüfen, was gerade “npm/node” ist

```bash
which node || true
which npm || true
echo $PATH
```


## Schritt 3: Node 20 LTS über nvm installieren (erst danach!)

Sobald `nvm` aktiv ist, dann:

```bash
nvm install 20
nvm use 20
nvm alias default 20
```
<img width="1653" height="267" alt="image" src="https://github.com/user-attachments/assets/0080a027-f077-4b4f-aeda-cb0aa5da0ea1" />


<img width="846" height="227" alt="image" src="https://github.com/user-attachments/assets/7aae13f0-c628-48fa-a712-a67ef8d32992" />

## Schritt 4: Yarn via Corepack

```bash
corepack enable
corepack prepare yarn@stable --activate
```
<img width="990" height="158" alt="image" src="https://github.com/user-attachments/assets/2822e32f-6e6e-4bfd-abb3-8c45fcfb9ddd" />



<img width="861" height="278" alt="image" src="https://github.com/user-attachments/assets/afd4930e-903f-4aac-b171-dcaa5f23c63e" />

## Schritt 5: Finaler Check

```bash
node -v
npm -v
yarn -v
```

Wenn `which npm` bei Schritt 2 **irgendwas mit `/mnt/c/` oder `C:\...`** zeigt, fixen wir als nächstes deinen `PATH` sauber, damit Linux-Tools zuerst kommen.


<img width="1666" height="474" alt="image" src="https://github.com/user-attachments/assets/98984d84-8146-4ce4-a02e-096de0c324bf" />



Sehr gut, jetzt ist die Ursache eindeutig: Dein `PATH` enthält viele Windows-Pfade (`/mnt/c/...`) und deshalb wird **Windows-npm** verwendet:

* `which npm` → `/mnt/c/Program Files/nodejs//npm`

Damit Backstage sauber läuft, müssen wir sicherstellen, dass **Linux Node/NPM/Yarn (über nvm)** verwendet werden, nicht Windows.

## Schritt 1: Node 20 via nvm installieren (damit Linux-node vorhanden ist)

Führe aus:

```bash
nvm install 20
nvm use 20
nvm alias default 20
```

Danach prüfen:

```bash
which node
which npm
node -v
npm -v
```

Wichtig: `which node` und `which npm` müssen jetzt auf etwas wie `/home/koffi/.nvm/...` zeigen.

## Schritt 2: Corepack/Yarn aktivieren

```bash
corepack enable
corepack prepare yarn@stable --activate
yarn -v
```

## Schritt 3: Backstage App erstellen

Wenn Schritt 1–2 sauber sind:

```bash
mkdir -p ~/idp
cd ~/idp
npx @backstage/create-app@latest
```

<img width="1499" height="830" alt="image" src="https://github.com/user-attachments/assets/6c1b0075-addf-48a5-ab68-18899c4c1adf" />


Danach starten:

```bash
cd my-idp
yarn install
yarn dev
```

<img width="1607" height="738" alt="image" src="https://github.com/user-attachments/assets/4ebe90f2-66f0-407e-a06e-60ddc7a69a77" />

<img width="1535" height="1138" alt="image" src="https://github.com/user-attachments/assets/ecdd3207-35cf-4484-bdb3-a1776d1457a9" />


Backstage sollte dann typischerweise hier erreichbar sein:

* Frontend: `http://localhost:3000`
* Backend: `http://localhost:7007`

---

### Jetzt du

Führe bitte **nur Schritt 1** aus und poste die Ausgabe von:

```bash
which node
which npm
node -v
npm -v
```

Dann gehen wir direkt weiter (Yarn → Create App).



Danke — deine Toolchain (gcc/g++/python/build-essential) ist jetzt sauber. Dass `isolated-vm` **trotzdem** auf **Node v20.20.0** failt, passt leider zu einem bekannten Backstage-Problem rund um `isolated-vm` Builds auf Node 20. In einem aktuellen Backstage-Issue wird explizit beschrieben, dass der Wechsel auf **Node 22** das Problem löst. ([GitHub][1])

Auch die offizielle Backstage-Installationsseite sagt: **Node “Active LTS”** (und Yarn 4.4.1) verwenden. ([backstage.io][2])

## Nächster Schritt (professionell, zielgerichtet): Auf Node 22 wechseln und neu installieren

Im Projektordner:

```bash
nvm install 22
nvm use 22
node -v
```
<img width="1111" height="433" alt="image" src="https://github.com/user-attachments/assets/24529053-8cb4-43cd-a80e-7fa4298dd7ad" />

Dann sicherstellen, dass Yarn korrekt ist (Backstage nutzt 4.4.1):

```bash
corepack enable
yarn set version 4.4.1
yarn -v
```


<img width="1019" height="201" alt="image" src="https://github.com/user-attachments/assets/1c320a6c-bff4-4db4-ba26-55deb82c29e5" />

Jetzt Dependencies erneut:

```bash
cd ~/idp/koffitapp
yarn install
```
<img width="1656" height="794" alt="image" src="https://github.com/user-attachments/assets/c57308ae-989e-445e-87db-381cb6293f63" />

Wenn das durchläuft, starten wir direkt:

```bash
yarn start
```
<img width="1510" height="533" alt="image" src="https://github.com/user-attachments/assets/a728cd89-2d88-45bf-a30f-963f4325c283" />



## Was ich als Nächstes von dir brauche


Parfait 👍 on analyse calmement.

Ce que ça montre :

* ❌ `systemctl status backstage` → normal (tu n’as jamais créé de service systemd)
* ❌ `ss -ltnp` → rien écoute sur 3000 / 7007
* ❌ `curl` → rien ne répond

👉 Donc **Backstage n’est plus en train de tourner**.

Très probablement tu as :

* fermé le terminal
* ou stoppé le process (Ctrl+C)

---

# ✅ Pour le relancer

Retourne dans ton projet :

```bash
cd ~/idp/koffitapp
yarn start
```

Attends jusqu’à voir :

```
Rspack compiled successfully
Listening on :7007
```

Ensuite dans un **autre terminal** :

```bash
ss -ltnp | egrep ':(3000|7007)\b'
```

Tu devrais voir node écouter sur ces ports.

Puis :

```bash
curl -I http://localhost:3000
curl -I http://localhost:7007/healthcheck
```

---

# 🧠 Important : pourquoi systemctl ne marche pas ?

Tu es sous **WSL** :

```
Linux 5.15.167.4-microsoft-standard-WSL2
```

👉 WSL n’utilise pas systemd par défaut
👉 Backstage n’est pas installé comme service
👉 Il tourne juste comme process Node

---

# 🚀 Si tu veux le garder en arrière-plan

Option simple :

```bash
nohup yarn start &
```

Option propre (recommandée pour dev) :

```bash
sudo apt install tmux
tmux
yarn start
```

Puis tu peux détacher avec :

```
Ctrl+B puis D
```

---


Parfait ! Git a bien identifié tous les fichiers non suivis. On va maintenant les **ajouter et faire le premier commit** pour sécuriser ton état actuel.

---

### **1️⃣ Ajouter tous les fichiers à Git**

```bash
git add -A
```


<img width="972" height="366" alt="image" src="https://github.com/user-attachments/assets/17851b88-c6c5-45b5-be87-7d6fd45ff4ff" />


> `-A` ajoute **tous les fichiers** (nouveaux, modifiés et supprimés) à l’index pour le commit.

---

### **2️⃣ Faire le premier commit**

```bash
git commit -m "Initial commit - snapshot before securing project"
```



<img width="1182" height="1126" alt="image" src="https://github.com/user-attachments/assets/48645709-7c33-4847-a9b7-a676e6c0d56d" />



2️⃣ Ajouter tous les fichiers et faire le commit initial

Toujours dans ton projet :
cd ~/idp/koffitapp
git add -A
git commit -m "Initial commit - snapshot before securing project"


<img width="984" height="1290" alt="image" src="https://github.com/user-attachments/assets/a188b62b-4911-40b3-875d-dfaed65b6448" />


> ✅ Ton projet est maintenant versionné localement.

---

### **3️⃣ Renommer la branche principale en `main`**

Actuellement, tu es sur `master`. On va renommer :

```bash
git branch -M main
```

Vérifie :

```bash
git branch
```

<img width="1081" height="424" alt="image" src="https://github.com/user-attachments/assets/beee1b83-51d5-42be-9383-e0a040019645" />




Tu devrais voir :

```
* main
```

---

### **4️⃣ Ajouter le repository GitHub distant**

1. Crée un **nouveau repository sur GitHub** (privé si tu veux garder le code confidentiel) nommé `koffitapp`.
2. Note l’URL HTTPS du repo, par exemple :

```
https://github.com/TonNomUtilisateur/koffitapp.git
```

3. Connecte ton dépôt local à GitHub :

```bash
git remote add origin https://github.com/TonNomUtilisateur/koffitapp.git
```
<img width="1020" height="314" alt="image" src="https://github.com/user-attachments/assets/c779adb1-39af-4240-afe7-4a1e8115cbb2" />

Vérifie que tout est bien configuré :

```bash
git remote -v
```

---

### **5️⃣ Envoyer ton projet sur GitHub**

```bash
git push -u origin main
```

<img width="1251" height="630" alt="image" src="https://github.com/user-attachments/assets/db2cda03-6d1f-457f-a98e-a940471711a1" />

---

https://github.com/aboudou123/koffitapp














