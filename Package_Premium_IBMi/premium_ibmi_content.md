# Bob for IBM i Premium Package

## Pourquoi le Premium Package IBM i ?

Le Premium Package IBM i centralise des capacités de maintenance, de modernisation, d’analyse et de compréhension des applications IBM i :

- **5 workflows Premium** guidés pas à pas
- **1 tableau de bord Understand** pour la cartographie applicative
- **4 Slash Commands Premium** exécutables depuis le chat Bob
- **71 skills IBM i listées ci-dessous**, activées automatiquement selon le contexte

> **Contrôle de cohérence (mise à jour novembre 2026 d'après la documentation officielle) :** les listes détaillées ci-dessous contiennent 71 skills : 22 RPG, 4 CL, 5 DDS, 12 COBOL for i, 19 Db2 for i & SQL, 5 IBM i IWS, 3 Understand et 1 Tests unitaires. Le message source initial annonçait 57 skills, puis 65 avant l'ajout de 4 skills COBOL, 1 skill Db2 et `rpgunit-testing`.

---

## 1. Les 5 Workflows Premium IBM i

Ces workflows pas à pas guident les développeurs dans l’automatisation des tâches complexes de maintenance et de modernisation.

### 1.1 RPG Modernization

**Rôle :** convertit et modernise le code source RPG existant, OPM RPG III ou ILE RPG IV au format fixe, vers du RPG libre (`**FREE`) en appliquant les standards modernes : `CTL-OPT`, procédures intégrées, suppression des `GOTO`, etc.

**Documentation officielle :** [RPG Modernization](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

### 1.2 SQL Index Strategy Advisor

**Rôle :** analyse les performances des requêtes SQL sur Db2 for i, notamment les recommandations d’Index Advisor, l’utilisation des MTI et les statistiques d’exécution. Il conseille et applique une stratégie d’indexation visant à améliorer les temps de réponse de la base de données.

**Documentation officielle :** [SQL Index Strategy Advisor](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

### 1.3 RPGUnit Test Plan Creation

**Rôle :** scanne les programmes RPG de l’application afin d’identifier les points d’entrée et les procédures exportées. Il génère ensuite un plan de tests unitaires structuré au format Markdown, avec les scénarios de test et les utilitaires partagés nécessaires.

**Documentation officielle :** [RPGUnit Test Plan Creation](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

### 1.4 RPGUnit Test Suite Implementation

**Rôle :** implémente les suites de tests unitaires RPGUnit planifiées en générant la structure du code de test, les assertions et les routines `setup`/`teardown`. Il exécute ensuite la suite de tests directement sur le système afin de corriger les erreurs logiques détectées.

**Documentation officielle :** [RPGUnit Test Suite Implementation](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

### 1.5 Business Rules Extraction

**Rôle :** analyse automatiquement le flux d’exécution et les spécifications de calcul des sources **RPG** afin d’isoler et de documenter en langage clair les règles métier, les validations et les règles de calcul décisionnelles de l’application. Périmètre officiel : RPG uniquement (workspaces Local, IFS, Library List).

**Documentation officielle :** [Business Rules Extraction](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

---

## 2. Tableau de bord Understand

Le tableau de bord **Understand** est le scanner d’application intégré qui analyse la structure globale du parc applicatif IBM i.

### Rôle

Il enregistre et cartographie les objets et leurs interdépendances :

- appels de programmes ;
- utilisation des fichiers de données DDS PF/LF et des tables SQL ;
- liaisons de programmes de service (`SRVPGM`) ;
- déclencheurs ;
- contraintes ;
- statistiques d’utilisation opérationnelles.

### Fonctionnalités clés

- **Analyse d’impact :** déterminer quels programmes ou fichiers seront affectés par la modification, le renommage ou la suppression d’un élément.
- **Objets morts :** identifier les objets obsolètes ou inutilisés dans une bibliothèque afin de nettoyer les environnements de production.
- **Visualisation d’architecture :** obtenir un graphe complet des dépendances sans lecture manuelle de chaque membre source.

### Accès

Le tableau de bord s’ouvre directement depuis l’IDE avec l’action :

```text
vscode-ibmi-bob.openUnderstandDashboard
```

**Documentation officielle :** [Understand Dashboard](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/understand)

---

## 3. Les 4 Slash Commands Premium IBM i

Ces commandes préfixées par `/` s’exécutent directement dans la zone de saisie du chat Bob et déclenchent une action prédéfinie sur l’IBM i connecté.

### 3.1 `/init`

**Rôle :** analyse le workspace IBM i actif, bibliothèque courante en QSYS ou répertoire de travail en IFS, puis génère automatiquement un fichier `AGENTS.md` adapté à l’application. Ce fichier documente l’architecture ILE, les conventions de nommage, les objets DDS/SQL et les standards du projet. Il est ensuite injecté en contexte dans les conversations Bob sans reconnexion.

### 3.2 `/erd`

**Rôle :** génère un diagramme de relations entre entités au format Mermaid depuis les tables Db2 for i du système connecté. La commande interroge `QSYS2.SYSTABLES`, `QSYS2.SYSCOLUMNS`, `QSYS2.SYSCST` et `QSYS2.SYSREFCST` pour produire un diagramme intégrant les types de données Db2 natifs, les marqueurs de colonnes PK, FK et UQ, ainsi que les cardinalités.

### 3.3 `/review_SQL`

**Rôle :** réalise une revue complète d’une requête SQL ou d’un membre source contenant du SQL. L’analyse couvre la syntaxe, la lisibilité, la sécurité, notamment l’injection SQL et le moindre privilège, les performances SQE et les possibilités de refactorisation.

**Structure du résultat :**

1. Synthèse
2. Constats confirmés
3. Recommandations conditionnelles
4. SQL révisé
5. Explication des changements

### 3.4 `/review_rpg`

**Rôle :** réalise une revue de code d’un membre source RPG au format fixe, libre ou mixte. L’analyse couvre la syntaxe, les standards de codage, la gestion des erreurs, la sécurité, les performances, notamment RLA par rapport au SQL embarqué, et les possibilités de modernisation.

**Structure du résultat :**

1. Format détecté
2. Synthèse
3. Problèmes critiques
4. Problèmes majeurs
5. Problèmes mineurs
6. Feuille de route de modernisation

**Documentation officielle commune :** [Slash Commands Bob for IBM i](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/slash-commands)

---

## 4. Skills IBM i

Les skills sont activées automatiquement par Bob selon le contexte. Elles sont regroupées en sept catégories.

### 4.1 RPG

| Skill | Description |
|---|---|
| `rpg-primer-basics` | Fondamentaux RPG toutes variantes : RPG II, RPG III, RPG/400, RPGLE et ILE. |
| `rpg-free-format-fundamentals` | Syntaxe et bonnes pratiques du RPGLE moderne au format libre. |
| `rpg-opm-understanding` | Architecture OPM, différences de modèle de programme et migration. |
| `rpg-ile-understanding` | Concepts ILE : binding, groupes d’activation, programmes de service et appels de procédures. |
| `rpgle-specs` | Référence colonne par colonne des spécifications RPGLE au format fixe : H, F, D, I, C et O. |
| `rpg-legacy-specs` | Spécifications RPG historiques : RPG II, RPG III, RPG36 et RPG38, avec positionnement des colonnes. |
| `rpg-data-structures` | Modèles de DS, DS décrites en externe, DS de statut programme et DS d’information fichier. |
| `rpg-procedures-functions` | Sous-procédures, prototypes, passage de paramètres, exports et programmes de service. |
| `rpg-file-operations` | Accès RLA, codes opération, déclarations de fichiers et gestion des erreurs. |
| `rpg-indicators` | Types d’indicateurs et modèles de remplacement par des variables booléennes. |
| `rpg-cycle-explained` | Cycle de programme RPG, ruptures de contrôle et situations dans lesquelles l’éviter en code moderne. |
| `rpg-embedded-sql` | SQL embarqué : opérations de curseur, variables hôtes et différences de compilation. |
| `rpg-sql-cursor` | Déclaration de curseurs SQL, curseurs scrollables et traitement multiligne en SQLRPGLE. |
| `rpg-fixed-to-free` | Conversion du RPGLE au format fixe vers le format libre, y compris I-specs et O-specs. |
| `rpg-pgmdb-conversion` | Modernisation des I-specs et O-specs vers RPGLE libre pour les fichiers décrits par programme. |
| `rpg-ospec-conversion` | Conversion des O-specs d’impression vers des fichiers imprimante décrits en externe. |
| `rpg-opm-to-ile` | Conversion OPM RPG vers ILE RPGLE : `CVTRPGSRC`, groupes d’activation et binding. |
| `rpg-refactor-legacy` | Refactorisation : suppression des `GOTO` et conversion des routines en sous-procédures. |
| `rpg-rla-to-sql` | Conversion de l’accès RLA vers le SQL embarqué pour améliorer les performances. |
| `rpg-code-review` | Revue qualité : dette technique, performances et possibilités de modernisation. |
| `rpg-debugging-techniques` | Débogage RPG : `STRDBG`, points d’arrêt, expressions surveillées et points d’entrée de service. |
| `rpg-api` | Appel d’API système IBM i depuis RPG : prototypes, correspondance des types et gestion des erreurs. |

**Documentation officielle :** [Skills RPG](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#rpg)

### 4.2 CL

| Skill | Description |
|---|---|
| `cl-primer-basics` | Fondamentaux CL : conventions de nommage, syntaxe des paramètres et commandes courantes. |
| `cl-programming` | Programmation CL : variables, logique conditionnelle, gestion des erreurs et structure de programme. |
| `cl-best-practices` | Bonnes pratiques : `MONMSG`, validation des paramètres et stratégies de journalisation. |
| `clle-api` | Appel d’API système depuis CL ILE : `CALL PGM/PARM`, correspondance `DCL VAR` et gestion `QUSEC`. |

**Documentation officielle :** [Skills CL](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#cl)

### 4.3 DDS

| Skill | Description |
|---|---|
| `dds-primer-basics` | Fondamentaux DDS pour expliquer et générer des définitions DDS. |
| `dds-physical-files` | Fichiers physiques PF : spécifications de champs, clés et formats d’enregistrement. |
| `dds-logical-files` | Fichiers logiques LF : jointures, `select/omit` et champs dérivés. |
| `dds-display-files` | Fichiers d’affichage DSPF : formats, indicateurs, sous-fichiers et conception d’écrans. |
| `dds-printer-files` | Fichiers d’impression PRTF : espacements, positionnement, codes d’édition et rapports. |

**Documentation officielle :** [Skills DDS](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#dds)

### 4.4 COBOL for i

| Skill | Description |
|---|---|
| `cobol-primer-basics` | Fondamentaux COBOL for i : structure, quatre divisions, verbes courants et standards ILE/OPM. |
| `cobol-data-division` | Data Division : numéros de niveau, clauses picture, working-storage et linkage section. |
| `cobol-procedure-division` | Procedure Division : verbes, logique conditionnelle, boucles `PERFORM`, I/O fichiers et interfaces `CALL`. |
| `cobol-embedded-sql` | SQL embarqué en COBOL : variables hôtes, indicateurs, curseurs et contrôle de transaction. |
| `cobol-ile-understanding` | COBOL ILE : groupes d’activation, binding, intégration des programmes de service et directives du compilateur. |
| `cobol-ile-intrinsic-functions` | Fonctions intrinsèques ILE COBOL : chaînes, arithmétique, dates/heures et conversions de types. |
| `cobol-opm-understanding` | Modèle de programme OPM COBOL, différences avec ILE et migration. |
| `cobol-file-io` | I/O fichiers COBOL : séquentiels, indexés, relatifs et entrées de contrôle de fichier. |
| `cobol-source-debugging` | Débogage COBOL : débogueur source ILE, points d’arrêt, exécution pas à pas, analyse du job log et des dumps, erreurs de compilation et d’exécution. |
| `cobol-code-refactoring` | Restructuration COBOL : simplification de la logique, suppression des `GO TO` et du code mort, réduction de la duplication, en préservant la logique métier. |
| `cobol-code-review` | Revue de code COBOL structurée : défauts, erreurs de logique, dette technique et opportunités de modernisation. |
| `cobol-module-service-program` | Modules et programmes de service ILE COBOL : imports/exports, binder language, binding directories, groupes d’activation, applications multi-modules. |

**Documentation officielle :** [Skills COBOL for i](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#cobol-for-i)

### 4.5 Db2 for i & SQL

| Skill | Description |
|---|---|
| `db2-sql-primer` | Fondamentaux SQL Db2 for i : CRUD, jointures, agrégats, procédures stockées et UDF. |
| `db2-stored-procedures` | Procédures SQL PL : paramètres `IN`, `OUT`, `INOUT`, structures de contrôle et curseurs. |
| `db2-functions-udfs` | UDF SQL, UDF externes, fonctions de table et fonctions sourcées. |
| `db2-sql-debugging` | Interprétation des erreurs SQL, `SQLCODE`/`SQLSTATE`, et techniques de diagnostic. |
| `db2-dds-understanding` | Fichiers DDS historiques : PF, LF et chemins d’accès clés. |
| `db2-dds-to-ddl` | Migration DDS vers DDL SQL : tables, vues et stratégies de conversion. |
| `db2-ddl-generation` | Syntaxe DDL : colonnes, contraintes PK, FK, CHECK et UNIQUE, index et vues. |
| `db2-ccsid-encoding` | Résolution des problèmes CCSID et d’encodage des caractères. |
| `db2-sql-performance-analysis` | Analyse du Plan Cache, interprétation Visual Explain et recommandations Index Advisor. |
| `db2-sqlpl-performance` | Optimisation SQL PL : optimisation `SET` et techniques d’inlining. |
| `db2-sql-stored-procedure-performance` | Bonnes pratiques SQL PL haute performance dans les procédures stockées, fonctions et déclencheurs. |
| `db2-sql-find-performance-data` | Localisation des données de performance SQL : instantanés Plan Cache et moniteurs de base de données. |
| `db2-sql-optimization` | Réécriture des requêtes, ordre des jointures, predicate pushdown et utilisation des index. |
| `db2-index-strategy` | Types d’index, notamment Binary Radix et EVI, syntaxe, sélection et maintenance. |
| `db2-security-best-practices` | Modèles d’autorité, deny-by-default, autorité adoptée, privilèges SQL et journal d’audit. |
| `db2-journaling-commitment` | Journal, contrôle d’engagement, niveaux d’isolation et procédures de reprise. |
| `db2-temporal-tables` | Tables temporelles : requêtes historiques et versioning automatique. |
| `db2-system-catalog` | Catalogues système Db2 for i pour la découverte des métadonnées. |
| `db2-services-catalog` | Exploration des IBM i SQL Services disponibles et de leurs paramètres. |

**Documentation officielle :** [Skills Db2 for i & SQL](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#db2-for-i--sql)

### 4.6 IBM i IWS (Integrated Web Services)

| Skill | Description |
|---|---|
| `iws-admin` | Administration IWS : opérations serveur/service, configuration, déploiement REST et sauvegarde. |
| `iws-server-ops` | Lister, créer, supprimer, démarrer et arrêter des serveurs IWS. |
| `iws-server-config` | Lire et modifier les propriétés des serveurs IWS. |
| `iws-service-ops` | Lister, installer, désinstaller, démarrer, arrêter et configurer des services web IWS. |
| `iws-backup` | Sauvegarder et restaurer des serveurs et services IWS vers ou depuis des save files IFS. |

**Documentation officielle :** [Skills IBM i IWS](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#ibm-i-iws)

### 4.7 Understand

> Ces skills sont actives uniquement si Understand est installé.

| Skill | Description |
|---|---|
| `understand-app-overview` | Architecture globale de l’application, hotspots, maturité de modernisation OPM/ILE, SQL dynamique, décommissionnement et ordre de conversion. |
| `understand-impact-analysis` | Analyse d’impact : dépendants directs et indirects, niveau de risque et ordre de reconstruction recommandé. |
| `understand-program-explainer` | Explication d’un programme ou programme de service : composition OPM/ILE, modules liés, exports, fichiers utilisés et appelants. |

**Documentation officielle :** [Skills Understand](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#understand)

### 4.8 Tests unitaires

| Skill | Description |
|---|---|
| `rpgunit-testing` | Structure d’une suite RPGUnit, nommage des cas de test, `setup`/`teardown`, API d’assertion et configuration des tests (mode IBM i Developer). |

**Documentation officielle :** [Skills Tests unitaires](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills)
