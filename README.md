# 🎯 PROJET Réalité Augmentée — WebAR, Flask & PythonAnywhere

**Campus Saint-Aspais (Melun)** — Titre de niveau 7 « Expert en architecture et développement logiciel »

> **Le pitch.** En cinq ateliers progressifs, vous partez d'un dépôt GitHub vide et vous
> terminez avec une **application de réalité augmentée en ligne**, accessible depuis
> n'importe quel smartphone en scannant un QR code — sans installer une seule
> application. Tout tient dans une page web servie par Flask sur PythonAnywhere, et
> déployée automatiquement à chaque `git push`.

| | |
|---|---|
| **Module** | Réalité Augmentée / Réalité Virtuelle |
| **Année** | 2ᵉ année |
| **Modalité** | **Travail individuel** sur les cinq ateliers |
| **Public** | Étudiants développeurs (bases solides en POO et développement d'applications) |
| **Fil rouge** | 5 ateliers progressifs : **déployer → comprendre → prototyper → développer → livrer un MVP** |
| **Technologies** | Flask · Jinja2 · GitHub Actions · PythonAnywhere · `<model-viewer>` · A-Frame + AR.js · MindAR |
| **Environnement** | **GitHub Codespaces** — aucune installation sur la machine de l'étudiant |

---

## Architecture cible

```
     GitHub Codespaces               GitHub                        PythonAnywhere
┌────────────────────────┐    ┌────────────────────────┐    ┌──────────────────────────┐
│  VS Code (navigateur)  │    │  Repository (fork)     │    │  Flask (WSGI)            │
│                        │    │           │            │    │   ├── templates/  Jinja2 │
│  git commit            │───▶│           ▼            │    │   ├── static/css         │
│  git push              │    │  GitHub Actions        │    │   └── static/models/*.glb│
└────────────────────────┘    │  deploy-*.yml          │───▶│                          │
                              │  + 4 secrets           │API │  🔒 HTTPS  ← indispensable│
                              │  (PA_USERNAME, TOKEN,  │Fil.│  monuser.pythonanywhere  │
                              │   TARGET_DIR, DOMAIN)  │    └─────────────┬────────────┘
                              └────────────────────────┘                  │
                                                                          │ https://
                                                                          ▼
                                                            ┌──────────────────────────┐
                                                            │  📱 Smartphone            │
                                                            │  Navigateur + caméra     │
                                                            │  model-viewer · AR.js ·  │
                                                            │  MindAR                  │
                                                            └──────────────────────────┘
```

**Le point clé de cette architecture :** le navigateur n'autorise l'accès à la caméra que
dans un *contexte sécurisé* (HTTPS). PythonAnywhere fournit un certificat valide sur
`*.pythonanywhere.com`. **Sans la chaîne de déploiement de l'atelier 0, il n'y a pas d'AR
du tout.** L'industrialisation continue n'est donc pas un préambule administratif : c'est
une dépendance technique du projet.

---

## 1. Objectifs pédagogiques du module

À l'issue du module, l'étudiant est capable de :

- distinguer réalité augmentée (RA), réalité virtuelle (RV) et réalité mixte (RM), et situer une expérience sur le **continuum de Milgram** ;
- expliquer les grands principes de **suivi** (*tracking*) et de compréhension de scène : marqueur, image, détection de plans, ancrage, estimation de lumière ;
- **choisir un moteur et un SDK** adaptés à un besoin, et justifier ce choix par des contraintes mesurables (parc matériel, coût, compatibilité) ;
- **mettre en place une chaîne de déploiement continue** (Git → CI → hébergement) et l'exploiter pour livrer une application immersive ;
- concevoir, développer et livrer une **application AR selon une logique MVP** ;
- conduire un projet de bout en bout en respectant des jalons et une démarche de
  prototypage.

Ces objectifs couvrent les trois sections suivantes : **Introduction**,
**Développement d'applications** et **Cas pratiques / prototypage**.

## 2. Compétences visées (référentiel niveau 7)

- Analyser un besoin et le traduire en spécifications techniques pour une application immersive.
- Concevoir l'architecture d'une application AR en couches (rendu, suivi, métier/UI).
- Mettre en œuvre une **chaîne de build et de déploiement automatisée**, et gérer des secrets.
- Intégrer un SDK tiers et en maîtriser les limites matérielles.
- Diagnostiquer un incident en production à partir des journaux (*logs*).
- Documenter, versionner (Git) et présenter une réalisation logicielle.

## 3. Prérequis et matériel

**Prérequis** : programmation Python, notions de HTML/CSS/JavaScript, notions de repères 3D
(position, rotation, échelle), utilisation de Git.

**Matériel requis** — volontairement minimal :

| Besoin | Détail |
|---|---|
| **Un navigateur, rien de plus** | Tout le développement se fait dans **GitHub Codespaces** : l'éditeur, Python et le terminal tournent dans le nuage. Windows, macOS, Linux ou tablette. |
| Un compte GitHub | Gratuit. Inclut le quota Codespaces (voir ci-dessous). |
| Un compte PythonAnywhere | Gratuit (offre *Beginner*). |
| **Un smartphone quelconque** | Android **ou** iOS. **Aucune compatibilité ARCore / ARKit n'est requise.** |
| Une imprimante (atelier 3a) | Pour imprimer le marqueur fiduciaire sur une feuille A4. |

> 🎉 **Rien à installer, nulle part.** Ni Unity (20 Go), ni Android SDK, ni Xcode, ni
> compte développeur Apple, ni câble USB, ni mode développeur sur le téléphone — et pas
> même Python ou Git sur votre machine, puisque tout vit dans le codespace.
>
> ⚠️ **Quota Codespaces** : un compte GitHub personnel gratuit inclut **120 core-heures
> par mois** (soit environ 60 h sur une machine 2 cœurs) et 15 Go de stockage. C'est
> largement suffisant pour le module, à deux conditions : **arrêter son codespace** quand
> on a fini (*Codespaces → Stop codespace*), et savoir que ce quota gratuit **n'existe pas
> sur les comptes d'organisation** — travaillez depuis votre compte personnel.

## 4. Choix technologique — et sa justification

> Cette section est elle-même un **objet de cours** : elle illustre comment justifier une
> décision d'architecture par des contraintes, et non par préférence. Elle est reprise et
> discutée en atelier 1.

### 4.1 Pourquoi le **WebAR** plutôt qu'Unity + AR Foundation

Unity est la référence industrielle de la RA mobile, et il reste étudié en **annexe A**.
Mais pour ce module, avec un parc matériel hétérogène, le WebAR l'emporte sur cinq
critères :

| Critère | WebAR (retenu) | Unity + AR Foundation |
|---|---|---|
| Temps avant la première démo | ~10 min | ~2 h (installation, SDK, build) |
| Compatibilité du parc | **Tout smartphone** | Liste ARCore/ARKit à vérifier appareil par appareil |
| Distribution aux étudiants | Une **URL / un QR code** | APK à transférer, ou Xcode + compte Apple pour iOS |
| Versionnement Git | Fichiers texte + `.glb` de quelques Mo | Git LFS, `.gitignore` complexe, binaires lourds |
| Exploite la chaîne PythonAnywhere | **Oui, c'est la condition du HTTPS** | Non — aucun hébergeur web ne compile un APK |

### 4.2 Les trois bibliothèques retenues, et ce que chacune enseigne

Le choix n'est pas « une techno », mais **trois couches qui correspondent chacune à une
famille de suivi**. C'est cette correspondance qui structure la progression des
ateliers.

| Atelier | Bibliothèque | Famille de suivi | Ce que l'étudiant apprend |
|---|---|---|---|
| **2** | **`<model-viewer>`** (Google) | Détection de plan **déléguée à l'OS** | Afficher et poser un modèle `glTF` à l'échelle réelle, sans écrire de code AR |
| **3a** | **A-Frame + AR.js** | **Marqueurs fiduciaires** (Hiro, *barcode*) | Le graphe de scène 3D en HTML déclaratif ; la notion de marqueur et de pose |
| **3b** | **MindAR** | **Suivi d'image** (*image tracking*) | Compilation de cibles, robustesse du suivi, contraintes sur l'image source |

**Justification du panachage :**

1. **`<model-viewer>`** est le seul de ces trois outils qui déclenche la **vraie AR native**
   du téléphone (Scene Viewer sur Android, AR Quick Look sur iOS) : la qualité de suivi est
   excellente, et c'est universel. Sa limite — on délègue tout à l'OS, donc aucune logique
   applicative dans la scène AR — en fait un « Hello World » idéal et rien de plus.
2. **AR.js** et **MindAR** font de la **vision par ordinateur en JavaScript**. Ils
   fonctionnent donc sur *n'importe quel* navigateur avec une caméra, indépendamment
   d'ARCore/ARKit — c'est ce qui garantit le fonctionnement sur l'ensemble du parc. En
   contrepartie, la scène est entièrement sous votre contrôle : on peut y brancher l'API
   Flask, de l'UI, de la logique métier.
3. **A-Frame est déclaratif** : la scène 3D s'écrit en balises HTML. Les étudiants
   *voient* le graphe de scène, les repères et les transformations dans le DOM, là où un
   éditeur graphique les cacherait dans des panneaux.

### 4.3 Ce que nous ne faisons **pas** — et pourquoi

**WebXR (`navigator.xr` + `immersive-ar`)** serait la voie la plus « noble » : c'est le
standard W3C, il donne accès au *hit-test*, à la détection de plans et aux ancrages — les
équivalents web exacts des API d'AR Foundation.

**Nous l'écartons : Safari sur iOS n'implémente pas WebXR** (vérifié en 2026 ; le module
`immersive-ar` n'est même pas activé sur Vision Pro). Un atelier WebXR laisserait de côté
tous les iPhone de la promotion.

👉 **Conséquence pédagogique assumée :** le **SLAM et l'ancrage** ne font donc pas l'objet
d'un TP. Ils sont traités **théoriquement en atelier 1** et **démontrés par votre enseignant**
sur un appareil Android (ou via `<model-viewer>`, qui en fait usage sans l'exposer). Les
étudiants qui veulent les manipuler trouveront la voie WebXR et Unity en **annexe A**.

### 4.4 Adéquation à PythonAnywhere

| Contrainte PythonAnywhere | Impact sur ce projet |
|---|---|
| **HTTPS avec certificat valide** | ✅ **C'est l'atout décisif** : condition d'accès à la caméra. |
| Pas de mise en veille / pas de *cold start* | ✅ Le QR code projeté en salle répond instantanément. |
| 100 CPU-secondes / jour | ✅ Sans effet : **tout le calcul AR se fait dans le navigateur**. Le serveur ne rend que du HTML et du JSON. |
| Système de fichiers persistant (512 Mo) | ✅ Les modèles `.glb` déposés par les étudiants survivent aux déploiements. |
| Pas de WebSockets | ⚠️ Pas d'AR collaborative temps réel. Hors périmètre du module. |
| Requêtes sortantes limitées à une liste blanche | ⚠️ **Ne concerne que le code serveur.** Les CDN (A-Frame, MindAR) sont chargés **par le navigateur de l'étudiant**, donc jamais filtrés. |
| Pas d'ASGI sur l'offre gratuite | ⚠️ Flask (WSGI) convient ; FastAPI est à éviter. |
| Web app inactive désactivée après ~1 mois | ⚠️ **À annoncer dès l'atelier 0** : les projets s'éteignent après le module sans reconnexion. |

> Les conditions des offres gratuites évoluent vite : **à revérifier en début de module**.

## 5. Progression

| # | Atelier | Section du programme | Format |
|---|---------|--------------------|--------|
| **0** | **Mettre sa chaîne AR en ligne** | 2. Développement | TP pas-à-pas |
| **1** | **Comprendre l'AR** | 1. Introduction | Recherche guidée + restitution |
| **2** | **« Hello World » AR** | 2. Développement | TP pas-à-pas |
| **3** | **Suivi par marqueur et par image** | 2. Développement | TD encadré par jalons |
| **4** | **Projet : MVP AR** | 3. Cas pratiques | Projet + soutenance |

> L'atelier 0 installe l'infrastructure et doit être terminé avant tout le reste : il
> conditionne l'accès à la caméra. Les ateliers 1 et 2 posent le socle conceptuel et
> technique ; l'atelier 3 consolide par la pratique guidée ; l'atelier 4 met les acquis en
> autonomie. Selon le besoin, le projet de l'atelier 4 peut être décliné **en RA ou
> en RV**.

## 6. Arborescence du dépôt

Le dépôt que vous allez forker est un **squelette à trous** : il se déploie tel quel et la
page d'accueil fonctionne immédiatement, ce qui vous permet de **valider la chaîne avant
d'écrire une ligne d'AR**. Les scènes AR, elles, sont à compléter — ce sont les ateliers 2
et 3.

```
Atelier_AR/
├── README.md                      ← ce document
├── flask_app.py                   ← application Flask : routes, API, authentification
├── requirements.txt
├── data/
│   └── catalogue.json             ← catalogue des modèles 3D (données du projet)
├── templates/
│   ├── base.html                  ✅ gabarit commun (fourni)
│   ├── index.html                 ✅ accueil + liste du catalogue (fourni, sert de test)
│   ├── camera.html                ✅ diagnostic caméra — séquence 0.4 (fourni)
│   ├── erreur.html                ✅ page d'erreur 403 / 404 / 501 (fournie)
│   ├── viewer.html                🚧 ATELIER 2  — <model-viewer>
│   ├── marqueur.html              🚧 ATELIER 3a — A-Frame + AR.js
│   ├── image.html                 🚧 ATELIER 3b — MindAR
│   └── fiche.html                 🚧 ATELIER 3c — fiche modèle
├── static/
│   ├── css/style.css              ✅ feuille de style (fournie)
│   ├── models/                    📦 vos fichiers .glb et .usdz  (+ mode d'emploi)
│   ├── markers/                   📦 vos marqueurs .patt         (+ mode d'emploi)
│   └── targets/                   📦 vos cibles image .mind      (+ mode d'emploi)
├── .devcontainer/
│   └── devcontainer.json          ✅ configuration du codespace (fournie)
├── .gitignore                     ✅ fourni
└── .github/
    └── workflows/
        └── deploy-pythonanywhere.yml   ✅ déploiement automatique (fourni)
```

Légende — ✅ fourni et fonctionnel · 🚧 **à compléter par vous** (cherchez les `TODO`) ·
📦 répertoire d'assets à alimenter.

> 💡 Chacun des trois répertoires d'assets contient son **propre mode d'emploi**
> (`README.md`) : formats attendus, où trouver des modèles libres, comment choisir une
> bonne image cible, comment imprimer un marqueur.

### Carte des routes

| Route | État | Atelier |
|---|---|---|
| `GET /` | ✅ | catalogue — sert de test de déploiement (atelier 0) |
| `GET /camera` | ✅ | diagnostic caméra (séquence 0.4) |
| `GET /sante` | ✅ | diagnostic HTTPS et catalogue (séquence 0.4) |
| `GET /api/modeles` | ✅ | catalogue en JSON |
| `GET /api/modeles/<slug>` | ✅ | un modèle en JSON — consommé au **jalon J4** |
| `GET /viewer/<slug>` | 🚧 | **atelier 2** — `<model-viewer>` |
| `GET /marqueur` | 🚧 | **atelier 3, J1–J2** — AR.js (`?modele=<slug>` en option) |
| `GET /image/<slug>` | 🚧 | **atelier 3, J3–J4** — MindAR |
| `GET /fiche/<slug>` | 🚧 | **atelier 3, J5** — fiche du modèle |
| `GET /fiche_nom/` | 🚧 | **atelier 3** — exercices 3.1 et 3.2 (renvoie `501` tant qu'elle n'est pas écrite) |

---

# 🧩 Atelier 0 — Mettre sa chaîne AR en ligne

**Rattachement** : Section 2 « Développement d'applications » ·
**Modalité** : TP individuel pas-à-pas.

## Objectifs

- Mettre en place un dépôt Git personnel à partir d'un projet existant.
- Créer un hébergement web et le raccorder à une application Flask.
- **Automatiser le déploiement** par intégration continue et **gérer des secrets**.
- Vérifier le prérequis fondamental du module : **HTTPS + accès caméra**.

---

### 🧩 Séquence 0.1 — GitHub

**Objectif** : créer votre dépôt de travail · **Difficulté** : très facile

**Faites un *fork* de ce projet.** Si besoin, voici une vidéo d'accompagnement pour vous
aider à « forker » un dépôt GitHub : [Forker ce projet](https://youtu.be/p33-7XQ29zQ)

**Ouvrez ensuite votre fork dans un codespace.** Sur la page de *votre* fork (et non de
l'original) : bouton vert **`<> Code`** → onglet **Codespaces** → **Create codespace on
main**.

Au bout d'une minute, VS Code s'ouvre dans votre navigateur, avec le dépôt déjà cloné,
Python installé et les dépendances en place. Vous n'aurez **jamais besoin de `git clone`**
ni d'installer quoi que ce soit sur votre machine.

Pour vérifier que vous êtes au bon endroit, dans le terminal du codespace :

```bash
git remote -v      # doit afficher VOTRE compte, pas le dépôt d'origine
```

> 💡 **Retrouver son codespace.** Il persiste entre les séances : retournez sur
> <https://github.com/codespaces> ou refaites `<> Code → Codespaces`, et reprenez où vous
> en étiez. **Pensez à l'arrêter en fin de séance** pour économiser votre quota.

**Notion acquise** : *fork*, dépôt distant (*remote*), environnement de développement
infonuagique.

---

### 🧩 Séquence 0.2 — Création d'un site chez PythonAnywhere

**Objectif** : créer un hébergement · **Difficulté** : faible

1. Rendez-vous sur **<https://www.pythonanywhere.com/>** et créez un compte gratuit
   (offre *Beginner*).
2. Onglet **Web** → **Add a new web app** → **Flask** → version de Python **3.10 ou
   supérieure**.
3. Notez les deux informations que PythonAnywhere vous affiche, vous en aurez besoin en
   séquence 0.3 :
   - le **Source code** (ex. `/home/monuser/mysite`),
   - le **domaine** de votre site (ex. `monuser.pythonanywhere.com`).

4. **Raccordez le WSGI à notre application.** Dans l'onglet **Web** → section *Code* →
   cliquez sur le lien du **WSGI configuration file**. Assurez-vous qu'il importe bien
   `flask_app` :

   ```python
   import sys

   path = '/home/monuser/mysite'          # ⚠️ adaptez à votre chemin
   if path not in sys.path:
       sys.path.insert(0, path)

   from flask_app import app as application    # noqa
   ```

5. **Déclarez les fichiers statiques.** Toujours dans l'onglet **Web**, section
   *Static files*, ajoutez :

   | URL | Directory |
   |---|---|
   | `/static/` | `/home/monuser/mysite/static/` |

   > 💡 Pourquoi ? Vos modèles 3D (`.glb`) peuvent peser plusieurs mégaoctets. Ce mapping
   > les fait servir directement par le serveur web, **sans passer par Python** : c'est
   > beaucoup plus rapide et cela ne consomme aucun de vos 100 CPU-secondes quotidiens.

6. Cliquez sur **Reload**, puis ouvrez votre site. Vous devez voir la page d'accueil du
   squelette.

**Notion acquise** : serveur WSGI, séparation contenu dynamique / fichiers statiques.

---

### 🧩 Séquence 0.3 — Les Actions GitHub (industrialisation continue)

**Objectif** : automatiser la mise à jour de votre hébergement · **Difficulté** : moyenne

Dans le dépôt que vous venez de forker, vous avez un fichier **`deploy-pythonanywhere.yml`**
déposé dans le répertoire `.github/workflows`. Ce fichier a pour objectif d'automatiser le
déploiement de votre code sur votre site PythonAnywhere.

Pour information, c'est ce que l'on appelle des **Actions GitHub**. Ce sont des scripts qui
s'exécutent automatiquement lors de chaque *commit* dans votre projet (c'est-à-dire à chaque
modification de votre code). Ces scripts sont au format `yml`, un format structuré proche
d'XML.

Concrètement, notre Action fait deux choses :
1. elle **envoie tous vos fichiers** vers PythonAnywhere via son **API Files** ;
2. elle **recharge** ensuite la web app pour que le nouveau code soit pris en compte.

Pour utiliser cette Action, **vous avez besoin de créer des secrets dans GitHub** afin de ne
pas divulguer d'informations sensibles aux internautes de passage dans votre dépôt, comme
vos identifiants par exemple.

**Vous avez 4 secrets à créer** dans votre dépôt GitHub :
**Settings → Secrets and variables → Actions → New repository secret**

| Secret | Valeur | Où la trouver |
|---|---|---|
| **`PA_USERNAME`** | votre nom d'utilisateur PythonAnywhere | en haut à droite du tableau de bord |
| **`PA_TOKEN`** | votre jeton d'API | **Account → API Token** → *Create a new API token* |
| **`PA_TARGET_DIR`** | le répertoire du code source (ex. `/home/monuser/mysite`) | **Web → Source code** |
| **`PA_WEBAPP_DOMAIN`** | votre site (ex. `monuser.pythonanywhere.com`) | **Web**, en haut de page |

> ⚠️ **Compte européen ?** Si vous vous êtes inscrit sur `eu.pythonanywhere.com`, ajoutez
> un 5ᵉ secret **`PA_HOST`** valant `eu.pythonanywhere.com`. Sinon, ne le créez pas : le
> workflow utilise `www.pythonanywhere.com` par défaut.

**Dernière étape** : pour engager l'automatisation de votre première Action, vous devez
cliquer sur le gros bouton vert dans l'onglet supérieur **[Actions]** de votre dépôt GitHub.
Le bouton s'intitule **« I understand my workflows, go ahead and enable them »**.

**Testez la chaîne de bout en bout :**

```bash
git commit --allow-empty -m "Test de la chaîne de déploiement"
git push
```

Puis suivez l'exécution dans l'onglet **Actions**. Le *job* doit se terminer en vert.

**Notions acquises** : intégration/déploiement continus, gestion de secrets, jeton d'API,
lecture d'un journal de CI.

---

### 🗺️ Séquence 0.4 — Mise en service et validation

**Objectif** : prouver que la chaîne AR est opérationnelle · **Difficulté** : faible

Cette séquence est un **jalon bloquant** : tant que les quatre vérifications ci-dessous ne
passent pas, les ateliers 2 et 3 sont impossibles.

**✅ Vérification 1 — le site répond.**
Ouvrez `https://<votre-domaine>.pythonanywhere.com/`. La page d'accueil du squelette
s'affiche.

**✅ Vérification 2 — le déploiement continu fonctionne.**
Modifiez le titre dans `templates/index.html`, faites un `git push`, attendez la fin de
l'Action, rechargez la page. Votre modification est visible **sans aucune manipulation
manuelle**.

**✅ Vérification 3 — le HTTPS est bien actif.**
L'URL affiche un **cadenas** 🔒 et commence par `https://`. Vérifiez aussi la route de
diagnostic fournie :

```
https://<votre-domaine>.pythonanywhere.com/sante
```

Elle doit répondre `{"https": true, "ok": true, ...}`. **Si `https` vaut `false`, la caméra
sera refusée** : tapez bien l'URL en `https://`, jamais en `http://`.

**✅ Vérification 4 — la caméra est autorisée.**
Depuis votre **smartphone**, ouvrez `https://<votre-domaine>.pythonanywhere.com/camera`.
Le navigateur doit vous demander l'autorisation d'accéder à la caméra, et vous devez voir
le flux vidéo.

> 🧠 **À retenir — le piège n°1 du module.** `getUserMedia()` (l'API d'accès à la caméra)
> n'est disponible que dans un **contexte sécurisé** : HTTPS, ou `localhost`. Une page
> servie en `http://` renvoie `undefined` pour `navigator.mediaDevices`, et **toutes** les
> bibliothèques AR échouent sans message clair. C'est très précisément le service que vous
> rendent PythonAnywhere **et** l'URL `app.github.dev` de votre codespace — toutes deux
> en HTTPS.

**💡 Astuce** Générez un QR code de votre URL
(<https://api.qrserver.com/> ou l'extension de votre navigateur) et collez-le dans votre
`README`. Tester sur téléphone devient instantané, et c'est ce que vous projetterez en
soutenance.

## Livrable de l'atelier 0

- L'URL de votre site en ligne.
- Une capture de l'onglet **Actions** montrant un déploiement réussi.
- Une capture de `/sante` renvoyant `"https": true`.

## Critères de réussite

| Critère | Acquis si… |
|---|---|
| Dépôt personnel | le fork existe et un codespace est ouvert dessus |
| Hébergement | l'URL PythonAnywhere répond en HTTPS |
| Déploiement continu | un `git push` met le site à jour sans intervention |
| Secrets | aucun identifiant n'apparaît en clair dans le dépôt |
| Caméra | le flux vidéo s'affiche sur smartphone |

---

# 🧩 Atelier 1 — Comprendre l'AR

**Rattachement** : Section 1 « Introduction à la RA et RV » ·
**Modalité** : travail individuel, recherche guidée puis restitution orale courte.

## Objectifs

- Poser un vocabulaire commun et rigoureux (RA / RV / RM, continuum de Milgram).
- Comprendre *comment* la RA « tient » dans le réel : les familles de suivi et la
  compréhension de scène.
- Cartographier l'écosystème des moteurs et SDK, et savoir lequel répond à quel besoin.
- Relier la technologie à des cas d'usage réels (marketing, gaming, formation).

## Contenu à explorer

1. **Définitions et frontières** : RA vs RV vs RM ; continuum réel–virtuel de Milgram ;
   RA *marker-based* vs *markerless*.
2. **Le suivi (*tracking*)** — les quatre familles :
   - **marqueurs fiduciaires** (type *Hiro*, *barcode*, QR) → atelier 3a,
   - **suivi d'image** (*image tracking*, *NFT*) → atelier 3b,
   - **détection de plans et SLAM** (*Simultaneous Localization and Mapping*) → théorie
     seulement (voir encadré ci-dessous),
   - **suivi géolocalisé** (AR « outdoor », boussole + GPS).
3. **Compréhension de scène** : ancrage (*anchors*), estimation de lumière, occlusion.
4. **Écosystème technique** : ARCore (Google), ARKit (Apple), AR Foundation (Unity),
   Vuforia, **WebXR**, **AR.js**, **MindAR**, **`<model-viewer>`**, 8th Wall, Unreal Engine.
5. **Cas d'usage** : un exemple documenté par catégorie (marketing, gaming, formation), avec
   bénéfice métier et limite technique associée.


## Travail demandé

Chaque étudiant produit une **fiche de synthèse (2 pages)** et un **mini-exposé (5 min)**
couvrant :

- un tableau comparatif **RA / RV / RM** ;
- un **schéma du principe de suivi** choisi (marqueur, image ou SLAM) ;
- une comparaison de **2 SDK** (forces, limites, plateformes, coût de licence) ;
- l'analyse d'**une application AR existante** du marché : cas d'usage, techno probable, ce
  qui marche / ce qui pêche ;
- 🆕 une **critique argumentée du choix technologique de ce module** (§ 4) : sur quel
  critère le WebAR est-il le bon choix ici ? Dans quel contexte professionnel
  recommanderiez-vous Unity à la place ? **Citez au moins une contrainte chiffrée.**

## Livrable

`atelier1-synthese-<nom>.pdf` (ou `.md`), déposé dans votre dépôt, + support de restitution.

## Grille d'évaluation (sur 20)

| Critère | Points |
|---|---|
| Exactitude des concepts (RA/RV/RM, familles de suivi) | 5 |
| Pertinence de la comparaison de SDK | 4 |
| Qualité de l'analyse de cas d'usage | 4 |
| **Critique argumentée du choix techno du module** | 3 |
| Clarté de la restitution orale et du support | 4 |

---

# 🧩 Atelier 2 — « Hello World » AR

**Rattachement** : Section 2 « Développement d'applications » ·
**Modalité** : TP individuel guidé pas-à-pas.

> ⛔ **Prérequis bloquant** : les 4 vérifications de la séquence 0.4 doivent passer.

## Technologie : `<model-viewer>`

`<model-viewer>` est un **composant web** de Google. Vous l'utilisez comme une balise HTML
ordinaire, et il vous offre gratuitement : un visualiseur 3D avec rotation à la souris ou au
doigt, et un **bouton « Voir en AR »** qui bascule vers la vraie AR native du téléphone
(*Scene Viewer* sur Android, *AR Quick Look* sur iOS).

C'est le meilleur rapport résultat/effort de tout le module : **zéro ligne de code AR** pour
un suivi de qualité professionnelle.

## Objectif du TP

Afficher un **modèle 3D** que l'on peut inspecter dans la page, puis **poser à l'échelle
réelle sur une surface** de la salle, depuis son propre smartphone.

## Déroulé pas-à-pas

### Étape 1 — Récupérer un modèle 3D

Téléchargez un modèle au format **`.glb`** (glTF binaire) — par exemple depuis
[Khronos glTF Sample Assets](https://github.com/KhronosGroup/glTF-Sample-Assets),
[Sketchfab](https://sketchfab.com/) (filtre *Downloadable*) ou
[Poly Pizza](https://poly.pizza/).

Déposez-le dans **`static/models/`**.

> ⚠️ **Contrainte à respecter : visez moins de 5 Mo par modèle.** Vous disposez de 512 Mo
> sur PythonAnywhere, et surtout vos camarades téléchargeront ce fichier sur le réseau de la
> salle. Compressez avec [gltf-transform](https://gltf-transform.dev/) ou
> [gltfpack](https://meshoptimizer.org/gltf/) si nécessaire.

### Étape 2 — Déclarer le modèle dans le catalogue

Ouvrez **`data/catalogue.json`** et complétez une entrée. C'est ce fichier que Flask lit
pour alimenter toutes les pages du projet.

### Étape 3 — Compléter le gabarit `templates/viewer.html`

Le fichier contient les `TODO` à remplir. Les points essentiels :

```html
<script type="module"
        src="https://cdn.jsdelivr.net/npm/@google/model-viewer@4.0.0/dist/model-viewer.min.js">
</script>

<model-viewer
    src="{{ url_for('static', filename='models/' ~ modele.glb) }}"
    ios-src="{{ url_for('static', filename='models/' ~ modele.usdz) }}"
    alt="{{ modele.nom }}"
    ar
    ar-modes="scene-viewer quick-look webxr"
    ar-scale="auto"
    camera-controls
    shadow-intensity="1">
  <button slot="ar-button">👁️ Voir dans ma pièce</button>
</model-viewer>
```

> 💡 **`poster` n'est pas dans la liste ci-dessus, et c'est voulu** : c'est
> l'objet de l'exercice 2.3. Attention, le catalogue livre `"poster": null` —
> un attribut rendu inconditionnellement afficherait `/static/models/None`.
> À vous de ne le produire que lorsque la valeur existe.

> 📌 **Le détail qui fait échouer la moitié de la classe : iOS.** Android lit le `.glb`,
> mais **AR Quick Look sur iPhone exige un fichier `.usdz`**. Convertissez votre modèle
> ([Reality Converter](https://developer.apple.com/augmented-reality/tools/) sur macOS, ou
> un convertisseur glTF→USDZ en ligne) et renseignez l'attribut `ios-src`. Sans lui, le
> bouton AR ne s'affiche tout simplement pas sur iPhone.

### Étape 4 — Déployer et tester

```bash
git add static/models/ data/catalogue.json templates/viewer.html
git commit -m "Atelier 2 : premier modèle en AR"
git push
```

Attendez le vert dans **Actions**, puis ouvrez la page depuis votre smartphone et appuyez
sur **« Voir dans ma pièce »**.

### Étape 5 — Régler l'échelle

Appuyez sur **« Voir dans ma pièce »**, balayez lentement le sol avec la caméra jusqu'à ce
que la surface soit détectée, puis touchez l'écran pour y déposer le modèle. Reculez de
quelques pas et jugez sa taille par rapport au mobilier réel.

C'est la seule façon de vérifier une échelle : dans le visualiseur de la page,
`<model-viewer>` recadre automatiquement le modèle, donc un objet de 1 cm et un objet de
100 m s'affichent identiquement. Une fois l'objet ancré dans la pièce, il n'y a plus de
recadrage : **les unités glTF sont des mètres**. Un modèle exporté en centimètres apparaît
donc 100 fois trop grand.

Corrigez via l'attribut `scale` de `<model-viewer>`, ou mieux, en réexportant le modèle aux
bonnes unités.

## Exercices

**Exercice 2.1 — Galerie.** Faites en sorte que la page d'accueil liste **tous** les modèles
du catalogue avec une vignette, chacune renvoyant vers sa page `/viewer/<slug>`.

**Exercice 2.2 — Diagnostic.** Ajoutez dans la page un message qui prévient l'utilisateur
lorsque l'AR n'est **pas** disponible sur son appareil (écoutez l'événement
`ar-status` de `<model-viewer>`), au lieu de le laisser devant un bouton inopérant.

**Exercice 2.3 — Poster.** Ajoutez une image `poster` pour éviter la zone blanche pendant le
chargement du modèle, et mesurez le gain perçu sur le réseau de la salle.

## Livrable

- Le code poussé sur votre dépôt et **déployé**.
- Une **capture vidéo (30 s)** montrant le modèle posé sur une surface réelle, filmée depuis
  votre téléphone.
- Un paragraphe dans votre `README` : modèle utilisé, appareil de test, difficultés
  rencontrées.

## Critères de réussite

Un modèle 3D **reste posé et stable** sur une surface réelle quand on déplace le téléphone,
et il est à **l'échelle plausible**. La page fonctionne sur **au moins un appareil Android
et un appareil iOS** (empruntez un appareil à un camarade si vous n'avez pas les deux).

---

# 🧩 Atelier 3 — Suivi par marqueur et par image

**Rattachement** : Section 2 « Développement d'applications » ·
**Modalité** : TD encadré individuel, jalons avec points de contrôle par l'enseignant.

## Principe

En atelier 2, l'OS faisait tout le travail et vous n'aviez aucun contrôle sur la scène AR.
Ici, **vous reprenez la main** : le suivi tourne dans le navigateur, la scène 3D est à vous,
et vous pouvez y brancher votre application Flask.

On transforme le « Hello World » en une **visionneuse de catalogue augmentée** :
un marqueur ou une image imprimée déclenche l'apparition d'un modèle 3D accompagné de sa
fiche d'information.

## Jalons

| Jalon | Fonctionnalité | Fichier | Compétence travaillée |
|---|---|---|---|
| **J1** | Afficher un cube sur le **marqueur Hiro** | `marqueur.html` | Graphe de scène A-Frame, notion de pose |
| **J2** | Remplacer le cube par un **modèle glTF** du catalogue, bien orienté et à l'échelle | `marqueur.html` | Repères 3D, transformations |
| **J3** | **Suivi d'image** : une image de votre choix déclenche le modèle | `image.html` | Compilation de cibles MindAR, robustesse |
| **J4** | **API + UI** : la scène lit `/api/modeles` et affiche un panneau d'info | `image.html`, `flask_app.py` | Séparation couche AR / métier, `fetch` |
| **J5** | **Route protégée** `/fiche/<slug>` (voir exercices) | `flask_app.py`, `fiche.html` | Authentification, contrôle d'accès |
| **J6** *(bonus)* | Plusieurs cibles, animations, ou son | libre | API avancée |

---

### 🔹 Jalon J1–J2 — Suivi par marqueur avec A-Frame + AR.js

**Imprimez le marqueur Hiro** (une feuille A4 suffit, motif noir sur fond blanc, marge
blanche conservée) :
<https://raw.githubusercontent.com/AR-js-org/AR.js/master/data/images/hiro.png>

Complétez `templates/marqueur.html`. Structure de référence :

```html
<script src="https://aframe.io/releases/1.6.0/aframe.min.js"></script>
<script src="https://cdn.jsdelivr.net/gh/AR-js-org/AR.js/aframe/build/aframe-ar.js"></script>

<a-scene embedded arjs="sourceType: webcam; debugUIEnabled: false;"
         vr-mode-ui="enabled: false">

  <a-marker preset="hiro">
    <!-- Tout ce qui est ici n'existe que si le marqueur est vu -->
    <a-box position="0 0.5 0" material="color: #4A90D9;"></a-box>
  </a-marker>

  <!-- Caméra obligatoire, et toujours en dehors du marqueur -->
  <a-entity camera></a-entity>
</a-scene>
```

**À comprendre — et c'est le cœur du jalon :** tout ce qui est enfant de `<a-marker>` vit
dans le **repère du marqueur**. `position="0 0.5 0"` signifie « 50 cm au-dessus du centre du
marqueur » — pas au-dessus de l'écran. C'est exactement la notion de *pose* vue en atelier 1.

> 🧠 **Repère A-Frame** : `X` vers la droite, `Y` vers le haut, `Z` vers vous. Unité = le
> **mètre**. Un marqueur Hiro imprimé sur A4 mesure ~10 cm : un modèle de `1` d'arête
> paraîtra donc énorme. C'est la source de confusion n°1 du jalon J2.

---

### 🔹 Jalon J3–J4 — Suivi d'image avec MindAR

Le marqueur Hiro est laid et peu « produit ». MindAR suit **n'importe quelle image** :
affiche, couverture de livre, planche de cours, étiquette.

**a) Choisissez une image source.** Une bonne cible est **riche en détails non répétitifs et
contrastés**. Une image trop uniforme, floue, symétrique ou très géométrique ne se suit pas.

**b) Compilez-la** avec le
[compilateur de cibles MindAR](https://hiukim.github.io/mind-ar-js-doc/tools/compile) :
vous obtenez un fichier **`.mind`** à déposer dans `static/targets/`.

**c) Complétez `templates/image.html`.** Structure de référence :

```html
<script src="https://aframe.io/releases/1.6.0/aframe.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-image-aframe.prod.js"></script>

<a-scene mindar-image="imageTargetSrc: {{ url_for('static', filename='targets/cibles.mind') }};"
         embedded color-space="sRGB"
         renderer="colorManagement: true, physicallyCorrectLights"
         vr-mode-ui="enabled: false" device-orientation-permission-ui="enabled: false">

  <a-assets>
    <a-asset-item id="modele"
                  src="{{ url_for('static', filename='models/' ~ modele.glb) }}"></a-asset-item>
  </a-assets>

  <a-camera position="0 0 0" look-controls="enabled: false"></a-camera>

  <a-entity mindar-image-target="targetIndex: 0">
    <a-gltf-model rotation="0 0 0" position="0 0 0" scale="0.5 0.5 0.5"
                  src="#modele" animation-mixer></a-gltf-model>
  </a-entity>
</a-scene>
```

**d) Jalon J4 — branchez votre API.** La scène doit récupérer les métadonnées du modèle
auprès de Flask et les afficher dans un panneau HTML superposé :

```js
const reponse = await fetch("/api/modeles/" + slug);
const modele  = await reponse.json();
document.querySelector("#panneau-info").textContent = modele.description;
```

C'est ici que se joue l'attendu d'architecture du niveau 7 : **la scène AR ne connaît pas
les données**, elle les demande. Le catalogue peut changer côté serveur sans toucher à la
scène.

---

### 🔹 Jalon J5 — Exercices : nouvelle fonctionnalité et protection

**Exercice 3.1 — Création d'une nouvelle fonctionnalité.**
Créez une nouvelle route dans votre application afin de faire une **recherche sur le nom
d'un modèle** du catalogue. Cette fonctionnalité sera accessible via la route
**`/fiche_nom/`** et renverra la fiche du modèle correspondant (ou un message clair si
aucun ne correspond).

**Exercice 3.2 — Protection.**
Cette nouvelle route est soumise à un **contrôle d'accès *User***, c'est-à-dire distinct des
identifiants administrateur. Pour accéder à cette fonctionnalité, l'utilisateur sera
authentifié sous les identifiants suivants : **`user` / `12345`**.

Le squelette fournit déjà une authentification HTTP Basic fonctionnelle dans `flask_app.py`
et un décorateur `@authentification_requise(...)`. **Votre travail** : introduire la
**distinction des rôles** (`user` vs `admin`), protéger `/fiche_nom/` au niveau `user`, et
réserver une route d'administration au niveau `admin`.

> 🔐 **Avertissement professionnel.** Des identifiants en clair dans le code sont
> **inacceptables en production** — c'est une facilité d'atelier, et le code vous le signale.
> Pour aller plus loin : variables d'environnement (voir le fichier WSGI de PythonAnywhere),
> hachage des mots de passe (`werkzeug.security`), et jamais de secret dans Git. **Faites le
> lien avec les 4 secrets GitHub de la séquence 0.3 : c'est exactement le même problème.**

## Consignes de conception

- Séparer clairement la **couche AR** (scène, suivi), la **couche présentation** (gabarits,
  CSS, panneaux d'UI) et la **logique métier** (catalogue, API, authentification) — attendu
  d'architecture au niveau 7.
- Gérer les **cas d'erreur** : permission caméra refusée, cible non détectée, modèle
  introuvable (404), navigateur non compatible.
- Versionner par petits *commits* signifiants et en français ou en anglais, mais de façon
  cohérente.

## Déroulé indicatif

1. **Cadrage** : rappel de l'architecture cible, prise en main des assets.
2. **Jalons J1–J2** : marqueur, modèle, échelle.
3. **Jalon J3** : suivi d'image MindAR, compilation de la cible.
4. **Jalon J4** : API et UI, puis gestion des cas d'erreur.
5. **Jalon J5** : exercices 3.1 et 3.2.
6. **Clôture** : bonus J6, nettoyage du dépôt, démonstration individuelle à l'enseignant.

## Livrable

- Dépôt Git de l'application **déployée et fonctionnelle** (README à jour, captures).
- Le **marqueur et l'image cible imprimés**, apportés en séance.
- Démonstration en séance des jalons J1 à J5.

## Grille d'évaluation (sur 20)

| Critère | Points |
|---|---|
| Jalons J1–J3 opérationnels (marqueur et image) | 7 |
| Jalon J4 : consommation de l'API, séparation des couches | 4 |
| Jalon J5 : authentification et gestion des rôles | 3 |
| Robustesse et gestion des cas d'erreur | 3 |
| Qualité du dépôt Git et de la documentation | 3 |
| Bonus J6 | +2 |

---

# 🧩 Atelier 4 — Projet : application AR (MVP individuel)

**Rattachement** : Section 3 « Cas pratiques : prototypage » ·
**Modalité** : projet individuel, **logique MVP**, clôturé par une soutenance-démo.

## Énoncé du sujet

> **Concevoir, développer et livrer le MVP d'une application de réalité augmentée**
> répondant à un besoin réel dans l'un des domaines suivants (**marketing, gaming,
> formation, éducation**). Vous appliquez une démarche projet complète : cadrage du
> besoin, périmètre MVP, développement itératif, démonstration.
>
> **Contrainte de livraison : l'application doit être en ligne**, accessible par URL
> publique et démontrable depuis le téléphone de l'enseignant.

Le projet peut être **décliné en RV** (WebXR sur casque) si vous le justifiez — mais la
contrainte de démonstration reste la même.

> ⚠️ **Le périmètre est calibré pour une personne seule.** Une seule *user story*, un seul
> mode de suivi (marqueur **ou** image, pas les deux), deux ou trois modèles 3D au
> maximum. Un MVP modeste et qui fonctionne vaut bien mieux qu'une ambition inachevée :
> c'est exactement ce que la grille d'évaluation récompense.

## Démarche imposée (approche MVP)

1. **Cadrage** : problème adressé, utilisateur cible, proposition de valeur, une *user story*
   principale.
2. **Périmètre MVP** : lister les fonctionnalités *must / should / could*, **ne développer
   que le *must***.
3. **Architecture** : schéma des couches (AR, UI, données) et inventaire des assets
   nécessaires.
4. **Développement itératif** : au moins deux itérations, chacune **déployée** et démontrée.
5. **Livraison & démo** : URL publique + soutenance.

## Banque de sujets proposés

Tous ces sujets sont réalisables avec la pile du module (marqueur ou suivi d'image).

| Domaine | Sujet | Cœur fonctionnel MVP | Techno la plus adaptée |
|---|---|---|---|
| Marketing | **Aperçu produit** (meuble, sneaker, objet déco chez soi) | Poser un modèle à l'échelle réelle, changer de variante | `<model-viewer>` |
| Éducation | **Manuel augmenté** : une image du cours déclenche une animation 3D | Suivi d'image → modèle animé + légende | MindAR |
| Formation | **Guide de maintenance AR** : étapes superposées sur un équipement | Suivi d'image + surcouche d'instructions séquencées | MindAR + API Flask |
| Gaming | **Chasse au trésor** dans l'établissement | Plusieurs marqueurs disséminés, score côté serveur | AR.js + Flask |
| Culture | **Visite augmentée** : un socle → l'œuvre en 3D + sa fiche | Marqueur → modèle + panneau d'info | AR.js ou MindAR |
| Sciences | **Explorateur 3D** (organe, molécule, système solaire) | Modèle manipulable + annotations interactives | `<model-viewer>` + API |
| Marketing | **Carte de visite augmentée** | Suivi d'image sur la carte → avatar + liens | MindAR |

> Vous pouvez proposer votre **propre sujet**, sous validation de l'enseignant
> (faisabilité pour une personne seule + périmètre MVP réaliste).

## Livrables attendus

- **Dépôt Git** du projet (code, `README`, instructions de déploiement).
- **Application en ligne et fonctionnelle**, avec son **QR code** dans le `README`.
- **Dossier projet court (2–3 pages)** : besoin, périmètre MVP, architecture, choix
  techniques, **limites et perspectives**.
- **Soutenance (10 min + démo live)**.

## Jalons de suivi

| Jalon | Attendu |
|---|---|
| **J0** (démarrage) | Sujet validé, *user story* principale, périmètre MVP arbitré |
| **J1** (mi-parcours) | Cœur fonctionnel **déployé** et démontrable (itération 1) |
| **J2** (fin) | MVP livré en ligne, dossier rendu, soutenance |

## Grille d'évaluation (sur 20)

| Critère | Points |
|---|---|
| Pertinence du besoin et cadrage MVP | 4 |
| Fonctionnalité et stabilité du MVP livré | 5 |
| Qualité technique (architecture, code, intégration de la bibliothèque AR) | 4 |
| **Chaîne de déploiement opérationnelle et application réellement en ligne** | 2 |
| Dossier projet et documentation | 2 |
| Soutenance et démonstration | 3 |

---

# 🧠 Troubleshooting

## Lire ses logs — le premier réflexe

Lors de vos développements, vous serez peut-être confronté à des erreurs système : erreurs
de syntaxe, mauvaises déclarations de fonctions, appels à des modules inexistants, secrets
mal renseignés, etc. Les causes d'erreurs sont quasi illimitées. **Vous devez donc vous
tourner vers les logs de votre système pour comprendre d'où vient le problème.**

Vos logs sont accessibles depuis l'onglet **Web** de PythonAnywhere, ou directement :

| Log | Fichier | À quoi il sert |
|---|---|---|
| **Access log** | `{site}.pythonanywhere.com.access.log` | Qui a demandé quoi, et avec quel code HTTP |
| **Error log** | `{site}.pythonanywhere.com.error.log` | 🎯 **Vos traces Python** — commencez toujours ici |
| **Server log** | `{site}.pythonanywhere.com.server.log` | Démarrage et rechargement du serveur |

Et côté client, le second réflexe : la **console JavaScript** du navigateur. Sur Android,
branchez le téléphone et ouvrez `chrome://inspect` sur votre poste. Sur iOS, activez
*Réglages → Safari → Avancé → Inspecteur web* puis utilisez Safari sur macOS.

## Symptômes fréquents

| Symptôme | Cause la plus probable | Solution |
|---|---|---|
| **L'Action GitHub échoue à l'étape *Validate secrets*** | un des 4 secrets est absent ou mal orthographié | vérifiez les noms **exactement** (majuscules comprises) dans *Settings → Secrets* |
| **L'Action échoue avec `401 Unauthorized`** | `PA_TOKEN` invalide ou expiré | régénérez le jeton (*Account → API Token*) et mettez le secret à jour |
| **L'Action échoue avec `404 Not Found` au *reload*** | `PA_WEBAPP_DOMAIN` incorrect, ou compte européen | le domaine **sans** `https://` ; si compte EU, ajoutez le secret `PA_HOST` = `eu.pythonanywhere.com` |
| **L'Action est verte mais le site n'a pas changé** | `PA_TARGET_DIR` ne pointe pas vers le répertoire de la web app | comparez avec **Web → Source code** |
| **Erreur 502 / « Something went wrong »** | exception Python au démarrage | lisez l'**error log**, puis **Reload** |
| **Page blanche, aucun log d'erreur** | le WSGI n'importe pas la bonne application | dans le fichier WSGI : `from flask_app import app as application` |
| **Le CSS ou les modèles ne se chargent pas (404)** | mapping des fichiers statiques absent | ajoutez le mapping `/static/` (séquence 0.2, étape 5) |
| **🎥 « Permission caméra refusée » ou rien ne s'affiche** | page servie en `http://`, ou permission bloquée | **forcez `https://`** ; puis réinitialisez l'autorisation dans les réglages du navigateur |
| **`navigator.mediaDevices` est `undefined`** | contexte non sécurisé | idem : HTTPS obligatoire. Vérifiez `/sante` |
| **Le bouton AR n'apparaît pas sur iPhone** | attribut `ios-src` (`.usdz`) manquant | convertissez le modèle en USDZ (atelier 2, étape 3) |
| **Le modèle est invisible, ou gigantesque** | unités ou échelle | les unités glTF sont en **mètres** ; ajustez `scale` |
| **Le marqueur Hiro n'est pas détecté** | impression trop petite, reflets, marge blanche rognée | imprimez plus grand, évitez l'éclairage direct, **conservez la bordure blanche** |
| **La cible MindAR décroche sans arrêt** | image source pauvre en détails ou répétitive | changez d'image : contrastée, détaillée, non symétrique |
| **Le site répondait, puis plus rien (après quelques semaines)** | web app gratuite désactivée pour inactivité | reconnectez-vous à PythonAnywhere et cliquez sur **Run until 3 months from today** |

## Tester dans son codespace avant de pousser

Inutile d'attendre un déploiement de 20 secondes pour voir une faute de frappe. Dans le
terminal du codespace :

```bash
pip install -r requirements.txt   # déjà fait au premier démarrage
python flask_app.py
```

Codespaces détecte le port 5000 et propose de l'ouvrir : cliquez sur **Open in Browser**,
ou allez dans l'onglet **PORTS** du terminal. L'URL a la forme
`https://<votre-codespace>-5000.app.github.dev`.

**Bonne nouvelle : cette URL est en HTTPS**, donc c'est un contexte sécurisé — la caméra
fonctionne, et vous pouvez tester vos scènes AR sans passer par PythonAnywhere.

> 🔑 **Pour tester depuis votre téléphone**, il faut rendre le port accessible : onglet
> **PORTS** → clic droit sur le port 5000 → **Port Visibility** → **Public**. Sans cela,
> l'URL exige une authentification GitHub et votre téléphone n'affichera qu'une page de
> connexion. Repassez le port en **Private** quand vous avez fini.

Arrêtez le serveur avec `Ctrl+C`. **PythonAnywhere reste le livrable** : c'est l'URL
stable que vous présenterez, et elle ne dépend pas d'un codespace allumé.

---

# 📎 Annexe A — Aller plus loin : Unity, AR Foundation et WebXR

Le module écarte volontairement ces technologies (cf. § 4.3), mais tout expert en
développement logiciel doit savoir qu'elles existent et ce qu'elles apportent.

## A.1 — WebXR : le SLAM dans le navigateur

L'API `navigator.xr` donne accès en JavaScript à la détection de plans, au *hit-test* et aux
ancrages. Elle fonctionne sur **Chrome Android** mais **pas sur Safari iOS**.

```js
// Vérifier la disponibilité avant toute chose
if (navigator.xr && await navigator.xr.isSessionSupported("immersive-ar")) {
  const session = await navigator.xr.requestSession("immersive-ar", {
    requiredFeatures: ["hit-test", "anchors"],
  });
  // … puis three.js ou Babylon.js pour le rendu
}
```

À explorer : [three.js + WebXR](https://threejs.org/examples/?q=webxr),
[Babylon.js WebXR](https://doc.babylonjs.com/features/featuresDeepDive/webXR).

## A.2 — Unity + AR Foundation : la référence industrielle

**Unity 6 LTS** avec le paquet **AR Foundation**, plus les plugins *ARCore XR Plugin*
(Android) et *ARKit XR Plugin* (iOS), constitue le standard de fait de la RA mobile. Un seul
code, deux cibles natives.

La scène minimale repose sur un **XR Origin (AR)** portant les composants **AR Session**,
**AR Plane Manager** et **AR Raycast Manager**. Le placement d'un objet au *tap* s'écrit
ainsi :

```csharp
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARSubsystems;

[RequireComponent(typeof(ARRaycastManager))]
public class PlaceOnPlane : MonoBehaviour
{
    [SerializeField] private GameObject prefabAPlacer;   // ex. un cube

    private ARRaycastManager raycastManager;
    private static readonly List<ARRaycastHit> hits = new();

    void Awake() => raycastManager = GetComponent<ARRaycastManager>();

    void Update()
    {
        if (Input.touchCount == 0) return;
        var touche = Input.GetTouch(0);
        if (touche.phase != TouchPhase.Began) return;

        if (raycastManager.Raycast(touche.position, hits, TrackableType.PlaneWithinPolygon))
        {
            Pose pose = hits[0].pose;                 // position + orientation du point visé
            Instantiate(prefabAPlacer, pose.position, pose.rotation);
        }
    }
}
```

**Comparez ce code au jalon J1 de l'atelier 3** : dans les deux cas, on obtient une **pose**
(position + orientation) et on y instancie un objet. Seule la **source** de la pose change —
un marqueur détecté par vision ici, un plan issu du SLAM là. C'est le même concept, et c'est
tout l'intérêt de l'avoir vu en WebAR d'abord.

> ⚠️ **Ce que PythonAnywhere ne fera jamais** : compiler un APK Unity. Si vous partez sur
> cette voie, la chaîne de déploiement change complètement (GitHub Actions avec licence
> Unity, ou build local puis distribution par store / TestFlight).

## A.3 — Autres pistes

- **8th Wall** (Niantic) : WebAR commercial avec SLAM propriétaire **fonctionnant sur iOS**.
  Payant, mais c'est la réponse industrielle au trou laissé par Safari.
- **Vuforia** : suivi d'objet et de cible très robuste, licence commerciale.
- **Unreal Engine** : moteur AAA, surdimensionné pour un premier projet AR.

---

# 📚 Annexe B — Ressources et bibliographie

## Les bibliothèques du module

- **`<model-viewer>`** — [documentation et exemples](https://modelviewer.dev/) ·
  [générateur d'attributs](https://modelviewer.dev/editor/)
- **A-Frame** — [documentation](https://aframe.io/docs/) ·
  [école A-Frame](https://aframe.io/aframe-school/)
- **AR.js** — [documentation](https://ar-js-org.github.io/AR.js-Docs/) ·
  [dépôt](https://github.com/AR-js-org/AR.js) ·
  [marqueur Hiro](https://raw.githubusercontent.com/AR-js-org/AR.js/master/data/images/hiro.png)
- **MindAR** — [documentation](https://hiukim.github.io/mind-ar-js-doc/) ·
  [compilateur de cibles](https://hiukim.github.io/mind-ar-js-doc/tools/compile) ·
  [dépôt](https://github.com/hiukim/mind-ar-js)

## Modèles 3D libres

- [Khronos glTF Sample Assets](https://github.com/KhronosGroup/glTF-Sample-Assets) — les modèles de référence du format
- [Poly Pizza](https://poly.pizza/) — modèles *low-poly* légers, idéaux pour le mobile
- [Sketchfab](https://sketchfab.com/features/free-3d-models) — filtrez sur *Downloadable*
- [gltf-transform](https://gltf-transform.dev/) et [gltfpack](https://meshoptimizer.org/gltf/) — compression de modèles

## Plateforme et outillage

- **Flask** — [documentation](https://flask.palletsprojects.com/) ·
  [gabarits Jinja2](https://jinja.palletsprojects.com/)
- **PythonAnywhere** — [aide Flask](https://help.pythonanywhere.com/pages/Flask/) ·
  [API Files](https://help.pythonanywhere.com/pages/API/) ·
  [sites autorisés (offre gratuite)](https://www.pythonanywhere.com/whitelist/)
- **GitHub Actions** — [documentation](https://docs.github.com/actions) ·
  [secrets chiffrés](https://docs.github.com/actions/security-guides/encrypted-secrets)

## Références SDK natifs

- **Google ARCore** — [concepts fondamentaux](https://developers.google.com/ar/develop/fundamentals) · [appareils compatibles](https://developers.google.com/ar/devices)
- **Apple ARKit** — [documentation développeur](https://developer.apple.com/augmented-reality/)
- **Unity AR Foundation** — [documentation du paquet `com.unity.xr.arfoundation`](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@latest)
- **WebXR Device API** — [spécification W3C](https://www.w3.org/TR/webxr/) · [état du support](https://caniuse.com/webxr)

## Cadre théorique

- P. Milgram & F. Kishino, *A Taxonomy of Mixed Reality Visual Displays*, 1994 — le
  continuum réel–virtuel.
- R. Azuma, *A Survey of Augmented Reality*, 1997 — les trois critères fondateurs de la RA.

> Les versions de bibliothèques et de CDN citées dans ce document, ainsi que les
> compatibilités matérielles, **évoluent vite : à vérifier en début de module**.

---

*Document pédagogique — Module RA/RV, Titre de niveau 7. À adapter selon le parc
matériel et le calendrier de la promotion.*
