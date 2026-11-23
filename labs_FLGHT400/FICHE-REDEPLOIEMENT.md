# Fiche Bob : redéployer l'IBM i + FLGHT400, puis jouer le lab

> **Note :** les dossiers `ibmi-lab/` et `scripts/` cités dans cette fiche (`tz.py`, `tunnel.sh`, `install_flght400.sh`, `.env`, etc.) **ne sont pas fournis dans ce dépôt**. Ils illustrent l'environnement de travail utilisé lors du lab. Les adresses IP, mots de passe et chemins locaux ont été masqués (`xxx`, `_set_password_`, `<WORKSPACE>`).

## Prompt de lancement

**Session solo (1 utilisateur) :**
```
Je voudrais lancer ces labs ibmi-lab/FICHE-REDEPLOIEMENT.md, peux-tu me monter un environnement de test sur la TZ ?
```

**Session multi-utilisateurs (remplacer X par le nombre de participants) :**
```
Je voudrais lancer ces labs ibmi-lab/FICHE-REDEPLOIEMENT.md, peux-tu me monter un environnement de test sur la TZ pour X utilisateurs ?
```

> Bob lit cette fiche, exécute la phase A (TechZone + FLGHT400), puis pour X > 1 enchaîne sur la section Multi-utilisateurs pour créer les bibliothèques `FLGHT401`…`FLGHT4XX`.
> Référentiel du lab : `flight400-demo/README.md` (exercices 1 à 7). Bibliothèque solo : **`FLGHT400`** ; multi : **`FLGHT4nn`** attribué par l'instructeur.

## Règles pour Bob (obligatoires)

1. Ne jamais afficher le mot de passe, la clé privée ni le jeton TechZone. Les scripts ci-dessous les écrivent dans des fichiers en mode 600 sans les afficher.
2. **Outils TechZone — règle stricte :**
   - **Création d'une réservation** : `scripts/tz.py create` **uniquement**. Les outils MCP TechZone (`platform-mcp-platform-list`, `request-mcp-techzone-create-request`, etc.) n'ont pas accès au catalogue IBM i PowerVS — les utiliser pour la création conduit à un échec garanti.
   - **Statut / credentials après création** : `scripts/tz.py status|wait|creds` en priorité. Le MCP `request-mcp-techzone-get-request` peut être utilisé **en secours** si `tz.py` échoue, en passant le `requestId` retourné par `tz.py create` (sa sortie contient les secrets — ne jamais l'afficher brute).
3. Avant `--force` (supprime et restaure `FLGHT400`, donc efface les modifications des exercices) : demander confirmation à l'utilisateur.
4. Si l'utilisateur dit **« nouvel environnement »** ou **« nouvel IBM i »** : créer une réservation TechZone sans vérifier l'existante. Ne pas proposer de réutiliser une réservation active.
5. Créer une réservation consomme du quota TechZone : demander confirmation avant `create`, sauf si l'utilisateur l'a déjà donnée dans le prompt.
6. Fermer les tunnels ouverts pendant la session (`scripts/tunnel.sh close`) avant de conclure.
7. En cas d'échec d'une étape : s'arrêter, donner le message exact, proposer (a) corriger, (b) continuer sans.
8. Si le prompt de lancement mentionne **« pour X utilisateurs »** (X > 1) : après la phase A, enchaîner automatiquement sur la section **Multi-utilisateurs** — vérifier A0c, uploader le script, dry-run, puis exécution réelle avec `-n X`.

## Prérequis : fichiers de déploiement dans `ibmi-lab/`

`ibmi-lab/` est auto-porteur : tout ce dont Bob a besoin doit être **dans ce dossier**. Avant de lancer la phase A, Bob vérifie la présence des deux fichiers suivants.

### A0 — Vérifier (et récupérer si besoin) `FLGHT400.FILE`

```bash
ls ibmi-lab/FLGHT400.FILE
```

**Si le fichier est absent :** il n'est pas dans le repo GitHub — il est uniquement sur IBM Box.

> ⬇️ **Téléchargement manuel (action humaine)** : ouvrir https://ibm.box.com/v/flight400-box et télécharger **`FLGHT400.FILE`** (≈ 275 Mo) directement dans `ibmi-lab/`.

Aucun raccourci CLI ne fonctionne sur Box sans authentification IBM. L'utilisateur doit le faire depuis son navigateur.

### A0b — Vérifier (et récupérer si besoin) `Install-Flight400.sql`

```bash
ls ibmi-lab/Install-Flight400.sql
```

**Si le fichier est absent :** Bob le récupère directement depuis GitHub :

```bash
curl -sL "https://raw.githubusercontent.com/bmarolleau/flight400-demo/main/instructor-setup/Install-Flight400.sql" \
     -o ibmi-lab/Install-Flight400.sql
```

> ℹ️ `install_flght400.sh` n'utilise pas `Install-Flight400.sql` directement — il exécute les CL lui-même. Ce fichier reste utile comme référence ou pour la méthode manuelle (Option 2 du README instructor).

### A0c — Vérifier (et récupérer si besoin) `scripts/setup_flight400_lab.sh` (mode multi-utilisateurs uniquement)

Nécessaire seulement pour les labs multi-participants (voir section **Multi-utilisateurs** en fin de fiche). Si absent :

```bash
curl -sL "https://raw.githubusercontent.com/bmarolleau/flight400-demo/main/instructor-setup/setup_flight400_lab.sh" \
     -o ibmi-lab/scripts/setup_flight400_lab.sh
chmod +x ibmi-lab/scripts/setup_flight400_lab.sh
```

---

## Phase A : déploiement (Bob seul, aucune action humaine sauf si un contrôle échoue)

Tous les chemins sont relatifs à `ibmi-lab/`. Prérequis : être connecté au VPN IBM si le réseau l'exige.

| # | Action de Bob | Commande | Résultat attendu |
|---|---|---|---|
| A1 | Contrôler l'ancienne réservation (**sauf si l'utilisateur a demandé un nouvel environnement → sauter directement à A2**) | `python3 scripts/tz.py status` | `Ready` avec une date de fin future → **sauter à A5** (rien à recréer). Sinon continuer. |
| A2 | Créer la réservation (confirmation requise, sauf si déjà donnée dans le prompt) | `python3 scripts/tz.py create --purpose Test --geography europe` | Affiche l'identifiant et le lien ; l'identifiant est mémorisé dans `.tz_request_id`. `Education`/`Learning` est bloqué par la politique TechZone (environnement non lié à un cours officiel) : utiliser `Test`, ou `Demo`/`Pilot` avec `--opportunity`. Avec `europe`, le serveur tire la région (fra04 ou mad02). `Test` donne une durée courte (environ 12 h le 27/10) : prolonger dans l'interface TechZone si besoin. |
| A3 | Attendre la fin du provisionnement (10 à 25 min) | `python3 scripts/tz.py wait` | `status=Ready` |
| A4 | Écrire les accès | `python3 scripts/tz.py creds` | Écrit `ssh/ssh_private_key.pem` et `.env`, sans rien afficher d'autre que l'hôte et l'utilisateur |
| A5 | Installer FLGHT400 | `scripts/install_flght400.sh` (ajouter `--node` seulement pour l'exercice 7) | `RESULT: OK`, `PGM=46 MENU=3` |
| A6 | Résumer à l'utilisateur | – | IP, utilisateur, date de fin de réservation, résultat A5 |

**Repli si `tz.py creds` échoue** (secours manuel, le minimum d'interaction humaine) : Bob demande à l'utilisateur
1. de coller l'utilisateur et le mot de passe dans `ibmi-lab/.env` (champs `IBMI_USER`, `IBMI_PASSWORD`, `IBMI_HOST`), en gardant les valeurs entre guillemets simples ;
2. de déposer la clé privée téléchargée depuis la page de la réservation sous `ibmi-lab/ssh/ssh_private_key.pem`, puis `chmod 600`.
Ensuite Bob reprend en A5. Le fichier modèle est `.env.example`.

Notes techniques vérifiées :
- La passerelle TechZone refuse l'agent HTTP par défaut de Python (HTTP 403, code 1010). `tz.py` envoie un autre agent : ne pas le retirer.
- `install_flght400.sh` utilise `ssh/known_hosts` : une nouvelle machine qui reprend une ancienne IP ne bloque pas la connexion.
- Chemin « première installation » (bibliothèque absente) : même séquence que celle exécutée à la main le 22/10 (`CRTSAVF`, `CPYFRMSTMF`, `RSTLIB`, `CHGOWN`). Le chemin `--force` et `create` n'ont pas été exécutés en conditions réelles.

## Phase B : réglages dans Bob IDE (interface graphique, action humaine)

Bob ne peut ni cliquer dans l'interface ni ouvrir la connexion IBM i (le sélecteur « How would you like to connect? » attend un clic). Il présente donc cette phase à l'utilisateur **avec les valeurs exactes à saisir**, puis attend « connecté » (après B2) ou « phase B faite » (après B5).

**Ne pas appeler `execute_sql_statement`, `execute_cl_command` ni aucun outil IBM i avant que l'utilisateur ait confirmé la connexion** : chaque appel rouvre le sélecteur et boucle.

### Source des valeurs : le dossier de l'IP (Bob le lit à chaque exécution)
Les accès de la réservation sont déposés à la main par l'utilisateur dans `ibmi-lab/<IP publique>/` (ex. `ibmi-lab/161.156.153.xxx/`). Ce dossier est ignoré par git. Il contient :
- `<IP>.txt` : le détail de la réservation, au format *libellé sur une ligne, valeur sur la ligne suivante* ;
- `ssh_private_key*.pem` : la clé privée de déploiement (le nom peut contenir `(1)`).

Procédure de Bob avant de présenter B2 :
1. Lister `ibmi-lab/` et prendre le dossier dont le nom est une adresse IPv4 (s'il y en a plusieurs, prendre le plus récent et le dire).
2. Lire `<IP>.txt` et en tirer, par leur libellé, uniquement : *Deployment user name* (utilisateur), *Virtual machine 1 public ip address* (hôte), *SSH port* (port), *IBM Cloud PowerVS datacenter* (ex. `eu-de-1` = fra04).
3. **Ne jamais afficher la valeur qui suit « Deployment password »**, qui est sur la ligne d'après le libellé : un masquage ligne par ligne ne suffit pas. Extraire les champs voulus par libellé (ex. `grep -A1 'Deployment user name'`), sans `cat` du fichier entier. Le mot de passe ne sert pas à la connexion SSH, qui utilise la clé.
4. Trouver la clé avec `ssh_private_key*.pem` dans ce dossier, faire `chmod 600` dessus (sinon SSH la refuse : « permissions too open »), puis donner à l'utilisateur son **chemin absolu**.
5. Contrôle : `ssh-keygen -lf` sur cette clé et sur `ibmi-lab/ssh/ssh_private_key.pem` doit donner la même empreinte (vérifié le 27/10). Sinon, le signaler.
6. Remplir le tableau de B2 avec ces valeurs.

### B1. Extensions
Bob vérifie avec `list_extensions` les extensions déjà installées et signale celles qui manquent.

Extensions présentes le 27/10 (rien à faire) :
- *IBM Bob Premium Package for i* (v2.0.0) : installée et active.
- *Code for IBM i* (v3.0.13) : installée et active.
- *IBM i Testing* (v1.3.4) : installée et active (nécessaire pour l'exercice 6).

Extension **manquante à installer avant l'étape 1a** :
- **IBM i Renderer** (éditeur : halcyontechltd) : absente le 27/10. Fournit les liens `Preview all` et `Preview XXXXXXXX` dans l'éditeur de sources DDS. Sans elle, aucun lien de prévisualisation n'apparaît dans `FRS001DF` ni `FRS021DF`. Installer : `Cmd+Shift+X` → chercher « IBM i Renderer » → **Install**.

Note : l'extension « IBM i Development Pack » n'apparaît pas dans la liste et n'est pas nécessaire. *Code for IBM i* la remplace.

### B2. Connexion à l'IBM i
Dans le sélecteur « How would you like to connect? », choisir **Open connection browser**.

Cas 1, une connexion `Flight 400 Demo` existe déjà : Bob met à jour son champ `host` dans `~/Library/Application Support/IBM Bob/User/settings.json` (clé `code-for-ibmi.connections`) après une copie de sauvegarde `settings.json.bak-*`, puis l'utilisateur la sélectionne. Si l'ancienne IP s'affiche, recharger la fenêtre.

Cas 2, elle n'existe pas : **Add new connection**, puis remplir le formulaire avec les valeurs lues dans le dossier de l'IP.

| Champ du formulaire | Valeur (exemple du 27/10, à relire dans le dossier de l'IP) |
|---|---|
| Connection Name | `Flight 400 Demo` |
| Host or IP Address | `161.156.153.xxx` (*Virtual machine 1 public ip address*) |
| Port (SSH) | `22` (*SSH port*) |
| Username | `ITZUSER` (*Deployment user name*) |
| Password | **laisser vide** : une seule méthode d'authentification |
| Save Password | décoché |
| Enable MFA | décoché |
| Private Key | chemin absolu du `.pem` du dossier de l'IP, ex. `<WORKSPACE>/ibmi-lab/161.156.153.xxx/ssh_private_key(1).pem` ; ou *Select File* |

Contrôle : la barre d'état ou la vue IBM i affiche `ITZUSER@<IP>`.

### B3. Liste des bibliothèques
Vue **User Library List** du panneau IBM i :
- bouton **+** : ajouter `FLGHT400` ;
- clic droit sur `FLGHT400` : *Set as Current Library*.

Contrôle obligatoire : `FLGHT400` apparaît dans la liste **et** porte l'indication de bibliothèque courante. Le réglage `currentLibrary` dans `settings.json` ne suffit pas — il peut ne pas être rattaché à la nouvelle connexion, et `/init` échoue alors (« No current library is configured for this connection »). Si besoin, utiliser **Change Current Library** dans la vue et choisir `FLGHT400`.

### B4. Filtre de l'Object Browser
Créer un filtre (ou réutiliser celui qui existe déjà) :

| Champ | Valeur |
|---|---|
| Name | `FLGHT400` |
| Library | `FLGHT400` |
| Object | `*` |
| Types | `*ALL` |

Contrôle : le filtre déplié montre les fichiers sources (`QRPGSRC`, `QRPGLESRC`, `QDDSSRCD`…) et les 46 programmes.

### B5. Nouvelle tâche Bob
| Étape | Valeur |
|---|---|
| Lancer | **+** → *New Task in Library List* |
| Mode | **IBM i Developer** |
| Permissions de la tâche | activer **Documentation** et **Understand** |

Bob contrôle par SSH, avec la clé et sans les outils IBM i, que `BOBLIB` existe (utilisée à l'exercice 1c pour les métadonnées d'*Understand*) : `ssh -i <clé> ITZUSER@<IP> 'system "DSPLIBD LIB(BOBLIB)"'`. Constat du 27/10 sur la machine neuve : **`BOBLIB` n'existe pas** (`CPF9810`). Avant l'exercice 1c, soit la créer (`CRTLIB LIB(BOBLIB)`, après accord de l'utilisateur), soit utiliser la bibliothèque `understand` configurée dans les réglages (`vscode-ibmi-bob.understand.installLibrary`). Constat du 27/10 : `UNDERSTAND` n'existe pas non plus (`CPF9810`). Les outils *Understand* ne répondront donc qu'après l'installation d'*Understand* depuis son tableau de bord (exercice 1c), à faire par l'utilisateur.

### B6. Règles personnalisées du mode IBM i Developer (avant l'exercice 3 uniquement)
> ⚠️ **À refaire sur chaque nouvel environnement** (nouveau déploiement TechZone, nouvelle machine, nouvelle installation de Bob). Ces règles sont stockées dans les paramètres locaux de Bob IDE — elles ne se transfèrent pas d'un poste à l'autre. Si B6 n'a pas été fait, l'exercice 3 produit des diffs avec de mauvaises colonnes et Bob compile sans attendre l'accord.

Sans ces règles, Bob risque d'insérer les lignes DDS et RPG avec de mauvaises colonnes et de compiler sans attendre ton accord. À faire une seule fois, à n'importe quel moment avant l'exercice 3 :

1. Cliquer sur l'**icône de réglages** en haut à droite du panneau Bob.
2. Cliquer sur **Modes** dans la barre latérale gauche, puis sélectionner **IBM i Developer**.
3. Cliquer sur l'**icône crayon** (modifier) en haut à droite pour ouvrir l'éditeur du mode.
4. Descendre jusqu'à la section **Custom Instructions**, ajouter le texte ci-dessous **sous** les « Interaction Guidelines » existantes, puis cliquer sur **Save**.

```
## When a user asks to plan or implement code changes
- Make insert-only changes using apply_diff with surrounding context; do not use insert_content.
- When inserting, anchor the diff on the line ABOVE the insertion point and do not include trailing structural lines (DDS key "K ..." or record-format "R ..." lines) in the replacement - leave those lines untouched.
- When adding a fixed-form entry that mirrors an existing one (RPG I-spec, O-spec, or DDS field), copy the model line's EXACT column positions and change only the field name and value. Re-pad so numbers stay right-justified in their original columns and the data-type letter stays in its column. If the new field name is a different length than the model, adjust the padding so every downstream column is unchanged. Never eyeball the spacing.
- Preserve exact DDS column positions for field-definition entries (name, length, type). Keyword continuation lines like COLHDG only need to start after position 44 - do not fixate on their exact alignment.
- Preserve the existing OPM RPG fixed-column style.
- Do not add ALWNULL, do not use SQL ALTER TABLE, do not change the field to 5P 1.
- COLHDG belongs only in the physical file, never in the display file.
- Always show the diff and wait for my approval before saving; never compile until I say so.
```

Contrôle : après l'enregistrement, ces lignes apparaissent dans les Custom Instructions du mode IBM i Developer. Source : [`flight400-demo/README.md`](https://github.com/bmarolleau/flight400-demo/blob/main/README.md) § 3a, recopié ici pour que la fiche se suffise.

### B7. Installer Understand et RPGUnit (à faire avant les exercices 1c et 6)

Ces deux composants s'installent de la même façon depuis le panneau IBM i. Comprend est nécessaire dès l'exercice 1c ; RPGUnit seulement pour l'exercice 6, mais autant tout installer d'un coup.

#### Ouvrir le panneau IBM i

Le panneau IBM i se trouve dans la **barre d'activité gauche** (icône IBM i). S'il a été fermé :
- passer la souris sur la **barre d'état en bas** (statusbar) → le nom de la connexion apparaît (ex. `Flight 400 Demo`) avec une icône de réglages → cliquer dessus, ou
- clic droit sur n'importe quelle icône de la barre d'activité gauche → cocher **IBM i** pour le réafficher.

Une fois le panneau ouvert : **clic droit sur la connexion** (ex. `Flight 400 Demo`) → **Connection Settings** → onglet **Components**.

#### Understand

État constaté le 27/10 et 28/10 : **`BOBLIB` et `UNDERSTAND` n'existent pas** sur la machine neuve. Le composant Understand crée sa bibliothèque de métadonnées lors de l'installation.

1. Dans l'onglet **Components** : **Add Component** → choisir **Understand** → **Install**.
2. Contrôle : le tableau de bord *Understand* (icône graphe dans la barre d'activité) est accessible sans erreur.

> **Constat 28/10 :** l'installeur **ne propose pas** de choisir la bibliothèque de métadonnées — il crée directement `UNDERSTAND` (valeur par défaut de `vscode-ibmi-bob.understand.installLibrary`). Ne pas anticiper un prompt qui n'apparaît pas. **À l'étape 1c, utiliser `UNDERSTAND` comme bibliothèque de métadonnées** (et non `BOBLIB`).

#### RPGUnit (exercice 6 uniquement)

État constaté le 27/10 : **`RPGUNIT` n'existe pas** (`CPF9810`) ; `QDEVTOOLS` existe déjà.

1. Extension **IBM i Testing** : ouvrir le panneau des extensions (`Cmd+Shift+X`), chercher « IBM i Testing » et vérifier qu'elle est *Installed* (elle l'est le 27/10, version 1.3.4).
2. Dans l'onglet **Components** : **Add Component** → choisir **RPGUnit** → **Install**.
3. Ajouter `RPGUNIT` puis `QDEVTOOLS` à la **User Library List** (bouton **+**, comme en B3). La liste doit contenir `FLGHT400`, `RPGUNIT` et `QDEVTOOLS`.

Contrôle (Bob, par SSH) : `ssh -i <clé> ITZUSER@<IP> 'system "DSPLIBD LIB(RPGUNIT)"'` ne renvoie plus `CPF9810`. Source : [`flight400-demo/README.md`](https://github.com/bmarolleau/flight400-demo/blob/main/README.md) § Prérequis exercice 6.

## Phase C : jouer le lab (instructions à donner à Bob, dans l'ordre)

Remplacer partout `FLGHT4nn` par `FLGHT400`. Durée totale environ 2 h. Les blocs entre guillemets sont à envoyer tels quels à Bob.

### Comment lancer chaque tâche (bouton **+** au-dessus du chat)
Le bouton **+** propose trois portées : *New task in Bob_L4*, *New task in Library List* et *New task in IFS Working Directory*. Dans le sélecteur de ta capture, elles s'affichent ainsi :

| Option affichée | Portée | À choisir pour |
|---|---|---|
| **New task in Library List** (`FLGHT400, QGPL, QTEMP`) | Bibliothèques IBM i (QSYS) | **Tous les exercices 1 à 6**, mode **IBM i Developer** (ou **IBM i Database** aux étapes 1e et 4). Seule option qui permet `/init`, `@FLGHT400`, la lecture des sources et la compilation. |
| New task in Bob_L4 | Dossier local du Mac (*Local Workspace*) | **Seulement l'exercice 7, étape 3** (créer le skill, en mode **Agent**). Jamais pour travailler sur FLGHT400. |
| New task in IFS Working Directory (`/home/ITZUSER`) | Répertoire personnel sur l'IBM i | Pas nécessaire pour le lab. L'exercice 7 n'en a pas besoin : l'application est créée dans l'IFS depuis la portée *Library List*. |

Règles à suivre :
- **Une nouvelle tâche pour chaque changement de portée** (bouton **+**). Pour un changement de mode seul (IBM i Developer vers IBM i Database), [`flight400-demo/README.md`](https://github.com/bmarolleau/flight400-demo/blob/main/README.md) § 4a dit de basculer le mode avec le sélecteur de modes du chat.
- Contrôle avant d'envoyer la première instruction : le mode affiché est le bon, la portée affichée est *Library List*, et `FLGHT400` est sur la liste de bibliothèques avec la mention bibliothèque courante (B3).
- Une tâche peut enchaîner plusieurs étapes d'un même exercice. Pour l'exercice 3, garder **une seule tâche** du début à la fin, afin que Bob conserve le plan validé à l'étape 3c.
- Les étapes « Humain » (tableau de bord *Understand*, icône *Workflows*) se font dans l'interface, pas dans le chat. Pour *Workflows*, choisir d'abord la portée *Library List* ([`flight400-demo/README.md`](https://github.com/bmarolleau/flight400-demo/blob/main/README.md) § 4d et § 6b).

### Exercice 1 : documentation (40 min)
| Étape | Instruction | Mode / remarque |
|---|---|---|
| 1a | Humain, sans Bob : parcourir l'application dans l'Object Browser. (1) Étendre **User Library List** et **Object Browser** ; filtre `FLGHT400`, types `*ALL`. (2) Sous **Source Files**, ouvrir quelques membres de `QRPGSRC` pour voir le style RPG fixe. (3) Dans `QDDSSRCD`, double-cliquer `FRS001DF` pour l'ouvrir : **Preview All** nécessite l'extension **IBM i Renderer** (non installée le 27/10). Sans elle, le lien n'apparaît pas du tout dans l'éditeur. Pour l'installer : `Cmd+Shift+X` → chercher « IBM i Renderer » (éditeur : halcyontechltd) → **Install**. Une fois installée, un lien `Preview all` apparaît **au-dessus de la ligne 1**, dans la gouttière de l'éditeur, en gris très petit (vérifié le 27/10 sur `FRS001DF`). Un lien `Preview XXXXXXXX` apparaît aussi sous chaque nom de format d'écran. Cliquer `Preview all` pour voir le rendu visuel de tous les écrans du fichier. Essayer aussi `FRS021DF` (écran de l'exercice 3). (4) Dans l'Object Browser, **clic droit sur `FRS000.PGM`** → **Detail** : un panneau latéral affiche les métadonnées de l'objet compilé, dont la date de compilation (1997). | Aucune instruction à Bob |
| 1b | **Avant** : dans la vue **User Library List**, clic droit sur `FLGHT400` → *Set as Current Library* (ou **Change Current Library**). Puis `/init` | IBM i Developer, portée *Library List*. Crée `FLGHT400/QBOBSRC(AGENTS)`. Sans bibliothèque courante, `/init` s'arrête sans rien créer (QGPL et QTEMP refusées) |
| 1c | **Prérequis : Understand installé (B7).** Humain : dans la barre d'activité, cliquer l'icône **Understand** (graphe) → **New Application**, nom `FLGHT400`, bibliothèque `FLGHT400`, bibliothèque de métadonnées **`UNDERSTAND`** (l'installeur ne propose pas de la choisir — il crée `UNDERSTAND` directement, confirmé le 28/10) → **Scan**. Puis : « What are the high-level functional areas of FLGHT400? » | Le scan prend 30 à 60 s. Attendu : environ 46 programmes, 0 SQL dynamique |
| 1d | « Generate a comprehensive architecture overview of the FLGHT400 application in QSYS in Markdown format. Include a high-level description, the main program flows, key programs and their roles, a Mermaid architecture diagram, and a summary of the database tables used. » | Enregistrer en `FLGHT400-Architecture.md` |
| 1e | `/erd FLGHT400` | Mode **IBM i Database** |
| 1f (option) | « Analyze the FLGHT400 application from the library list and generate a draw.io architecture diagram showing the main programs, menus, and database files. Save the file as `FLGHT400-architecture.drawio` in `$HOME/docs/` on IBM i. » | Extension Draw.io Integration |
| 1g (option) | Humain : icône *Workflows* → *Business Rules Extraction*. Bibliothèque `FLGHT400`, fichier `QRPGLESRC`, membre `FRS401.RPGLE`, sortie `/home/ITZUSER/flght400/docs/business-rules/FRS401-report.md` | |

### Exercice 2 : moderniser `FRS409` (15 min)
1. Ouvrir `FLGHT400/QRPGSRC(FRS409)`, puis : « What does this program do? »
2. « Can you modernize this program? » → *Start workflow* (RPG Modernization), source `FLGHT400/QRPGSRC`, membre `FRS409`, sortie `FLGHT400/QRPGLESRC/FRS409.RPGLE`.

### Exercice 3 : ajouter *Total Flight Hours* (30 min)
> ⚠️ Cet exercice **modifie** la bibliothèque. Pour le rejouer, restaurer avec `--force` puis refaire B6.

**Prérequis :** B6 faite (règles personnalisées enregistrées dans le mode IBM i Developer). Mode **IBM i Developer**, portée **Library List (QSYS)**.

**Étape 3a — Règles personnalisées :** = **B6** (déjà fait en phase B). Pas d'action supplémentaire ici.

**Spec du champ à ajouter** (informatif — c'est la spec que Bob va implémenter aux étapes suivantes) :
| Couche | Nom | Définition |
|---|---|---|
| Base (PF) | `FLHRS` | 4P 0 (packed, 4 chiffres, 0 décimales) |
| Logique / RPG | `FHRS` | `RENAME(FLHRS)` |
| Écran (DDS) | `SFLHRS` | 4/0, input/output, après Mileage |

**Étape 3b — Explorer l'écran FRS021DF :**
```
Preview the display file FRS021DF from FLGHT400/QDDSSRCD and list every field on the Flight Maintenance screen, noting the input/output type of each and the screen-field naming convention (e.g. SFLGHT, SMILES, SSEATS, SPRICE). Do not modify anything.
```

**Étape 3c — Planifier (ne rien écrire) :**
```
Let's plan this change together first — do not write, edit, save, or compile any code; just produce the plan with me. Produce a step-by-step implementation plan to add "Total Flight Hours" to the Flight Maintenance application in FLGHT400.

Spec: database field FLHRS (4P 0); expose to RPG as FHRS RENAME(FLHRS); screen field SFLHRS (4 digits, 0 decimals, input/output) placed immediately after Mileage with the visible label "Flight Hours".

Base the plan on how the existing Mileage field flows today: FLIGHTS.MILEAGE → FLIGHTSZ.MILES → FRS021 → FRS021DF.SMILES. Mirror that exact path for the new field.

The plan must cover and bake in these constraints:
- FLGHT400/QDDSSRCF(FLIGHTS): add FLHRS 4P 0 with a COLHDG consistent with the existing PF DDS.
- FLGHT400/QDDSSRCF(FLIGHTSZ): add FHRS RENAME(FLHRS).
- FLGHT400/QDDSSRCD(FRS021DF): add input/output field SFLHRS with visible label "Flight Hours", CHECK(RZ) to match nearby numeric fields, no COLHDG, no overlap with existing fields/message/function-key areas.
- FLGHT400/QRPGSRC(FRS021): increase the FLIGHTSZ record length from 233 to 236 for the 3-byte packed field; add FHRS to the input spec at positions 234-236 — note FHRS is packed (4P 0, 3 bytes), so use the P data-type in column 43 of the I-spec, unlike the neighboring FMILES which is binary (B); after every successful CHAIN that loads an existing flight, explicitly move FHRS to SFLHRS; move SFLHRS to FHRS on add/update; include FHRS in the ADDFLT and UPDFLT output specs — mirror the exact column alignment of the neighboring FMILES O-spec entry so the end position stays right-justified; validate consistently with existing numeric screen fields.
- Build order at the end: CHGPF FLIGHTS, CRTLF FLIGHTSZ, CRTDSPF FRS021DF, CRTRPGPGM FRS021.

Only these four objects may change: FLIGHTS, FLIGHTSZ, FRS021DF, FRS021. Any other affected program is follow-up work only. Flag any other programs that use FLIGHTS or FLIGHTSZ as follow-up work only — do not include them in the plan's changes.
```
> ✅ Checkpoint : le plan ne touche que `FLIGHTS`, `FLIGHTSZ`, `FRS021DF`, `FRS021`, utilise bien `FLHRS`/`FHRS`/`SFLHRS`, mentionne le passage 233→236 et le move `FHRS→SFLHRS` post-CHAIN. Approuver : *"I approve this plan — let's implement it."*

**Étape 3d — Implémenter les DDS (un membre à la fois) :**
```
Execute the approved plan for the DDS members only, one at a time: (1) FLIGHTS, (2) FLIGHTSZ, (3) FRS021DF. For each, show the diff, wait for my approval, then save the source member only — do not compile. Follow the standing rules exactly.
```
Vérifier pour chaque membre avant d'approuver :
- `FLIGHTS` : `FLHRS 4P 0` avec `COLHDG` ; ligne de clé `K FLIGH00001` intacte.
- `FLIGHTSZ` : `FHRS RENAME(FLHRS)` après `MILES` ; ligne de clé `K FLGHTN` intacte.
- `FRS021DF` : `SFLHRS` 4/0 I/O après Mileage ; label "Flight Hours" ; `CHECK(RZ)` présent ; **pas** de `COLHDG`.

**Étape 3e — Implémenter le programme RPG (3 diffs séparés) :**

*5.1 — F-spec et I-spec :*
```
Apply just the F-spec and I-spec changes to FLGHT400/QRPGSRC(FRS021): increase the FLIGHTSZ record length from 233 to 236, and add the FHRS input-spec entry at positions 234-236 using data-type P (packed) in column 43, mirroring the exact column alignment of the neighboring FMILES entry. Show the diff and wait for my approval; do not compile.
```
Vérifier : F-spec lit `236` ; I-spec `FHRS` à 234-236 avec `P` dans la colonne data-type (pas `B`).

*5.2 — C-specs :*
```
Now apply just the C-spec changes to FRS021: after every successful CHAIN that loads an existing flight, move FHRS to SFLHRS; move SFLHRS to FHRS on both the add and update paths (mirroring the existing Z-ADD SMILES/FMILES lines); and add an SFLHRS validation consistent with the SMILES check using the next free indicator. Show the diff and wait for my approval; do not compile.
```

*5.3 — O-specs (partie la plus sensible à l'alignement) :*
```
Finally apply just the O-spec changes to FRS021: add FHRS to both the ADDFLT and UPDFLT output records. Copy the exact column positions of the neighboring FMILES O-spec line and change only the field name and end position — the end position must stay right-justified in the same columns, with the packed data-type letter in the same column as FMILES's B. Do not eyeball the spacing. Show the diff and wait for my approval; do not compile.
```
> ⚠️ La position de fin de `FHRS` doit s'aligner sur celle de `FMILES` (`233`) — sinon le compilateur lit `238` et échoue.

**Étape 3f — Compiler les 4 objets dans l'ordre :**
```
Build only the four demo objects in order: (1) apply DDS for FLIGHTS, (2) rebuild FLIGHTSZ, (3) compile FRS021DF, (4) compile FRS021. Use the correct IBM i command for each source type. If one fails, stop, report the compile error briefly, and wait for my direction. Do not touch unrelated programs.
```
Commandes attendues :
```
CHGPF FILE(FLGHT400/FLIGHTS) SRCFILE(FLGHT400/QDDSSRCF) SRCMBR(FLIGHTS)
CRTLF FILE(FLGHT400/FLIGHTSZ) SRCFILE(FLGHT400/QDDSSRCF) SRCMBR(FLIGHTSZ)
CRTDSPF FILE(FLGHT400/FRS021DF) SRCFILE(FLGHT400/QDDSSRCD) SRCMBR(FRS021DF)
CRTRPGPGM PGM(FLGHT400/FRS021) SRCFILE(FLGHT400/QRPGSRC) SRCMBR(FRS021) REPLACE(*YES)
```

**Étape 3g — Valider :**
```
Validate the completed change: confirm FLHRS exists in FLGHT400/FLIGHTS, FHRS is available through FLIGHTSZ, SFLHRS appears immediately after Mileage in the FRS021DF Previewer, FRS021 compiled, and FRS021DF has no COLHDG. Finish with a one-line end-to-end field mapping and list any additional impacted programs as follow-up. Do not modify anything.
```

### Exercice 4 : optimisation SQL (15 min)
**Étape 4a — Changer de mode :** passer en **IBM i Database** via le sélecteur de mode dans le panneau Bob.

**Étape 4b — Revue de requête :**
1. Dans le chat Bob, taper `/review_sql` (le slash-command doit être visible), puis coller la requête suivante (remplacer `FLGHT400` par votre bibliothèque) :

```sql
SELECT
    f.FLIGH00001                                    AS FLIGHT_NUMBER,
    f.DEPARTURE                                     AS FROM_CITY,
    f.ARRIVAL                                       AS TO_CITY,
    f.AIRLINES                                      AS AIRLINE,
    f.DAY_O00001                                    AS DAY_OF_WEEK,
    f.DEPAR00002                                    AS DEPARTURE_TIME,
    f.ARRIV00002                                    AS ARRIVAL_TIME,
    f.MILEAGE,
    f.TICKE00001                                    AS TICKET_PRICE,
    f.SEATS00001                                    AS SEATS_AVAILABLE,
    ag.AGENT_NAME,
    COUNT(DISTINCT o.CUSTO00001)                    AS UNIQUE_CUSTOMERS,
    SUM(o.TICKE00001)                               AS TOTAL_TICKETS_SOLD,
    SUM(CASE WHEN o.CLASS = 'F' THEN o.TICKE00001 ELSE 0 END) AS FIRST_CLASS_TICKETS,
    SUM(CASE WHEN o.CLASS = 'B' THEN o.TICKE00001 ELSE 0 END) AS BUSINESS_TICKETS,
    SUM(CASE WHEN o.CLASS = 'E' THEN o.TICKE00001 ELSE 0 END) AS ECONOMY_TICKETS,
    MIN(o.DEPAR00001)                               AS EARLIEST_BOOKING_DATE,
    MAX(o.DEPAR00001)                               AS LATEST_BOOKING_DATE
FROM FLGHT400/FLIGHTS       f
JOIN FLGHT400/ORDERS        o  ON o.FLIGH00001  = f.FLIGH00001
JOIN FLGHT400/AGENTS        ag ON ag.AGENT_NO   = o.AGENT_NO
LEFT JOIN FLGHT400/CUSTOMERS c  ON c.CUSTO00001  = o.CUSTO00001
WHERE o.DEPAR00001 >= TIMESTAMP('2004-02-08-00.00.00')
  AND o.DEPAR00001 <  TIMESTAMP('2004-02-11-00.00.00')
GROUP BY
    f.FLIGH00001, f.DEPARTURE, f.ARRIVAL, f.AIRLINES,
    f.DAY_O00001, f.DEPAR00002, f.ARRIV00002,
    f.MILEAGE, f.TICKE00001, f.SEATS00001, ag.AGENT_NAME
ORDER BY
    o.DEPAR00001,
    f.FLIGH00001
FETCH FIRST 100 ROWS ONLY;
```
> 💡 Problème intentionnel : `ORDER BY o.DEPAR00001` référence une colonne non présente dans le `GROUP BY`. Bob doit le détecter et proposer `MIN(o.DEPAR00001)`.

**Étape 4c — Performance (optionnel) :**
Demander : *"Is any table a performance bottleneck and why?"*
Attendu : Bob identifie `ORDERS` comme table volumineuse, le filtre sur `DEPARTURE_DATE` comme très sélectif, et recommande un index.

**Étape 4d — Index Advisor Workflow :**
1. Icône *Workflows* (haut du panneau Bob) → scope **Library List** → **SQL Index Strategy Advisor**.
2. Paramètres :

| Paramètre | Valeur |
|---|---|
| Data Source | Capture New Performance Data |
| Capture Method | `DUMP_PLAN_CACHE_TOPN` |
| Output Library | `FLGHT400` |
| Output Object Name | `FLGHT400P` |
| Top N Queries | 20 |
| Top N Category | Runtime |

3. Quand Bob propose les index : *"Apply the highest-priority index only for FLGHT400"*
   Résultat attendu :
   ```sql
   CREATE INDEX FLGHT400.ORDERS_IDX_DEPDT_FLT
       ON FLGHT400.ORDERS (DEPARTURE_DATE, FLIGHT_NUMBER);
   ```

### Exercice 5 : questions système (10 min), mode IBM i Developer
1. « Which active jobs have accumulated the most CPU time? For the top jobs, distinguish cumulative CPU time from their current elapsed CPU percentage. »
2. « Which programs in the FLGHT400 library have not been recompiled in the last 5 years? »

### Exercice 6 : tests RPGUnit (20 min)
**Prérequis :** B7 faite (RPGUnit installé, `RPGUNIT` et `QDEVTOOLS` dans la library list). Mode **IBM i Developer**.

**Étape 6a — Créer le membre source :**
Dans l'Object Browser, `FLGHT400/QRPGLESRC` → clic droit → **New Member** → nom `CUSTCHK.SQLRPGLE`. Coller le code suivant et sauvegarder (`Cmd/Ctrl+S`) :

```rpgle
**free
ctl-opt nomain;

dcl-proc checkCustomerExists export;
  dcl-pi *n ind;
    custId packed(9:0) const;
  end-pi;

  dcl-s rowCount int(10);

  exec sql
    select count(*)
      into :rowCount
      from CUSTOMRZ
      where CUSTNO = :custId;

  return (SQLCODE = 0 and rowCount > 0);
end-proc;
```

**Étape 6b — Workflow RPGUnit Test Plan Creation :**
Icône *Workflows* → **RPGUnit Test Plan Creation** (scope : library list).
1. **Get Started** → sélectionner la bibliothèque `FLGHT400`.
2. Dossier IFS : `/home/ITZUSER/flght400`.
3. Choisir **create a new test plan** → **default recommended path** → laisser Bob trouver les fichiers automatiquement (il doit trouver `CUSTCHK.SQLRPGLE`).
4. Sélectionner la procédure exportée → **validate the environment**.
> 💡 Si Bob propose de relancer le workflow Test Plan Creation, répondre **No thanks**.

**Étape 6c — Workflow RPGUnit Test Suite Implementation :**
Icône *Workflows* → **RPGUnit Test Suite Implementation** (scope : library list).
1. **Proceed** (le plan de 6b est déjà prêt).
2. Bibliothèque : `FLGHT400`.
3. Dossier IFS : `/home/ITZUSER/flght400` (même qu'en 6b). Confirmer le chemin vers les suites.
4. **validate the environment**. Si Bob propose de télécharger RPGUnit, cliquer **Install** puis **Cancel** sur le popup suivant.
> 💡 Si un test échoue : « Explain the failure and help fix it. »

### Exercice 7 (option) : application React depuis l'écran vert (30 min)
> 📌 **Valeurs par défaut (démo solo) : bibliothèque `FLGHT400`, port `3001`.**
> Si le participant a un numéro attribué (ex. `403`), remplacer `FLGHT400` → `FLGHT403` et port `3001` → port assigné dans la Setup table.

**Prérequis :** tunnel SSH actif sur le port applicatif + Node 22 installé sur l'IBM i.

> ⚠️ **Pourquoi un tunnel ?** Le serveur Vite tourne dans PASE sur une IP interne de l'IBM i (type `192.168.x.x`) qui n'est **jamais** accessible directement depuis ton Mac. Le seul chemin est un tunnel SSH via l'IP publique TechZone. **Ne pas utiliser l'IP interne dans la commande SSH** : cela échoue. La cible du tunnel doit être `localhost:<port>`.

**Étape 7.0 — Installer Node 22 (si pas encore fait) :**
```bash
scripts/install_flght400.sh --node
```

**Étape 7.1 — Générer l'app React :**
Mode **IBM i Developer**, cliquer **+** → bibliothèque `FLGHT400`. Joindre l'image `flight400-demo/docs/img/flight400.png`, puis envoyer :
```
Given this screenshot of the 5250 flight order screen from the Application Flight400 in @FLGHT400, Build a single-page React 18 + Vite 4 app on IBM i (PASE) using @carbon/react ^1.x with the g100 dark theme that modernises the IBM i 5250 screen shown in the attached screenshot. Create the app in the IFS at $HOME/flight400-frontend-apps/screen-name/. Use the g100 dark theme. All fields should have a list of values to select from. The dev server must run in the background using nohup ... & and write output to /tmp/vite-dev.log. Pin the Vite dev server to port 3001 if available.
```

**Étape 7.2 — Créer la skill helper :**
Mode **Agent**, cliquer **+** → **Local Workspace**. Copier le contenu de `flight400-demo/SAMPLE-SKILL.md`, le coller dans le chat, puis envoyer :
> *"Create a skill from the pasted text."*

**Étape 7.3 — Démarrer le dev server :**
```
Please start the dev server for the app you just created.
```
Vérifier dans le log de Bob que Vite confirme le démarrage :
```
VITE vX.X.X  ready in XXXX ms
➜  Local:   http://localhost:3001/
```

**Étape 7.4 — Ouvrir le tunnel SSH depuis ton Mac :**
Dans un terminal sur ton Mac, depuis `ibmi-lab/` :
```bash
scripts/tunnel.sh open 3001
```
Réponse attendue :
```
port 3001: open (pid XXXXX) -> http://localhost:3001
```
Si le tunnel est déjà ouvert (`port 3001: already open`) → passer directement à l'étape suivante.

Commande équivalente sans le script (au premier plan, arrêt avec `Ctrl+C`) :
```bash
set -a; . ./.env; set +a
ssh -i "$IBMI_SSH_KEY" -p "$IBMI_SSH_PORT" -L 3001:localhost:3001 -N "$IBMI_USER@$IBMI_HOST"
```

**Étape 7.5 — Ouvrir dans le navigateur :**
```
http://localhost:3001/
```

**Pièges courants**

| Symptôme | Cause | Correction |
|---|---|---|
| `192.168.x.x:3001` timeout | IP interne PASE — inaccessible depuis le Mac | Utiliser le tunnel, pas l'IP directe |
| `ssh -L 3001:192.168.x.x:3001` échoue | La cible doit être `localhost`, pas l'IP interne | Utiliser `localhost:3001` comme cible |
| `bind: Address already in use` | Tunnel déjà ouvert | C'est OK — aller directement au navigateur |
| L'IP change entre deux réservations TechZone | `.env` stocke `IBMI_HOST` — à mettre à jour si besoin | Re-sourcer `.env` après chaque nouvelle réservation |

**Étape 7.6 — Fermer le tunnel après la démo :**
```bash
scripts/tunnel.sh close 3001
```

**Étape 7.7 — Arrêter l'app :**
```
Stop the web service for FLGHT400 on port 3001.
```

### Exercice 8 (option) : FLGHT400 en mode web (5250 vers HTML)
Prototype des écrans 5250 en HTML, consulté par tunnel SSH. Tunnel nécessaire sur le port **57700** (voir ci-dessous). Durée estimée : 30 min. Aucune modification de la bibliothèque `FLGHT400`.

**Objectif :** montrer qu'un écran 5250 peut être reconstruit comme page web sans toucher au code RPG. Le prototype émule le terminal (fond noir, texte vert, police à chasse fixe, champs de saisie, barre des touches de fonction) à partir des sources DDS. Écrans visés : `FRS000DF` (connexion) et `FRS001DF` (création d'une commande). Le prototype ne manipule aucune donnée : il ne reproduit que l'affichage.

#### Étape 1 — Ouvrir le tunnel (port 57700)
Le serveur Apache écoute sur le port **57700** mais n'est joignable que depuis l'IBM i. Ouvrir le tunnel depuis `ibmi-lab/` :
```bash
scripts/tunnel.sh open
```
Le script lit `.env`, utilise la clé de `ssh/`, ouvre le port 57700 en arrière-plan et affiche `port 57700: open`. Fermeture : `scripts/tunnel.sh close`. Commande équivalente sans le script, au premier plan (arrêt avec `Ctrl+C`) :
```bash
set -a; . ./.env; set +a
ssh -i "$IBMI_SSH_KEY" -p "$IBMI_SSH_PORT" -L 57700:localhost:57700 -N "$IBMI_USER@$IBMI_HOST"
```

#### Étape 2 — Générer le prototype
Mode **IBM i Developer**, portée *Library List* avec `FLGHT400`. Envoyer à Bob :

```
Cette application FLGHT400 utilise des écrans 5250. Lis les sources DDS de
FLGHT400/QDDSSRCD, en commençant par FRS000DF et FRS001DF, puis génère un
prototype web en HTML, CSS et JavaScript sans framework qui émule fidèlement le
terminal vert sur noir (champs de saisie, barre de touches de fonction).
Déploie les fichiers dans l'IFS, dans /www/qiccqxml/htdocs/flght400/ : le
serveur Apache QICCQXML existant les sert déjà. Ne démarre aucun autre serveur.
Ne modifie aucun objet de la bibliothèque FLGHT400.
```
Résultat attendu : `index.html`, `style.css` et `app.js` sous `/www/qiccqxml/htdocs/flght400/`.

#### Étape 3 — Consulter le prototype
Ouvrir dans le navigateur : <http://localhost:57700/flght400/index.html>

#### Notes techniques
| Point | Détail |
|---|---|
| Ce qui marche | Tunnel SSH avec la clé de `ibmi-lab/ssh/` vers le port 57700. Vérifié le 27/10 : la page répond `HTTP 200` via le tunnel. |
| Serveur web | Instance Apache `QICCQXML` (sous-système `QHTTPSVR`). `/www/qiccqxml/conf/httpd.conf` : `Listen *:57700`, `DocumentRoot /www/qiccqxml/htdocs`. |
| Accès direct `http://<IP>:57700` | Ne répond pas depuis le Mac (délai dépassé, vérifié). Passer par le tunnel. |
| Serveur Python | À éviter : même problème et inutile puisqu'Apache sert déjà le dossier. |
| Clé SSH | La clé de l'IBM i est identique à celle de la réservation TechZone (même empreinte SHA256 vérifiée). Ne jamais l'afficher avec `cat`. |
| Autre port | `scripts/tunnel.sh open 3001` pour l'exercice 7 ; plusieurs ports en même temps : `open 57700 3001`. |

## Remise à zéro entre deux passages
- Rejouer les exercices sur la même machine : `scripts/install_flght400.sh --force` (confirmation requise). Cela ne restaure que la bibliothèque `FLGHT400` : les fichiers du lab web (`/www/qiccqxml/htdocs/flght400/`) et ceux de l'exercice 7 (dans le répertoire personnel) restent en place.
- Nouvelle machine : reprendre à A2. Les fichiers web et ceux de l'exercice 7 sont à régénérer (ils vivent sur l'ancienne machine, supprimée à la fin de la réservation).
- Prolonger la réservation : à faire dans l'interface TechZone ; aucun outil MCP ne le permet.

## Test live 5250 — valeurs de saisie pour FRS000 et FRS001

> ⚠️ Ces valeurs sont propres à l'application de démonstration FLGHT400. Elles n'ont aucune valeur en production.

Pour accéder aux écrans 5250, utiliser ACS (IBM Access Client Solutions) ou tout émulateur 5250 connecté à l'IBM i de la réservation TechZone (IP et mot de passe dans `ibmi-lab/.env` et `ibmi-lab/<IP>/<IP>.txt`). Démarrer `FRS000` depuis le menu `FLGHT400/QMNUSRC(FRSMNUDF)` ou par `CALL FLGHT400/FRS000`.

### Écran 1 — `FRS000DF` : LOGON

| Champ | Description | Valeur à saisir |
|---|---|---|
| **Agent Name** (`AGNTNM`) | 10 car., réf. AGENTS | `Julia` |
| **Password** (`PASSWD`) | 10 car., réf. AGENTS | `_set_password_` |

Tous les agents partagent le mot de passe **`_set_password_`**. Noms valides : `Julia`, `Mary`, `Alex`, `Debby`, `Amanda`, `Sharon`, `Suzanne`, `Helen`, `Terry`.

Valider avec **F10** (LOGON).

### Écran 2 — `FRS001DF` : Create Order (réservation de vol)

Deux parties : **Flight Information** (gauche) et **Ticket Order Information** (droite).

#### Partie gauche — Informations de vol

| Champ | Description | Valeur à saisir |
|---|---|---|
| **DOFMON / DOFDAY / DOFYR** | Date du vol MM / JJ / AAAA | `07` / `15` / `2025` |
| **FCITY** | From City | `Albany` |
| **TCITY** | To City | `Albuquerque` |

Après la saisie de FCITY et TCITY, appuyer sur **F6** (Flights) pour afficher les vols disponibles. Exemples issus de la base :

| Flight# | Airline | De | Vers | Départ | Arrivée | Prix |
|---|---|---|---|---|---|---|
| `1100001` | AMA | Albany (ABY) | Albuquerque (ALB) | 09:01 AM | 03:01 PM | $169 |
| `1200002` | AMA | Albany (ABY) | Albuquerque (ALB) | 09:03 AM | 11:03 AM | $179 |
| `1300003` | AMA | Albany (ABY) | Albuquerque (ALB) | 10:06 AM | 12:06 PM | $199 |
| `1500005` | AMA | Albany (ABY) | Albuquerque (ALB) | 12:55 PM | 02:55 PM | $299 |

#### Partie droite — Ticket Order

| Champ | Description | Valeur à saisir |
|---|---|---|
| **CUSNAM** | Nom du client (25A) | `Aaronson, Lynn` |
| **COSFIR** | First class (1A) | `Y` ou vide |
| **COSBUS** | Business class (1A) | `Y` ou vide |
| **COSECO** | Economy class (1A) | `Y` |
| **TCKETS** | Nombre de tickets (2D) | `2` |

Appuyer sur **F7** pour afficher la liste des clients et sélectionner `Aaronson, Lynn`. Valider avec **F10** (Make Reservation).

### Touches de fonction

| Touche | `FRS000` | `FRS001` |
|---|---|---|
| F2 | Refresh | Refresh |
| F3 | Exit | — |
| F4 | Agent Prompt | FROM Cities |
| F5 | — | TO Cities |
| F6 | — | Flights |
| F7 | — | Customers |
| F10 | LOGON | Make Reservation |
| F12 | Cancel | Cancel |

## Fichiers
| Fichier | Rôle |
|---|---|
| `scripts/tz.py` | Créer, suivre et récupérer les accès d'une réservation (sans afficher les secrets) |
| `scripts/install_flght400.sh` | Envoyer le fichier de sauvegarde, restaurer `FLGHT400`, vérifier |
| `scripts/tunnel.sh` | Ouvrir, fermer ou lister les tunnels SSH vers l'IBM i (port 57700 pour le site web, 3001 pour l'exercice 7) |
| `.env`, `ssh/ssh_private_key.pem` | Accès (secrets). Ignorés par git |
| `.tz_request_id`, `.tunnels/` | État local (identifiant de réservation, PID des tunnels). Sans secret, ignorés par git |
| `FLGHT400.FILE` | Sauvegarde de l'application (275 Mo). **Doit être dans `ibmi-lab/`** — si absent, télécharger depuis https://ibm.box.com/v/flight400-box (voir A0) |
| `Install-Flight400.sql` | Script SQL de restauration manuelle (Option 2). Récupérable sur GitHub si absent (voir A0b) |
| `scripts/setup_flight400_lab.sh` | Script de clonage multi-utilisateurs. Récupérable sur GitHub si absent (voir A0c) |

## Multi-utilisateurs (Bobathon / formation)

Cette section s'applique quand **plusieurs participants** jouent le lab en même temps sur le **même IBM i**.

### Principe

- `FLGHT400` = **master intact**. Ne jamais l'assigner à un participant. Bob le restaure via `install_flght400.sh` (phase A5).
- Chaque participant reçoit une copie isolée : `FLGHT401`, `FLGHT402`… jusqu'à `FLGHT450`.
- La copie se fait via `CPYLIB` (CL IBM i). Chaque participant utilise **uniquement sa bibliothèque** tout au long du lab.
- Pour l'exercice 7, chaque numéro `nn` a son port dédié : participant 1 → port `3001`, participant 2 → port `3002`, etc.

### Avant d'installer : annoncer le nombre de participants

Décider du nombre **avant** de lancer l'installation, pour préparer les clones en une seule passe.

### Option 1 — Script automatique (recommandé, ≥ 2 participants)

Vérifier que `scripts/setup_flight400_lab.sh` est présent (voir A0c si absent).

**Étape 1 — Uploader le save file et le script sur l'IBM i**

```bash
cd ibmi-lab
set -a; . ./.env; set +a
# Upload du save file (si pas déjà fait par install_flght400.sh)
scp -i "$IBMI_SSH_KEY" -P "$IBMI_SSH_PORT" FLGHT400.FILE "$IBMI_USER@$IBMI_HOST:$IBMI_IFS_DIR/FLGHT400.FILE"
# Upload du script
scp -i "$IBMI_SSH_KEY" -P "$IBMI_SSH_PORT" scripts/setup_flight400_lab.sh "$IBMI_USER@$IBMI_HOST:$IBMI_IFS_DIR/setup_flight400_lab.sh"
```

**Étape 2 — Dry run (prévisualisation, aucun changement)**

```bash
ssh -i "$IBMI_SSH_KEY" -p "$IBMI_SSH_PORT" "$IBMI_USER@$IBMI_HOST" \
  "bash $IBMI_IFS_DIR/setup_flight400_lab.sh -f $IBMI_IFS_DIR/FLGHT400.FILE -n <N> --dry-run"
```

Remplacer `<N>` par le nombre de participants. Vérifier que les bibliothèques affichées sont `FLGHT401` … `FLGHT4NN`.

**Étape 3 — Exécution réelle**

```bash
ssh -i "$IBMI_SSH_KEY" -p "$IBMI_SSH_PORT" "$IBMI_USER@$IBMI_HOST" \
  "bash $IBMI_IFS_DIR/setup_flight400_lab.sh -f $IBMI_IFS_DIR/FLGHT400.FILE -n <N>"
```

Le script :
1. Restaure `FLGHT400` depuis le save file (si pas déjà fait).
2. Clone `FLGHT401` … `FLGHT4NN` via `CPYLIB`.
3. Affiche le tableau participants / bibliothèque / port / URL React — à distribuer aux participants.

**Variantes utiles**

| Besoin | Commande |
|---|---|
| Master déjà restauré (phase A5 faite), clones seulement | `setup_flight400_lab.sh --skip-restore -n <N>` |
| Restaurer le master seulement, pas de clones | `setup_flight400_lab.sh -f <IFS_PATH> --restore-only` |
| Plage explicite (ex. participants 3 à 8) | `setup_flight400_lab.sh -f <IFS_PATH> --range 3 8` |
| Écraser des bibliothèques existantes | ajouter `--force` |

### Option 2 — CPYLIB manuel via Bob (petit groupe)

Dans une tâche **IBM i Developer** (portée *Library List*) :

> « Run `CPYLIB FROMLIB(FLGHT400) TOLIB(FLGHT401)` and so on so each participant gets their own isolated copy. I have X participants. »

Bob enchaîne les `CPYLIB` jusqu'à `FLGHT4XX`.

### Instructions aux participants

Chaque participant :
1. Reçoit son numéro `nn` (01, 02…).
2. Ajoute **uniquement** `FLGHT4nn` à sa User Library List (pas `FLGHT400`).
3. Remplace `FLGHT400` par `FLGHT4nn` dans **toutes** les instructions du lab.
4. Pour l'exercice 7, utilise le port `30nn` (ex. participant 3 → port `3003`).

### Remise à zéro d'un participant

Si un participant corrompt sa bibliothèque :

```bash
ssh -i "$IBMI_SSH_KEY" -p "$IBMI_SSH_PORT" "$IBMI_USER@$IBMI_HOST" \
  "bash $IBMI_IFS_DIR/setup_flight400_lab.sh --skip-restore --range <nn> <nn> --force"
```

Cela recrée uniquement `FLGHT4nn` depuis le master `FLGHT400`, sans toucher aux autres.

> Source : [`flight400-demo/instructor-setup/README.md`](https://github.com/bmarolleau/flight400-demo/blob/main/instructor-setup/README.md)

