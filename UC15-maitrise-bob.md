# UC 15 — Maîtrise de Bob

> **Catégorie :** Utilisation de BOB  
> **Priorité dans le POC :** 1 (premier use case — prérequis à tous les autres)  
> **Durée estimée :** 1 à 2 journées (selon le niveau initial de l'équipe)  
> **Mode Bob utilisé :** ASK, Code, Plan, Agent, IBM i Developer, IBM i Database  

---

## Objectif

Rendre chaque membre de l'équipe ACME autonome sur Bob avant d'aborder tout autre use case :

- Comprendre les **modes natifs** de Bob et savoir lequel utiliser selon la situation
- Savoir **créer, importer et utiliser des modes personnalisés** (custom modes)
- Comprendre la notion de **contexte** (scope) et comment le fournir à Bob
- Savoir utiliser les **workflows** et les **Skills**
- Maîtriser la gestion des **Bob Coins** (consommation, surveillance)
- Pratiquer sur une application IBM i réelle grâce au **lab Flight400**

---

## 1. Accès au lab Flight400

### Référence officielle

- **Lab complet :** [IBM i Application Modernization with Bob — Flight400](https://github.com/bmarolleau/flight400-demo#readme)
- **Guide applicatif Flight400 :** [FLIGHT400-GUIDE.md](https://github.com/bmarolleau/flight400-demo/blob/main/FLIGHT400-GUIDE.md)

### Accès à un IBM i pour le lab

Deux situations possibles :

| Situation | Action |
|-----------|--------|
| **L'admin IBM i de ACME accorde des droits** sur l'IBM i de test | Utiliser directement cet IBM i. Vérifier : profil utilisateur, accès aux bibliothèques sources, droits `*USE` et `*CHANGE` sur les objets, port SSH ouvert pour Code for IBM i. |
| **Droits non disponibles ou IBM i de test non accessible** | Provisionner un IBM i TechZone en **urgence** (voir section ci-dessous). |

### Provisionner un IBM i sur TechZone (si nécessaire)

> **Documentation de référence :** [How to Get an IBM i Virtual Machine (LPAR)](https://github.com/bmarolleau/flight400-demo#how-to-get-an-ibm-i-virtual-machine-aka-lpar)

**Étapes :**

1. Aller sur [techzone.ibm.com](https://techzone.ibm.com) et se connecter avec l'IBM ID du POC.
2. Rechercher **"IBM i"** dans le catalogue → sélectionner **IBM i 7.6 - Sandbox**.
3. Cliquer **Reserve** :
   - Objectif : *Practice / Self-Education*
   - Durée : minimum 8 heures (extensible)
   - Région : la plus proche géographiquement
4. Dans quelques minutes, un e-mail fournit : hostname, port, profil utilisateur, mot de passe.

> ⚠️ **Durée maximale de réservation :** 4 à 6 semaines avec un **OPPY valide** (Opportunity valide dans Salesforce). Des extensions sont possibles — vérifier les conditions en vigueur sur TechZone au moment de la réservation. Ne pas attendre la fin de la réservation pour demander une extension.

---

## 2. Description du lab Flight400

Le lab Flight400 est l'application de référence pour pratiquer Bob sur IBM i. Il s'agit d'une application de réservation de vols (style années 90) comprenant :

- Des programmes **RPG OPM** (fixe colonné, style RPG III/IV)
- Des fichiers **DDS** (écrans 5250, fichiers physiques et logiques)
- Une base de données **Db2 for i** avec les tables FLIGHTS, ORDERS, CUSTOMERS, AGENTS
- Des programmes **CL** de pilotage

Le lab est structuré en **6 exercices progressifs** couvrant tous les axes de modernisation IBM i avec Bob.

---

## 3. Étapes clés du lab — ce qu'il faut faire et retenir

### Phase de setup (Prérequis terrain)

| Étape | Action | Lesson apprise |
|-------|--------|---------------|
| S1 | Créer le workspace local dans Bob (`File → Open Folder`) | Bob travaille dans un **workspace local** — toujours ouvrir un dossier avant de démarrer. |
| S2 | Télécharger `FLGHT400.FILE` (save file) depuis le Box Folder | Les sources IBM i ne sont pas dans Git — elles s'installent via un save file restauré sur le LPAR. |
| S3 | Connecter Bob à l'IBM i (`IBM i icon → New Connection`) | La connexion SSH/ODBC est la **fondation** de tout le travail. Si elle ne fonctionne pas, rien ne fonctionne. Tester immédiatement après configuration. |
| S4 | Déployer les fichiers vers l'IFS (`Deploy Selected Files`) | L'IFS est le "pont" entre le workspace local et IBM i. Le chemin IFS déployé est affiché dans le panneau de sortie — le noter. |
| S5 | Exécuter le script SQL `Install-Flight400.sql` via `Run SQL Statements` | On peut **exécuter du SQL directement depuis Bob** via le menu contextuel — pas besoin de passer par un terminal 5250. |
| S6 | Vérifier que la librairie `FLGHT400` est dans la liste de bibliothèques | Le **Library List** dans les paramètres Code for IBM i est crucial : Bob y cherche les membres sources et objets en premier. |

---

### Exercice 1 (Optionnel) — Générer une app React depuis un écran 5250

> Cet exercice est optionnel mais très **démonstratif** pour convaincre le management. À réaliser si le temps le permet.

| Étape | Action | Lesson apprise |
|-------|--------|---------------|
| E1.1 | Créer un **Skill** local depuis `SAMPLE-SKILL.md` en mode Agent | Un Skill est un fichier Markdown qui enrichit le contexte de Bob. On peut en créer des **skills maison** pour les normes ACME. |
| E1.2 | Basculer en mode **IBM i Developer**, ajouter `FLGHT400` comme scope (`+` button) | Le **scope** (contexte) est fondamental : sans le bon scope, Bob ne "voit" pas les sources IBM i. Toujours définir le scope avant de promter. |
| E1.3 | Coller une capture d'écran 5250 dans le chat et demander la génération React | Bob accepte des **images dans le prompt** (mode multimodal). Très utile pour décrire des écrans existants. |
| E1.4 | Lire le résultat : app React générée, fichiers créés dans l'IFS | En mode **Agent**, Bob peut **écrire des fichiers directement sur l'IFS** IBM i — pas seulement générer du texte. |

---

### Exercice 2 — Documentation et architecture (UC 4-5-6 en pratique)

| Étape | Action | Lesson apprise |
|-------|--------|---------------|
| E2.1 | Parcourir l'Object Browser : `*PGM`, `*FILE`, `*MENU`, membres sources | L'**Object Browser** de Code for IBM i est intégré dans Bob. On navigue dans QSYS sans terminal 5250. |
| E2.2 | Ouvrir `FRS001DF` et cliquer **Preview All** | Le **DDS Previewer** affiche visuellement l'écran 5250 — indispensable avant de moderniser un display file. |
| E2.3 | Ajouter **QSYS Library List** comme scope, demander l'architecture en Markdown | Avec le bon scope, Bob génère une **doc architecture complète** en quelques secondes — diagramme Mermaid inclus. |
| E2.4 | Passer en mode **IBM i Database**, taper `/erd FLGHT400` | La **slash command `/erd`** génère un diagramme ERD Mermaid automatiquement — aucun SQL manuel requis. |
| E2.5 | (Optionnel) Installer **Draw.io Integration** et générer un `.drawio` | Les extensions VS Code tierces fonctionnent dans Bob. Draw.io génère des diagrammes éditables directement dans l'IDE. |

---

### Exercice 3 — Modernisation RPG OPM → ILE Free Format (UC 3-7-8 en pratique)

| Étape | Action | Lesson apprise |
|-------|--------|---------------|
| E3.1 | Ouvrir `FRS409` dans l'éditeur, taper *"What does this program do?"* | Bob comprend le **code RPG OPM fixe colonné** sans aucune préparation. Le simple fait d'avoir le fichier ouvert suffit comme contexte. |
| E3.2 | Taper *"Can you modernize this program?"* | Bob propose le **workflow RPG Modernization** — un workflow guidé spécialisé, différent du mode Agent générique. |
| E3.3 | Choisir entre **Agentic mode** et **Workflow mode** | Il existe deux approches : workflow (structuré, rapide) vs agent (flexible, itératif). Workflow recommandé pour les patterns connus. |
| E3.4 | Bob compile avec `CRTBNDRPG` et retourne le résultat | Bob peut **déclencher des compilations IBM i** directement depuis le chat. Le résultat de compilation s'affiche dans le panneau de sortie. |
| E3.5 | Consulter le **Modernization Summary Report** généré automatiquement | Bob génère un **rapport de modernisation** Markdown automatiquement après chaque conversion — à conserver comme documentation. |
| E3.6 | Cliquer sur **File Changed** en bas du panneau Bob | Le panneau **File Changed** affiche le diff avant/après du source modifié. Toujours le vérifier avant de committer. |

---

### Exercice 4 — Field Expansion : ajout d'un champ de bout en bout (UC 3-7-8 avancé)

| Étape | Action | Lesson apprise |
|-------|--------|---------------|
| E4.1 | Demander à Bob d'afficher l'écran `FRS021DF` et lister ses champs | On peut combiner **navigation dans l'IDE et questions dans le chat** en un seul prompt. |
| E4.2 | Demander l'ajout du champ `SFLHRS` dans le DDS + compilation | Bob modifie le DDS source ET compile — tout dans un seul échange conversationnel. |
| E4.3 | Demander une **analyse d'impact** complète avant de toucher les programmes | L'analyse d'impact via Bob (`QSYS2.BOUND_MODULE_INFO`, `SYSCOLUMNS`, `SYSKEYS`) est **systématique et non négociable** avant tout changement structurel. |
| E4.4 | Bob exécute `ALTER TABLE` pour ajouter la colonne SQL | Bob génère et exécute le DDL SQL directement — pas besoin de passer par ACS ou un terminal. |
| E4.5 | Bob propage le changement dans les programmes RPG impactés | Bob traite l'impact en **cascade** : DDS → DB → RPG → compilation → validation. C'est le workflow "full stack IBM i". |
| E4.6 | Validation finale : colonne en DB + programme compilé + écran prévisualisé | Bob **valide lui-même** en fin de cycle — toujours demander cette validation explicitement dans le prompt. |

---

### Exercice 5 — Optimisation SQL et Index Advisor (UC 5-11 en pratique)

| Étape | Action | Lesson apprise |
|-------|--------|---------------|
| E5.1 | Passer en mode **IBM i Database** | Le mode IBM i Database est **distinct** du mode IBM i Developer — il est optimisé pour l'analyse et la génération SQL Db2 for i. |
| E5.2 | Utiliser la slash command `/review` sur une requête SQL | La commande `/review` est une **slash command spécialisée** — apprendre les slash commands disponibles dans chaque mode. |
| E5.3 | Demander l'exécution du **Index Advisor** | Bob peut interroger le **Index Advisor Db2 for i** et générer les `CREATE INDEX` DDL recommandés. |
| E5.4 | Utiliser le **workflow Index Advisor** (workflow picker) | Le workflow Index Advisor est un workflow guidé distinct des slash commands — accès via le **workflow picker** dans le chat. |

---

### Exercice 6 — Questions système en langage naturel

| Étape | Action | Lesson apprise |
|-------|--------|---------------|
| E6.1 | *"Quels jobs consomment le plus de CPU ?"* | Bob interroge `QSYS2.ACTIVE_JOB_INFO` — on peut poser des **questions système** en français ou anglais, sans connaître le SQL. |
| E6.2 | *"Quels programmes n'ont pas été recompilés depuis 5 ans ?"* | Bob interroge `QSYS2.OBJECT_STATISTICS` — puissant pour **constituer un backlog de modernisation** automatiquement. |

---

## 4. Synthèse des lessons apprises pour la maîtrise de Bob

| # | Concept clé | Ce qu'il faut savoir |
|---|-------------|----------------------|
| L1 | **Choisir le bon mode** | ASK = questions générales ; IBM i Developer = code RPG/CL/DDS ; IBM i Database = SQL Db2 for i ; Agent = actions multi-étapes autonomes. Ne jamais rester en mode ASK pour générer du code IBM i. |
| L2 | **Toujours définir le scope** | Sans scope (`+` button → Library List ou QSYS), Bob n'a pas accès aux sources IBM i. Le scope est le premier geste avant tout prompt de production. |
| L3 | **Workspace local obligatoire** | Bob travaille dans un dossier local ouvert. Sans workspace, les fichiers générés n'ont pas d'emplacement. |
| L4 | **Connexion IBM i = fondation** | Si la connexion SSH/ODBC est down, aucune action IBM i n'est possible. Toujours vérifier la connexion en début de session. |
| L5 | **Library List dans Code for IBM i** | La liste de bibliothèques détermine où Bob cherche. `FLGHT400` (ou la lib du client) doit y figurer avant tout travail. |
| L6 | **Workflows vs Agent vs Slash commands** | Trois niveaux d'interaction : slash commands (actions rapides), workflows (guidés structurés), agent (itératif autonome). Commencer par les workflows pour les tâches connues. |
| L7 | **Skills = contexte persistant** | Un Skill est un fichier `.md` qui injecte du contexte dans chaque session. Créer un Skill "ACME" avec les normes de développement du client dès le début du POC. |
| L8 | **File Changed = validation obligatoire** | Toujours vérifier le diff dans le panneau "File Changed" avant d'accepter une modification générée par Bob. |
| L9 | **Bob génère ET compile** | Bob peut déclencher `CRTBNDRPG`, `CRTDSPF`, etc. directement depuis le chat. Vérifier les droits de compilation sur l'IBM i de test. |
| L10 | **Gestion des Bob Coins** | Chaque appel au LLM consomme des Coins. Les modes Agent et la génération d'apps (ex. React) sont les plus consommateurs. Surveiller le compteur dans les settings du profil TechZone. |
| L11 | **Images dans le prompt** | Bob accepte des captures d'écran (5250, architecture, etc.) dans le prompt en mode multimodal. Très utile pour décrire l'existant. |
| L12 | **Analyse d'impact avant tout changement structurel** | Demander systématiquement une analyse d'impact avant de modifier un fichier DDS, une table ou un programme qui a des dépendants. |
| L13 | **Contexte implicite vs scope explicite** | Quand un fichier est **ouvert dans l'éditeur**, Bob l'utilise automatiquement comme contexte implicite — `"What does this program do?"` fonctionne sans rien préciser. Le scope (`+`) est nécessaire quand Bob doit naviguer dans plusieurs sources. Comprendre cette différence évite des résultats inattendus. → *Détails : voir § 6.1* |
| L14 | **Syntaxe `@mention`** | Dans les prompts, on peut référencer explicitement une bibliothèque ou un objet avec `@NOM` (ex. `@FLGHT400`). Cette syntaxe cible précisément les sources et améliore la qualité des réponses. → *Détails : voir § 6.2* |
| L15 | **Taille du contexte (fenêtre de tokens)** | Fournir une bibliothèque entière en scope peut saturer la fenêtre de contexte du LLM. Symptômes : résultats tronqués, réponses vagues. Remède : réduire le scope au fichier ou à la sous-bibliothèque concernée. → *Détails : voir § 6.3* |
| L16 | **Approbation des actions en mode Agent** | En mode Agent, Bob demande une **confirmation explicite** avant d'exécuter chaque action (écriture de fichier, compilation, appel CL…). Ne jamais tout approuver sans lire. → *Détails : voir § 6.4* |

---

## 5. Mode personnalisé recommandé pour ACME

Dès la Phase 0, créer un **custom mode "ACME IBM i Developer"** dans Bob qui intègre :

- Les **normes de développement** RPG de ACME (nommage, structure, copyrights)
- Les **conventions de nommage** des objets (bibliothèques, membres, variables)
- Le **contexte métier** (secteur grande distribution, vocabulaire spécifique)
- La **langue de travail** (français)

Ce mode sera réutilisé pour tous les UC suivants et évitera de répéter ces informations dans chaque prompt.

> 💡 Pour créer un custom mode : dans Bob, ouvrir la palette de commandes (`Cmd+Shift+P`) → *"Bob: Create Custom Mode"* → remplir le formulaire → sauvegarder. Le mode est alors disponible dans le sélecteur de mode pour toute l'équipe (si partagé via Git ou export).

---

## 6. Maîtrise avancée — Ce que le lab n'explique pas assez

### 6.1 Contexte implicite vs scope explicite

**Pourquoi ça existe ?**
Bob est un LLM : il ne peut répondre qu'à partir de ce qu'on lui donne. Deux mécanismes lui fournissent du contexte :

- **Contexte implicite** : le fichier actuellement **ouvert et actif dans l'éditeur** est automatiquement transmis à Bob dans chaque prompt. C'est pourquoi `"What does this program do?"` fonctionne sans rien préciser — Bob "voit" le source affiché.
- **Scope explicite** : le bouton `+` (en haut du panneau chat) permet d'ajouter des sources supplémentaires : une bibliothèque entière, la Library List, l'IFS, ou un dossier local. Bob peut alors naviguer et référencer ces sources dans sa réponse.

**Comment les utiliser correctement ?**

| Situation | Mécanisme à utiliser |
|-----------|----------------------|
| Poser une question sur le programme ouvert | Contexte implicite suffit — ne pas ajouter de scope inutile |
| Demander les dépendances entre plusieurs programmes | Scope → Library List (`FLGHT400`) |
| Générer une architecture globale | Scope → QSYS Library List |
| Modifier un programme en référençant ses copybooks | Scope → dossier source ou Library List |

**Erreurs fréquentes :**
- Ajouter un scope trop large (toute la QSYS) alors que seul le fichier ouvert est nécessaire → ralentissement + résultats dilués.
- Oublier de définir le scope pour une tâche multi-sources → Bob répond sur le seul fichier ouvert et manque le contexte global.
- Changer de fichier actif sans s'en rendre compte → le contexte implicite change silencieusement entre deux prompts.

**Bonne pratique :** En début de session, ouvrir le fichier cible ET définir le scope Library List. Pour les questions ponctuelles sur un programme isolé, le fichier ouvert seul suffit.

---

### 6.2 La syntaxe `@mention`

**Pourquoi ça existe ?**
Dans un prompt, Bob doit savoir à quel objet on fait référence. La syntaxe `@NOM` est un **ancrage explicite** : elle force Bob à chercher et charger cet objet précis dans son contexte, indépendamment de ce qui est ouvert ou du scope défini.

**Comment l'utiliser ?**

```
@FLGHT400          → référence la bibliothèque FLGHT400
@FRS409            → référence le membre source FRS409 (dans la lib du scope)
@FLIGHTS           → référence la table/fichier FLIGHTS
```

Exemples de prompts avec `@mention` :

```
"Génère la documentation de @FRS409 et liste ses dépendances dans @FLGHT400."
"Compare les champs de @FLIGHTS avec ceux utilisés dans @FRS021."
"Effectue une analyse d'impact de la suppression de @FRCITY dans @FLGHT400."
```

**À quoi ça sert concrètement ?**
- Évite les ambiguïtés quand plusieurs programmes portent des noms similaires.
- Permet de croiser plusieurs objets dans un seul prompt sans les ouvrir manuellement.
- Accélère les prompts complexes (impact analysis, cross-référence) en guidant explicitement Bob.

**Erreurs fréquentes :**
- Utiliser `@NOM` d'un objet qui n'est pas dans le scope → Bob ne le trouve pas et peut halluciner ou ignorer la mention silencieusement. Toujours s'assurer que l'objet référencé est accessible via le scope défini.
- Confondre le nom du membre source avec le nom de l'objet compilé (ex. `@FRS409` vs `@FRS409.PGM`) → préciser le type si Bob remonte le mauvais objet.

---

### 6.3 Gestion de la fenêtre de contexte (tokens)

**Pourquoi c'est un problème ?**
Le LLM sous-jacent à Bob a une **fenêtre de contexte limitée** (nombre maximum de tokens traités en une seule fois). Quand le scope contient trop de sources ou que le programme est très long, Bob atteint cette limite. Les conséquences sont insidieuses : pas de message d'erreur clair, mais des réponses incomplètes, tronquées ou génériques.

**Symptômes à reconnaître :**

| Symptôme | Cause probable |
|----------|---------------|
| Réponse vague alors que le code est précis | Scope trop large — Bob n'a pas tout lu |
| Documentation générée incomplète (s'arrête en cours) | Programme trop long pour un seul prompt |
| Bob "oublie" ce qui a été dit 10 prompts plus tôt | Historique de conversation trop long |
| Résultat différent d'une session à l'autre sur le même code | Contexte inconsistant entre les sessions |

**Comment éviter le problème :**

- **Réduire le scope** : ne charger que la bibliothèque ou le fichier strictement nécessaire, pas toute la QSYS.
- **Découper le travail** : ne pas demander la documentation de 50 programmes en un seul prompt — travailler programme par programme.
- **Nouvelles sessions** : pour les longues sessions de travail, ouvrir une nouvelle conversation Bob pour repartir avec un contexte propre. L'historique de conversation s'accumule et consomme des tokens.
- **Limiter les fichiers ouverts** : fermer les onglets inutiles dans l'éditeur — les fichiers ouverts peuvent être inclus dans le contexte implicite selon le mode.

**Comment gérer si ça arrive déjà :**
1. Ouvrir une **nouvelle conversation** Bob (`+` en haut du panneau chat).
2. Redéfinir le scope minimal nécessaire.
3. Reformuler le prompt en étant plus ciblé (un programme, une question précise).
4. Si le programme source est très long (> 500 lignes), demander à Bob de traiter section par section : `"Explique uniquement la section de calcul des prix dans ce programme."`.

---

### 6.4 Approbation des actions en mode Agent

**Pourquoi ce mécanisme existe ?**
Le mode Agent permet à Bob d'exécuter des **actions réelles et irréversibles** sur l'IBM i : écriture de fichiers dans l'IFS, modification de membres sources, compilation, exécution de commandes CL. Avant chaque action, Bob présente un **plan d'approbation** que l'utilisateur doit valider explicitement. C'est le filet de sécurité principal.

**Comment ça fonctionne :**

1. Bob analyse la demande et décompose en actions unitaires.
2. Pour chaque action, Bob affiche :
   - **Ce qu'il va faire** (ex. : `Écrire FRS409.RPGLE dans FLGHT400/QRPGSRC`)
   - **Pourquoi** (justification)
   - Un bouton **Approve** / **Reject**
3. L'utilisateur approuve ou rejette chaque action individuellement.
4. Bob n'exécute que les actions approuvées.

**Ce qu'il faut vérifier avant d'approuver :**

| Point de contrôle | Question à se poser |
|-------------------|---------------------|
| Chemin de destination | Est-ce la bonne bibliothèque / le bon membre ? |
| Type d'action | Écriture ? Remplacement ? Suppression ? |
| Commande CL proposée | La commande est-elle correcte et sans effet de bord ? |
| Périmètre | Bob touche-t-il plus d'objets que demandé ? |

**Erreurs fréquentes :**
- **Tout approuver sans lire** (clic rapide sur Approve) → risque d'écraser un source existant ou de compiler sur le mauvais LPAR.
- **Rejeter sans lire non plus** → bloquer Bob sur une action légitime et se retrouver avec une tâche à moitié faite.
- **Ne pas vérifier la bibliothèque cible** → en environnement multi-LPAR (dev/test/prod), Bob peut proposer la mauvaise destination si le scope n'est pas correctement défini.

**Bonne pratique :** Lire systématiquement le **chemin complet** de chaque action avant d'approuver. En cas de doute sur une action, la rejeter et reformuler le prompt avec plus de précision sur la destination.

---

## 7. Check-list de validation UC 15

Avant de passer à l'UC suivant, chaque membre de l'équipe doit pouvoir répondre **oui** à chaque point :

- [ ] Je sais basculer entre les modes ASK, IBM i Developer, IBM i Database et Agent
- [ ] Je sais ajouter un scope (Library List, QSYS) avant un prompt
- [ ] Je sais ouvrir l'Object Browser et naviguer dans une bibliothèque IBM i
- [ ] Je sais utiliser le DDS Previewer pour visualiser un écran 5250
- [ ] J'ai utilisé au moins une slash command (`/erd`, `/review`)
- [ ] J'ai déclenché au moins un workflow (RPG Modernization ou Index Advisor)
- [ ] Je sais créer ou importer un Skill
- [ ] Je sais lire le panneau "File Changed" pour valider un diff
- [ ] Je comprends comment Bob Coins est consommé et où le surveiller
- [ ] J'ai exécuté au moins un exercice complet du lab Flight400
- [ ] Je comprends la différence entre contexte implicite (fichier ouvert) et scope explicite (`+`)
- [ ] Je sais utiliser la syntaxe `@mention` dans un prompt
- [ ] Je sais reconnaître les symptômes d'une fenêtre de contexte saturée et comment y remédier
- [ ] Je sais lire le plan d'approbation en mode Agent et je vérifie le chemin cible avant d'approuver

---

*Fiche UC 15 — Document évolutif à mettre à jour au fil du POC.*
