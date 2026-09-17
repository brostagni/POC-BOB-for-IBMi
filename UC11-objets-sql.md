# UC 11 — Génération d'objets SQL

> **Catégorie :** Développement
>
> **Priorité dans le POC :** 11b — Track B Phase 4, après UC 9, peut être mené en parallèle avec UC 9 sur des objets différents ; complémentaire de UC 14
>
> **Durée POC (avec Bob) :** 3 à 5 heures — 6 types d'objets couverts (table, vue, index, procédure, trigger, fonction) sur un périmètre représentatif
>
> **Durée PROD (avec Bob) :** 10 à 30 min / objet selon complexité — table simple en 10 min ; procédure avec curseurs et gestion d'erreur en 30 min
>
> **Durée PROD (sans Bob) :** 1 à 4 heures / objet — rédaction DDL manuelle, gestion des contraintes, gestion d'erreur dans les procédures, tests de création
>
> **Gain Bob estimé :** ~10× — une procédure stockée avec curseur, gestion d'erreur et paramètres IN/OUT générée en 20 min au lieu de 3 heures ; gain fort sur les triggers qui sont complexes à écrire correctement à la main
>
> **Mode initial :** Ask ou ACME IBM i Review.
> **Mode d'exécution :** Agent ou ACME IBM i Execute.

---

## Objectif

Générer des **objets SQL nouveaux** — tables, vues, index, procédures stockées, triggers, fonctions — conformes à Db2 for i, *ex nihilo*, à partir d'un besoin exprimé.

**Ce UC se différencie fondamentalement de UC 14 :** UC 14 **convertit** une structure DDS existante en DDL — il part d'un objet IBM i legacy et produit son équivalent SQL. UC 11 **crée** un objet SQL nouveau à partir d'un besoin fonctionnel — il n'y a pas de source DDS d'origine. Les deux UC produisent des scripts DDL, mais leurs origines sont opposées.

> ⚠️ **Règle de routage :** si un fichier physique (PF) ou logique (LF) DDS existe déjà et doit être modernisé → utiliser **UC 14**. Si un objet SQL doit être créé pour un nouveau besoin → utiliser **UC 11**.

**Six types d'objets couverts :**
- **Table** (`CREATE TABLE`) — modèle de données nouveau, avec contraintes modernes (PK, FK, CHECK, DEFAULT)
- **Vue** (`CREATE OR REPLACE VIEW`) — accès sécurisé ou lecture simplifiée d'une ou plusieurs tables
- **Index** (`CREATE INDEX`) — performance d'accès ou contrainte d'unicité
- **Procédure stockée** (`CREATE OR REPLACE PROCEDURE`) — logique métier encapsulée côté base de données
- **Trigger** (`CREATE TRIGGER`) — automatisation déclenchée par un événement DML (INSERT, UPDATE, DELETE)
- **Fonction** (`CREATE OR REPLACE FUNCTION`) — fonction scalaire ou table utilisable dans un SELECT

**Convention de nommage des fichiers générés :**
```
{appArcad}-{fonction}-{composant}-objet-sql-{YYYYMMDD-HHmm}.md       ← table, vue, index
{appArcad}-{fonction}-{composant}-procedure-sql-{YYYYMMDD-HHmm}.md   ← procédure stockée
{appArcad}-{fonction}-{composant}-trigger-sql-{YYYYMMDD-HHmm}.md     ← trigger
{appArcad}-{fonction}-{domaine}-plan-sql-{YYYYMMDD-HHmm}.md          ← plan périmètre
```

Exemples :
```
acme-APPVTE-FCOMMANDES-objet-sql-20250625-0900.md      ← CREATE TABLE FCOMMANDES
acme-APPVTE-FCOMMANDES-procedure-sql-20250625-1100.md  ← CREATE PROCEDURE sur FCOMMANDES
acme-APPVTE-FCOMMANDES-trigger-sql-20250625-1400.md    ← CREATE TRIGGER sur FCOMMANDES
acme-APPVTE-APPVTE-plan-sql-20250625-1600.md           ← plan périmètre SQL complet
```

> 💡 Cette convention est valable en dehors du contexte POC — réutilisable en production tel quel.

---

## Démarrer par un objet que vous connaissez

> **Recommandation forte avant d'aborder les objets critiques de ACME.**

Commencer par une **table simple** (moins de 10 colonnes, pas de clé étrangère complexe) dont un DBA ou développeur ACME peut valider immédiatement que les types SQL sont corrects et les noms cohérents avec les conventions du client.

Pourquoi une table d'abord ? Parce que les tables sont la fondation des autres objets : les procédures accèdent des tables, les triggers se définissent sur des tables, les fonctions peuvent retourner des données de tables. Valider les types SQL et les conventions de nommage sur une table simple calibre toutes les générations suivantes.

| Étape | Objet à choisir | Catégorie attendue | Objectif |
|-------|----------------|--------------------|---------|
| 1 | Table simple, < 10 colonnes, PK évidente, pas de FK | SIMPLE | Calibrer les types SQL Db2 for i, la convention de nommage ACME, valider que `CREATE TABLE` s'exécute sans erreur |
| 2 | Table avec contraintes (FK, CHECK, DEFAULT) | STANDARD | Valider la gestion des contraintes référentielles, vérifier la présence de la table cible (FK) |
| 3 | Procédure CRUD simple (INSERT / SELECT sur la table générée à l'étape 1) | SIMPLE | Valider la structure de base d'une procédure, les paramètres IN/OUT, la gestion SQLSTATE |
| 4 | Trigger d'audit sur la table générée | SIMPLE | Valider EVENT, TIMING, FOR EACH ROW, accès à NEW et OLD |
| 5 | Fonction scalaire utilitaire | STANDARD | Valider RETURNS, DETERMINISTIC, utilisation dans un SELECT |

---

## Impact de la complexité sur la stratégie de génération

La complexité UC 11 se mesure **par type d'objet** — les critères sont différents pour une table, une procédure, un trigger ou une fonction.

### Tables

| Catégorie | Critères |
|-----------|----------|
| **SIMPLE** | < 15 colonnes, pas de FK, types SQL directs (CHAR, VARCHAR, DECIMAL, INTEGER) |
| **STANDARD** | FK vers une table existante, contraintes CHECK ou DEFAULT, types DECFLOAT ou DATE/TIME, colonne GENERATED ALWAYS |
| **COMPLEXE** | Partitionnement, colonnes calculées (AS expression), versioning de ligne (ROW CHANGE TIMESTAMP), table avec plusieurs FK imbriquées |

### Procédures stockées

| Catégorie | Critères |
|-----------|----------|
| **SIMPLE** | 1 à 3 paramètres IN, pas de curseur, logique linéaire (1 SELECT ou 1 INSERT) |
| **STANDARD** | Paramètres IN/OUT, 1 curseur, gestion SQLSTATE avec DECLARE HANDLER |
| **COMPLEXE** | Curseurs imbriqués, tables temporaires (DECLARE GLOBAL TEMPORARY TABLE), CALL d'autres procédures, logique conditionnelle sur plusieurs tables |

### Triggers

| Catégorie | Critères |
|-----------|----------|
| **SIMPLE** | 1 EVENT (INSERT ou UPDATE ou DELETE), 1 TIMING (BEFORE ou AFTER), logique d'audit simple (INSERT dans une table de log) |
| **COMPLEXE** | Logique métier conditionnelle (IF/CASE sur NEW.colonne), appel de procédure depuis le trigger, plusieurs EVENT combinés |

> ⚠️ Pour les triggers COMPLEXE avec logique métier : évaluer si cette logique ne devrait pas être dans le programme RPG appelant plutôt que dans le trigger (point à soulever au Prompt 0 — un trigger difficile à tester et à déboguer est souvent un signe que la logique appartient à la couche applicative).

### Fonctions

| Catégorie | Critères |
|-----------|----------|
| **SIMPLE** | Fonction scalaire, 1 à 2 paramètres, retourne une valeur calculée sans accès à une table |
| **STANDARD** | Fonction scalaire avec accès à 1 table, DETERMINISTIC selon les paramètres, utilisable dans un SELECT ou un WHERE |
| **COMPLEXE** | Fonction table (RETURNS TABLE), plusieurs sous-requêtes, impact sur les performances si utilisée dans un prédicat WHERE sur un grand volume |

---

## Démarrer une session Bob

> **À lire avant chaque session UC 11 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

Démarrer chaque session sur un **type d'objet** dans une **nouvelle conversation Bob** (bouton `+` en haut du panneau Chat). Ne pas mélanger les types d'objets dans la même conversation — le contexte d'un trigger et celui d'une procédure stockée sont distincts et peuvent se contrarier.

**Exception :** enchaîner une table et la procédure qui l'accède dans la même session si la procédure est directement liée à la table générée — la table est déjà dans le contexte, Bob peut vérifier les types et noms de colonnes sans relire QSYS2.

**Mode à sélectionner :** Ask ou ACME IBM i Review (génération), Agent ou ACME IBM i Execute (RUNSQLSTM et sauvegarde).

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Avant de lancer le Prompt 0, ouvrir dans l'éditeur Bob les fichiers de contexte utiles à Bob pour valider les dépendances.

**Procédure :** utiliser **Add File to Chat** (icône trombone dans le chat) pour les fichiers `.md` du workspace, ou **Open in Editor** depuis le panneau IBM i Object Browser pour les membres sources.

Fichiers à ouvrir pour chaque session UC 11 :
- Le fichier `*-ddl-complet-*.md` (UC 14) si le nouvel objet référence des tables DDL déjà produites — ouvrir pour que Bob puisse vérifier les types et noms exacts des colonnes cibles (FK, paramètres de procédures)
- Le fichier `*-spec-tech-*.md` (UC 6) si l'objet s'intègre dans une application documentée

### 3. Fichiers de contexte à charger

| Fichier | Produit par | Obligatoire / Recommandé |
|---------|-------------|--------------------------|
| `{appArcad}-{fonction}-{composant}-spec-tech-{date}.md` | UC 6 | **Recommandé** — modèle de données de référence, normes SQL ACME |
| `{appArcad}-{fonction}-{composant}-ddl-complet-{date}.md` | UC 14 | **Obligatoire si FK** — si le nouvel objet référence une table DDL déjà produite, Bob doit connaître les noms exacts des colonnes cibles ; sans ce fichier, Bob invente les noms |
| Normes SQL ACME (conventions de nommage, schémas cibles, types préférés) | — | **Recommandé** — évite que Bob applique ses conventions par défaut |

> 💡 **Tip avant le Prompt 1 (tables avec FK) :** vérifier dans `QSYS2.TABLES` et `QSYS2.SYSCOLUMNS` que les tables référencées existent déjà sur l'IBM i de test avant de générer les contraintes `FOREIGN KEY`. Une FK vers une table inexistante produit une erreur SQL `-204` à l'exécution de `CREATE TABLE`.

> ⚠️ **Risque de réduction de contexte :** une session UC 11 avec plusieurs types d'objets (table + procédure + trigger) peut atteindre la limite de contexte. Si Bob semble oublier une décision prise au Prompt 0, sauvegarder le livrable en cours en mode Agent et commencer une nouvelle conversation pour l'objet suivant.

---

## Prérequis

- UC 15 (maîtrise de Bob) et UC 12 Phase 0 (MCP) complétés
- **IBM i Database MCP actif** — indispensable pour le Prompt 1-bis (RUNSQLSTM / EXEC SQL) et l'introspection QSYS2 avant génération des FK
- UC 14 complété si les nouveaux objets référencent des tables DDL — vérifier la présence du fichier `*-ddl-complet-*.md` avant de démarrer
- Droits SQL sur l'IBM i de test : l'utilisateur IBM i associé au MCP doit disposer des autorités DDL pour les six types d'objets du périmètre — `CREATE TABLE`, `CREATE VIEW`, `CREATE INDEX`, `CREATE PROCEDURE`, `CREATE TRIGGER`, `CREATE FUNCTION`. Valider avec le DBA ACME avant de démarrer les sessions de génération.

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | **IBM i Database** (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : Ask pour la qualification et la génération, Agent pour RUNSQLSTM et la sauvegarde. |
| **Scope** | Library List → schéma SQL / bibliothèque cible ACME |
| **MCP central** | IBM i Database MCP (test de création RUNSQLSTM, introspection QSYS2) |
| **MCP secondaire** | IBM i MCP (lecture de sources RPG si la procédure migre de la logique d'un programme existant — lien avec UC 3) |
| **MCP différés** | Confluence MCP (publication des scripts DDL, si token disponible) |

### Pourquoi le mode IBM i Database pour la génération d'objets SQL ?

Le mode **IBM i Database** est spécialisé pour Db2 for i : curseurs SQL, SQLCODE, QSYS2, DDL, Index Advisor, procédures stockées. Bob génère le DDL dans le chat — l'équipe le relit, le corrige, itère sans risque avant d'autoriser l'exécution. Exécuter un `CREATE TABLE` incorrect peut écraser un objet existant.

| Phase | Comportement attendu | Ce que Bob fait |
|-------|----------------------|----------------|
| Qualification de l'objet (Prompt 0) | Génère dans le chat — pas d'écriture | Analyse le besoin, détermine le type et la catégorie, liste les dépendances, recommande la séquence |
| Génération des DDL (Prompts 1, 1V, 1I, 2, 3, 4) | Génère dans le chat — pas d'écriture | Génère `CREATE TABLE`, `CREATE VIEW`, `CREATE INDEX`, `CREATE PROCEDURE`, `CREATE TRIGGER`, `CREATE FUNCTION` dans le chat |
| Test de création (Prompt 1-bis) | Écriture et exécution autorisées — après validation | Exécute le script DDL sur l'IBM i de test, analyse les messages SQL, propose les corrections |
| Plan périmètre (Prompt 5) | Génère dans le chat — pas d'écriture | Génère le plan de génération dans le chat |
| Sauvegarde des livrables | Écriture autorisée — après validation | Écrit les fichiers `.md` dans le workspace — uniquement une fois validés |

> 💡 **Règle d'or pour UC 11 :** Bob génère dans le chat. L'exécution SQL (`RUNSQLSTM`) et la sauvegarde ne sont autorisées qu'après validation explicite de l'équipe.

> 💡 **Sans Premium Package IBM i :** utiliser le mode Ask pour la qualification et la génération (Prompts 0 à 5), puis basculer en mode Agent uniquement pour le test de création (Prompt 1-bis) et la sauvegarde finale.

### Intégration ARCAD

Le MCP ARCAD n'était pas disponible dans le contexte de ce POC de référence (version ARCAD non compatible avec le MCP). Si le MCP ARCAD est disponible dans votre environnement, les étapes manuelles de réintégration décrites ci-dessous peuvent être automatisées. N'hésitez pas à demander à Bob de modifier cette fiche UC en intégrant la disponibilité du MCP ARCAD.

**Impact sur UC 11 : faible à moyen.** Les scripts DDL générés sont de nouveaux objets SQL — ils devront être versionnés et enregistrés dans ARCAD (ou dans un gestionnaire de migrations SQL) après validation.

| Sans MCP ARCAD (contexte de ce POC) | Avec MCP ARCAD disponible |
|--------------------------------------|---------------------------|
| Vérifier manuellement dans ARCAD si un objet de même nom est déjà géré | IBM i Database MCP peut vérifier via `QSYS2.TABLES` / `QSYS2.SYSROUTINES` dans les deux cas |
| Versionner et enregistrer manuellement les scripts DDL dans ARCAD | Le MCP ARCAD peut versionner automatiquement les scripts après validation |
| Ajouter `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` dans chaque script | Le placeholder n'est plus nécessaire — la réintégration est pilotée par Bob |

---

## Prompts clés

### Prompt 0 — Qualification de l'objet SQL à générer

> **Ce prompt est le point d'entrée obligatoire de UC 11 pour chaque objet.**
> Il traduit le besoin fonctionnel en type d'objet, catégorie et séquence de prompts adaptée.
> Il se lance **avant** tout prompt de génération.

```
Je dois créer un nouvel objet SQL dans le schéma [NOM_LIB] sur IBM i.

Besoin fonctionnel : [décrire ce que doit faire l'objet — en termes métier]
[Si disponible : Le fichier *-ddl-complet-*.md (UC 14) est dans le contexte.]
[Si disponible : Le fichier *-spec-tech-*.md (UC 6) est dans le contexte.]

Produis en français, en markdown, une fiche de qualification :

## Qualification UC 11 — [NOM_OBJET_CIBLE]

### 1. Type d'objet recommandé
- [ ] Table (CREATE TABLE) — modèle de données
- [ ] Vue (CREATE OR REPLACE VIEW) — accès sécurisé ou lecture simplifiée d'une table
- [ ] Index (CREATE INDEX) — performance ou unicité
- [ ] Procédure stockée (CREATE OR REPLACE PROCEDURE) — logique encapsulée
- [ ] Trigger (CREATE TRIGGER) — automatisation sur événement DML
- [ ] Fonction (CREATE OR REPLACE FUNCTION) — valeur calculée ou table retournée

### 2. Catégorie
- [ ] SIMPLE
- [ ] STANDARD
- [ ] COMPLEXE

### 3. Dépendances et points d'attention
| Dépendance | Objet concerné | Statut |
|------------|---------------|--------|
| FK vers une table existante | [NOM_TABLE] | À vérifier dans QSYS2.TABLES |
| Procédure appelée depuis un trigger | [NOM_PROCEDURE] | À créer avant le trigger |
| Type SQL non standard | [NOM_TYPE] | Vérifier la version IBM i |

### 4. Points d'attention spécifiques
- [ ] Trigger avec logique métier complexe → évaluer si la logique appartient au programme appelant
- [ ] Fonction dans un prédicat WHERE → évaluer l'impact sur les performances
- [ ] Procédure avec curseurs imbriqués → planifier la gestion des SQLSTATE

### 5. Séquence recommandée
Selon le type et la catégorie, indiquer les prompts à enchaîner dans l'ordre.
```

**Analyse ligne à ligne :**

- `Besoin fonctionnel en termes métier` → décrire ce que doit faire l'objet (pas la syntaxe SQL). "Calculer la remise applicable à une commande selon le code client" est plus utile que "fonction scalaire avec 2 paramètres". Le second force Bob à reproduire une implémentation supposée ; le premier laisse Bob proposer le type d'objet le plus adapté.

- `Type d'objet recommandé` → question délibérément ouverte. Bob peut parfois recommander un type différent de celui supposé : une logique de validation que l'équipe pensait mettre dans un trigger est parfois mieux placée dans une procédure appelée par le programme. La qualification rend cette décision explicite.

- `Dépendances` → les FK et les appels de procédures depuis les triggers sont les deux principales sources d'erreur SQL à la création. Les lister au Prompt 0 permet de vérifier leur existence via `QSYS2.TABLES` / `QSYS2.SYSROUTINES` avant de lancer la génération.

- `Trigger avec logique complexe` → point d'attention structurel. Un trigger COMPLEXE est souvent le symptôme d'une logique qui appartient à la couche applicative. Forcer Bob à soulever ce point au Prompt 0 évite de générer un trigger difficile à tester et à déboguer.

> 💡 **Sauvegarder la fiche de qualification** (mode Agent) si la session couvre plusieurs objets :
> ```
> "Sauvegarde cette fiche de qualification dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-objet-sql-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-FCOMMANDES-objet-sql-20250625-0900.md"
> ```

> ⚠️ **Piège évité :** sans le Prompt 0, l'équipe démarre directement la génération d'une procédure et découvre en milieu de session que la table cible n'existe pas encore ou que les noms de colonnes diffèrent du `*-ddl-complet-*.md`. Le Prompt 0 déplace cette découverte au tout début.

---

### Prompt 1 — Génération d'une table (CREATE TABLE)

```
Sur la base de la qualification UC 11 que nous venons de faire pour [NOM_TABLE],
génère en français, en markdown, le script DDL SQL CREATE TABLE pour Db2 for i :

## Script DDL — CREATE TABLE [NOM_TABLE]

-- ============================================================
-- TABLE : [NOM_LIB].[NOM_TABLE]
-- Besoin fonctionnel : [résumé en 1 ligne]
-- Généré le : [DATE]
-- Statut : À valider et tester sur IBM i de test avant tout usage production
-- ⚠️ Réintégration ARCAD — à effectuer manuellement après validation
-- ============================================================

-- Si l'objet existe déjà, ce script s'arrête avec -601 (objet en conflit).
-- Signaler l'erreur et demander une décision explicite avant de continuer.
-- N'utiliser CREATE OR REPLACE TABLE que sur un schéma POC jetable et de manière explicitement destructive.
CREATE TABLE [NOM_LIB].[NOM_TABLE] (
  -- [NOM_COLONNE]  [TYPE_SQL]  [NULL/NOT NULL]  DEFAULT [valeur]  -- [commentaire métier]
  ...
  CONSTRAINT [NOM_TABLE]_PK PRIMARY KEY ([COLONNES_CLE])
  [, CONSTRAINT [NOM_TABLE]_FK_[NOM_COL] FOREIGN KEY ([NOM_COL])
       REFERENCES [NOM_LIB].[NOM_TABLE_REF] ([NOM_COL_REF])
       ON DELETE [RESTRICT/CASCADE/SET NULL]  ]
  [, CONSTRAINT [NOM_TABLE]_CHK_[NOM_COL] CHECK ([condition])  ]
)
;

Règles de génération :
- Types SQL Db2 for i natifs uniquement : CHAR, VARCHAR, DECIMAL, INTEGER, SMALLINT, DATE, TIME, TIMESTAMP, DECFLOAT
- CHAR(n) si n ≤ 30 ; VARCHAR(n) si n > 30 ou champ de texte libre
- DECIMAL(n,d) pour les montants et zones décimales
- DATE, TIME, TIMESTAMP pour les colonnes temporelles (pas CHAR(8) sauf si les données existantes l'imposent)
- CREATE TABLE (pas OR REPLACE) — si l'objet existe déjà, le script doit s'arrêter avec -601 et demander une décision explicite. Pour modifier une table existante, utiliser ALTER TABLE. CREATE OR REPLACE TABLE peut être accepté uniquement sur un schéma POC jetable annoncé explicitement dans le prompt
- Ne pas inventer de contraintes non exprimées dans le besoin
- Commenter chaque colonne avec son rôle métier
- Après la création de la table, générer les instructions LABEL ON COLUMN :
  LABEL ON COLUMN [NOM_LIB].[NOM_TABLE]
  (
    [COLONNE1] TEXT IS '[LIBELLÉ MÉTIER]',
    [COLONNE2] TEXT IS '[LIBELLÉ MÉTIER]'
  );
```

**Analyse ligne à ligne :**

- `CREATE TABLE` → utilisé pour un objet neuf. Si l'objet existe déjà, le script s'arrête avec `-601` — le signaler et demander une décision explicite avant de continuer. `CREATE OR REPLACE TABLE` peut être utilisé dans un exercice explicitement destructif sur un schéma POC jetable, mais pas comme mécanisme d'idempotence standard. Pour une évolution de structure, utiliser `ALTER TABLE`.

- `CONSTRAINT [NOM_TABLE]_PK / _FK_ / _CHK_` → convention de nommage explicite pour les contraintes. Un nom de contrainte sans convention produit des messages d'erreur SQL illisibles (`-204 : CONSTRAINT1234 not found`) — le nommage systématique facilite le diagnostic.

- `ON DELETE [RESTRICT/CASCADE/SET NULL]` → forcer Bob à choisir explicitement le comportement de suppression pour chaque FK. `RESTRICT` est le plus sûr par défaut (interdit la suppression si des lignes enfants existent) ; `CASCADE` est dangereux sans validation métier préalable.

- `LABEL ON COLUMN` → sur Db2 for i, `COLUMN_TEXT` dans `QSYS2.SYSCOLUMNS` est alimenté par une instruction `LABEL ON COLUMN`, pas par les commentaires SQL `--`. Après la création de la table, Bob doit générer les instructions `LABEL ON COLUMN [NOM_LIB].[NOM_TABLE] ([COL] TEXT IS '[libellé]')` pour renseigner les descriptifs visibles dans l'Object Browser. Les commentaires `--` dans le DDL restent utiles pour la lisibilité du script mais ne populent pas `COLUMN_TEXT`.

> 💡 **Sauvegarder ce script** (mode Agent) avant de lancer le Prompt 1-bis :
> ```
> "Sauvegarde ce script CREATE TABLE dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-objet-sql-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-FCOMMANDES-objet-sql-20250625-0900.md"
> ```

> ⚠️ **Piège évité :** sans la règle `VARCHAR(n)` pour les champs longs, Bob peut générer `CHAR(200)` pour des colonnes de texte libre — gaspillage d'espace et dégradation des performances d'indexation.

---

### Prompt 1V — Génération d'une vue (CREATE OR REPLACE VIEW)

```
Sur la base de la qualification UC 11 pour [NOM_VUE],
génère en français, en markdown, le script DDL de la vue pour Db2 for i :

## Script DDL — CREATE VIEW [NOM_VUE]

-- ============================================================
-- VIEW : [NOM_LIB].[NOM_VUE]
-- Tables sources : [NOM_LIB].[NOM_TABLE1] [, NOM_TABLE2...]
-- Besoin fonctionnel : [résumé en 1 ligne]
-- Généré le : [DATE]
-- Statut : À valider et tester sur IBM i de test avant tout usage production
-- ⚠️ Réintégration ARCAD — à effectuer manuellement après validation
-- ============================================================

CREATE OR REPLACE VIEW [NOM_LIB].[NOM_VUE] AS
  SELECT
    [alias1].[colonne1],
    [alias1].[colonne2],
    [alias2].[colonne3]
  FROM [NOM_LIB].[NOM_TABLE1] [alias1]
  [JOIN [NOM_LIB].[NOM_TABLE2] [alias2]
    ON [alias1].[col_cle] = [alias2].[col_cle] ]
  [WHERE [condition_filtre]]
;

Règles :
- CREATE OR REPLACE VIEW pour l'idempotence — une vue ne contient pas de données de table ; son remplacement ne détruit pas de lignes. Attention : OR REPLACE remplace la définition existante et peut casser les objets ou programmes dépendants si le contrat de colonnes change.
- Ne sélectionner que les colonnes nécessaires — pas de SELECT * (une vue créée avec SELECT * ne se met pas automatiquement à jour si une colonne est ajoutée à la table source ; le contrat de colonnes dérive silencieusement sans recréation)
- Préfixer les colonnes ambiguës par l'alias de table
- Si la vue est destinée uniquement à la lecture, ajouter WITH READ ONLY
- Documenter le critère de filtre WHERE en commentaire si non évident
```

**Analyse ligne à ligne :**

- `CREATE OR REPLACE VIEW` → contrairement à `CREATE TABLE`, le remplacement d'une vue ne détruit pas de données de table. Mais `OR REPLACE` réécrit la définition — cela peut casser les objets ou programmes dépendants si le contrat de colonnes change. IBM décrit le remplacement comme un drop/recreate logique de la définition existante.

- `WITH READ ONLY` → empêche toute tentative d'INSERT/UPDATE/DELETE via la vue — erreur SQL `-150` immédiate si tentée. À utiliser pour les vues destinées aux rapports ou aux accès lecture seule.

- `Pas de SELECT *` → une vue créée avec `SELECT *` ne propagera pas automatiquement une nouvelle colonne ajoutée à la table source sans recréation. Le contrat de colonnes dérive silencieusement — ce n'est pas une erreur de compilation, c'est une dérive fonctionnelle.

> 💡 **Sauvegarder ce script** (mode Agent) avant de lancer le Prompt 1-bis :
> ```
> "Sauvegarde ce script CREATE VIEW dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-objet-sql-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-FCOMMANDES-objet-sql-20250625-0920.md"
> ```

> ➡️ **Après la sauvegarde : lancer le Prompt 1-bis** pour tester la création de cette vue sur l'IBM i de test.

> ⚠️ **Piège évité :** une vue sur une table non encore créée échoue au `CREATE VIEW` avec `-204`. Vérifier via `QSYS2.TABLES` que toutes les tables sources existent avant de lancer le Prompt 1-bis.

---

### Prompt 1I — Génération d'un index (CREATE INDEX)

```
Sur la base de la qualification UC 11 pour [NOM_INDEX],
génère en français, en markdown, le script DDL de l'index pour Db2 for i :

## Script DDL — CREATE INDEX [NOM_INDEX]

-- ============================================================
-- INDEX : [NOM_LIB].[NOM_INDEX]
-- Table cible : [NOM_LIB].[NOM_TABLE]
-- Colonnes indexées : [COL1 [ASC/DESC], COL2 [ASC/DESC]...]
-- Besoin fonctionnel : [résumé en 1 ligne]
-- Généré le : [DATE]
-- Statut : À valider et tester sur IBM i de test avant tout usage production
-- ⚠️ Réintégration ARCAD — à effectuer manuellement après validation
-- ============================================================

CREATE [UNIQUE] INDEX [NOM_LIB].[NOM_INDEX]
  ON [NOM_LIB].[NOM_TABLE]
  ( [COL1] [ASC/DESC] [, COL2 [ASC/DESC]] )
;

Règles :
- UNIQUE si les valeurs de la combinaison de colonnes doivent être uniques (contrainte fonctionnelle)
- Sans UNIQUE si l'index est destiné à accélérer les recherches (index de performance)
- Ne pas créer un index UNIQUE sur des colonnes admettant des doublons — la création échoue avec -603 sur une table existante avec doublons ; un INSERT/UPDATE ultérieur viole la contrainte avec -803
- Documenter pourquoi cet index est créé (requête ou programme qui en bénéficie)
- Sur IBM i, les index DRI (créés par les contraintes FK) sont gérés automatiquement — ne pas créer un index manuel en doublon d'une FK existante
```

**Analyse ligne à ligne :**

- `UNIQUE vs non UNIQUE` → un index UNIQUE est une contrainte fonctionnelle (les doublons sont interdits). Un index non UNIQUE est un outil de performance (l'optimiseur peut l'utiliser, les données peuvent se répéter). Attention : `-603` est l'erreur retournée lors de la création de l'index si des doublons existent déjà dans la table ; `-803` est la violation de contrainte lors d'un INSERT/UPDATE ultérieur.

- `Ne pas dupliquer une FK` → sur IBM i, chaque contrainte de clé étrangère crée automatiquement un index interne sur la colonne référençant. Créer un deuxième index manuel sur la même colonne est un gaspillage de ressources.

- `Documenter le bénéficiaire` → un index sans documentation de la requête ou du programme qui en bénéficie devient un objet orphelin. Le commentaire `-- Besoin fonctionnel` est obligatoire pour la maintenabilité.

> 💡 **Sauvegarder ce script** (mode Agent) avant de lancer le Prompt 1-bis :
> ```
> "Sauvegarde ce script CREATE INDEX dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-objet-sql-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-FCOMMANDES-objet-sql-20250625-0930.md"
> ```

> ➡️ **Après la sauvegarde : lancer le Prompt 1-bis** pour tester la création de cet index sur l'IBM i de test.

> ⚠️ **Piège évité :** un `CREATE UNIQUE INDEX` sur une table déjà peuplée avec des doublons échoue avec `-603`. Vérifier l'unicité des valeurs via `SELECT [COL1], COUNT(*) FROM [NOM_LIB].[NOM_TABLE] GROUP BY [COL1] HAVING COUNT(*) > 1` avant de créer un index UNIQUE sur une table existante.

---



### Prompt 1-bis — Test de création RUNSQLSTM

> **Ce prompt s'exécute en mode Agent**, immédiatement après chaque prompt de génération (Prompts 1, 1V, 1I, 2, 3, 4).
> **Ce que Bob peut faire :** exécuter le script DDL sur l'IBM i de test et rapporter les erreurs SQL.
> **Ce que Bob ne peut pas faire :** valider que l'objet est fonctionnellement correct — la logique (types, contraintes, comportement du trigger) reste à vérifier par un développeur.
> **Quand l'utiliser :** après le Prompt 1 (`CREATE TABLE`), après le Prompt 1V (`CREATE VIEW`), après le Prompt 1I (`CREATE INDEX`), après le Prompt 2 (`CREATE PROCEDURE`), après le Prompt 3 (`CREATE TRIGGER`), après le Prompt 4 (`CREATE FUNCTION`) — avant toute sauvegarde du script final.

```
Le script [CREATE TABLE / CREATE VIEW / CREATE INDEX / CREATE PROCEDURE / CREATE TRIGGER / CREATE FUNCTION] [NOM_OBJET]
vient d'être généré.

Exécute ce script sur l'IBM i de test [NOM_LIB_TEST] via IBM i Database MCP :

Option 1 (script court) : exécuter directement via EXEC SQL
Option 2 (script long) : soumettre via RUNSQLSTM SRCSTMF('[CHEMIN_SCRIPT]') COMMIT(*NONE) NAMING(*SQL)

Note : COMMIT(*NONE) signifie que le commitment control n'est pas utilisé pour cette exécution. Ce choix doit être réservé au schéma POC — valider avec le DBA pour les scripts de production.

Analyse le résultat et produis en français :
1. Statut : CRÉATION RÉUSSIE / ERREURS SQL
2. Si erreurs :
   | Code SQL | Description | Objet concerné | Cause probable | Correction proposée |
3. Si création réussie : confirmer le nom complet de l'objet créé ([NOM_LIB_TEST].[NOM_OBJET])
4. Si réussi : contrôle selon le type d'objet créé :
   - Table ou vue  → QSYS2.SYSCOLUMNS (vérifier le nombre de colonnes)
   - Index         → QSYS2.SYSINDEXES (vérifier le nom et la table cible)
   - Procédure ou fonction → QSYS2.SYSROUTINES (vérifier le nom SPECIFIC et les paramètres)
   - Trigger       → QSYS2.SYSTRIGGERS (vérifier l'événement et le timing)

Ne pas modifier les données existantes — création de structure uniquement.
```

**Analyse ligne à ligne :**

- `COMMIT(*NONE)` → signifie que le commitment control n'est pas utilisé pour cette exécution. Ce choix doit être réservé au schéma POC — valider avec le DBA pour les scripts de production.

- `| Code SQL | ... | Correction proposée |` → Bob connaît les codes SQL courants : `-601` (objet déjà existant), `-204` (table ou objet cible introuvable), `-104` (erreur de syntaxe), `-551` (droits insuffisants). La colonne "Correction proposée" permet une correction immédiate dans le chat sans sortir de la session.

- `Contrôle selon le type d'objet` → la vue à interroger dépend du type d'objet créé : `SYSCOLUMNS` pour les tables et vues, `SYSINDEXES` pour les index, `SYSROUTINES` pour les procédures et fonctions, `SYSTRIGGERS` pour les triggers. Un type invalide dans une `CREATE TABLE` provoque une erreur de création, pas une omission silencieuse.

> 💡 Ce prompt s'exécute en mode **Agent** — vérifier que l'utilisateur IBM i associé au MCP a les droits `*CHANGE` sur la bibliothèque cible et les droits DDL correspondants dans le schéma SQL.

---

### Prompt 2 — Génération d'une procédure stockée (CREATE OR REPLACE PROCEDURE)

```
Sur la base de la qualification UC 11 pour [NOM_PROCEDURE],
génère en français, en markdown, le script DDL de la procédure stockée pour Db2 for i :

## Script DDL — CREATE PROCEDURE [NOM_PROCEDURE]

-- ============================================================
-- PROCEDURE : [NOM_LIB].[NOM_PROCEDURE]
-- Besoin fonctionnel : [résumé en 1 ligne]
-- Généré le : [DATE]
-- Statut : À valider et tester sur IBM i de test avant tout usage production
-- ⚠️ Réintégration ARCAD — à effectuer manuellement après validation
-- ============================================================

CREATE OR REPLACE PROCEDURE [NOM_LIB].[NOM_PROCEDURE] (
  IN  [param1]  [TYPE],
  IN  [param2]  [TYPE],
  OUT [param3]  [TYPE]
)
LANGUAGE SQL
SPECIFIC [NOM_LIB].[NOM_PROCEDURE]
BEGIN
  -- 1. Déclarations variables locales (toujours en premier)
  DECLARE [var1]           [TYPE]        DEFAULT [valeur] ;
  DECLARE v_done           SMALLINT      DEFAULT 0 ;
  DECLARE v_message        VARCHAR(2048) DEFAULT '' ;
  DECLARE [SQLSTATE_LOCAL] CHAR(5)       DEFAULT '00000' ;

  -- 2. Déclarations de curseurs (obligatoirement AVANT les handlers)
  -- Si curseur nécessaire :
  DECLARE [cur1] CURSOR FOR
    SELECT [colonnes] FROM [NOM_LIB].[NOM_TABLE]
    WHERE [condition] ;

  -- 3. Handlers (APRÈS variables et curseurs — ordre imposé par SQL PL)
  -- Handler fin de curseur (NOT FOUND)
  DECLARE CONTINUE HANDLER FOR NOT FOUND
    SET v_done = 1 ;

  -- Handler erreurs SQL non récupérables (EXIT — arrêt du bloc)
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    GET DIAGNOSTICS CONDITION 1
      [SQLSTATE_LOCAL] = RETURNED_SQLSTATE,
      v_message        = MESSAGE_TEXT ;
    SET [param3] = [SQLSTATE_LOCAL] ;
  END ;

  -- [Corps de la procédure]
  OPEN [cur1] ;
  FETCH [cur1] INTO [variables] ;
  WHILE v_done = 0 DO
    -- traitement
    FETCH [cur1] INTO [variables] ;
  END WHILE ;
  CLOSE [cur1] ;

  -- Retour du statut
  SET [param3] = [SQLSTATE_LOCAL] ;
END
;

Règles :
- CREATE OR REPLACE PROCEDURE pour l'idempotence (rejeu sans DROP)
- Ordre SQL PL impératif : variables → curseurs → handlers (toute inversion provoque une erreur de compilation)
- DECLARE CONTINUE HANDLER FOR NOT FOUND pour la fin de curseur (condition normale)
- DECLARE EXIT HANDLER FOR SQLEXCEPTION avec GET DIAGNOSTICS pour les erreurs non récupérables
- SPECIFIC attribue un nom unique à une instance précise de la procédure. En cas de surcharge, utiliser un nom SPECIFIC distinct pour chaque signature, ou laisser Db2 générer ce nom.
- Privilégier SQLSTATE pour les nouveaux développements — SQLSTATE est le standard SQL portable (5 caractères). SQLCODE reste disponible sur Db2 but n'est pas portable de façon homogène entre produits.
- Paramètres OUT de type CHAR(5) pour retourner un SQLSTATE au programme appelant
```

**Analyse ligne à ligne :**

- `CREATE OR REPLACE PROCEDURE` → idempotence : le script peut être rejoué sans `DROP PROCEDURE` préalable. Essentiel dans un contexte de tests itératifs sur l'IBM i de test.

- `Ordre SQL PL : variables → curseurs → handlers` → SQL PL impose un ordre strict dans le BEGIN. Toute déclaration de curseur après un handler produit une erreur de compilation. Le template respecte cet ordre.

- `DECLARE CONTINUE HANDLER FOR NOT FOUND` → signale la fin normale de lecture d'un curseur. Ce n'est pas une erreur — `CONTINUE` est le comportement correct.

- `DECLARE EXIT HANDLER FOR SQLEXCEPTION` → gère les erreurs SQL non récupérables. `EXIT` est correct pour une procédure de mise à jour : en cas d'erreur inattendue, le traitement s'arrête immédiatement sans poursuivre dans un état potentiellement incohérent. `GET DIAGNOSTICS` récupère le SQLSTATE réel et le message avant que l'information ne soit écrasée.

- `SPECIFIC [NOM_PROCEDURE]` → le nom SPECIFIC permet de référencer la procédure sans ambiguïté (utile en cas de surcharge). Sur IBM i, le SPECIFIC name est aussi utilisé dans les vues `QSYS2.SYSROUTINES` pour l'audit et le suivi des procédures.

- `SQLSTATE vs SQLCODE` → SQLSTATE est le standard SQL privilégié pour les nouveaux développements. SQLCODE reste disponible sur Db2, mais n'est pas portable de façon homogène entre les produits Db2 — à éviter dans les nouveaux scripts.

- `Paramètres OUT pour le statut` → le programme RPG appelant doit connaître le statut d'exécution de la procédure. Un paramètre OUT contenant le SQLSTATE final est la convention la plus simple et la plus portable.

> 💡 **Sauvegarder ce script** (mode Agent) :
> ```
> "Sauvegarde ce script dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-procedure-sql-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-FCOMMANDES-procedure-sql-20250625-1100.md"
> ```

> ➡️ **Après la sauvegarde : lancer le Prompt 1-bis** pour tester la création de cette procédure sur l'IBM i de test avant de passer à l'objet suivant.

> ⚠️ **Piège évité :** une procédure sans DECLARE HANDLER peut laisser un curseur ouvert en cas d'erreur SQL. Le prochain appel retourne alors `-502` (curseur déjà ouvert) — difficile à diagnostiquer si la gestion d'erreur est absente.

---

### Prompt 3 — Génération d'un trigger (CREATE TRIGGER)

```
Sur la base de la qualification UC 11 pour [NOM_TRIGGER],
génère en français, en markdown, le script DDL du trigger pour Db2 for i :

## Script DDL — CREATE TRIGGER [NOM_TRIGGER]

-- ============================================================
-- TRIGGER : [NOM_LIB].[NOM_TRIGGER]
-- Table concernée : [NOM_LIB].[NOM_TABLE]
-- Événement : [INSERT / UPDATE / DELETE]
-- Timing : [BEFORE / AFTER]
-- Besoin fonctionnel : [résumé en 1 ligne]
-- Généré le : [DATE]
-- Statut : À valider et tester sur IBM i de test avant tout usage production
-- ⚠️ Réintégration ARCAD — à effectuer manuellement après validation
-- ============================================================

CREATE OR REPLACE TRIGGER [NOM_LIB].[NOM_TRIGGER]
  [BEFORE / AFTER] [INSERT / UPDATE / DELETE]
  ON [NOM_LIB].[NOM_TABLE]
  REFERENCING [NEW AS new_row] [OLD AS old_row]
  FOR EACH ROW
  MODE DB2ROW
BEGIN ATOMIC
  -- [Corps du trigger — accès à new_row et old_row]
END
;

Règles :
- BEFORE pour validation (modifier NEW ou lever une erreur avant l'écriture)
- AFTER pour audit (écrire dans une table de log après l'écriture)
- FOR EACH ROW obligatoire sur IBM i pour accéder à NEW et OLD
- MODE DB2ROW (standard IBM i)
- BEGIN ATOMIC pour que le trigger soit transactionnel
- Ne pas appeler de procédures avec une logique métier complexe depuis le trigger
  si la catégorie est COMPLEXE — noter un TODO et orienter vers le programme appelant
- Signaler si le trigger peut être récursif (UPDATE qui déclenche UPDATE sur la même table)
```

**Analyse ligne à ligne :**

- `BEFORE vs AFTER selon le cas d'usage` → BEFORE : valider ou modifier les données avant l'écriture (ex. vérifier qu'un montant est positif, calculer un champ dérivé). AFTER : journaliser après l'écriture (ex. insérer dans une table d'audit). Le choix est fonctionnel, pas technique — forcer Bob à le justifier dans la sortie du Prompt 0.

- `REFERENCING NEW AS new_row OLD AS old_row` → syntaxe IBM i pour accéder aux valeurs avant et après la modification. `OLD` n'est pas disponible sur un INSERT (il n'y a pas de valeur avant). `NEW` n'est pas disponible sur un DELETE. Bob doit inclure uniquement les clauses applicables selon l'EVENT.

- `FOR EACH ROW` → obligatoire pour accéder à NEW et OLD sur IBM i. Sans cette clause, le trigger est en mode statement-level — un seul déclenchement par instruction DML, sans accès aux lignes individuelles.

- `BEGIN ATOMIC` → assure que le corps du trigger est exécuté dans la transaction de l'instruction DML déclencheuse. Si le trigger échoue, l'instruction DML est annulée — comportement attendu pour les triggers de validation.

- `Signaler si le trigger peut être récursif` → un trigger sur UPDATE qui fait un UPDATE sur la même table se déclenche lui-même à l'infini. Bob doit détecter ce cas au Prompt 0 et l'indiquer explicitement.

> 💡 **Sauvegarder ce script** (mode Agent) :
> ```
> "Sauvegarde ce script dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-trigger-sql-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-FCOMMANDES-trigger-sql-20250625-1400.md"
> ```

> ➡️ **Après la sauvegarde : lancer le Prompt 1-bis** pour tester la création de ce trigger sur l'IBM i de test avant de passer à l'objet suivant.

> ⚠️ **Piège évité :** un trigger AFTER UPDATE qui fait un UPDATE sur la même table → récursivité infinie. Toujours vérifier que le corps du trigger n'écrit pas dans la table déclencheuse sans condition d'arrêt.

---

### Prompt 4 — Génération d'une fonction (CREATE OR REPLACE FUNCTION)

```
Sur la base de la qualification UC 11 pour [NOM_FONCTION],
génère en français, en markdown, le script DDL de la fonction pour Db2 for i :

## Script DDL — CREATE FUNCTION [NOM_FONCTION]

-- ============================================================
-- FUNCTION : [NOM_LIB].[NOM_FONCTION]
-- Type : [Scalaire / Table]
-- Besoin fonctionnel : [résumé en 1 ligne]
-- Généré le : [DATE]
-- Statut : À valider et tester sur IBM i de test avant tout usage production
-- ⚠️ Réintégration ARCAD — à effectuer manuellement après validation
-- ============================================================

[Pour une fonction scalaire :]
CREATE OR REPLACE FUNCTION [NOM_LIB].[NOM_FONCTION] (
  [param1] [TYPE],
  [param2] [TYPE]
)
RETURNS [TYPE_RETOUR]
LANGUAGE SQL
[DETERMINISTIC / NOT DETERMINISTIC]
SPECIFIC [NOM_LIB].[NOM_FONCTION]
BEGIN
  DECLARE [result] [TYPE_RETOUR] ;
  -- [Corps de la fonction]
  RETURN [result] ;
END
;

[Pour une fonction table :]
CREATE OR REPLACE FUNCTION [NOM_LIB].[NOM_FONCTION] (
  [param1] [TYPE]
)
RETURNS TABLE (
  [col1] [TYPE],
  [col2] [TYPE]
)
LANGUAGE SQL
NOT DETERMINISTIC
BEGIN
  RETURN
    SELECT [col1], [col2]
    FROM [NOM_LIB].[NOM_TABLE]
    WHERE [condition] ;
END
;

Règles :
- DETERMINISTIC si le même appel avec les mêmes paramètres retourne toujours le même résultat quel que soit l'état de la base
- NOT DETERMINISTIC si le résultat de la fonction peut varier pour les mêmes paramètres selon l'état des données — une fonction dont le résultat dépend de données de table modifiables est généralement NOT DETERMINISTIC. Le déterminisme est distinct de la classification d'accès SQL (READS SQL DATA, MODIFIES SQL DATA) ; évaluer selon la stabilité du résultat, pas selon le seul fait d'accéder à une table.
- Si utilisée dans un prédicat WHERE, demander à Bob de signaler l'impact potentiel sur les performances et de recommander un plan d'accès approprié — l'impact réel dépend de la requête et doit être vérifié avec le plan d'accès SQL, pas présumé systématique
```

**Analyse ligne à ligne :**

- `Scalaire vs Table` → deux cas d'usage distincts. Scalaire : retourne une valeur unique, utilisable dans n'importe quel SELECT ou expression. Table : retourne un ensemble de lignes, utilisable dans un FROM — utile pour encapsuler une requête complexe réutilisée dans plusieurs programmes.

- `DETERMINISTIC vs NOT DETERMINISTIC` → DETERMINISTIC indique que le même appel avec les mêmes paramètres retourne toujours le même résultat — Db2 sépare la déterminisme de la classification d'accès SQL (`READS SQL DATA`, `MODIFIES SQL DATA`, etc.). Une fonction qui lit une table est généralement NOT DETERMINISTIC, mais le choix doit être justifié dans le Prompt 0. Forcer Bob à justifier son choix explicitement.

- `Impact sur les performances dans un WHERE` → une fonction NOT DETERMINISTIC dans un `WHERE` peut entraîner un appel par ligne selon le plan d'accès. L'impact réel dépend de la requête, des index disponibles et du coût estimé par l'optimiseur. Bob doit signaler le risque dès le Prompt 0 si la qualification indique une utilisation dans un prédicat — et recommander de vérifier le plan d'accès via le **Visual Explain** (outil ACS) ou `QSYS2.ACTIVE_QUERY_INFO` (requêtes SQE actives).

> 💡 **Sauvegarder ce script** (mode Agent) dans le fichier `*-objet-sql-*` si la fonction est liée à la table générée au Prompt 1, ou dans un fichier dédié si elle est indépendante.

> ➡️ **Après la sauvegarde : lancer le Prompt 1-bis** pour tester la création de cette fonction sur l'IBM i de test avant de passer à l'objet suivant.

---

### Prompt 5 — Plan de génération périmètre SQL

```
Sur la base des besoins identifiés pour l'application [NOM_APPLICATION] et des objets SQL
déjà générés dans cette session, produis en français, en markdown, le plan de génération
du périmètre SQL complet :

## Plan de génération SQL — [NOM_APPLICATION]

### Inventaire des objets SQL à créer
| Objet | Type | Catégorie (S/St/C) | Dépendances | Droits requis | Priorité | Statut |
|-------|------|--------------------|-------------|---------------|----------|--------|
| [NOM_TABLE] | Table | ? | — | CREATE TABLE | 1 | À générer |
| [NOM_VUE] | Vue | ? | [NOM_TABLE] | CREATE VIEW | 2 | À générer |
| [NOM_INDEX] | Index | ? | [NOM_TABLE] | CREATE INDEX | 3 | À générer |
| [NOM_PROC] | Procédure | ? | [NOM_TABLE] | CREATE PROCEDURE | 4 | À générer |
| [NOM_TRIG] | Trigger | ? | [NOM_TABLE] | CREATE TRIGGER | 5 | À générer |
| [NOM_FUNC] | Fonction | ? | [NOM_TABLE] | CREATE FUNCTION | 6 | À générer |

Catégorie : S = SIMPLE / St = STANDARD / C = COMPLEXE

### Ordre de création obligatoire
1. Tables sans dépendances (pas de FK)
2. Tables avec FK (leurs tables cibles doivent exister)
3. Vues (leurs tables sources doivent exister)
4. Index (leur table cible doit exister)
5. Procédures (leurs tables doivent exister)
6. Triggers (leur table cible doit exister ; les procédures appelées doivent exister)
7. Fonctions (leurs objets référencés doivent exister — le déterminisme DETERMINISTIC/NOT DETERMINISTIC dépend de la stabilité du résultat pour les mêmes paramètres, pas uniquement de l'accès à une table)

### Droits à accorder après création (GRANT)
| Objet | Profil bénéficiaire | Droits à accorder |
|-------|-------------------|-------------------|
| [NOM_TABLE] | [PROFIL_APPLICATIF] | SELECT, INSERT, UPDATE, DELETE |
| [NOM_VUE] | [PROFIL_APPLICATIF] | SELECT (ajouter INSERT/UPDATE/DELETE uniquement si la vue est modifiable et que le besoin l'exige explicitement) |
| [NOM_INDEX] | — | Aucun GRANT — l'index est utilisé implicitement par l'optimiseur ; aucun droit d'utilisation à accorder |
| [NOM_PROC] | [PROFIL_APPLICATIF] | EXECUTE |
| [NOM_TRIG] | — | (automatique — le trigger s'exécute sous le profil de l'instruction DML) |
| [NOM_FUNC] | [PROFIL_APPLICATIF] | EXECUTE |

Ne pas inventer d'objets non exprimés dans le besoin ou non visibles dans les sources disponibles.
```

**Analyse ligne à ligne :**

- `Ordre de création obligatoire` → sur IBM i, une FK vers une table inexistante lève une erreur SQL `-204` à la création. Un trigger sur une procédure non encore créée lève une erreur `-204` également. L'ordre est non négociable — Bob doit le respecter et l'expliquer dans le plan.

- `Droits à accorder après création (GRANT)` → sur IBM i, un objet créé par l'administrateur n'est pas automatiquement accessible aux profils applicatifs. La section GRANT est souvent oubliée dans un contexte POC et découverte en production quand les programmes retournent `-551` (droits insuffisants).

- `Statut "À générer"` → le plan est un document vivant. Au fur et à mesure de la session, chaque objet passe de "À générer" à "Généré" à "Testé" à "Validé". Ce suivi est le livrable de pilotage de UC 11.

> 💡 **Sauvegarder ce plan** (mode Agent) :
> ```
> "Sauvegarde ce plan dans un fichier nommé
>  {appArcad}-{fonction}-{domaine}-plan-sql-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-APPVTE-plan-sql-20250625-1600.md"
> ```

> ⚠️ **Ce prompt est en mode Ask.** Le plan est généré dans le chat — le DBA ACME valide l'ordre, les dépendances et les droits avant que Bob ne sauvegarde (mode Agent).

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Ouverture des fichiers `*-ddl-complet-*.md` dans l'éditeur pour fournir le contexte de tables existantes ; navigation Object Browser pour vérifier les objets créés sur l'IBM i de test |
| **IBM i Languages** | Coloration syntaxique SQL — indispensable pour relire les scripts DDL générés dans l'éditeur avant exécution |
| **Markdown All in One** | Prévisualisation des scripts DDL et du plan de génération sauvegardés en markdown |

> 💡 Contrairement à UC 9, Mermaid Preview n'est pas nécessaire pour UC 11 — les dépendances entre objets SQL sont exprimées en tableau dans le Prompt 5, pas en diagramme.

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i Database MCP** | **MCP central de cet UC** — exécution des scripts DDL via RUNSQLSTM ou EXEC SQL (Prompt 1-bis), introspection `QSYS2.TABLES`, `QSYS2.SYSCOLUMNS`, `QSYS2.SYSROUTINES`, `QSYS2.SYSTRIGGERS` pour valider les dépendances et les objets créés |
| **IBM i MCP** | Lecture de sources RPG existants si la procédure stockée migre de la logique d'un programme (lien avec UC 3) |
| **Confluence MCP** *(si disponible)* | Publication des scripts DDL et du plan de génération dans l'espace POC |

> ⚠️ **IBM i Database MCP est l'outil central de cet UC** — contrairement à UC 9 où c'est IBM i MCP (compilation RPG). Vérifier que le MCP Database est actif et que l'utilisateur a les droits DDL avant de démarrer.

> 💡 **Requêtes QSYS2 utiles pour UC 11 :**
> ```sql
> -- Vérifier qu'une table existe avant de créer une FK
> SELECT TABLE_SCHEMA, TABLE_NAME, TABLE_TYPE
> FROM QSYS2.TABLES
> WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_TABLE]';
>
> -- Colonnes d'une table (pour vérifier les types des FK et des paramètres de procédures)
> SELECT COLUMN_NAME, DATA_TYPE, LENGTH, NUMERIC_SCALE, IS_NULLABLE
> FROM QSYS2.SYSCOLUMNS
> WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_TABLE]'
> ORDER BY ORDINAL_POSITION;
>
> -- Procédures et fonctions existantes (éviter les conflits de noms)
> SELECT ROUTINE_SCHEMA, ROUTINE_NAME, ROUTINE_TYPE, SPECIFIC_NAME
> FROM QSYS2.SYSROUTINES
> WHERE ROUTINE_SCHEMA = '[NOM_LIB]'
> ORDER BY ROUTINE_NAME;
>
> -- Triggers existants sur une table
> SELECT TRIGGER_NAME, EVENT_MANIPULATION, ACTION_TIMING, TRIGGER_BODY
> FROM QSYS2.SYSTRIGGERS
> WHERE EVENT_OBJECT_SCHEMA = '[NOM_LIB]'
>   AND EVENT_OBJECT_TABLE  = '[NOM_TABLE]';
> ```

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Confondre UC 11 et UC 14 | UC 11 est utilisé pour convertir un DDS existant → la table créée peut dupliquer une table DDL déjà produite par UC 14 | Si un fichier DDS existe pour cet objet, utiliser UC 14 (conversion) ; UC 11 s'applique uniquement aux nouveaux objets sans source DDS |
| Générer une FK sans vérifier que la table cible existe | Erreur SQL `-204` à l'exécution du `CREATE TABLE` — le Prompt 1-bis échoue et la table n'est pas créée | Vérifier via `QSYS2.TABLES` que la table cible existe avant le Prompt 1 ; ou enchaîner la création de la table parent en premier |
| Utiliser des types SQL non supportés par la version IBM i du client | Le `CREATE TABLE` échoue avec une erreur de syntaxe (-104) ou le type est interprété différemment | Vérifier la version IBM i avant de générer du `DECFLOAT` (disponible depuis IBM i 6.1) ou des colonnes `GENERATED ALWAYS AS IDENTITY` / `ROW CHANGE TIMESTAMP` (disponibles depuis IBM i 7.1). Lancer `SELECT OS_NAME, OS_VERSION, OS_RELEASE FROM QSYS2.SYSTEM_STATUS_INFO` pour confirmer la version avant le Prompt 1. |
| Trigger récursif | Un trigger sur UPDATE qui fait un UPDATE sur la même table se déclenche à l'infini → IBM i lève `-723` (récursivité maximale atteinte) | Vérifier au Prompt 0 que le corps du trigger n'écrit pas dans la table déclencheuse sans condition d'arrêt ; demander à Bob de le signaler explicitement |
| Oublier le test RUNSQLSTM (Prompt 1-bis) | Un script DDL sauvegardé mais non testé peut contenir des erreurs silencieuses — découvertes en production lors d'un chargement de données | Le Prompt 1-bis est obligatoire après chaque prompt de génération avant toute sauvegarde du script final |
| Procédure sans DECLARE HANDLER | Une erreur SQL non gérée peut laisser un curseur ouvert (prochain appel `-502`) ou poser des verrous non relâchés | Inclure `DECLARE CONTINUE HANDLER FOR NOT FOUND` (fin de curseur) et `DECLARE EXIT HANDLER FOR SQLEXCEPTION` avec `GET DIAGNOSTICS` (erreurs non récupérables) — ne pas utiliser `CONTINUE HANDLER FOR SQLEXCEPTION` comme handler générique sur une procédure de mise à jour |
| Travailler en mode Agent pendant la génération | Bob peut exécuter un `CREATE TABLE` incorrect sur l'IBM i de test ou sauvegarder un script intermédiaire non validé | Rester en mode **Ask** pendant les Prompts 0 à 5 ; mode Agent uniquement pour Prompt 1-bis et sauvegarde finale |

---

## Check-list de validation UC 11

Avant de passer à UC 10 (voir `UC10-generation-app.md`), valider chaque point :

- [ ] **Pour chaque objet SQL généré : le Prompt 0 a été exécuté** — le type (table / vue / index / procédure / trigger / fonction), la catégorie et les dépendances sont documentés dans le fichier `*-objet-sql-*.md`
- [ ] Les dépendances entre objets ont été vérifiées via `QSYS2.TABLES`, `QSYS2.SYSROUTINES` et `QSYS2.SYSTRIGGERS` avant chaque génération — aucun objet dépendant d'une table, d'une procédure ou d'un autre objet inexistant
- [ ] Chaque script DDL a été **testé sur l'IBM i de test via le Prompt 1-bis** — sans erreur SQL
- [ ] Chaque objet créé a été validé via le catalogue approprié : `QSYS2.SYSCOLUMNS` (tables et vues), `QSYS2.SYSINDEXES` (index), `QSYS2.SYSROUTINES` (procédures et fonctions), `QSYS2.SYSTRIGGERS` (triggers) — la structure créée correspond au script généré
- [ ] **Pour chaque table : les instructions `LABEL ON COLUMN` ont été générées et exécutées** — vérifier via `QSYS2.SYSCOLUMNS.COLUMN_TEXT` que les descriptifs sont présents
- [ ] **Pour chaque procédure stockée, fonction et trigger : un appel ou déclenchement nominal a été exécuté** sur l'IBM i de test avec des données représentatives — le résultat ou l'effet de bord a été vérifié. Pour les procédures : le paramètre OUT SQLSTATE retourné est `'00000'` sur le cas nominal. Pour les triggers : l'effet de bord (insertion d'audit, modification de colonne) est visible dans la table cible. La compilation seule ne suffit pas à déclarer un objet validé.
- [ ] **Les `GRANT` ont été exécutés** pour chaque objet créé et validé — `SELECT/INSERT/UPDATE/DELETE` sur les tables selon le besoin ; `SELECT` par défaut sur les vues, avec des droits DML uniquement si la vue est modifiable et si le besoin l'exige explicitement ; `EXECUTE` sur les procédures et fonctions — pour le profil applicatif ACME (vérifier via `QSYS2.SYSROUTINEAUTH` et `QSYS2.SYSTABAUTH`)
- [ ] Le plan de génération périmètre SQL (Prompt 5) est produit et validé avec le DBA ACME — input direct de UC 13 pour les tests des objets SQL générés
- [ ] Les scripts DDL sont sauvegardés avec la convention de nommage correcte (`*-objet-sql-*`, `*-procedure-sql-*`, `*-trigger-sql-*`, `*-plan-sql-*`) et publiés sur Confluence (si MCP disponible)

---

## Points à compléter avant passage en production

> Ces points ne bloquent pas le POC — ils concernent des cas avancés à traiter avant d'industrialiser la génération d'objets SQL sur l'ensemble du parc en production.

### Droits SQL après création (GRANT)

**Contexte :** sur IBM i, un objet créé par un administrateur n'est pas automatiquement accessible aux profils applicatifs. Le plan de génération (Prompt 5) liste les droits à accorder — mais ces `GRANT` doivent être exécutés manuellement après chaque création sur l'IBM i de test, et inclus dans les scripts de déploiement.

```sql
-- Exemple de GRANT après création
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE [NOM_LIB].[NOM_TABLE] TO [PROFIL_APPLICATIF] ;
GRANT EXECUTE ON SPECIFIC PROCEDURE [NOM_LIB].[NOM_PROCEDURE] TO [PROFIL_APPLICATIF] ;
GRANT EXECUTE ON SPECIFIC FUNCTION  [NOM_LIB].[NOM_FONCTION]   TO [PROFIL_APPLICATIF] ;
```

Avant production, vérifier avec l'équipe sécurité ACME que les profils applicatifs ont les droits minimum requis — ni trop (risque de modification non contrôlée) ni trop peu (erreurs `-551` en production).

### Stratégie de versioning des scripts DDL

**Contexte :** le MCP ARCAD n'est pas actif dans ce POC. Les scripts DDL produits par UC 11 ne sont pas automatiquement versionnés dans ARCAD.

**À faire avant production :** définir avec l'équipe ACME une stratégie de migration SQL :
- Intégration manuelle dans ARCAD après validation, avec un membre source dédié (ex. `QSQLSRC`)
- Ou adoption d'un gestionnaire de migrations SQL (Liquibase, Flyway) pour séquencer et versionner les scripts DDL indépendamment d'ARCAD

Chaque script généré par UC 11 contient déjà le placeholder `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` — s'assurer que cette étape est tracée dans JIRA avant de marquer un objet comme "Validé".

### Triggers — validation des cas limites avec le métier

**Contexte :** un trigger AFTER UPDATE déclenché par un `UPDATE` de masse (ex. recalcul de prix sur toute une table de 500 000 lignes) se déclenchera 500 000 fois. Si la logique du trigger est lourde (INSERT dans une table de log, appel de procédure), l'impact performance peut bloquer l'opération.

Avant production, valider avec le métier ACME les cas limites de déclenchement :
- Un UPDATE en masse est-il possible sur la table déclenchante ?
- Le trigger doit-il s'exécuter même lors d'imports batch ou uniquement lors de modifications unitaires ?
- Si le déclenchement en masse est inacceptable, envisager un trigger conditionnel (WHEN clause) ou une désactivation temporaire du trigger lors des imports.

---

*Fiche UC 11 — Document évolutif à mettre à jour au fil du POC.*
