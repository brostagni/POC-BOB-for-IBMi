# UC 3 — Accès natif base de données → SQL embarqué

> **Catégorie :** Modernisation
>
> **Priorité dans le POC :** 7 — suit UC 14 (tables DDL cibles connues), peut être mené en parallèle si l'équipe est suffisante
>
> **Durée POC (avec Bob) :** 3 à 6 heures — conversion d'un périmètre représentatif (3 à 5 programmes avec accès natifs), itérations et tests de non-régression
>
> **Durée PROD (avec Bob) :** 1 à 3 heures / programme — analyse des accès natifs, génération SQL embarqué, revue développeur, test fonctionnel
>
> **Durée PROD (sans Bob) :** 2 à 5 jours / programme — identification manuelle de chaque opcode d'accès, rédaction des requêtes SQL équivalentes, gestion des curseurs, tests
>
> **Gain Bob estimé :** ~6× — un programme de 500 lignes avec 20 accès natifs converti en une demi-journée au lieu de 2 à 3 jours ; gain plus fort sur les programmes avec logique de navigation complexe (SETLL/READE en boucle)
>
> **Mode Bob recommandé :** IBM i Developer (mode Ask pour l'analyse et la génération, Agent pour la sauvegarde)

---

## Objectif

Remplacer les **accès natifs aux fichiers IBM i** (`CHAIN`, `READ`, `READE`, `READP`, `SETLL`, `SETGT`, `WRITE`, `UPDATE`, `DELETE`…) par des **instructions SQL embarqué** (`EXEC SQL SELECT`, `INSERT`, `UPDATE`, `DELETE`, curseurs) dans les programmes RPG.

**Ce UC modernise l'accès aux données, pas la structure.** UC 14 a converti les définitions DDS en tables DDL — UC 3 convertit la façon dont les programmes accèdent à ces tables. Les deux sont complémentaires : un programme converti en SQL embarqué qui accède encore à un PF DDS fonctionne, mais un programme qui accède en SQL à une table mal typée produit des résultats incorrects. UC 14 d'abord, UC 3 ensuite.

**Ce UC diffère de UC 7 (Optimisation) et UC 8 (Restructuration) :** UC 3 ne change pas la logique fonctionnelle du programme — il change uniquement le mécanisme d'accès aux données. Les règles métier restent les mêmes, les résultats doivent être identiques avant et après conversion. C'est le critère de succès principal.

**Livrable attendu :** Pour chaque programme converti : le source RPG modifié avec les accès natifs remplacés par du SQL embarqué, les déclarations de curseurs, la suppression des F-specs des fichiers convertis, et un fichier de traçabilité documentant chaque remplacement effectué.

**Convention de nommage des fichiers générés :**
```
{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md

Types pour cet UC :
  analyse-acces   → fiche de qualification (Prompt 0) + inventaire des accès natifs (Prompt 1)
  sql-embarque    → source RPG converti ou diff des modifications (Prompts 2, 3, 4)
  plan-conversion → plan de conversion périmètre applicatif complet (Prompt 5)
```

Exemples :
```
acme-APPVTE-GESCMD-analyse-acces-20250618-0900.md    ← inventaire des CHAIN/READ/WRITE
acme-APPVTE-GESCMD-sql-embarque-20250618-1100.md     ← source converti ou diff
acme-APPVTE-APPVTE-plan-conversion-20250618-0800.md  ← plan pour le périmètre complet
```

> 💡 Cette convention est valable en dehors du contexte POC — réutilisable en production tel quel.

---

## Démarrer par un programme que vous connaissez

> **Recommandation forte avant d'aborder les programmes critiques de ACME.**

Commencer par un **programme simple avec peu d'accès natifs** — idéalement un programme dont un développeur de l'équipe connaît la logique et peut valider que le comportement après conversion est identique.

Pourquoi ? Parce que la première conversion sert à **calibrer deux choses** :
- La qualité de la traduction opcode → SQL par Bob (est-ce que `CHAIN` sur clé composite devient bien un `SELECT ... WHERE champ1 = :var1 AND champ2 = :var2` ?)
- La capacité de l'équipe à tester la non-régression (quels jeux de données, quels cas limites valider ?)

Si la première conversion est validée fonctionnellement, la confiance est établie pour les programmes plus complexes.

**Progression recommandée :**

> 💡 Pour chaque programme de cette progression, **commencer par le Prompt 0** — il donne la catégorie (SIMPLE / STANDARD / COMPLEXE) et la séquence exacte à suivre. Ne pas aller directement au Prompt 1.

| Étape | Programme à choisir | Catégorie attendue | Objectif |
|-------|--------------------|--------------------|---------|
| 1 | Programme avec CHAIN et WRITE simples (< 10 accès natifs) | SIMPLE | Calibrer CHAIN → SELECT, WRITE → INSERT ; valider que le Prompt 0 classe correctement |
| 2 | Programme avec boucle READ/READE | STANDARD | Valider la conversion en curseur SQL |
| 3 | Programme avec SETLL/READE sur LF | STANDARD à COMPLEXE | Valider que l'index DDL remplace correctement le LF |
| 4 | Programme critique avec accès mixtes (lecture, écriture, mise à jour dans une même transaction) | COMPLEXE | Valider la gestion des transactions SQL et le comportement en erreur |

---

## Impact de la taille du programme sur la stratégie de conversion

La complexité d'UC 3 ne dépend pas du nombre de lignes total du programme, mais du **nombre et de la diversité des opcodes d'accès** et de la **logique de navigation** (boucles imbriquées, lectures conditionnelles, accès multi-fichiers dans une même subroutine).

Les vrais facteurs qui compliquent la conversion :
- Les **boucles SETLL/READ/READE** : navigation séquentielle positionnée — l'équivalent SQL est un curseur avec `ORDER BY` et une clause `WHERE` de départ, parfois délicat à reproduire exactement
- Les **accès multi-fichiers corrélés** : un `CHAIN` sur un fichier dont la clé vient du résultat d'un `READ` sur un autre fichier — devient un `JOIN` ou des sous-requêtes corrélées
- Les **verrous explicites** (`LOCK` en RPG natif) : n'ont pas d'équivalent direct en SQL embarqué — nécessitent une décision de conception (transaction SQL, `SELECT ... FOR UPDATE`)
- Les **indicateurs de statut d'accès** (`%FOUND`, `%EOF`, `%EQUAL`) : doivent être remplacés par des vérifications du `SQLCODE` ou `SQLSTATE`
- Les **sous-fichiers** (SUBFILE) alimentés par des lectures natives : la conversion de la boucle de chargement est souvent la partie la plus délicate

### Programmes SIMPLE — < 10 accès natifs, pas de boucle complexe

Bob gère sans difficulté. **Séquence : Prompt 0 → Prompt 1 → Prompt 2 → Prompt 2-bis (test compilation) → Prompt 4 (nettoyage).**

> 💡 Le Prompt 0 confirme la catégorie SIMPLE et recommande directement cette séquence — pas de décision manuelle requise.

### Programmes STANDARD — 10 à 30 accès natifs, ou boucles simples sans imbrication

Le Prompt 0 identifie les groupes fonctionnels (par fichier ou par subroutine). **Séquence : Prompt 0 → Prompt 1 → Prompt 2 répété par groupe → Prompt 2-bis après chaque groupe → Prompt 4 (nettoyage).**

> 💡 La cartographie des subroutines produite en UC 4 est le guide de navigation — le Prompt 0 s'en sert pour définir les groupes. Convertir et tester groupe par groupe, pas tout le programme en une seule passe.

### Programmes COMPLEXE — > 30 accès natifs, ou boucles imbriquées, ou accès corrélés, ou verrous

Un programme COMPLEXE est souvent un programme qui fait trop de choses. Le Prompt 0 pose explicitement la question : **faut-il restructurer avant de convertir ?** (UC 8 avant UC 3 sur ce programme spécifique). Si le Prompt 0 conclut "restructuration recommandée", ne pas démarrer la conversion — ouvrir une issue UC 8 d'abord.

Si la décision est de convertir sans restructurer : **Séquence : Prompt 0 → Prompt 1 → Prompt 3 (curseurs) pour les boucles → Prompt 2 pour les accès simples → Prompt 2-bis → Prompt 4.**

> ⚠️ Sur un programme COMPLEXE, une conversion globale en une seule passe produit un résultat incomplet ou avec des erreurs subtiles (SQLCODE non testé, curseur non fermé, ordre des opérations modifié). Le Prompt 0 découpe toujours le travail en groupes.

---

## Démarrer une session Bob

> **À lire avant chaque session UC 3 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

Chaque session de travail sur un programme RPG doit démarrer dans une **nouvelle conversation Bob** (bouton `+` en haut du panneau Chat). Ne pas réutiliser une conversation UC 14 ou d'un programme précédent — le contexte de conversion d'un autre programme bruite les réponses sur le programme courant.

**Mode à sélectionner :** `IBM i Developer`

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Avant de lancer le Prompt 0, ouvrir dans l'éditeur Bob le programme RPG à convertir. L'ouverture dans l'éditeur le rend accessible au MCP IBM i sans copier-coller.

**Procédure :** dans le panneau **IBM i — Object Browser** (extension Code for IBM i), naviguer jusqu'à la bibliothèque source, faire un clic droit sur le membre RPG → **Open in Editor**.

Fichiers à ouvrir pour chaque session UC 3 :
- Le programme RPG source (`[NOM_LIB]/QRPGSRC([NOM_PROGRAMME])`)
- Le fichier DDL complet de la bibliothèque (`*-ddl-complet-*.md`) pour les noms de tables cibles
- Le fichier de compréhension du programme (`*-comprehension-*.md`) depuis UC 4

### 3. Fichiers de contexte à charger

Ces fichiers produits par les UC précédents doivent être disponibles dans le workspace Bob **avant** de démarrer. Utiliser **Add File to Chat** (icône trombone) ou les ouvrir dans l'éditeur.

| Fichier | Produit par | Obligatoire / Recommandé |
|---------|-------------|--------------------------|
| `{appArcad}-{fonction}-{fonction}-ddl-complet-{date}.md` | UC 14 | **Obligatoire** — noms des tables SQL cibles pour les requêtes générées |
| `{appArcad}-{fonction}-{composant}-comprehension-{date}.md` | UC 4 | **Obligatoire** — liste des fichiers accédés et leur nature PF/LF |
| `{appArcad}-{fonction}-{composant}-spec-tech-{date}.md` | UC 6 | **Recommandé** — interfaces du programme (paramètres, fichiers) |
| `{appArcad}-{fonction}-{fonction}-matrice-{date}.md` | UC 6 | **Recommandé** — programmes qui partagent les mêmes fichiers (cohérence des noms DDL) |
| `{appArcad}-{fonction}-{composant}-analyse-acces-{date}.md` | UC 3 (session précédente) | **Si reprise** — inventaire des accès déjà produit, évite de relancer le Prompt 1 |

> 💡 **Prérequis bloquant :** si `*-ddl-complet-*.md` est absent, le Prompt 0 le détecte et bloque la session — compléter UC 14 sur le périmètre avant de continuer.

> ⚠️ **Risque de réduction de contexte — sauvegarde intermédiaire recommandée :** une session UC 3 avec plusieurs prompts consécutifs peut atteindre la limite de contexte sur les programmes STANDARD ou COMPLEXE. Si Bob semble oublier une décision prise au Prompt 0 (catégorie, groupes fonctionnels identifiés, périmètre retenu), c'est un signal de compression de contexte. Sauvegarder le livrable en cours en mode Agent après chaque prompt majeur — pas seulement en fin de session. À chaque reprise de passe, commencer le prompt par : "Le fichier [NOM_FICHIER] contient les décisions prises — continuer à partir de [ÉTAPE]."

> 💡 **Reprise de session :** si la conversion d'un programme est interrompue, ouvrir le fichier `*-analyse-acces-*.md` déjà produit — Bob retrouve l'inventaire des opcodes sans relire le source depuis zéro. Indiquer dans le prompt "l'inventaire est disponible dans [NOM_FICHIER] — reprendre à partir du groupe [N°]".

---

## Prérequis

- UC 14 complété sur le périmètre à convertir : les tables SQL DDL (`CREATE TABLE`, `CREATE INDEX`) existent sur l'IBM i de test — les noms de tables cibles sont connus
- Les fichiers `*-comprehension-*.md` (UC 4) des programmes à convertir sont présents : ils fournissent la liste des fichiers accédés et leur nature (PF/LF)
- Les fichiers `*-spec-tech-*.md` (UC 6) sont disponibles : ils documentent les interfaces des programmes et les paramètres échangés
- IBM i MCP actif (lecture des sources RPG)
- IBM i Database MCP actif (validation des requêtes SQL générées via `QSYS2`, exécution de tests)
- Accès à l'IBM i de test pour compiler et tester les programmes convertis
- Un développeur RPG disponible pour valider la non-régression fonctionnelle

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | IBM i Developer — mode **Ask** pour l'analyse et la génération SQL, **Agent** pour la sauvegarde |
| **Scope** | Library List → bibliothèque applicative ACME |
| **MCP actifs** | IBM i MCP (lecture sources RPG) + IBM i Database MCP (validation SQL, QSYS2) |
| **MCP différés** | Confluence MCP (publication des sources convertis, si token disponible) |

### Pourquoi IBM i Developer — Ask pour la génération SQL embarqué ?

La conversion d'accès natifs en SQL embarqué est un travail **itératif** : la première version générée par Bob est rarement parfaite — des cas limites, des indicateurs de statut manquants, ou des curseurs mal fermés nécessitent des corrections dans le chat avant de sauvegarder. Le mode Ask permet ces itérations sans risque d'écriture intermédiaire.

| Phase | Mode | Ce que Bob fait |
|-------|------|----------------|
| Qualification du programme (Prompt 0) | **Ask** | Compte les opcodes, détecte les facteurs de complexité, recommande la stratégie |
| Inventaire des accès natifs (Prompt 1) | **Ask** | Lit le source RPG via IBM i MCP, produit le tableau détaillé des opcodes |
| Génération SQL embarqué (Prompts 2, 3) | **Ask** | Génère le source converti dans le chat — itérations possibles |
| Test de compilation (Prompt 2-bis) | **Agent** | Lance `CRTBNDRPG` via IBM i MCP, rapporte les erreurs — compilation uniquement |
| Test fonctionnel | **Humain** | Exécution sur IBM i de test, comparaison des résultats — non délégable à Bob |
| Sauvegarde du source converti | **Agent** | Écrit le fichier `.md` (diff ou source complet) dans le workspace |

> 💡 **Règle d'or pour UC 3 :** Ne passer en mode **Agent** pour la génération SQL qu'une fois la conversion relue et validée par le développeur. Un source RPG sauvegardé avec un `SQLCODE` non testé peut compiler sans erreur mais produire un comportement silencieusement incorrect en production.

> ⚠️ Le mode Agent est autorisé **uniquement** pour deux opérations précises : le test de compilation via le Prompt 2-bis, et la sauvegarde du source validé. Pendant toute la phase de génération et d'itération SQL (Prompts 0, 1, 2, 3), rester en mode Ask.

### Spécificité ARCAD — MCP non disponible

Le MCP ARCAD n'est **pas actif** dans ce POC (incompatibilité de version).

**Impact sur UC 3 : faible à moyen.** Les sources RPG modifiés par Bob devront être réintégrés dans ARCAD manuellement après validation.

| Ce que l'absence du MCP ARCAD change | Ce qui fonctionne quand même |
|--------------------------------------|------------------------------|
| Impossible de lire l'historique des versions du source RPG dans ARCAD | IBM i MCP lit la version courante du source dans les bibliothèques ARCAD normalement |
| Les sources convertis ne sont pas automatiquement versionnés dans ARCAD | L'analyse et la génération SQL embarqué sont intégralement fonctionnelles |
| Réintégration manuelle dans ARCAD après chaque session de conversion | Ajouter le placeholder `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` dans l'en-tête de chaque fichier de source converti |

> 💡 **Contournement :** définir dès le début de la session un workflow de réintégration ARCAD — par exemple, nommer les membres sources convertis avec un suffixe `_SQL` dans une bibliothèque de travail, puis les promouvoir dans ARCAD depuis cette bibliothèque après validation.

---

## Prompts clés

### Prompt 0 — Qualification et choix de stratégie

> **Ce prompt est le point d'entrée obligatoire de UC 3 pour chaque programme.**
> Il remplace la décision manuelle sur la stratégie (Conversion directe / Par groupes / Analyse architecturale).
> Il se lance **avant** le Prompt 1 — son résultat conditionne toute la séquence suivante.

```
Le programme [NOM_PROGRAMME] se trouve dans [NOM_LIB]/QRPGSRC.

Analyse ce programme et produis en français, en markdown, une fiche de qualification
pour la conversion en SQL embarqué :

## Qualification UC 3 — [NOM_PROGRAMME]

### 1. Comptage des accès natifs
Compte le nombre total d'occurrences de chaque opcode d'accès aux fichiers :
| Opcode | Nb occurrences |
| CHAIN  | ?              |
| READ   | ?              |
| READE  | ?              |
| READP / READPE | ?      |
| SETLL  | ?              |
| SETGT  | ?              |
| WRITE  | ?              |
| UPDATE | ?              |
| DELETE | ?              |
| EXCEPT (vers fichier) | ? |
| **TOTAL** | **?**      |

### 2. Détection des facteurs de complexité
Réponds par OUI / NON / À CONFIRMER pour chaque facteur :
- [ ] Boucles de navigation (SETLL + DOW + READ/READE)
- [ ] Boucles imbriquées (une boucle READ dans une boucle READ)
- [ ] Accès multi-fichiers corrélés (clé d'un READ utilisée pour un CHAIN)
- [ ] Verrous explicites (opcode LOCK ou F-spec avec OFLIND/locking)
- [ ] Sous-fichiers (SUBFILE) alimentés par des boucles de lecture
- [ ] Accès à des fichiers logiques (LF) — vérifier que le LF a un index DDL dans UC 14

### 3. Fichiers DDL cibles — vérification des prérequis
Pour chaque fichier déclaré en F-spec, vérifier son existence dans
{appArcad}-{fonction}-{fonction}-ddl-complet-{YYYYMMDD-HHmm}.md (UC 14) :
| Fichier DDS natif | Table DDL cible | DDL disponible ? |
| [NOM_FICHIER]     | [NOM_TABLE_SQL] | OUI / NON / ?    |

Si un DDL est manquant : signaler le blocage — ce fichier doit être traité
dans UC 14 avant de continuer.

### 4. Catégorie et stratégie recommandée
Sur la base du comptage et des facteurs de complexité, conclure :

**Catégorie :**
- [ ] SIMPLE — < 10 accès, aucun facteur de complexité
- [ ] STANDARD — 10 à 30 accès, ou boucles simples sans imbrication
- [ ] COMPLEXE — > 30 accès, ou boucles imbriquées, ou accès corrélés, ou verrous

> Note : UC 3 utilise SIMPLE / STANDARD / COMPLEXE (critère : nombre d'opcodes et type de boucles).
> UC 14 utilise les mêmes catégories (critère : nombre de champs et nature des LF).
> Les deux échelles sont indépendantes — un fichier DDS SIMPLE peut être accédé par un programme COMPLEXE.

**Recommandation :**
- SIMPLE   → Séquence directe : Prompt 1 → Prompt 2 → Prompt 2-bis → Prompt 4
- STANDARD → Conversion par groupes fonctionnels : identifier les groupes ci-dessous,
              appliquer Prompt 1 → Prompt 2 groupe par groupe → Prompt 2-bis → Prompt 4
- COMPLEXE → Analyse architecturale préalable recommandée :
              Évaluer UC 8 (restructuration) avant UC 3 sur ce programme.
              Si décision de convertir sans restructurer :
              Prompt 1 → Prompt 3 (curseurs) + Prompt 2 (accès simples) → Prompt 2-bis → Prompt 4

**Si catégorie STANDARD ou COMPLEXE — groupes fonctionnels identifiés :**
| Groupe | Subroutine(s) | Fichier(s) concerné(s) | Nb accès | Prompts à utiliser |

Signale clairement ce que tu ne peux pas déterminer sans exécuter le programme.
```

**Analyse ligne à ligne :**

- `## Qualification UC 3 — [NOM_PROGRAMME]` → titre normalisé dans la sortie. La fiche de qualification peut être sauvegardée directement — elle devient la première page du fichier `*-analyse-acces-*.md`.

- `Comptage des opcodes` → tableau exhaustif avec TOTAL. Le total est le critère de décision principal — Bob le calcule depuis le source, sans approximation. C'est ce qui remplace le jugement "à vue" du développeur.

- `Facteurs de complexité OUI / NON / À CONFIRMER` → trois états. "À CONFIRMER" est délibéré — certains facteurs (comme un verrou implicite via un attribut de F-spec) nécessitent une lecture attentive que Bob peut rater. Le statut "À CONFIRMER" force une vérification humaine sur ces points précis.

- `Fichiers DDL cibles — vérification des prérequis` → vérification du prérequis UC 14 intégrée dans le Prompt 0. Si un DDL est manquant, le programme est bloqué et la séquence s'arrête — pas de conversion incohérente.

- `Catégorie SIMPLE / STANDARD / COMPLEXE` → les trois seuils de la section "Impact de la taille" traduits en décision actionnable. Bob coche lui-même la case — l'équipe n'a plus à lire les critères et trancher manuellement. Ces noms sont identiques dans UC 14 (critère : champs DDS) et UC 3 (critère : opcodes d'accès) — vocabulaire unifié pour les deux UC de la Phase 2.

- `Recommandation avec la séquence exacte de prompts` → la sortie du Prompt 0 est un plan d'action précis, pas une recommandation vague. L'équipe sait exactement quels prompts enchaîner et dans quel ordre.

- `Groupes fonctionnels identifiés` → pour les catégories STANDARD et COMPLEXE, Bob découpe lui-même le travail en groupes. Ces groupes deviennent les paramètres du Prompt 2 ("convertis les accès du groupe [NOM_GROUPE]").

> 💡 **Sauvegarder la fiche de qualification** (mode Agent) dès la fin du Prompt 0 :
> ```
> "Sauvegarde cette fiche de qualification dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-analyse-acces-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-analyse-acces-20250618-0900.md"
> ```
> Le Prompt 1 (inventaire détaillé) complète ensuite ce même fichier.

> ⚠️ **Piège évité :** sans le Prompt 0, l'équipe démarre la conversion sans savoir si le programme nécessite une restructuration préalable. Découvrir en milieu de conversion qu'un programme a 45 accès natifs avec des boucles imbriquées oblige à tout recommencer depuis une position plus difficile.

---

### Prompt 1 — Inventaire des accès natifs d'un programme

```
Le programme [NOM_PROGRAMME] se trouve dans [NOM_LIB]/QRPGSRC.

Analyse ce programme et produis en français, en markdown, l'inventaire complet
de tous ses accès natifs aux fichiers IBM i :

1. Liste des fichiers déclarés en F-specs (ou DCL-F en free) :
   | Nom fichier | Type (PF/LF) | Mode (INPUT/OUTPUT/UPDATE/DELETE) | Accès (KEYED/SEQ) | Fichier clé ? |

2. Inventaire de tous les opcodes d'accès natifs, par occurrence dans le source :
   | N° | Ligne | Opcode | Fichier cible | Clé / Condition | Subroutine / Procédure | Équivalent SQL pressenti |
   Opcodes à détecter : CHAIN, READ, READE, READP, READPE, SETLL, SETGT,
   WRITE, UPDATE, DELETE, EXCEPT (vers fichier), OPEN, CLOSE

3. Boucles de navigation identifiées :
   Pour chaque boucle DOW/DOU/FOR avec READ/READE à l'intérieur :
   - Type de boucle (séquentielle / par clé / EOF)
   - Fichier parcouru
   - Condition de sortie
   - Stratégie SQL pressentie : curseur simple / curseur avec filtre WHERE / fetch unique

4. Accès multiples corrélés :
   Les cas où la valeur lue sur un fichier sert de clé pour accéder à un autre fichier
   (candidats à une jointure SQL)

5. Indicateurs de statut utilisés après les opcodes d'accès :
   %FOUND, %EOF, %EQUAL, *IN et les indicateurs de résultat — ils devront être
   remplacés par des tests SQLCODE / SQLSTATE

Ne pas générer le SQL dans ce prompt.
Signale clairement les accès dont la conversion sera complexe ou nécessitera une décision.
```

**Analyse ligne à ligne :**

- `Opcodes à détecter : CHAIN, READ, READE, READP, READPE, SETLL, SETGT, WRITE, UPDATE, DELETE, EXCEPT, OPEN, CLOSE` → liste exhaustive explicite. Sans cette liste, Bob peut oublier certains opcodes moins courants (`READPE`, `SETGT`, `EXCEPT` vers fichier) qui ont des équivalents SQL non triviaux.

- `| N° | Ligne | Opcode | Fichier cible | Clé / Condition | Subroutine / Procédure | Équivalent SQL pressenti |` → tableau structuré avec la colonne "Équivalent SQL pressenti". Cette colonne force Bob à réfléchir à la conversion dès l'inventaire — ce qui révèle les cas complexes avant de les attaquer.

- `Boucles de navigation identifiées` → section dédiée aux boucles. Sur IBM i, la navigation séquentielle (`SETLL` + boucle `READ/READE`) est la construction la plus fréquente et la plus délicate à convertir en SQL. L'identifier séparément permet de lui appliquer le Prompt 3 dédié.

- `Accès multiples corrélés` → les candidats à la jointure SQL. Deux `CHAIN` imbriqués (un pour lire le client, un pour lire la commande) peuvent souvent être remplacés par un seul `SELECT ... JOIN` — plus performant et plus lisible.

- `Indicateurs de statut` → point souvent oublié lors des conversions manuelles. Un `%FOUND` après `CHAIN` doit devenir `IF SQLCODE = 0`. Un `%EOF` après `READ` doit devenir `IF SQLCODE = 100`. Si on oublie ces conversions, le programme compile mais se comporte différemment.

- `Ne pas générer le SQL dans ce prompt` → séparation inventaire / conversion. L'inventaire sert de feuille de route — l'équipe peut le relire, le corriger, et identifier les cas qui nécessitent une décision avant de lancer la conversion.

> ⚠️ **Piège évité :** sans inventaire préalable, Bob génère du SQL embarqué en une seule passe et peut manquer des opcodes isolés dans des subroutines secondaires — l'accès natif manqué reste dans le source converti et provoque une erreur à la compilation.

---

### Prompt 2 — Conversion des accès simples (CHAIN, WRITE, UPDATE, DELETE)

```
Sur la base de l'inventaire des accès natifs de [NOM_PROGRAMME] que nous venons de faire,
convertis les accès suivants en SQL embarqué pour Db2 for i, en RPG Free :

[Lister ici les N° d'accès du tableau Prompt 1 à convertir dans cette passe]

Pour chaque accès converti, produis :

1. Le code RPG natif original (commenté, conservé pour référence) :
   // NATIF : [OPCODE] [FICHIER] [CLE]

2. Les déclarations SQL à ajouter (variables hôtes, si nouvelles) :
   DCL-S [NOM_VAR] [TYPE] ;

3. Le remplacement SQL embarqué :
   EXEC SQL [INSTRUCTION_SQL] INTO / FROM [VARIABLES_HOTES] ;
   IF SQLCODE <> 0 ;  // ou SQLCODE = 100 pour NOT FOUND
     [GESTION EQUIVALENTE A L'INDICATEUR D'ORIGINE]
   ENDIF ;

Règles de conversion à respecter :
- CHAIN [CLE] [FICHIER] → EXEC SQL SELECT [CHAMPS] INTO [VARS] FROM [TABLE] WHERE [CLE]
  Après : IF SQLCODE = 0 → équivalent %FOUND *ON ; IF SQLCODE = 100 → %FOUND *OFF
- WRITE [FICHIER] → EXEC SQL INSERT INTO [TABLE] ([CHAMPS]) VALUES ([VARS])
  Après : IF SQLCODE <> 0 → gérer l'erreur (duplication de clé : SQLCODE = -803)
- UPDATE [FICHIER] → EXEC SQL UPDATE [TABLE] SET [CHAMPS = VARS] WHERE [CLE]
  Après : IF SQLCODE <> 0 → gérer l'erreur
- DELETE [FICHIER] → EXEC SQL DELETE FROM [TABLE] WHERE [CLE]
  Après : IF SQLCODE <> 0 → gérer l'erreur

Utiliser les noms de tables DDL issus du fichier
{appArcad}-{fonction}-{fonction}-ddl-complet-{YYYYMMDD-HHmm}.md (UC 14).

Ne pas supprimer les F-specs dans ce prompt — elles seront retirées
uniquement une fois tous les accès au fichier convertis et validés.
Ne pas inventer de champs ou de colonnes non visibles dans le source original.
Signaler les cas où la conversion n'est pas directe.
```

**Analyse ligne à ligne :**

- `[Lister ici les N° d'accès du tableau Prompt 1 à convertir dans cette passe]` → référence explicite à l'inventaire. Travailler sur un sous-ensemble numéroté évite que Bob convertisse des accès hors périmètre de la passe courante.

- `// NATIF : [OPCODE] [FICHIER] [CLE]` → conserver le code natif en commentaire. Pendant la phase de validation, un développeur peut comparer l'original et la version SQL côte à côte dans le même source. Ces commentaires sont retirés une fois la validation terminée.

- `IF SQLCODE <> 0` → vérification SQLCODE systématique après chaque instruction SQL. Sur Db2 for i, une instruction SQL qui échoue ne lève pas d'exception si on ne teste pas le SQLCODE — le programme continue silencieusement. C'est la différence fondamentale avec les accès natifs où les indicateurs de résultat sont intégrés dans l'opcode.

- `SQLCODE = 100` pour NOT FOUND → valeur standard SQL pour "aucun enregistrement trouvé". `SQLCODE = -803` pour violation de clé primaire en INSERT. Ces valeurs précises dans le prompt évitent que Bob utilise des constantes non standard.

- `Utiliser les noms de tables DDL issus du fichier *-ddl-complet-*.md (UC 14)` → ancrage sur le livrable de UC 14. Les noms de tables SQL peuvent différer des noms DDS d'origine (en particulier si l'équipe a choisi de renommer lors de la conversion DDL). Référencer le fichier DDL évite les incohérences.

- `Ne pas supprimer les F-specs dans ce prompt` → discipline de conversion progressive. Supprimer la F-spec d'un fichier alors que tous ses accès n'ont pas encore été convertis provoque une erreur de compilation. Les F-specs sont retirées uniquement une fois **tous** les accès à ce fichier convertis et validés.

- `Ne pas inventer de champs ou de colonnes non visibles dans le source original` → garde-fou anti-hallucination. Bob ne doit pas ajouter des colonnes "utiles" dans un SELECT qui n'étaient pas dans le CHAIN original — cela change le contrat de la structure de données chargée.

> 💡 **Sauvegarder ce livrable** (mode Agent) :
> ```
> "Sauvegarde ce diff de conversion SQL embarqué dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-sql-embarque-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-sql-embarque-20250618-1100.md"
> ```

> ⚠️ **Piège évité :** sans le test `IF SQLCODE`, un `UPDATE` qui n'a trouvé aucun enregistrement (SQLCODE = 100) passe silencieusement — en RPG natif, l'indicateur d'erreur aurait été activé et la logique de l'application l'aurait détecté.

---

### Prompt 2-bis — Test de compilation par Bob (après chaque groupe converti)

> **Ce que Bob peut faire :** lancer la compilation via IBM i MCP en mode **Agent** et rapporter les erreurs.
> **Ce que Bob ne peut pas faire :** valider que le résultat fonctionnel est correct — c'est une vérification humaine irréductible.
> **Quand l'utiliser :** après chaque Prompt 2 (groupe d'accès simples) ou Prompt 3 (curseur), avant de passer au groupe suivant.

```
Le source RPG de [NOM_PROGRAMME] dans [NOM_LIB]/QRPGSRC a été modifié
avec les conversions SQL embarqué que nous venons de faire.

Lance la compilation sur l'IBM i de test :
CRTBNDRPG PGM([NOM_LIB_TEST]/[NOM_PROGRAMME])
          SRCFILE([NOM_LIB]/QRPGSRC)
          SRCMBR([NOM_PROGRAMME])
          OPTION(*EVENTF *LIST)
          DBGVIEW(*SOURCE)

Analyse le résultat de la compilation et produis en français :

1. Statut : COMPILATION RÉUSSIE / ERREURS DE COMPILATION
2. Si erreurs : liste des erreurs avec numéro de ligne, code erreur IBM i et description
   | Ligne | Code erreur | Description | Cause probable |
3. Si avertissements SQL (SQLCODE, SQLSTATE non testés) : les lister séparément
4. Recommandation pour corriger les erreurs avant de continuer

Si la compilation réussit sans erreur :
- Confirmer que le groupe [NOM_GROUPE] est compilé et prêt pour le test fonctionnel
- Rappeler que la compilation réussie ne garantit pas la non-régression fonctionnelle —
  le test fonctionnel sur l'IBM i de test reste obligatoire (vérification humaine)
```

**Analyse ligne à ligne :**

- `CRTBNDRPG` avec `OPTION(*EVENTF *LIST)` → les options qui produisent la liste de compilation complète et le fichier d'événements. Sans `*LIST`, les erreurs de compilation ne sont pas retournées en détail. `*EVENTF` active le fichier d'événements utilisé par Code for IBM i pour afficher les erreurs directement dans l'éditeur.

- `DBGVIEW(*SOURCE)` → inclure les informations de débogage source dans l'objet compilé. Indispensable pour utiliser le débogueur IBM i lors des tests fonctionnels — permet de naviguer ligne par ligne dans le source lors d'un test en erreur.

- `| Ligne | Code erreur | Description | Cause probable |` → tableau structuré des erreurs. La colonne "Cause probable" est importante : Bob connaît les codes erreur IBM i courants et peut souvent identifier immédiatement si une erreur vient d'une F-spec restante, d'un champ non déclaré, ou d'un SQLCODE manquant.

- `Si avertissements SQL (SQLCODE, SQLSTATE non testés)` → les avertissements SQL sont distincts des erreurs de compilation. Sur IBM i, un `EXEC SQL` sans déclaration de SQLSTATE génère un avertissement, pas une erreur — le programme compile mais avec un comportement indéfini en cas d'erreur SQL.

- `la compilation réussie ne garantit pas la non-régression fonctionnelle` → rappel obligatoire dans la sortie elle-même. La distinction compilation (automatisable par Bob) / test fonctionnel (validation humaine) doit être visible à chaque utilisation de ce prompt, pas seulement dans la documentation.

> 💡 **Ce prompt s'exécute en mode Agent** — Bob doit pouvoir lancer la commande `CRTBNDRPG` via IBM i MCP. Vérifier que l'utilisateur IBM i associé au MCP a le droit `*USE` sur `CRTBNDRPG` et les droits d'écriture sur la bibliothèque cible de test.

> 💡 **En cas d'erreur de compilation :** ne pas passer au groupe suivant. Corriger les erreurs dans le chat (mode Ask), puis relancer ce prompt. La boucle "corriger → recompiler → valider" peut se faire entièrement dans Bob jusqu'à la compilation propre.

> ⚠️ **Ce prompt ne remplace pas le test fonctionnel.** Une compilation réussie sur un programme converti signifie que le code est syntaxiquement valide — pas qu'il se comporte identiquement à l'original. Les tests suivants restent obligatoires et ne peuvent pas être délégués à Bob :
> - Exécuter le programme sur l'IBM i de test avec un jeu de données réel
> - Comparer les résultats (enregistrements lus, créés, mis à jour) avec les résultats du programme natif d'origine
> - Vérifier les cas limites : enregistrement non trouvé, fin de fichier, violation de clé

---

### Prompt 3 — Conversion des boucles de navigation (SETLL/READ/READE → curseur SQL)

```
Sur la base de l'inventaire de [NOM_PROGRAMME], convertis la boucle de navigation
suivante en curseur SQL pour Db2 for i, en RPG Free :

Boucle identifiée :
[Copier ici le bloc de code natif : SETLL + DOW/DOU + READ/READE + contenu + ENDDO]

Stratégie de conversion attendue :

1. Déclaration du curseur SQL (à placer dans la section de déclaration du programme) :
   EXEC SQL DECLARE [NOM_CURSEUR] CURSOR FOR
     SELECT [CHAMPS_NECESSAIRES]
     FROM [TABLE_DDL]
     WHERE [CONDITION_EQUIVALENTE_AU_SETLL_OU_READE]
     ORDER BY [CLE_DU_LF_OU_PF]
   ;

2. Remplacement de la boucle :
   EXEC SQL OPEN [NOM_CURSEUR] ;
   IF SQLCODE <> 0 ;
     [GESTION ERREUR OUVERTURE]
   ENDIF ;

   EXEC SQL FETCH [NOM_CURSEUR] INTO [VARIABLES_HOTES] ;
   DOW SQLCODE = 0 ;
     [CORPS DE LA BOUCLE — logique inchangée]
     EXEC SQL FETCH [NOM_CURSEUR] INTO [VARIABLES_HOTES] ;
   ENDDO ;

   EXEC SQL CLOSE [NOM_CURSEUR] ;

3. Suppression du SETLL et du READ/READE d'origine (commentés pour référence)

Règles :
- SETLL [CLE] [FICHIER] + READ/READE [FICHIER] → curseur avec ORDER BY + WHERE >= [CLE]
- SETGT [CLE] [FICHIER] + READ [FICHIER] → curseur avec WHERE > [CLE]
- Boucle jusqu'à %EOF → DOW SQLCODE = 0 (SQLCODE = 100 = fin de curseur)
- Boucle jusqu'à changement de clé (READE) → ajouter une condition WHERE sur la clé dans le curseur
- Si la boucle modifie des enregistrements en cours de lecture : utiliser SELECT ... FOR UPDATE et UPDATE ... WHERE CURRENT OF [CURSEUR]

Nommer le curseur de façon explicite : [NOM_FICHIER]_[SUFFIXE_CONTEXTE]_CUR
(ex. : FCOMMANDES_ENCOURS_CUR)

Ne pas inventer de colonnes ou de conditions non présentes dans le source original.
Signaler si la boucle contient des opérations incompatibles avec un curseur simple.
```

**Analyse ligne à ligne :**

- `[Copier ici le bloc de code natif]` → fournir le code exact à convertir dans le prompt. Pour les boucles, travailler sur le code précis évite que Bob généralise et produise un curseur qui ne correspond pas à la logique réelle.

- `EXEC SQL DECLARE [NOM_CURSEUR] CURSOR FOR` → la déclaration du curseur doit être dans la section de déclaration du programme (avant la section de calcul principale en RPG). Bob le sait mais l'instruction explicite évite qu'il place la déclaration dans la boucle elle-même.

- `DOW SQLCODE = 0` → équivalent exact de `DOW NOT %EOF`. SQLCODE = 0 signifie "FETCH réussi" ; SQLCODE = 100 signifie "plus d'enregistrements". Le pattern `FETCH avant la boucle + FETCH en fin de boucle` reproduit fidèlement la sémantique du `READ/READE`.

- `EXEC SQL CLOSE [NOM_CURSEUR]` → fermeture explicite du curseur. En RPG natif, un fichier ouvert en boucle se ferme automatiquement en fin de programme. Un curseur SQL ouvert et non fermé consomme des ressources sur l'IBM i — et peut bloquer d'autres processus sur la même table si le curseur tient un verrou.

- `SELECT ... FOR UPDATE` → pour les boucles qui modifient les enregistrements en cours de lecture. Sans `FOR UPDATE`, un `UPDATE ... WHERE CURRENT OF` échoue. Le cas est fréquent sur IBM i (lire et mettre à jour dans la même boucle).

- `Nommer le curseur de façon explicite` → convention de nommage intégrée dans le prompt. Les curseurs anonymes (`CUR1`, `CUR2`) rendent le source illisible dès qu'il y en a plusieurs. Le pattern `[NOM_FICHIER]_[CONTEXTE]_CUR` est mémorisable et traçable.

> ⚠️ **Piège évité :** une boucle `SETLL/READE` lit uniquement les enregistrements dont la clé **égale** la valeur positionnée. Le curseur doit avoir `WHERE clé_partielle = :var` et non `WHERE clé_partielle >= :var` — une nuance qui change complètement le périmètre des données lues.

> ⚠️ **Piège évité :** oublier `CLOSE [NOM_CURSEUR]` en fin de boucle est une erreur classique. Sur un programme batch qui traite des milliers d'enregistrements, un curseur non fermé peut saturer les ressources de l'IBM i.

> 💡 Si la boucle contient un `EXCEPT` (écriture vers un fichier print ou d'interface), le `EXCEPT` est converti séparément — il ne fait pas partie du curseur de lecture.

---

### Prompt 4 — Nettoyage final : suppression des F-specs et OPEN/CLOSE natifs

```
Tous les accès natifs au fichier [NOM_FICHIER] dans [NOM_PROGRAMME] ont été
convertis en SQL embarqué et validés.

Effectue le nettoyage final pour ce fichier :

1. Supprimer la F-spec (ou DCL-F) du fichier [NOM_FICHIER]
   - Vérifier d'abord qu'aucun accès natif restant ne référence ce fichier
     (sinon, la suppression provoquera une erreur de compilation)
   - Conserver la F-spec en commentaire une session supplémentaire
     avant suppression définitive

2. Supprimer les OPEN et CLOSE natifs explicites pour ce fichier (s'ils existent)

3. Vérifier les déclarations de structures de données liées à ce fichier :
   - DS externées (E DS EXTNAME([NOM_FICHIER])) : à remplacer par des DS SQL
     ou par des variables hôtes individuelles selon la complexité
   - Les champs de ces DS sont-ils encore référencés après la conversion ?
     Si oui : les conserver sous forme de variables indépendantes
     Si non : les supprimer

4. Produire un récapitulatif des modifications :
   | Élément supprimé | Ligne d'origine | Raison |
   | Élément modifié  | Ligne d'origine | Changement effectué |

Ne pas supprimer d'autres éléments que ceux directement liés à [NOM_FICHIER].
Signaler tout élément dont la suppression nécessite une vérification manuelle.
```

**Analyse ligne à ligne :**

- `Vérifier d'abord qu'aucun accès natif restant ne référence ce fichier` → garde-fou de cohérence. Si la conversion a été faite par passes et qu'un opcode a été manqué, la suppression de la F-spec provoquera une erreur de compilation sur cet opcode — plus facile à corriger avant suppression qu'après.

- `Conserver la F-spec en commentaire une session supplémentaire` → filet de sécurité lors de la validation. Si un test révèle un comportement incorrect après suppression, retrouver la F-spec commentée dans le source facilite le diagnostic.

- `DS externées (E DS EXTNAME)` → les structures de données externées sont la forme la plus courante de liaison entre un programme RPG et un fichier DDS. Après conversion SQL, ces DS doivent être adaptées — soit converties en DS SQL (`EXEC SQL DECLARE [NOM] TABLE (...)`) soit remplacées par des variables hôtes individuelles. Bob doit être explicitement invité à les traiter.

- `Produire un récapitulatif des modifications` → le tableau de traçabilité du nettoyage. Utilisé lors de la revue de code pour vérifier que rien n'a été supprimé par erreur.

> 💡 **Sauvegarder le source nettoyé** (mode Agent) :
> ```
> "Sauvegarde le source RPG nettoyé (après suppression des F-specs de [NOM_FICHIER])
>  dans un fichier nommé {appArcad}-{fonction}-{composant}-sql-embarque-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-sql-embarque-20250618-1400.md"
> ```

> ⚠️ **Piège évité :** supprimer la F-spec avant d'avoir converti **tous** les accès au fichier est l'erreur la plus fréquente. Le compilateur IBM i signale l'erreur, mais le programme partiellement converti est dans un état incohérent qui peut être difficile à démêler.

---

### Prompt 5 — Plan de conversion pour un périmètre applicatif complet

```
Sur la base des fichiers de compréhension ({appArcad}-{fonction}-*-comprehension-*.md)
et des spécifications techniques ({appArcad}-{fonction}-*-spec-tech-*.md) pour
l'application [NOM_APPLICATION] dans [NOM_LIB], génère un plan de conversion
SQL embarqué en français, en markdown.

## Plan de conversion SQL embarqué — [NOM_APPLICATION]

### 1. Périmètre des programmes à convertir
Tableau :
| Programme | Nb accès natifs estimés | Fichiers accédés | Catégorie (S/St/C) | Ordre de conversion recommandé |

Catégorie : S  = SIMPLE   (< 10 accès, pas de boucle complexe)
            St = STANDARD (10–30 accès, boucles simples)
            C  = COMPLEXE (> 30 accès, boucles imbriquées, accès corrélés)

### 2. Dépendances entre programmes
Les programmes qui accèdent aux mêmes fichiers partagés (identifiés dans la matrice
*-matrice-*.md) doivent être convertis de manière cohérente :
- Un fichier partagé entre 3 programmes doit avoir la même table DDL cible pour tous
- Les programmes qui se passent des enregistrements via des structures partagées
  doivent être convertis ensemble

### 3. Fichiers DDL cibles
Vérifier que chaque fichier DDS accédé en natif a son équivalent DDL dans
{appArcad}-{fonction}-{fonction}-ddl-complet-{YYYYMMDD-HHmm}.md (UC 14).
Signaler les cas où le DDL est manquant — ces programmes ne peuvent pas être
convertis avant que UC 14 soit complété sur leur périmètre.

### 4. Risques identifiés
Les 3 à 5 points de conversion les plus risqués de ce périmètre :
boucles complexes, accès corrélés, verrous explicites, sous-fichiers alimentés
par des boucles de lecture natives.

### 5. Estimation d'effort
| Programme | Prompts nécessaires | Durée estimée (avec Bob) | Priorité |

Ne pas inventer de programmes ou de fichiers non visibles dans les sources disponibles.
```

**Analyse ligne à ligne :**

- `Catégorie : S / St / C` → trois niveaux alignés sur les catégories SIMPLE / STANDARD / COMPLEXE du Prompt 0. Le critère est le **nombre d'accès natifs** et la **présence de boucles complexes** — deux données que Bob peut extraire depuis les fichiers de compréhension produits en UC 4. Utiliser les abréviations dans le tableau pour des colonnes lisibles.

- `Dépendances entre programmes` → section critique pour les conversions de périmètre. Si deux programmes accèdent au même PF et que l'un est converti en SQL avec un nom de table différent de celui utilisé par l'autre, les deux programmes ne sont plus cohérents. La matrice de références croisées de UC 6 est la source de vérité.

- `Vérifier que chaque fichier DDS accédé a son équivalent DDL dans *-ddl-complet-*.md` → vérification de prérequis intégrée dans le plan. Si UC 14 est incomplet sur une partie du périmètre, le plan doit l'identifier avant de démarrer — pas pendant la conversion.

- `Signaler les cas où le DDL est manquant` → garde-fou anti-blocage. Un programme dont le DDL cible n'existe pas ne peut pas être converti — l'identifier dans le plan permet de débloquer UC 14 sur ce périmètre avant de démarrer UC 3.

> 💡 **Sauvegarder ce plan** (mode Agent) :
> ```
> "Sauvegarde ce plan de conversion dans un fichier nommé
>  {appArcad}-{fonction}-{fonction}-plan-conversion-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-APPVTE-plan-conversion-20250618-0800.md"
> ```

> 💡 Ce plan est le document de pilotage de UC 3. Il permet de suivre l'avancement programme par programme et de prioriser les conversions selon le risque et la dépendance entre programmes.

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Ouverture des membres sources RPG, navigation Object Browser, compilation des sources convertis |
| **IBM i Languages** | Coloration syntaxique RPG Free — indispensable pour lire et valider le code converti dans l'éditeur |
| **Markdown All in One** | Prévisualisation des fichiers de diff et de traçabilité sauvegardés |
| **SQL Notebook** *(si disponible)* | Test interactif des requêtes SQL générées avant intégration dans le source RPG |

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture des membres sources RPG (`QRPGSRC`) via `read_member` ; compilation des sources convertis sur l'IBM i de test |
| **IBM i Database MCP** | Validation des requêtes SQL générées (`EXPLAIN`, `VALUES SQLCODE`, vérification des tables DDL via `QSYS2.SYSTABLES`) ; exécution de requêtes de test |
| **Confluence MCP** *(si disponible)* | Publication du plan de conversion et des sources convertis dans l'espace POC |

> 💡 **Requêtes QSYS2 utiles pour UC 3 :**
> ```sql
> -- Vérifier qu'une table DDL existe avant de démarrer la conversion
> SELECT TABLE_NAME, TABLE_TYPE, TABLE_TEXT
> FROM QSYS2.SYSTABLES
> WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_TABLE]';
>
> -- Lister les colonnes d'une table DDL cible (valider les noms avant génération SQL)
> SELECT COLUMN_NAME, DATA_TYPE, LENGTH, NUMERIC_SCALE, IS_NULLABLE
> FROM QSYS2.SYSCOLUMNS
> WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_TABLE]'
> ORDER BY ORDINAL_POSITION;
>
> -- Vérifier les index disponibles sur une table (pour valider les ORDER BY)
> SELECT INDEX_NAME, COLUMN_NAMES, IS_UNIQUE
> FROM QSYS2.SYSINDEXES
> WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_TABLE]';
> ```
> Ces requêtes permettent de valider les noms de colonnes et d'index avant de les utiliser dans les requêtes SQL embarqué générées — évite les erreurs de compilation dues à des noms inexacts.

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Sauter le Prompt 0 et aller directement au Prompt 1 sur un programme COMPLEXE | La conversion démarre sans plan — les boucles imbriquées et les accès corrélés sont découverts en cours de route, la session doit être interrompue et recommencée | Toujours lancer le Prompt 0 en premier — il prend 2 minutes et évite de perdre 2 heures |
| Démarrer UC 3 sans que UC 14 soit complet sur le périmètre | Les requêtes SQL ciblent des PF DDS et non des tables DDL — la conversion est incohérente et devra être refaite | Vérifier que `QSYS2.SYSTABLES` retourne toutes les tables DDL attendues avant de démarrer (Prompt 5, section 3) |
| Oublier de tester le SQLCODE après chaque instruction SQL | Un `UPDATE` qui ne trouve pas d'enregistrement passe silencieusement — le programme continue sans signaler l'anomalie | Inclure systématiquement `IF SQLCODE <> 0` (et `SQLCODE = 100` pour NOT FOUND) après chaque `EXEC SQL` |
| Supprimer la F-spec avant d'avoir converti tous les accès au fichier | Erreur de compilation sur les opcodes non encore convertis — source dans un état incohérent | Supprimer les F-specs uniquement avec le Prompt 4, après vérification complète de l'inventaire |
| Lancer le Prompt 2-bis alors que des F-specs n'ont pas encore été traitées | Erreur de compilation sur les opcodes natifs restants — le diagnostic est difficile car le message IBM i pointe vers le natif et non vers le SQL | Utiliser l'inventaire du Prompt 1 comme checklist : cocher chaque opcode converti avant de lancer le Prompt 2-bis |
| Convertir une boucle SETLL/READE en `SELECT ... WHERE clé >= :var` | Tous les enregistrements à partir de la clé sont renvoyés, pas seulement ceux avec la même clé partielle | Pour READE : utiliser `WHERE clé = :var` (égalité stricte), pas `>=` |
| Oublier de fermer le curseur SQL après la boucle | Le curseur reste ouvert, consomme des ressources, peut bloquer d'autres processus sur la même table | Toujours inclure `EXEC SQL CLOSE [NOM_CURSEUR]` immédiatement après le `ENDDO` de la boucle |
| Convertir sans valider la non-régression fonctionnelle | Le programme compile mais retourne des résultats différents (ordre différent, enregistrements manquants) | Tester chaque conversion avec un jeu de données réel sur l'IBM i de test — comparer les sorties avant/après |
| Utiliser des noms de tables DDS dans le SQL embarqué au lieu des noms DDL | Fonctionne techniquement (Db2 for i accepte les deux), mais contourne l'objectif de la modernisation | Référencer systématiquement les noms de tables du fichier `*-ddl-complet-*.md` de UC 14 |
| Ignorer les DS externées (EXTNAME) lors du nettoyage | Les DS référencent encore l'ancien fichier DDS — erreur de compilation ou comportement inattendu si la définition DDS change | Traiter les DS externées explicitement dans le Prompt 4 (nettoyage) |
| Travailler en mode Agent pendant la génération SQL | Bob peut sauvegarder des sources partiellement convertis (F-specs encore présentes, SQLCODE non testés) | Rester en mode **Ask** pendant toute la phase de génération (Prompts 0 à 4) — mode Agent uniquement pour Prompt 2-bis (compilation) et sauvegarde finale |

---

## Check-list de validation UC 3

Avant de passer à UC 7 (Optimisation — voir `UC07-optimisation-code.md`), valider chaque point :

- [ ] Le plan de conversion (Prompt 5) est produit et sauvegardé dans `{appArcad}-{fonction}-{fonction}-plan-conversion-{YYYYMMDD-HHmm}.md` — tous les programmes du périmètre sont listés avec leur catégorie (SIMPLE / STANDARD / COMPLEXE) et leur ordre de conversion
- [ ] **Pour chaque programme converti : le Prompt 0 a été exécuté** — la fiche de qualification `{appArcad}-{fonction}-{composant}-analyse-acces-{YYYYMMDD-HHmm}.md` existe et mentionne la catégorie et la stratégie retenue
- [ ] Chaque programme converti a un inventaire des accès natifs complet (Prompt 1) dans son fichier `*-analyse-acces-*` — aucun opcode oublié
- [ ] Tous les accès `CHAIN`, `WRITE`, `UPDATE`, `DELETE` simples ont été convertis avec les tests `SQLCODE` correspondants
- [ ] Toutes les boucles `SETLL/READ/READE` ont été converties en curseurs SQL — les curseurs sont nommés explicitement, ouverts, parcourus et fermés
- [ ] Les F-specs des fichiers entièrement convertis ont été supprimées (Prompt 4) — plus aucun opcode natif ne référence ces fichiers
- [ ] Les DS externées (`EXTNAME`) liées aux fichiers convertis ont été traitées — converties en DS SQL ou en variables hôtes individuelles
- [ ] Chaque programme converti a été **compilé sans erreur** sur l'IBM i de test
- [ ] Chaque programme converti a été **testé fonctionnellement** sur l'IBM i de test avec un jeu de données réel — les résultats avant et après conversion sont identiques
- [ ] Les sources convertis sont sauvegardés avec la convention de nommage `{appArcad}-{fonction}-{composant}-sql-embarque-{YYYYMMDD-HHmm}.md` dans le workspace ET publiés sur Confluence (si MCP disponible)
- [ ] La mention `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` est présente dans l'en-tête de chaque source converti
- [ ] Si un programme COMPLEXE a été orienté vers UC 8 avant UC 3 (décision du Prompt 0), la tâche UC 8 correspondante est ouverte — voir `UC08-restructuration-code.md`

---

## Points à compléter avant passage en production

> Ces points ne bloquent pas le POC — ils concernent des cas avancés peu probables sur les programmes pilotes. Ils deviennent critiques dès que la conversion s'étend à des programmes de gestion complexes en production.

### Gestion des transactions SQL (COMMIT / ROLLBACK)

**Contexte :** certains programmes RPG effectuent plusieurs `WRITE` / `UPDATE` / `DELETE` qui doivent réussir ou échouer ensemble (ex. : créer une commande + mettre à jour le stock + écrire une ligne de journal). En RPG natif, le journal IBM i gère la cohérence implicitement selon le niveau de commit configuré. En SQL embarqué, il faut gérer explicitement les transactions :

```rpg
EXEC SQL SET TRANSACTION ISOLATION LEVEL CS ;
// ... accès SQL ...
EXEC SQL COMMIT ;
// en cas d'erreur :
EXEC SQL ROLLBACK ;
```

**Pourquoi absent de la fiche POC :** les programmes pilotes du POC sont choisis parmi les programmes SIMPLE et STANDARD — les blocs transactionnels apparaissent principalement sur les programmes COMPLEXE abordés en fin de parcours.

**À faire avant production :** ajouter un Prompt 3-ter dédié à la détection et conversion des blocs transactionnels. Points à couvrir : identification des séquences multi-fichiers sans commit explicite, choix du niveau d'isolation (`*NONE` / `*CS` / `*ALL`), pattern `COMMIT` / `ROLLBACK` + `SQLCODE`, interaction avec les journaux IBM i existants.

> ⚠️ **Signal d'alerte sur le terrain :** si le Prompt 0 détecte la combinaison `WRITE + UPDATE` sur des fichiers différents dans la même subroutine, ou la présence d'un opcode `ROLBK` / `COMMIT` dans le source natif — traiter ce programme comme un candidat à la gestion transactionnelle avant de démarrer la conversion.

---

*Fiche UC 3 — Document évolutif à mettre à jour au fil du POC.*
