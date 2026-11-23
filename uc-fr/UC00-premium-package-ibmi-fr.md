# UC 00 — Bob for IBM i Premium Package : présentation et mise en œuvre

> **Cette fiche est le point d'entrée de tous les autres UC du POC.**
> Le Premium Package IBM i prime sur tous les prompts historiques. Avant d'aborder n'importe quel UC,
> vérifier que le Premium Package est installé et que ses capacités sont comprises.
>
> **Source de référence :** [IBM Bob Premium Package for i — Overview](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/bob-for-i-index). Fiche alignée sur la documentation en ligne consultée début novembre 2026 (aucun changelog daté n'est publié).

---

## ⭐ Les capacités du Premium Package IBM i

Le Premium Package étend Bob avec des capacités dédiées à IBM i, qui s'activent directement depuis l'IDE Bob :

| Capacité | Type | Déclenchement |
|---|---|---|
| 2 modes : IBM i Developer, IBM i Database | Modes | Sélecteur de mode Bob |
| RPG Modernization | Workflow guidé | Bouton *Start Workflow* en haut du chat |
| SQL Index Strategy Advisor | Workflow guidé | Bouton *Start Workflow* en haut du chat |
| RPGUnit Test Plan Creation | Workflow guidé | Bouton *Start Workflow* en haut du chat |
| RPGUnit Test Suite Implementation | Workflow guidé | Bouton *Start Workflow* en haut du chat |
| Business Rules Extraction | Workflow guidé | Bouton *Start Workflow* en haut du chat |
| Understand (Dashboard + 6 outils + 3 skills) | Cartographie applicative | Commande *Open Understand Dashboard* ou icône du Dashboard dans la barre de titre du chat Bob |
| `/init` | Slash command | Zone de saisie du chat Bob |
| `/erd` | Slash command | Zone de saisie du chat Bob |
| `/review_SQL` | Slash command | Zone de saisie du chat Bob |
| `/review_rpg` | Slash command | Zone de saisie du chat Bob |
| 71 skills IBM i | Skills contextuelles | Activées automatiquement par Bob selon le contexte |
| Outils IBM i (lecture/écriture QSYS-IFS, CL, SQL, PASE, build, test, doc) | Outils natifs (connexion native du Premium Package) | Appelés par Bob dans les deux modes IBM i |
| Recherche documentaire RAG | Outil | Appelé par Bob (ou demandé explicitement) |

---

## Prérequis

Source : [Prerequisites](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/prerequisites).

- **Abonnements :** IBM Bob **et** IBM Bob Premium Package for i actifs
- **Poste :** Windows x64 ou macOS ARM64 (Apple Silicon). Windows ARM64, macOS Intel et Linux peuvent fonctionner mais ne sont pas supportés
- **Versions :** IBM Bob IDE **2.0.0 ou supérieure** ; extension Premium Package for i **1.0.0 ou supérieure**
- **Installation :** Marketplace Bob → installer le package **Premium IBM i** (voir *installing premium packages*)
- **Extensions open source installées automatiquement** (versions minimales) : Code for IBM i 3.0.10, Db2 for IBM i 2.0.1, RPGLE 0.33.7, CLLE 1.2.6, IBM i Testing 1.3.4, IBMi Languages 0.6.25
- **IBM i :** version **7.4 ou supérieure** ; Database PTF Group à jour (pour 7.4 : SF99704 niveau 13 minimum) ; démon SSH démarré (`STRTCPSVR *SSHD`, produit 5733-SC1, profil `QSSHD` activé). Code for IBM i se connecte **uniquement en SSH**
- **Understand :** à installer sur l'IBM i via le Dashboard (voir §2) ; le réglage `vscode-ibmi-bob.understand.installLibrary` doit d'abord désigner une bibliothèque dédiée
- **RPGUnit :** installé si utilisation des workflows RPGUnit (bibliothèques `RPGUNIT` et `QDEVTOOLS` dans la library list ; installation proposée par l'extension IBM i Testing)
- **Bob Coins :** prévoir ● ● ○ pour les workflows, ● ○ ○ pour les slash commands

---

## 0. Modes, workspaces et AGENTS.md

### Les deux modes IBM i

Sources : [Modes](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/modes), [Workspaces](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workspaces).

| Mode | Usage | Remarque |
|---|---|---|
| **IBM i Developer** | Expliquer, générer, compiler, tester et documenter du code (RPG OPM/ILE, CL, DDS, SQL, COBOL), en local ou sur IBM i (QSYS/IFS) | Remplace le mode Agent pour tout cas d'usage IBM i |
| **IBM i Database** | Écrire, moderniser, optimiser et revoir du SQL Db2 for i ; analyser les index | Idem pour les questions SQL |

Les deux modes injectent automatiquement le contexte de la connexion Code for IBM i (hôte, profil, version d'OS, CCSID, bibliothèque courante, library list, `AGENTS.md`). Le mode Database y ajoute le job SQL actif. Ils sont personnalisables comme tout mode Bob. On peut basculer vers Plan ou Agent si la tâche sort d'IBM i.

### Les trois workspaces

| Workspace | Contenu | Prérequis |
|---|---|---|
| **Local** | Dossier du poste (défaut) | — |
| **Library List** ⭐ | Bibliothèques, objets et membres de la User Library List Code for IBM i | Connexion IBM i |
| **IFS Working Directory** | Fichiers et répertoires du répertoire IFS configuré (défaut `/home/<utilisateur>`) | Connexion IBM i |

Pour tout exercice IBM i, le workspace **Library List** est l'espace par défaut. Les tâches sont rattachées à un workspace et filtrables par workspace dans l'historique. Les mentions `@` permettent de cibler une bibliothèque, un membre ouvert, ou un fichier de l'IFS.

### AGENTS.md

Bob charge automatiquement `AGENTS.md` dans les deux modes IBM i :

| Workspace | Emplacement |
|---|---|
| QSYS | `{bibliothèque courante}/QBOBSRC/AGENTS.MD` |
| IFS | `{répertoire de travail IFS}/.bob/AGENTS.md` |

`/init` le génère ou le met à jour (voir §3.1).

---

## 1. Les 5 workflows guidés

Source : [Workflows](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows).

Chaque workflow enchaîne des étapes **agentiques** (sous-agents ou sous-tâches où Bob raisonne et utilise ses outils) et des étapes **non agentiques** (opérations déterministes, qui réduisent la consommation de tokens et rendent le résultat plus prévisible).

**Comment les lancer :**
- Bouton **Start Workflow** en haut du chat → choisir le workspace → **Start** sur le workflow voulu (le lab FLGHT400 parle d'icône *Workflows*) ; ou
- Bob peut **suggérer** un workflow dans le chat quand la demande s'y prête → **Start workflow**.

> La plupart des workflows ne sont visibles que lorsqu'on est connecté à un IBM i.

| Workflow | Workspaces supportés |
|---|---|
| RPG Modernization | Local, Library List |
| Business Rules Extraction | Local, IFS Working Directory, Library List |
| RPGUnit Test Plan Creation | Local, Library List |
| RPGUnit Test Suite Implementation | Local, Library List |
| SQL Index Strategy Advisor | Library List uniquement |

### 1.1 RPG Modernization

**Ce qu'il fait :** convertit un programme source RPG (OPM RPG II/III/400 ou ILE RPG IV colonné) vers le format libre en suivant les bonnes pratiques. Il détecte automatiquement la variante RPG et choisit le chemin de conversion. Il détecte aussi les programmes à cycle (fichiers Primary/Secondary) et charge alors la skill `rpg-cycle-explained`. La conversion se fait en **deux phases** : (1) externalisation des I-specs et O-specs, (2) conversion des spécifications H, F, D et C restantes. Une compilation préalable optionnelle valide le source, et un rapport de synthèse clôt le workflow.

**Comment le lancer :**
1. Ouvrir le source à moderniser dans l'éditeur
2. **Start Workflow** → **RPG Modernization**
3. Saisir : bibliothèque source, fichier source, membre, bibliothèque de sortie, fichier de sortie, membre de sortie
4. Laisser Bob exécuter le workflow pas à pas

**Documentation officielle :** [Workflows — RPG Modernization](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

**UC concernés :** UC13 (RPG Colonné→FREE), UC11 (Restructuration). UC12 (COBOL→RPG) n'utilise pas ce workflow, seulement comme référence des standards cibles.

**Exemple FLGHT400 :** Modernisation de `FRS409` (RPG OPM → FREE ILE) — résultat dans [`labs_FLGHT400_results/FRS409-Modernisation-report.md`](../labs_FLGHT400_results/FRS409-Modernisation-report.md). Le workflow a analysé le source RPG fixe, produit le code FREE correspondant avec `CTL-OPT DFTACTGRP(*NO) ACTGRP(*NEW)`, et documenté les transformations appliquées.

---

### 1.2 SQL Index Strategy Advisor

**Ce qu'il fait :** analyse les données de performance SQL de Db2 for i — recommandations d'Index Advisor, utilisation des MTI (Maintained Temporary Indexes), statistiques d'exécution — et établit une stratégie d'indexation. Il peut **capturer de nouvelles données** ou exploiter des **snapshots de Plan Cache** ou un **moniteur de base de données** existants. Il propose des index sur tables physiques **et** fichiers logiques, les présente pour **approbation sélective**, les crée sur l'IBM i après accord, puis produit un rapport final (source de données, constats, index créés, prochaines étapes).

**Comment le lancer :**
1. Mode **IBM i Database**, workspace **Library List** (seul workspace supporté)
2. **Start Workflow** → **SQL Index Strategy Advisor**
3. Saisir : Data Source (`Capture New Performance Data`), Capture Method, Output Library, Output Object Name, Top N Queries
4. Bob analyse le Plan Cache, identifie les requêtes coûteuses et propose les index prioritaires
5. Valider : *"Apply the highest-priority index only"*

**Documentation officielle :** [Workflows — SQL Index Strategy Advisor](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

**UC concernés :** UC09 (SQL embarqué), UC10 (Optimisation), UC16 (Objets SQL)

**Exemple FLGHT400 :** Exercice 4 du lab — analyse de la requête sur `ORDERS` × `FLIGHTS` × `AGENTS`. Bob a identifié `ORDERS` comme table volumineuse, le filtre sur `DEPARTURE_DATE` comme très sélectif, et proposé :
```sql
CREATE INDEX FLGHT400.ORDERS_IDX_DEPDT_FLT
    ON FLGHT400.ORDERS (DEPARTURE_DATE, FLIGHT_NUMBER);
```
Résultat complet : [`labs_FLGHT400_results/FLGHT403_sql-review-bottleneck-analysis.md`](../labs_FLGHT400_results/FLGHT403_sql-review-bottleneck-analysis.md)

---

### 1.3 RPGUnit Test Plan Creation

**Ce qu'il fait :** analyse comment le code est utilisé, exécute les suites de tests existantes pour connaître l'état courant, puis génère **quatre types de documents Markdown** : des *templates* réutilisables, un document par *module* (chaque API exportée : rôle, paramètres, retour, effets de bord, organigramme des branches), un document de *suites de tests* par module (cas, setup, entrées, assertions, statut d'implémentation) et un document d'*utilitaires de test* partagés (ex. préparation de tables mock).

**Comment le lancer :**
1. Mode **IBM i Developer**
2. **Start Workflow** → **RPGUnit Test Plan Creation**
3. Sélectionner la bibliothèque cible
4. Saisir le dossier IFS de sortie
5. Choisir **create a new test plan** → Bob trouve les membres sources automatiquement
6. Sélectionner les procédures exportées à tester → **validate the environment**

**Documentation officielle :** [Workflows — RPGUnit Test Plan Creation](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

**UC concernés :** UC14 (Tests unitaires RPGUnit)

**Exemple FLGHT400 :** Exercice 6 du lab — création du plan de tests pour `CUSTCHK.SQLRPGLE` (procédure `checkCustomerExists`). Bob a scanné `FLGHT400/QRPGLESRC`, trouvé le membre, identifié la procédure exportée et généré le plan de tests structuré. Résultat : [`labs_FLGHT400_results/FLGHT403_tests_RPGUnit.md`](../labs_FLGHT400_results/FLGHT403_tests_RPGUnit.md)

---

### 1.4 RPGUnit Test Suite Implementation

**Ce qu'il fait :** lit les documents du plan de tests (produits par le workflow 1.3, **prérequis**), écrit ou met à jour les suites de tests, crée ou met à jour le fichier de configuration RPGUnit `testing.json`, exécute les suites et **itère jusqu'à ce que les nouveaux tests passent**. Les échecs préexistants sont signalés et Bob demande s'il faut les corriger. Il termine par un résumé (fichiers de suites créés, nombre de cas implémentés).

**Comment le lancer :**
1. Mode **IBM i Developer** (après avoir terminé le workflow Test Plan Creation)
2. **Start Workflow** → **RPGUnit Test Suite Implementation**
3. **Proceed** (le plan précédent est déjà prêt)
4. Confirmer la bibliothèque et le dossier IFS
5. **validate the environment** → Bob génère les membres sources de test
6. En cas d'échec : *"Explain the failure and help fix it."*

**Documentation officielle :** [Workflows — RPGUnit Test Suite Implementation](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

**UC concernés :** UC14 (Tests unitaires RPGUnit)

**Exemple FLGHT400 :** Suite de l'exercice 6 — implémentation de la suite de tests pour `CUSTCHK`. Résultat : [`labs_FLGHT400_results/FLGHT403_tests_RPGUnit.md`](../labs_FLGHT400_results/FLGHT403_tests_RPGUnit.md)

---

### 1.5 Business Rules Extraction

**Ce qu'il fait :** analyse les sources **RPG** (OPM RPG II, RPG III, RPG/400, RPGLE, SQLRPGLE) et extrait règles métier, logique de décision, validations et connaissance du processus dans un rapport Markdown lisible par le métier. Le workflow identifie les dépendances externes (fichiers, programmes, programmes de service, copy members, SQL, API système), ignore les sources sans construction RPG significative (avec avertissement), découpe le source en sections (procédures, sous-routines, mainline) traitées par des **sous-agents en parallèle**, enrichit avec les champs des fichiers lus, puis synthétise un rapport exécutif en **8 sections** avec jusqu'à **3 diagrammes Mermaid** (flux de processus, logique de décision, transitions d'état). Le rapport est enregistré en local ou dans l'IFS.

> ⚠️ **Périmètre selon la documentation officielle :** RPG uniquement. Pour COBOL et CL, utiliser les prompts de l'UC06 (logique métier) et les skills `cobol-*` / `cl-*`.

**Comment le lancer :**
1. Mode **IBM i Developer**
2. **Start Workflow** → **Business Rules Extraction**
3. Saisir : bibliothèque, fichier source, membre, chemin de sortie IFS
4. Bob analyse le programme et génère un rapport structuré : résumé exécutif, règles détaillées, validations, exceptions, logique de décision, flux du processus

**Documentation officielle :** [Workflows — Business Rules Extraction](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

**UC concernés :** UC06 (Logique métier), UC05 (Compréhension), UC07 (Documentation)

**Exemple FLGHT400 :** Exercice 1g du lab — extraction des règles métier de `FRS401.RPGLE` (fenêtre de sélection d'agent). Le workflow a produit un rapport complet en anglais ([`labs_FLGHT400_results/FRS401-business-rules.md`](../labs_FLGHT400_results/FRS401-business-rules.md)) et en français ([`labs_FLGHT400_results/FRS401-business-rules-FR.md`](../labs_FLGHT400_results/FRS401-business-rules-FR.md)), couvrant : 10 règles métier détaillées, 5 exceptions, 5 règles de calcul, 6 décisions principales, 6 règles d'intégrité des données, et un diagramme Mermaid du flux processus.

---

## 2. Understand : Dashboard, outils et skills

Source : [Understand](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/understand).

**Ce qu'il fait :** composant de « code intelligence » inclus dans le Premium Package. Il **scanne les bibliothèques d'applications** choisies et construit un modèle de l'application : programmes, programmes de service, modules, fichiers, commandes, data areas, data queues, et leurs relations (appels, accès fichiers, liaisons SRVPGM, références SQL, déclencheurs, clés étrangères, journalisation, contraintes). Une fois le scan fait, Bob répond aux questions structurelles **directement depuis les données du scan**, sans relire chaque membre source.

> ⚠️ **Limite importante :** le scan lit les **objets compilés**, pas le source. Si les sources ne sont pas synchronisés avec les objets, le scan reflète les objets. Les **appels dynamiques** et le **SQL dynamique** ne sont pas capturés (les objets concernés sont signalés pour revue manuelle). Les sources de type fichier physique source ne sont pas listés comme objets.

### Mise en place

1. Définir le réglage `vscode-ibmi-bob.understand.installLibrary` : bibliothèque **dédiée**, non existante, à accès restreint (10 caractères max, pas de point, pas une bibliothèque système)
2. Ouvrir le Dashboard (commande *Open Understand Dashboard*, ou icône du Dashboard dans la barre de titre du chat Bob). Pour découvrir l'outil : commande *Get Started with Understand* (walkthrough intégré)
3. Onglet **Overview** → **Install Understand** (une action **Update Understand** apparaît quand une mise à jour de l'extension embarque une version plus récente)
4. Onglet **Applications** → **New application** : nom, description, liste ordonnée de bibliothèques (les bibliothèques système QSYS, QSYS2, SYSIBM, SYSIBMADM, SYSPROC et SYSTOOLS sont exclues) → **Scan**

Les scans tournent dans leurs **propres jobs SQL** à paramètres fixes (vos réglages de nommage, commit et isolation n'ont pas d'effet) et le Dashboard reste utilisable pendant le scan. Après modification d'une bibliothèque, un **scan incrémental** par bibliothèque suffit (onglet Libraries). Le scan peut recommander des bibliothèques référencées mais absentes de l'application.

### Les onglets du Dashboard

| Onglet | Rôle |
|---|---|
| Overview | Installer, mettre à jour et surveiller le composant |
| Applications | Créer, scanner et gérer les applications |
| Libraries | Ajouter/retirer des bibliothèques ; scan incrémental |
| Objects | Rechercher/filtrer les objets scannés ; voir dépendances entrantes et sortantes, date de dernière utilisation |
| Insights | Compteurs d'objets et de dépendances, objets les plus référencés, risques de SQL dynamique, déclencheurs, clés étrangères, objets journalisés ; bouton **Explore the Application with Bob** |
| Messages | Diagnostics de chaque étape du scan |
| Settings | Accès au réglage de la bibliothèque d'installation et liste des bibliothèques Understand accessibles |
| Access Control | Audit et gestion des profils ayant accès à la bibliothèque Understand (voir ci-dessous) |

**Ouvertures contextuelles :** clic droit sur un profil de connexion Code for IBM i → *Open Understand Dashboard* (l'application dont la library list correspond au profil actif est marquée *Active Profile*) ; clic droit sur un membre ou un objet de l'Object Browser → *Open in Understand*.

### Utiliser Bob avec Understand

Quand Understand est installé et qu'une application est scannée, Bob dispose dans **les deux modes IBM i** de **6 outils Understand** et de **3 skills Understand**. Il les utilise en priorité pour toute question de structure, de relations ou de dépendances.

| Outil | Rôle |
|---|---|
| `understand_list_apps` | Liste les applications enregistrées |
| `understand_app_summary` | Vue d'architecture en un appel : inventaire par type et langage, dépendances, objets les plus couplés, signaux de risque et de modernisation (OPM, SQL dynamique, autorité adoptée, points d'entrée, fichiers sans journal), règles de base (clés étrangères, cascades, déclencheurs) |
| `understand_find_objects` | Recherche filtrée : nom, type, bibliothèque, langage, OPM/ILE, SQL dynamique, sans dépendance entrante, non utilisé depuis N jours |
| `understand_describe_object` | Profil complet d'un objet (métadonnées, dépendances, liaisons, exports/imports, colonnes, déclencheurs, clés étrangères, journalisation) |
| `understand_trace_dependencies` | Dépendances entrantes ou sortantes jusqu'à une profondeur donnée, filtrables par type (`FILE_OPEN`, `PGM_CALL`, `BOUND_SRVPGM`…) |
| `understand_find_symbol` | Où une procédure ou un symbole exporté est défini et qui l'importe (scans IBM i 7.6+) |

**Questions types :**
- *Analyse d'impact :* "Which programs will be affected if I modify FLIGHTS?" — "Is it safe to delete file CUSTMST? What would break?"
- *Objets morts :* "Which objects in FLGHT400 haven't been used in the last 5 years?"
- *Vue d'architecture :* "What are the high-level functional areas of FLGHT400?" — "Which programs are OPM? Which use dynamic SQL?"
- *Programme de service :* "What does service program DBSRV export, and who uses those procedures?"

La permission **Understand** de la tâche (auto-approve) contrôle si Bob peut utiliser ces outils sans demander.

### Sécurité d'Understand

Les données du scan sont stockées dans la bibliothèque d'installation : **un utilisateur qui y a accès peut voir des données d'objets sur lesquels il n'a pas d'autorité directe**. Restreindre l'accès aux seuls utilisateurs autorisés. L'onglet **Access Control** (**Audit Access**) liste les profils, signale les anomalies d'autorité et permet d'accorder, modifier (`*USE`, `*CHANGE`, `*ALL`) ou révoquer un accès. Ces actions **modifient les autorités sur l'IBM i** : relire avant de confirmer.

**Documentation officielle :** [Understand](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/understand)

**Périmètre :** Understand fait partie d'UC00 et sert les UC existants (aucun UC dédié) : UC05 (Compréhension), UC07 (Documentation), UC11 (Restructuration).

**Exemple FLGHT400 :** Exercice 1c du lab — scan de `FLGHT400` (~46 programmes, 0 SQL dynamique). Résultat architectural : [`labs_FLGHT400_results/FLGHT403_functional_areas.md`](../labs_FLGHT400_results/FLGHT403_functional_areas.md), [`labs_FLGHT400_results/FLGHT403_architecture.md`](../labs_FLGHT400_results/FLGHT403_architecture.md).

---

## 3. Les 4 slash commands Premium

Source : [Slash commands](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/slash-commands).

Les slash commands s'exécutent directement dans la zone de saisie du chat Bob (taper `/` pour les lister). Elles nécessitent une connexion IBM i active.

| Commande | Argument | Modes |
|---|---|---|
| `/init` | Aucun (consignes libres possibles) | IBM i Developer, IBM i Database |
| `/erd` | `[schema.table ou bibliothèque/fichier]` | IBM i Developer, IBM i Database |
| `/review_SQL` | `[instruction SQL ou chemin source]` | IBM i Database |
| `/review_rpg` | `[membre source ou chemin de stream file]` | IBM i Developer |

### 3.1 `/init`

**Ce qu'il fait :** analyse le workspace IBM i actif (bibliothèque courante en QSYS ou répertoire de travail en IFS) et génère un `AGENTS.md` qui documente l'architecture non évidente, les structures de sources, les conventions de base de données et les standards du projet : patterns ILE (programmes de service, répertoires de liage, groupes d'activation), définitions IWS, objets de base de données, conventions de nommage. Pour les grandes bibliothèques, Bob résume par comptages et patterns. Si `AGENTS.md` existe déjà, il est mis à jour en **préservant vos ajouts manuels**. Les règles sont immédiatement chargées et injectées dans toutes les conversations des deux modes IBM i, sans reconnexion.

**Comment l'utiliser :**
1. Dans la vue **User Library List**, clic droit sur la bibliothèque → *Set as Current Library*
2. Dans le chat Bob (mode IBM i Developer, portée *Library List*) : `/init`
3. Bob analyse la bibliothèque et écrit `{BIBLIO}/QBOBSRC/AGENTS.MD` (workspace QSYS) ou `{répertoire IFS}/.bob/AGENTS.md` (workspace IFS)

> ⚠️ Sans bibliothèque courante définie, `/init` s'arrête sans rien créer (QGPL et QTEMP refusées : Bob propose de basculer sur une bibliothèque applicative). En workspace IFS, un répertoire de travail doit être configuré (pas la racine `/`).
>
> 💡 **Personnaliser :** `/init and make sure to enforce free-format RPG and our standard error-handling subprocedures`, ou éditer `AGENTS.md` à la main.
>
> 📝 **À vérifier :** le lab FLGHT400 a aussi produit une copie `/home/ITZUSER/AGENTS.md` ; la documentation actuelle ne mentionne que les deux emplacements ci-dessus. Les UC02, UC03 et UC05 citent encore cette copie IFS (`/home/{USER}/AGENTS.md`).

**Documentation officielle :** [Slash commands — /init](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/slash-commands)

**UC concernés :** UC02 (Maîtrise Bob), UC03 (Bob en équipe)

**Exemple FLGHT400 :** Exercice 1b du lab — `/init` sur `FLGHT403` a produit un `AGENTS.md` documentant 122 membres sources, l'architecture ILE, les dépendances cross-library sur `FLGHT400`, les patterns de nommage des logical files et les styles de code modernisé. Résultat : [`labs_FLGHT400_results/FLGHT403_init_report.md`](../labs_FLGHT400_results/FLGHT403_init_report.md), [`labs_FLGHT400_results/FLGHT403_AGENTS.md`](../labs_FLGHT400_results/FLGHT403_AGENTS.md).

---

### 3.2 `/erd`

**Ce qu'il fait :** génère un diagramme Entités-Relations au format Mermaid depuis les tables Db2 for i du système connecté. Interroge `QSYS2.SYSTABLES`, `QSYS2.SYSCOLUMNS`, `QSYS2.SYSCST` et `QSYS2.SYSREFCST` pour produire un diagramme avec types de données Db2 natifs, marqueurs PK/FK/UQ et cardinalités. Sans argument, Bob demande le schéma à analyser.

**Comment l'utiliser :**
1. Mode **IBM i Database** (ou Developer)
2. Dans le chat : `/erd {BIBLIOTHÈQUE}` (ex. `/erd FLGHT400`, `/erd MYAPP.ORDERS`, `/erd MYPROJ/CUSTOMER`)
3. Bob génère le diagramme Mermaid directement dans la conversation

**UC concernés :** UC07 (Documentation), UC16 (Objets SQL), UC08 (DDS→DDL)

**Exemple FLGHT400 :** Exercice 1e du lab — `/erd FLGHT400` a produit le diagramme ERD complet de l'application Flight Reservation System. Résultat : [`labs_FLGHT400_results/FLGHT403_erd.md`](../labs_FLGHT400_results/FLGHT403_erd.md), [`labs_FLGHT400_results/FLGHT403_erd_report.md`](../labs_FLGHT400_results/FLGHT403_erd_report.md).

---

### 3.3 `/review_SQL`

**Ce qu'il fait :** réalise une revue complète d'une requête SQL ou d'un membre source contenant du SQL, selon une checklist : correction et simplification, lisibilité, qualification et nommage, bonnes pratiques (fonctions natives, CTE), sécurité (moindre privilège, injection SQL), performances SQE, guidage selon le type d'instruction (DDL, DML, SQL embarqué et dynamique), diagnostic (Visual Explain, plan cache, conseils d'index), logique réutilisable (UDF, procédures) et isolation transactionnelle. Produit : (1) Synthèse, (2) Constats confirmés, (3) Recommandations conditionnelles, (4) SQL révisé, (5) Explication des changements.

**Comment l'utiliser :**
1. Mode **IBM i Database**
2. Dans le chat : `/review_SQL` puis coller la requête, ou `/review_SQL mylib/mysrc(mycode)` ou `/review_SQL /home/user/src/mypgm.rpglesrc`

**UC concernés :** UC09 (SQL embarqué), UC10 (Optimisation), UC16 (Objets SQL)

**Exemple FLGHT400 :** Exercice 4b du lab — revue de la requête analytique sur `ORDERS × FLIGHTS × AGENTS`. Bob a détecté le problème intentionnel (`ORDER BY o.DEPAR00001` sur une colonne absente du `GROUP BY`) et proposé `MIN(o.DEPAR00001)`. Résultat : [`labs_FLGHT400_results/FLGHT403_sql-review-bottleneck-analysis.md`](../labs_FLGHT400_results/FLGHT403_sql-review-bottleneck-analysis.md).

---

### 3.4 `/review_rpg`

**Ce qu'il fait :** réalise une revue de code d'un membre source ou d'un stream file RPG (fixe, libre ou mixte). Bob **refuse** les sources non RPG (CL, COBOL, DDS, SQL pur) avant de charger les règles de revue. Il détecte le format, charge les skills adaptées, puis évalue syntaxe, standards, gestion des erreurs, sécurité (autorité adoptée, injection), performances (RLA vs SQL embarqué, blocage, curseurs) et modernisation. Produit : (1) Format détecté, (2) Synthèse, (3) Problèmes critiques, (4) Majeurs, (5) Mineurs, (6) Feuille de route de modernisation.

**Comment l'utiliser :**
1. Ouvrir le membre source RPG dans l'éditeur
2. Mode **IBM i Developer**
3. Dans le chat : `/review_rpg` ou `/review_rpg MYLIB/QRPGLESRC/MYPGM.RPGLE`

**UC concernés :** UC12 (COBOL→FREE, sur le RPG généré uniquement), UC13 (RPG→FREE), UC10 (Optimisation), UC11 (Restructuration)

---

## 4. Les 71 skills IBM i — référence rapide

Source : [Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills).

Les skills sont activées automatiquement par Bob selon le contexte. Elles n'ont pas besoin d'être invoquées manuellement. Chacune est disponible dans le mode IBM i Developer, le mode IBM i Database, ou les deux :

| Domaine | Nombre | Skills représentatives |
|---|---|---|
| RPG | 22 | `rpg-fixed-to-free`, `rpg-cycle-explained`, `rpg-rla-to-sql`, `rpg-opm-to-ile`, `rpg-code-review` |
| CL | 4 | `cl-programming`, `cl-best-practices` |
| DDS | 5 | `dds-physical-files`, `dds-logical-files`, `dds-display-files` |
| COBOL for i | 12 | `cobol-ile-understanding`, `cobol-embedded-sql`, `cobol-opm-understanding`, `cobol-source-debugging`, `cobol-code-refactoring`, `cobol-code-review`, `cobol-module-service-program` |
| Db2 for i & SQL | 19 | `db2-dds-to-ddl`, `db2-sql-performance-analysis`, `db2-sql-stored-procedure-performance`, `db2-index-strategy`, `db2-security-best-practices` |
| IBM i IWS | 5 | `iws-service-ops`, `iws-server-config` |
| Understand | 3 | `understand-app-overview`, `understand-impact-analysis`, `understand-program-explainer` |
| Tests unitaires | 1 | `rpgunit-testing` |

**Points d'attention :**
- Les skills Understand ne sont actives que si Understand est installé.
- Skills **réservées au mode IBM i Database** : `db2-dds-understanding`, `db2-ddl-generation`, `db2-sqlpl-performance`, `db2-sql-find-performance-data`, `db2-index-strategy`, `db2-journaling-commitment`, `db2-temporal-tables`.
- Skills disponibles dans les **deux modes** : `rpg-rla-to-sql`, `cl-*` (sauf `clle-api`), `dds-primer-basics`, `dds-physical-files`, `dds-logical-files`, plusieurs skills `db2-*` et les 3 skills `understand-*`. Les autres skills RPG, COBOL, IWS, `dds-display-files`, `dds-printer-files`, `clle-api` et `rpgunit-testing` relèvent d'IBM i Developer.

**Documentation complète :** [Skills IBM i](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills)

---

## 5. Les outils IBM i, la recherche documentaire (RAG) et les garde-fous

### 5.1 Outils IBM i

Source : [Tools](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/tools). Ces outils sont appelés par Bob ; ils sont regroupés par permission d'auto-approbation.

| Famille | Outils | À retenir pour le POC |
|---|---|---|
| Lecture | `read_member`, `search_qsys`, `read_stream_file`, `search_ifs` | Recherche par regex dans les membres et stream files |
| Écriture | `write_member`, `write_stream_file`, `generate_dds` | `generate_dds` : DDS pour PF, LF, DSPF, PRTF avec validation de mise en page (UC08, UC17) |
| Exécution | `execute_cl_command`, `execute_sql_statement`, `execute_pase_command`, `manage_library_list`, `convert_rpg_source` | `convert_rpg_source` = `CVTRPGSRC` (RPG III → RPG IV) ; `manage_library_list` lit/modifie la library list et la bibliothèque courante |
| Build | `get_compile_actions`, `execute_compile_action` | Actions Code for IBM i (`.vscode/actions.json` en local) |
| Test | `generate_rpg_unit_test_stub`, `run_rpg_unit_test_suite` | Couverture de code `*LINE` ou `*PROC` (UC14) |
| SQL | `get_schema_info`, `get_file_info`, `get_table_columns`, `get_cl_command_library` | Résolution des noms système/SQL, noms longs, alias, identifiants délimités |
| Documentation | `search_ibm_i_docs_with_rag`, `fetch_cl_command_doc`, `search_sql_examples`, `fetch_sql_example` | Documentation officielle et exemples SQL Services |
| Understand | 6 outils `understand_*` | Voir §2 |

### 5.2 Recherche documentaire (RAG)

Source : [Retrieval-augmented generation](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/retrieval-augmented-generation). Les deux modes IBM i disposent de `search_ibm_i_docs_with_rag`, qui interroge un corpus indexé de documentation IBM i, de Redbooks et de manuels de référence (ILE, Db2 for i, IFS/QSYS, sécurité, performance, réseau, dépannage). Bob l'utilise pour les questions conceptuelles ou quand il boucle (ex. erreurs de compilation répétées). On peut le demander explicitement : *"Search the IBM i docs for…"* ou *"Check the documentation before answering"*.

### 5.3 Garde-fous et réglages

Sources : [Settings](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/settings), [Security guidelines](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/security-guidelines).

| Réglage | Défaut | Effet |
|---|---|---|
| `vscode-ibmi-bob.guardrails.enforceQualifiedCLCommands` | `true` | Les commandes CL doivent être qualifiées (`QSYS/…`) ; sinon erreur avec alternatives qualifiées |
| `vscode-ibmi-bob.guardrails.deniedSQLStatementTypes` | 17 types dont `UPDATE`, `DROP`, `TRUNCATE`, `GRANT`, `CALL`, `MERGE` | Approbation explicite **toujours** demandée, même si l'auto-approve d'exécution est accordé |
| `vscode-ibmi-bob.general.maxRowsToFetch` | `100` | Plafond de lignes remontées par `execute_sql_statement` |
| `vscode-ibmi-bob.understand.installLibrary` | aucun (obligatoire) | Bibliothèque d'installation d'Understand |

**Sécurité :** Bob agit avec le **profil utilisateur de la connexion** Code for IBM i : tout ce que ce profil peut faire, Bob peut le faire → appliquer le moindre privilège. L'activité de Bob est traçable via le registre spécial `CLIENT_APPLNAME` des jobs SQL. Limiter l'accès à la bibliothèque `RPGUNIT` (développeurs en lecture, administrateurs en mise à jour) et à la bibliothèque et au répertoire temporaires de Code for IBM i.

### 5.4 Dépannage

Source : [Troubleshooting](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/troubleshooting). Le canal Output **IBM Bob Premium Package for i** journalise chaque appel d'outil (INFO au démarrage, ERROR en cas d'échec, WARN au-delà de 30 secondes, durée finale). Pour isoler les messages dans le canal *IBM Bob*, chercher `[source:ibmi]`. Autres canaux utiles : *Code for IBM i* (CL, SQL, PASE exécutés) et *IBM i Testing*.

---

## 6. Carte d'utilisation par UC

Le tableau suivant indique quelles capacités Premium sont mobilisées par chaque UC du POC (numérotation actuelle des fiches `uc-fr/`) :

| UC | Titre | Workflow | Slash Cmd | Understand | Skills clés |
|---|---|---|---|---|---|
| UC01 | Connexion IBM i et MCP tiers (optionnels) | — | — | — | — |
| UC02 | Maîtrise de Bob | — | `/init` | — | — |
| UC03 | Bob en équipe | — | `/init` → AGENTS.md | — | — |
| UC04 | Stratégie Skills | — | — | — | (lien skill `poc-ibmi-premium-uc-writer`) |
| UC05 | Compréhension de code | Business Rules Extraction | `/init` | ✅ | `understand-app-overview`, `understand-program-explainer` |
| UC06 | Logique métier | **Business Rules Extraction** (RPG) | — | ✅ | `rpg-code-review`, `cobol-procedure-division` |
| UC07 | Documentation | — | `/erd`, `/init` | ✅ | `understand-impact-analysis`, `db2-system-catalog` |
| UC08 | DDS→DDL | — | `/erd` | — | `db2-dds-to-ddl`, `dds-physical-files` |
| UC09 | SQL embarqué | SQL Index Strategy (partiel) | `/review_SQL` | — | `rpg-rla-to-sql`, `rpg-embedded-sql`, `db2-sql-stored-procedure-performance` |
| UC10 | Optimisation | SQL Index Strategy (partiel) | `/review_rpg`, `/review_SQL` | — | `rpg-code-review`, `rpg-rla-to-sql`, `db2-sql-stored-procedure-performance` |
| UC11 | Restructuration | RPG Modernization (partiel) | `/review_rpg` | ✅ | `rpg-fixed-to-free`, `rpg-refactor-legacy`, `cobol-code-refactoring` |
| UC12 | COBOL→FREE | — (démarche guidée avec les skills COBOL ; RPG Modernization non applicable aux sources COBOL) | `/review_rpg` (membres RPG générés uniquement) | — | `rpg-opm-to-ile`, `cobol-*` (dont `cobol-code-review`, `cobol-code-refactoring`) |
| UC13 | RPG Colonné→FREE | RPG Modernization | `/review_rpg` | — | `rpg-fixed-to-free`, `rpg-cycle-explained` |
| UC14 | Tests RPGUnit | **RPGUnit Test Plan + Impl.** | — | — | `rpgunit-testing`, `rpg-debugging-techniques` |
| UC15 | Génération code | — | — | — | `rpg-procedures-functions`, `rpg-free-format-fundamentals` |
| UC16 | Objets SQL | SQL Index Strategy (partiel) | `/review_SQL` | — | `db2-stored-procedures`, `db2-index-strategy`, `db2-sql-stored-procedure-performance` |
| UC17 | Génération apps | — | — | — | `iws-service-ops`, `dds-display-files` |
| UC18 | Exposition API REST (IWS) | — | `/init`, `/review_rpg` | — | `iws-admin`, `iws-server-ops`, `iws-server-config`, `iws-service-ops`, `iws-backup` |
| UC19 | Exploitation Ops | — | `/init` | — | `db2-security-best-practices`, `db2-system-catalog`, `db2-services-catalog`, `db2-sql-performance-analysis` |

---

## 7. Lab FLGHT400 — guide de mise en œuvre

Le lab Flight400 est le support pratique officiel du Premium Package IBM i. Il se joue sur une instance TechZone et couvre les 8 exercices ci-dessous, chacun illustrant une ou plusieurs capacités Premium.

**Référence déploiement :** [`labs_FLGHT400/FICHE-REDEPLOIEMENT.md`](../labs_FLGHT400/FICHE-REDEPLOIEMENT.md)
**GitHub du lab :** https://github.com/bmarolleau/flight400-demo

| Exercice | Durée | Capacités Premium utilisées | Résultats disponibles |
|---|---|---|---|
| 1a — Exploration Object Browser | 5 min | — (exploration manuelle) | — |
| 1b — `/init` sur la bibliothèque | 5 min | `/init` slash command | [`FLGHT403_init_report.md`](../labs_FLGHT400_results/FLGHT403_init_report.md) |
| 1c — Scan Understand | 10 min | Understand Dashboard | [`FLGHT403_functional_areas.md`](../labs_FLGHT400_results/FLGHT403_functional_areas.md) |
| 1d — Architecture overview | 10 min | Understand + skills | [`FLGHT403_architecture.md`](../labs_FLGHT400_results/FLGHT403_architecture.md) |
| 1e — `/erd` | 5 min | `/erd` slash command | [`FLGHT403_erd.md`](../labs_FLGHT400_results/FLGHT403_erd.md) |
| 1g — Business Rules | 10 min | **Business Rules Extraction** | [`FRS401-business-rules-FR.md`](../labs_FLGHT400_results/FRS401-business-rules-FR.md) |
| 2 — Moderniser FRS409 | 15 min | **RPG Modernization** | [`FRS409-Modernisation-report.md`](../labs_FLGHT400_results/FRS409-Modernisation-report.md) |
| 3 — Ajouter Total Flight Hours | 30 min | IBM i Developer (outils natifs IBM i) | [`FLGHT403_Change_Total_Flight_Hours.md`](../labs_FLGHT400_results/FLGHT403_Change_Total_Flight_Hours.md) |
| 4 — Optimisation SQL | 15 min | `/review_SQL` + **SQL Index Strategy Advisor** | [`FLGHT403_sql-review-bottleneck-analysis.md`](../labs_FLGHT400_results/FLGHT403_sql-review-bottleneck-analysis.md) |
| 5 — Questions système | 10 min | IBM i Database (outils natifs IBM i) | [`FLGHT403_cpu_jobs_programs_audit.md`](../labs_FLGHT400_results/FLGHT403_cpu_jobs_programs_audit.md) |
| 6 — Tests RPGUnit | 20 min | **RPGUnit Test Plan + Implementation** | [`FLGHT403_tests_RPGUnit.md`](../labs_FLGHT400_results/FLGHT403_tests_RPGUnit.md) |
| 7 — App React (option) | 30 min | Génération apps | [`FLGHT403_App_React.md`](../labs_FLGHT400_results/FLGHT403_App_React.md) |
| 8 — 5250 vers HTML (option) | 5 min | Génération apps | [`FLGHT403_App_5250_web.md`](../labs_FLGHT400_results/FLGHT403_App_5250_web.md) |

---

## Livrables de cet UC

| Fichier produit | Description | Consommé par |
|---|---|---|
| `{app}-init-all-{date}.md` | Rapport `/init` — AGENTS.md documenté | Tous les UC suivants (contexte) |
| `{app}-erd-all-{date}.md` | Diagramme ERD Mermaid complet | UC07, UC08, UC16 |
| `{app}-understand-overview-{date}.md` | Vue d'architecture Understand | UC05, UC07, UC11 |

---

## Sources

Documentation IBM Bob Premium Package for i : [Overview](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/bob-for-i-index) · [Prerequisites](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/prerequisites) · [Modes](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/modes) · [Workspaces](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workspaces) · [Understand](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/understand) · [Tools](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/tools) · [Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills) · [Slash commands](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/slash-commands) · [Workflows](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows) · [RAG](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/retrieval-augmented-generation) · [Settings](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/settings) · [Security guidelines](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/security-guidelines) · [Troubleshooting](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/troubleshooting)
