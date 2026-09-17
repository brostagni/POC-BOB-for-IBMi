# UC 13 — Tests unitaires et automatisation (RPGUnit)

> **Catégorie :** Tests
>
> **Priorité dans le POC :** 12 — déclenché dès qu'il existe du code modernisé à valider (au plus tôt après UC 7-8)
>
> **Durée POC (avec Bob) :** 3 à 6 heures — génération et exécution des tests sur un périmètre représentatif (3 à 5 programmes modernisés), itérations et corrections
>
> **Durée PROD (avec Bob) :** 1 à 2 heures / programme — génération du programme de test RPGUnit, exécution, analyse du rapport
>
> **Durée PROD (sans Bob) :** 1 à 2 jours / programme — conception manuelle des cas de test, rédaction du programme RPGLE de test, exécution, analyse des résultats, correction des régressions
>
> **Gain Bob estimé :** ~10× — un programme de test RPGUnit complet généré en 20 minutes au lieu d'une journée ; gain multiplicatif si le périmètre à couvrir est important (track parallèles UC 1, 2, 7, 8, 9, 10, 11)
>
> **Mode Bob recommandé :** IBM i Developer (mode Ask pour l'analyse et la génération, Agent pour la compilation et l'exécution des tests)

---

## Objectif

Générer, exécuter et analyser des **tests unitaires RPGUnit** pour les programmes IBM i modernisés au cours du POC : programmes optimisés (UC 7), restructurés (UC 8), convertis (UC 1, UC 2), générés (UC 9, UC 10, UC 11). C'est le **filet de non-régression** qui prouve que la modernisation n'a pas cassé la logique fonctionnelle.

**Ce UC ne teste pas l'ensemble d'une application.** L'ambition est ciblée : tester les programmes qui ont été modifiés, en vérifiant que leur comportement observé est identique à celui décrit dans les fiches de compréhension (UC 4), les règles métier (UC 5) et les livrables de diff produits par les UC précédents.

**Livrable attendu :** Pour chaque programme testé : un programme de test RPGUnit compilé dans la bibliothèque de test, le rapport d'exécution des tests, et un fichier de traçabilité documentant les cas de test couverts et les résultats obtenus.

**Convention de nommage des fichiers générés :**
```
{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md

Types pour cet UC :
  analyse-test    → fiche de qualification (Prompt 0) — inventaire des cas de test identifiables
  programme-test  → source RPGUnit généré ou diff du programme de test (Prompts 1, 2)
  plan-test       → plan de tests pour un périmètre applicatif complet (Prompt 3)
  rapport-test    → rapport d'exécution des tests RPGUnit (Prompt 4)
```

Exemples :
```
acme-APPVTE-GESCMD-analyse-test-20250620-0900.md   ← qualification des cas de test
acme-APPVTE-GESCMD-programme-test-20250620-1100.md ← source RPGUnit généré
acme-APPVTE-APPVTE-plan-test-20250620-0800.md      ← plan pour le périmètre complet
acme-APPVTE-GESCMD-rapport-test-20250620-1400.md   ← rapport d'exécution
```

> 💡 Cette convention est valable en dehors du contexte POC — réutilisable en production tel quel. Les programmes de test RPGUnit eux-mêmes sont sauvegardés dans la bibliothèque de test IBM i (ex. `APPVTETEST`) avec le nom `{NOM_PROGRAMME}T` — la convention de nommage `.md` s'applique uniquement aux livrables de documentation.

---

## Démarrer par un programme que vous connaissez

> **Recommandation forte avant d'aborder les programmes critiques de ACME.**

Avant de tester des programmes inconnus ou des restructurations complexes, **commencer par un programme simple dont un développeur de l'équipe connaît le comportement attendu** — idéalement un programme dont les résultats ont déjà été validés manuellement lors de UC 7 ou UC 8.

Pourquoi ? Parce que la première session de test UC 13 sert à **calibrer deux choses** :
- La qualité des programmes de test générés par Bob (est-ce que les cas de test couvrent les vrais comportements du programme ?)
- La capacité de l'équipe à exécuter RPGUnit sur l'IBM i de test (accès, compilation, rapport)

Si le premier test fonctionne et détecte correctement un comportement attendu, la confiance est établie pour les programmes plus complexes.

> 💡 **Ne pas commencer par un programme issu de UC 8 (Restructuration)** comme premier test — ces programmes ont subi les changements les plus profonds et ont le plus de cas de test à couvrir. Commencer par un programme UC 7 (optimisation) ou UC 9 (génération).

**Progression recommandée :**

> 💡 Pour chaque programme de cette progression, **commencer par le Prompt 0** — il donne la catégorie (SIMPLE / STANDARD / COMPLEXE) et la liste des cas de test à couvrir. Ne pas aller directement au Prompt 1.

| Étape | Programme à choisir | Catégorie attendue | Objectif |
|-------|--------------------|--------------------|---------|
| 1 | Programme simple optimisé (UC 7), logique de calcul directe, < 5 sorties observables | SIMPLE | Calibrer la génération de tests et valider l'exécution RPGUnit sur l'IBM i de test |
| 2 | Programme converti (UC 2 RPG colonné → FREE), avec quelques règles de validation | STANDARD | Valider que la conversion n'a pas changé les résultats — tester les cas limites identifiés en UC 5 |
| 3 | Programme restructuré (UC 8), décomposé en modules de service | COMPLEXE | Tester chaque procédure exportée du `*SRVPGM` indépendamment, puis les flux d'appel |
| 4 | Procédure stockée ou trigger SQL (UC 11) | SQL | Valider le comportement SQL avec données de test, cas d'erreur SQLCODE |

---

## Impact du programme sur la stratégie de test

La complexité d'UC 13 ne se mesure pas au nombre de lignes du programme testé, mais au **nombre de sorties observables**, à la **présence d'état externe** (base de données, date système, compteur), et à la **nature des effets de bord** (le programme modifie-t-il des données en base que d'autres programmes consultent ?).

Les vrais facteurs qui compliquent la génération de tests :
- **Sorties non déterministes** — un programme qui lit la date courante ou génère un numéro séquentiel produit une valeur différente à chaque exécution : les assertions sur valeur fixe sont impossibles. Stratégie : tester la forme (le champ est non vide, le format est YYYYMMDD) plutôt que la valeur exacte.
- **Effets de bord sur la base de données** — un programme qui écrit dans plusieurs tables ne peut être testé qu'avec des données de test isolées dans une bibliothèque de test. Sans TEARDOWN rigoureux, les exécutions successives du test se contaminent.
- **Dépendances d'appel entre programmes** — un programme qui `CALL` trois sous-programmes doit les avoir tous compilés et disponibles dans la bibliothèque de test pour que son programme de test RPGUnit fonctionne. Tester de bas en haut dans la pile (les sous-programmes d'abord).
- **Programmes interactifs 5250** — les programmes avec écrans `EXFMT` ne peuvent pas être testés en mode headless par RPGUnit. Stratégie : extraire la logique métier dans des procédures autonomes (résultat de UC 8) et tester ces procédures — pas le programme interactif directement.
- **Programmes batch avec `SBMJOB`** — le programme testé peut lui-même soumettre un job. RPGUnit exécute le programme en interactif — le `SBMJOB` sera bien soumis, mais le résultat du job soumis ne sera pas disponible pendant l'exécution du test. Signaler ces cas dans le Prompt 0.

### Programmes SIMPLE — < 5 sorties observables, pas d'état externe, pas d'appel externe

Bob gère sans difficulté. **Séquence : Prompt 0 → Prompt 1 → Prompt 1-bis (compilation du programme de test) → Prompt 4 (exécution).**

> 💡 Le Prompt 0 confirme la catégorie SIMPLE et recommande directement cette séquence — pas de décision manuelle requise.

### Programmes STANDARD — 5 à 15 cas de test, état DB, ou appels de sous-programmes

Le Prompt 0 identifie les groupes de cas et les dépendances. **Séquence : Prompt 0 → Prompt 1 répété par groupe → Prompt 1-bis après chaque groupe → Prompt 4.**

> 💡 Construire les données de test (SETUP SQL) groupe par groupe. Un SETUP trop grand qui crée 20 enregistrements de test est difficile à maintenir — préférer un SETUP minimal par groupe de cas.

### Programmes COMPLEXE — > 15 cas de test, effets de bord multiples, ou programmes interactifs

Un programme COMPLEXE en UC 13 est souvent un programme issu de UC 8 (restructuré). Le Prompt 0 pose explicitement la question : **les procédures exportées du *SRVPGM sont-elles testables indépendamment ?** Si oui, tester procédure par procédure avant le flux complet. Si non (trop de variables globales partagées non exportées), signaler le cas et limiter UC 13 aux flux principaux.

**Séquence : Prompt 0 → Prompt 1 par procédure exportée → Prompt 1-bis → Prompt 4 par procédure → Prompt 1 pour le flux complet → Prompt 4 flux complet.**

> ⚠️ Sur un programme COMPLEXE, un programme de test unique qui couvre 20 cas en une seule TESTCASE ne compile généralement pas (taille limite du membre source). Découper les cas de test en plusieurs programmes de test liés par une TESTSUITE RPGUnit.

### Programmes batch — traitement séquentiel de fichiers, pas d'entrée interactive

Les programmes batch (lancement par CL, `SBMJOB`, `RUNQRY`) ont des entrées implicites (paramètres du job, fichiers présents dans la bibliothèque). La stratégie de test change :

```
RUCALLTST TSTPGM([NOM_LIB_TEST]/[NOM_PROGRAMME]T)
          OUTPUT(*SYSOUT)
          DETAIL(*ALL)
-- ne couvre que l'exécution interactive.
-- Pour tester le programme en mode batch réel, utiliser SBMJOB depuis le programme de test
-- et interroger le statut du job soumis avec QSYS2.JOB_INFO après un délai.
```

> ⚠️ Le test d'un programme batch via RPGUnit est une approximation — RPGUnit exécute en interactif. Les différences d'attributs de job (CCSID, LANGID, DATFMT) entre interactif et batch peuvent produire des résultats différents. Signaler ces cas dans le rapport UC 13 comme "à valider en batch réel".

---

## Démarrer une session Bob

> **À lire avant chaque session UC 13 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

Chaque session de test d'un programme doit démarrer dans une **nouvelle conversation Bob** (bouton `+` en haut du panneau Chat). Ne pas tester deux programmes différents dans la même conversation — les comportements attendus d'un programme pollueraient les assertions générées pour l'autre.

**Exception :** si on génère un plan de tests périmètre (Prompt 3), rester dans la même conversation exploite la liste complète des programmes à tester.

**Mode à sélectionner :** `IBM i Developer` — rester en **Ask** pour les Prompts 0 à 3 (analyse et génération), basculer en **Agent** pour le Prompt 1-bis (compilation), le Prompt 4 (exécution RPGUnit), le Prompt 5 (automatisation batch) et la sauvegarde.

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Avant de lancer le Prompt 0, ouvrir dans l'éditeur Bob :
- Le programme RPG à tester — dans sa version **après modernisation** (`[NOM_LIB_SOURCE]/QRPGSRC([NOM_PROGRAMME])`)
- Les livrables de diff du programme — `*-diff-restr-*.md` (UC 8), `*-diff-optim-*.md` (UC 7), ou `*-rpg-converti-*.md` (UC 2) — ils documentent ce qui a changé et guident la génération des assertions

**Procédure :** dans le panneau **IBM i — Object Browser** (extension Code for IBM i), naviguer jusqu'à la bibliothèque source, faire un clic droit sur le membre → **Open in Editor**. Pour les fichiers `.md` : dans l'explorateur de fichiers Bob (panneau Explorer) → **Add File to Chat**.

### 3. Fichiers de contexte à charger

Ces fichiers sont les inputs directs des prompts UC 13. Les charger dans le chat via **Add File to Chat** avant de lancer le prompt correspondant.

| Fichier | Produit par | Utilisé dans | Obligatoire / Recommandé |
|---------|-------------|-------------|--------------------------|
| `*-diff-restr-*.md` | UC 8 | Prompt 0, Prompt 1 | **Obligatoire si programme restructuré** — identifie ce qui a changé → ce qu'il faut tester |
| `*-diff-optim-*.md` | UC 7 | Prompt 0, Prompt 1 | **Obligatoire si programme optimisé** |
| `*-rpg-converti-*.md` | UC 2 | Prompt 0, Prompt 1 | **Obligatoire si RPG colonné converti** |
| `*-cobol-converti-*.md` | UC 1 | Prompt 0, Prompt 1 | **Obligatoire si COBOL converti** |
| `*-programme-genere-*.md` | UC 9 | Prompt 0, Prompt 1 | **Obligatoire si programme généré UC 9** |
| `*-comprehension-*.md` | UC 4 | Prompt 1 | **Recommandé** — comportements attendus et dépendances |
| `*-regles-*.md` | UC 5 | Prompt 1 | **Recommandé** — règles métier à vérifier dans les assertions |
| `*-objet-sql-*.md` / `*-procedure-sql-*.md` | UC 11 | Prompt 2 | **Obligatoire pour tests procédures SQL** |
| `*-analyse-test-*.md` | UC 13 (P0) | Prompts 1, 2, 4 | **Si reprise** — reprendre les cas de test déjà identifiés |
| `*-programme-test-*.md` | UC 13 (P1/P2) | Prompt 4 | **Si reprise** — programme de test déjà généré à exécuter ou compléter |

> ⚠️ **Prérequis bloquant :** ne pas démarrer UC 13 sur un programme sans avoir au moins un fichier de diff (`*-diff-restr-*`, `*-diff-optim-*`, `*-rpg-converti-*` ou `*-cobol-converti-*`) — sinon il est impossible de savoir quoi tester.

> ⚠️ **Risque de réduction de contexte — sauvegarde intermédiaire recommandée :** une session UC 13 avec génération + exécution + correction peut atteindre la limite de contexte sur les programmes COMPLEXE. Sauvegarder le programme de test `.md` après le Prompt 1, avant de passer à l'exécution (Prompt 4).

---

## Prérequis

- UC 15 complété (maîtrise de Bob, modes Ask et Agent)
- UC 12 Phase 0 complété (IBM i MCP + IBM i Database MCP actifs)
- **Au moins un UC de modernisation complété sur le périmètre** : UC 7 (optimisation) ou UC 8 (restructuration) au minimum ; UC 1, UC 2, UC 9, UC 10, UC 11 selon le périmètre
- Les livrables de diff des UC précédents sont présents dans le workspace (voir section "Fichiers de contexte à charger")
- **RPGUnit installé sur l'IBM i de test** — la bibliothèque RPGUnit est dans la Library List et la commande `RUCALLTST` est disponible
- L'utilisateur de connexion IBM i dispose des autorités nécessaires pour compiler dans la bibliothèque de test (ex. `APPVTETEST`)
- **Version de RPGUnit identifiée** — différentes distributions ont des interfaces différentes (copybook, commande de compilation, noms des assertions, type d'objet produit). Le Prompt 0 doit confirmer ces éléments avant toute génération de code.

> ⚠️ **RPGUnit — identification de version obligatoire avant le Prompt 1 :** les exemples de cette fiche supposent une interface RPGUnit classique (`RUCALLTST`, `CRTBNDRPG`). Certaines distributions modernes (iRPGUnit) produisent un `*SRVPGM` compilé via `RUCRTRPG`. Si la version installée diffère, les commandes des Prompts 1-bis et 4 doivent être adaptées. Le Prompt 0 est le point de vérification — ne jamais supposer l'interface depuis le nom de la bibliothèque.

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | **IBM i Developer** (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : Ask pour l'analyse/génération, Agent pour la compilation, l'exécution et la sauvegarde. |
| **Scope** | Library List → bibliothèque applicative ACME + bibliothèque de test |
| **MCP actifs** | IBM i MCP (lecture sources, création membre de test, exécution RPGUnit) + IBM i Database MCP (données de test, requêtes QSYS2) |
| **MCP non utilisés** | Confluence MCP (les rapports de test peuvent être publiés si token disponible, mais ce n'est pas prioritaire) |

### Pourquoi le mode IBM i Developer pour les tests unitaires ?

Le mode **IBM i Developer** pré-charge le contexte IBM i (RPG, CL, IBM i MCP, RPGUnit) dans chaque conversation. Bob génère dans le chat — l'équipe valide le programme de test avant toute écriture ou compilation sur l'IBM i. Cette validation est le rempart contre la création d'un programme de test défectueux en bibliothèque de production.

| Phase | Comportement attendu | Ce que Bob fait |
|-------|----------------------|----------------|
| Qualification (Prompt 0) | Génère dans le chat — pas d'écriture | Lit le source modernisé, interroge QSYS2 pour la version RPGUnit, identifie les cas de test |
| Génération du programme de test (Prompts 1, 2 SQL) | Génère dans le chat — pas d'écriture | Génère le source RPGUnit dans le chat — aucune écriture sur l'IBM i |
| Plan périmètre (Prompt 3) | Génère dans le chat — pas d'écriture | Lit les livrables de diff pour construire le plan |
| Compilation seule — Prompt 1-bis | Écriture et compilation autorisées — après validation | Crée le membre source de test dans `QTESTSRC`, compile — pas d'exécution |
| Exécution des tests (Prompt 4) | Exécution autorisée — après compilation réussie | Exécute `RUCALLTST`, lit le rapport spool via `SYSTOOLS.SPOOLED_FILE_DATA` |
| Génération du script CL de suite (Prompt 5) | Génère dans le chat — pas d'écriture | Génère le source CL dans le chat |
| Exécution batch de la suite | Exécution autorisée — après validation | Lance `SBMJOB`, suit le job via `QSYS2.JOB_INFO`, lit les résultats |
| Sauvegarde des livrables | Écriture autorisée — après validation | Sauvegarde les fichiers `.md` dans `LIVRABLES/` |

> 💡 **Règle d'or pour UC 13 :** Bob génère dans le chat. L'écriture (`write_member`), la compilation et l'exécution ne sont autorisées qu'après validation explicite de l'équipe. Ne jamais exécuter sur la bibliothèque de **production** — toujours en bibliothèque de test.

> ⚠️ **Bibliothèque cible :** les programmes de test RPGUnit doivent être créés dans une bibliothèque de test dédiée (ex. `APPVTETEST`) — jamais dans les bibliothèques source gérées par ARCAD. Cette séparation est critique pour ne pas polluer le versioning ARCAD.

> 💡 **Sans Premium Package IBM i :** utiliser le mode Ask pour les Prompts 0-3 et la génération P5, puis basculer en mode Agent uniquement pour le Prompt 1-bis (compilation), le Prompt 4 (exécution), l'exécution batch P5 et la sauvegarde.

### Intégration ARCAD

Le MCP ARCAD n'était pas disponible dans le contexte de ce POC de référence (version ARCAD non compatible avec le MCP). Si le MCP ARCAD est disponible dans votre environnement, les étapes manuelles de réintégration décrites ci-dessous peuvent être automatisées. N'hésitez pas à demander à Bob de modifier cette fiche UC en intégrant la disponibilité du MCP ARCAD.

**Impact sur UC 13 : limité.** Les tests RPGUnit s'exécutent directement sur l'IBM i de test via IBM i MCP, indépendamment d'ARCAD.

| Sans MCP ARCAD (contexte de ce POC) | Avec MCP ARCAD disponible | Remarque |
|--------------------------------------|---------------------------|----------|
| Réintégrer manuellement les sources de test dans ARCAD après chaque session | Le MCP ARCAD peut enregistrer les sources de test dans ARCAD directement | Même workflow UC 7-8 |
| Inclure manuellement le **manifest de traçabilité** dans chaque rapport | Le MCP ARCAD peut alimenter automatiquement la traçabilité version/test | Le fichier `*-diff-restr-*.md` fournit le delta couvert dans les deux cas |
| Déclencher manuellement un pipeline ARCAD post-test | Le MCP ARCAD peut déclencher le pipeline depuis Bob | L'intégration pipeline complète est l'objet de UC 16 |

**Manifest de traçabilité obligatoire dans chaque `*-rapport-test-*.md` :**

```markdown
## Traçabilité d'exécution

- Objet testé : [NOM_PROGRAMME] / [NOM_LIB] / type : *PGM ou *SRVPGM
- Objet testé (bibliothèque/source de compilation) : [OBJECT_STATISTICS — SOURCE_LIBRARY/SOURCE_FILE(SOURCE_MEMBER)]
- Date de création / recréation de l'objet testé : [OBJECT_STATISTICS — OBJCREATED]
- Date de dernière modification de l'objet testé : [OBJECT_STATISTICS — CHANGE_TIMESTAMP]
- Horodatage du source utilisé lors de la compilation : [OBJECT_STATISTICS — SOURCE_TIMESTAMP]
- Source de test : [NOM_LIB_TEST]/QTESTSRC([NOM_PROGRAMME]T)
- Version RPGUnit utilisée : [identifiée en Prompt 0]
- Nom qualifié du job d'exécution : [JOB_NAME depuis QSYS2.OUTPUT_QUEUE_ENTRIES]
- Numéro et nom du spool file : [FILE_NUMBER / SPOOLED_FILE_NAME]
- Livrable de diff de référence : [*-diff-restr-* ou *-rpg-converti-* utilisé]
- Statut de réintégration ARCAD du source de test : [ ] À faire / [x] Fait
```

> ⚠️ **Pour UC 16 (DevOps) :** un rapport sans manifest de traçabilité n'est pas une preuve exploitable — il ne peut pas être rattaché à l'objet exact qui sera promu. Compléter le manifest avant de déclarer le programme VALIDÉ.

---

## Prompts clés

### Prompt 0 — Qualification et inventaire des cas de test

```
Le programme [NOM_PROGRAMME] dans [NOM_LIB]/QRPGSRC a été modernisé dans le cadre du POC
(UC [N_UC] — [TYPE_MODERNISATION]).
[Si reprise : charger le fichier *-diff-restr-*.md ou *-rpg-converti-*.md dans le contexte avant ce prompt]

Analyse ce programme et produis en français, en markdown, une fiche de qualification
pour la création de tests unitaires RPGUnit :

## Qualification UC 13 — [NOM_PROGRAMME]

### 1. Identification du programme et de l'environnement de test
| Indicateur | Valeur |
|------------|--------|
| Type d'objet IBM i | *PGM / *SRVPGM |
| Modèle | OPM (RPG III / RPG400) / ILE |
| Activation group | nommé / *CALLER / *NEW / défaut |
| Programme remet *INLR à ON | OUI / NON / À CONFIRMER |
| Procédures exportées (si *SRVPGM) | liste ou AUCUNE |
| Autorité adoptée | OUI / NON |
| Programme interactif (EXFMT) | OUI → logique métier testable via procédures extraites seulement / NON |
| Programme batch (lancé par SBMJOB) | OUI / NON |

Interroger QSYS2.OBJECT_STATISTICS et QSYS2.PROGRAM_INFO pour confirmer.
Ne pas déduire ces informations depuis le nom du programme.

### 2. Vérification de l'environnement RPGUnit
Identifier la version et l'interface installées en interrogeant les objets IBM i réels :
DSPCMD et DSPOBJD pour localiser les commandes et objets, puis inspection des membres source,
copybooks, templates ou exemples installés pour les assertions et les hooks.
Ne pas déduire l'API depuis le seul nom d'une commande ou d'un objet — les noms peuvent être identiques
entre versions avec des interfaces différentes.

Répondre À VÉRIFIER si l'information ne peut pas être confirmée depuis l'installation réelle :
- [ ] Commande RUCALLTST disponible — version exacte : [À RENSEIGNER]
- [ ] Copybook d'assertions : nom de fichier et membre exacts confirmés depuis la bibliothèque RPGUnit — [À RENSEIGNER]
- [ ] Commande de compilation : CRTBNDRPG ou RUCRTRPG — [À RENSEIGNER selon version]
- [ ] Type d'objet produit par la compilation : *PGM ou *SRVPGM — [À RENSEIGNER selon version]
- [ ] Bibliothèque de test [NOM_LIB_TEST] existe et l'utilisateur a autorité de compilation
- [ ] Journalisation active sur les tables de [NOM_LIB_TEST] (nécessaire pour isolation transactionnelle)
- [ ] Périmètre de commitment control applicable au programme testé :
      même job / même activation group / à confirmer

### 3. Interface de cycle de vie RPGUnit confirmée
Inspecter les exemples et la documentation installés pour identifier les hooks disponibles.
Ne pas supposer les noms depuis une connaissance générale de RPGUnit.

| Hook | Nom confirmé depuis l'installation | Disponible ? | Doit être exporté ? |
|------|-------------------------------------|-------------|---------------------|
| Avant toute la suite | [À RENSEIGNER] | OUI / NON / À VÉRIFIER | OUI / NON / À VÉRIFIER |
| Avant chaque test | [À RENSEIGNER] | OUI / NON / À VÉRIFIER | OUI / NON / À VÉRIFIER |
| Après chaque test | [À RENSEIGNER] | OUI / NON / À VÉRIFIER | OUI / NON / À VÉRIFIER |
| Après toute la suite | [À RENSEIGNER] | OUI / NON / À VÉRIFIER | OUI / NON / À VÉRIFIER |

⚠️ Ne pas inventer la disponibilité d'une commande, d'un copybook, d'un hook ou d'une assertion depuis son nom.
Si un élément ne peut pas être confirmé depuis l'installation réelle, marquer À VÉRIFIER
et bloquer la génération du programme de test jusqu'à résolution.

### 4. Sorties observables du programme
Liste toutes les sorties que ce programme produit :
| Sortie | Type (fichier écrit / paramètre retourné / impression / écran mis à jour) | Déterministe ? |
Déterministe : OUI si la sortie dépend uniquement des entrées, NON si elle dépend d'un état extérieur
(date système, compteur auto-incrémenté, etc.)

Pour les sorties non déterministes : décrire la stratégie de test (asserter la forme, pas la valeur).

### 5. Cas de test identifiables depuis le source
Pour chaque règle de gestion ou branche conditionnelle visible dans le source :
| N° cas | Description du cas de test | Entrées nécessaires | Sortie attendue | Règle métier source (fichier *-regles-*) |

### 6. Catégorie et séquence recommandée
- [ ] SIMPLE — sorties directes, < 5 cas de test, pas de boucle complexe ni d'état externe
- [ ] STANDARD — 5 à 15 cas de test, quelques branches conditionnelles, données de test nécessaires
- [ ] COMPLEXE — > 15 cas de test, état externe (DB, date), modules de service à tester séparément

Séquence recommandée :
- SIMPLE   → Prompt 1 → Prompt 1-bis (compilation) → Prompt 4 (exécution)
- STANDARD → Prompt 1 répété par groupe de cas → Prompt 1-bis → Prompt 4 après chaque groupe
- COMPLEXE → Prompt 1 par procédure exportée → Prompt 1-bis → Prompt 4 par procédure →
              Prompt 2 si procédures SQL incluses → Prompt 4 flux complet

Signale clairement ce que tu ne peux pas déterminer sans exécuter le programme.
Ne pas générer de programme de test si les sections 2 ou 3 contiennent des points À VÉRIFIER non résolus.
Ne pas référencer les sections par leur numéro dans ce prompt — utiliser leur titre (ex. "section Vérification de l'environnement RPGUnit").
```

**Analyse ligne à ligne :**

- `Type d'objet, modèle OPM/ILE, activation group, périmètre commitment control` → **qualification préalable indispensable**. Un programme OPM (RPG III) ne peut pas être testé de la même façon qu'un `*SRVPGM` ILE. Un programme OPM qui ne remet pas `*INLR` à ON conserve son état entre deux appels dans le même job — les tests suivants obtiennent l'état du test précédent. Le périmètre de commitment control détermine si le SAVEPOINT du programme de test englobe les opérations du programme testé.

- `Inspection des membres source et copybooks installés — ne pas déduire depuis le nom` → **garde-fou anti-hallucination sur l'environnement**. `DSPCMD RUCALLTST` confirme que la commande existe, pas l'interface du copybook ni les noms des assertions. Bob doit inspecter les exemples installés ou le source du copybook, pas compléter depuis sa connaissance générale de RPGUnit.

- `Tableau des hooks avec nom confirmé` → **même principe de neutralisation que pour les assertions**. Les noms `setUp`/`tearDown`/`setUpSuite`/`tearDownSuite` sont des conventions courantes, mais pas universelles selon la version installée. Le tableau force une confirmation explicite avant que le Prompt 1 ne génère ces procédures.

- `Journalisation + périmètre commitment control` → **deux prérequis liés pour l'isolation transactionnelle**. Sans journalisation, ROLLBACK n'est pas disponible. Sans participation au même périmètre de commitment control, un SAVEPOINT défini dans le programme de test n'englobe pas les opérations effectuées par le programme testé — l'isolation est alors illusoire.

- `Ne pas générer de code si sections 2 ou 3 contiennent des À VÉRIFIER` → **blocage explicite**. Sans cette instruction, Bob peut générer avec les meilleures hypothèses disponibles — ce qui produit un source non compilable ou fonctionnellement incorrect présenté comme correct.

> ⚠️ **Piège évité :** sans la section 3 (hooks), le Prompt 1 peut générer des procédures `setUpSuite`/`tearDownSuite` avec des noms que la version installée de RPGUnit ne reconnaît pas — le programme compile mais les hooks ne s'exécutent jamais, laissant des données résiduelles sans erreur visible.

---

### Prompt 1 — Génération du programme de test RPGUnit

```
Sur la base de la fiche de qualification [NOM_FICHIER_ANALYSE_TEST]
pour le programme [NOM_PROGRAMME] dans [NOM_LIB]/QRPGSRC,
[Si nouvelle session : charger le fichier *-analyse-test-*.md et le source du programme dans le contexte]

La fiche de qualification a confirmé :
- Copybook RPGUnit à utiliser : [NOM_COPYBOOK tel qu'identifié en section 2 du Prompt 0]
- Noms des assertions disponibles : [tels qu'identifiés — ne pas substituer]
- Type d'objet cible : [*PGM via CRTBNDRPG / *SRVPGM via RUCRTRPG]
- Hooks cycle de vie disponibles : [noms confirmés section 3 du Prompt 0 — ne pas supposer]
- Hooks de suite disponibles : [OUI avec noms / NON → intégrer dans les hooks de test disponibles]
- Journalisation disponible : [OUI → SAVEPOINT possible / NON → DELETE par clé]
- Périmètre commitment control : [même job / même activation group / à confirmer]

Génère en FREE RPG (ILE) un programme de test RPGUnit couvrant les cas de test [LISTE_CAS_OU_GROUPE].

Le programme de test doit :

1. Déclaration et en-tête
- Nom du programme de test : [NOM_PROGRAMME]T (suffixe T — convention RPGUnit)
- Inclure le copybook identifié en Prompt 0 (ne pas supposer le nom)
- Déclarer le prototype du programme testé ou de la procédure exportée

2. Hook avant suite (nom confirmé en Prompt 0, section 3 — si disponible)
- Vérifier que la bibliothèque de test [NOM_LIB_TEST] est accessible
- Vérifier la journalisation et le périmètre de commitment control
- Initialiser les constantes et clés de corrélation uniques (ex. préfixe horodaté)
- Si le hook de suite n'est pas disponible dans cette version, intégrer ces vérifications
  dans les premières lignes du hook avant chaque test

3. Hook avant chaque test (nom confirmé en Prompt 0, section 3)
- Insérer uniquement les données du cas de test courant dans [NOM_LIB_TEST]
- Si journalisation et même périmètre commitment control : définir un SAVEPOINT
- Initialiser les paramètres d'entrée

4. Procédures de test (une par cas de test)
- Nom : test[NomCas] (ex. testCalculRemise, testClientInconnu)
- Appel du programme ou de la procédure testée
- Assertions avec les noms exacts identifiés en Prompt 0
- Asserter la forme pour les sorties non déterministes (NON vide, format YYYYMMDD…)

5. Hook après chaque test (nom confirmé en Prompt 0, section 3 — déclenché même en cas d'échec)
- Stratégie selon le contexte identifié en Prompt 0 :
  a) SAVEPOINT disponible ET programme testé sans COMMIT propre :
     → ROLLBACK TO SAVEPOINT
  b) Sinon (journalisation absente, ou programme testé committe) :
     → DELETE par clé de corrélation unique sur toutes les tables affectées
     → Le programme testé reste testable dans les deux cas ; seule la stratégie de nettoyage change
- Vérifier l'absence de données résiduelles

6. Hook après suite (nom confirmé en Prompt 0, section 3 — si disponible)
- Nettoyage de sécurité sur les données de test
- Si le programme OPM ne remet pas *INLR à ON (identifié en Prompt 0, section 1) :
  signaler qu'il conserve potentiellement un état ou des ressources entre les appels.
  Définir avec l'équipe IBM i une stratégie d'isolation validée :
  exécution dans un job dédié, mécanisme de libération compatible avec l'environnement,
  ou exclusion des tests successifs dans le même job.
  Ne pas prétendre modifier directement le *INLR du programme testé.

7. Lisibilité
- Commentaires en français sur chaque cas de test
- Un cas de test par procédure — ne pas regrouper des assertions non liées

Ne pas inventer de comportements non visibles dans le source ou dans la fiche de qualification.
Ne pas supposer les noms d'assertions ou de hooks — utiliser uniquement ceux identifiés en Prompt 0.
Signaler les cas où une assertion nécessite des données de référence non disponibles.
Si le programme testé exécute lui-même un COMMIT :
- ne pas déclarer le programme non testable ;
- basculer vers la stratégie b) du hook après chaque test (DELETE par clé de corrélation) ;
- signaler que le SAVEPOINT ne protège plus au-delà du COMMIT.
N'utiliser la stratégie SAVEPOINT/ROLLBACK que si le programme testé participe
au même périmètre de commitment control que le programme de test.
```

**Analyse ligne à ligne :**

- `Copybook, assertions, type d'objet — tels qu'identifiés en Prompt 0` → **l'interface RPGUnit est déclarée par le Prompt 0, pas supposée par le Prompt 1**. Cette instruction force une chaîne de dépendance explicite entre les deux prompts et élimine le risque de générer du code avec des noms d'assertions fictifs.

- `Hooks confirmés en Prompt 0, pas supposés` → **même principe de neutralisation que pour les assertions**. Le Prompt 1 reçoit les noms de hooks depuis la fiche de qualification ; si le hook de suite n'est pas disponible, la logique de `setUpSuite` est intégrée dans le hook par test — sans inventer de procédure inexistante. Un hook avec un nom incorrect compile mais ne s'exécute jamais, laissant des données résiduelles sans erreur visible.

- `Clés de corrélation uniques (préfixe horodaté)` → **isolation sans dépendance à la journalisation**. Si les tables ne sont pas journalisées (cas fréquent sur les bibliothèques de test légacy), le ROLLBACK n'est pas disponible. La clé de corrélation unique permet un `DELETE WHERE CLE_TEST = :cle` ciblé même si d'autres tests ont tourné en parallèle.

- `Si journalisation : SAVEPOINT / Sinon : DELETE par clé` → **deux stratégies explicites selon l'environnement**. Le Prompt 0 a identifié laquelle est disponible. Bob choisit la bonne stratégie sans décider à la place de l'équipe.

- `Si le programme testé exécute un COMMIT — stratégie b) pas blocage` → **un COMMIT interne rend le SAVEPOINT inopérant au-delà, mais ne rend pas le programme non testable**. La stratégie d'isolation bascule vers le DELETE par clé de corrélation unique. Déclarer un programme avec COMMIT comme non testable exclurait une large partie des programmes batch IBM i — ce n'est pas la bonne conclusion.

- `Périmètre de commitment control` → **prérequis silencieux**. Si le programme de test et le programme testé ne partagent pas le même périmètre, le SAVEPOINT du test n'englobe pas les opérations du programme appelé — l'isolation semble correcte mais ne l'est pas. Le Prompt 0 identifie ce cas ; le Prompt 1 propage la décision.

> ⚠️ **Piège évité :** un `DELETE` en TEARDOWN qui échoue silencieusement (contrainte FK, verrou) laisse des données résiduelles qui faussent le test suivant sans erreur visible. Le `tearDownSuite` de sécurité détecte ce cas.

> 💡 **Variante pour les modules de service (*SRVPGM) :** générer une procédure de test par procédure exportée du `*SRVPGM` — et un `test[Flux]` de flux complet qui appelle le programme principal. Les tests de procédures exportées donnent une couverture fine ; le test de flux détecte les régressions d'intégration entre modules.

---

### Prompt 2 — Génération des tests pour procédures stockées et triggers SQL (UC 11)

```
Appliquer l'interface RPGUnit confirmée au Prompt 0 :
- Copybook exact : [nom identifié en Prompt 0, section "Vérification de l'environnement RPGUnit"]
- Assertions exactes : [noms confirmés en Prompt 0 — ne pas substituer]
- Hooks disponibles : [noms et disponibilité confirmés en Prompt 0, section "Interface de cycle de vie"]
- Type d'objet produit par la compilation : [*PGM / *SRVPGM selon Prompt 0]
- Stratégie d'isolation : [SAVEPOINT/ROLLBACK si journalisation + même périmètre CC / DELETE par clé de corrélation sinon]
Ne pas substituer une interface RPGUnit connue à celle identifiée au Prompt 0.

Sur la base des livrables SQL de UC 11 ([NOM_FICHIER_PROCEDURE_SQL] et [NOM_FICHIER_OBJET_SQL])
pour la procédure stockée / trigger [NOM_OBJET_SQL] dans [NOM_LIB],
[Si nouvelle session : charger *-procedure-sql-*.md et *-objet-sql-*.md dans le contexte]

Vérifier préalablement que l'objet existe :
SELECT ROUTINE_NAME FROM QSYS2.SYSROUTINES WHERE ROUTINE_SCHEMA = '[NOM_LIB]'

Génère en FREE RPG (ILE) avec SQL embarqué un programme de test RPGUnit couvrant les cas de test [LISTE_CAS].

Le programme de test doit :

1. En-tête
- Inclure le copybook identifié en Prompt 0 (ne pas supposer le nom)
- Variables hôte (DCL-S) pour les paramètres IN/OUT de la procédure

2. Hook avant chaque test (nom confirmé en Prompt 0)
- Insérer les données de test dans [NOM_LIB_TEST] (INSERT)
- Si journalisation + même périmètre de commitment control : définir un SAVEPOINT
- Sinon : initialiser une clé de corrélation unique (préfixe horodaté)
- Initialiser les paramètres de la procédure

3. Procédures de test (une procédure de test par scénario, selon la convention de nommage des tests confirmée depuis l'installation RPGUnit)
Pour chaque scénario :
- Cas nominal : CALL de la procédure / déclenchement du trigger → vérification SQLCODE = 0 + assertions sur les sorties
- Cas d'erreur attendu : données invalides → vérification SQLCODE attendu (ex. -803 pour doublon)
- Assertions avec les noms exacts identifiés en Prompt 0

4. Hook après chaque test (nom confirmé en Prompt 0)
- Stratégie selon le contexte identifié en Prompt 0 :
  a) SAVEPOINT disponible ET procédure testée sans COMMIT propre → ROLLBACK TO SAVEPOINT
  b) Sinon → DELETE par clé de corrélation unique sur toutes les tables affectées

5. Cas spécifiques aux triggers
- Vérifier que le trigger s'est déclenché (interroger la table de log ou les colonnes calculées)
- Vérifier qu'il ne se déclenche pas dans les cas exclus

Ne pas inventer de paramètres ou de comportements non définis dans les livrables UC 11.
Signaler si QSYS2.SYSROUTINES ne retourne pas l'objet — ce prérequis doit être résolu avant de générer les tests.
```

**Analyse ligne à ligne :**

- `Variables hôte pour les paramètres IN/OUT` → les variables hôtes RPGLE (préfixées `:`) sont la seule façon de passer des paramètres à une procédure SQL depuis RPGLE. Bob doit les déclarer explicitement pour que le programme compile.

- `Cas d'erreur attendu : vérification SQLCODE attendu` → **validation des cas négatifs**. Une procédure robuste doit rejeter les données invalides avec le bon SQLCODE. Tester uniquement les cas nominaux donne une fausse confiance.

- `Vérifier que le trigger s'est déclenché` → les triggers sont particulièrement difficiles à tester car ils ne retournent rien directement. La stratégie consiste à vérifier l'effet de bord (colonne calculée, table de log, compteur).

> 💡 **Prérequis UC 11 :** le Prompt 2 nécessite que les procédures stockées et triggers aient été créés sur l'IBM i de test (UC 11 complété). Vérifier via IBM i Database MCP que les objets existent : `SELECT ROUTINE_NAME FROM QSYS2.SYSROUTINES WHERE ROUTINE_SCHEMA = '[NOM_LIB]'`.

---

### Prompt 3 — Plan de tests pour un périmètre applicatif complet

```
Sur la base des livrables de modernisation pour l'application
[NOM_APPLICATION] dans [NOM_LIB] (fichiers *-diff-restr-*.md, *-rpg-converti-*.md,
*-diff-optim-*.md, *-programme-genere-*.md, *-objet-sql-*.md disponibles),
génère un plan de tests en français, en markdown.

## Plan de tests UC 13 — [NOM_APPLICATION]

### 1. Périmètre des programmes à tester
| Programme | UC de modernisation | Nb cas de test estimés | Catégorie (S/St/C) | Prérequis données de test | Ordre recommandé |
Catégorie : S = SIMPLE / St = STANDARD / C = COMPLEXE (critères UC 13)
Ordre : les programmes sans dépendances d'appel en premier — les programmes appelants en dernier

### 2. Programmes à reporter ou exclure
Les programmes pour lesquels les données de test ne sont pas disponibles, ou dont la
logique de test dépasse le périmètre du POC — avec la justification.

### 3. Dépendances entre tests
Les programmes qui s'appellent mutuellement : l'ordre dans lequel les tester pour ne
pas masquer une régression dans un programme avec une erreur dans son appelant.

### 4. Stratégie de données de test
Pour chaque famille de données nécessaires :
| Donnée de test | Programme(s) concerné(s) | Source (existante en lib test / à créer par SQL INSERT) | Sensibilité (données de production à anonymiser) |

### 5. Estimation d'effort
| Programme | Prompts nécessaires | Cas de test couverts | Priorité |

Ne pas inventer de programmes ou de comportements non visibles dans les sources disponibles.
Signaler les cas où les livrables de modernisation sont insuffisants pour planifier les tests.
```

**Analyse ligne à ligne :**

- `fichiers *-diff-restr-*.md, *-rpg-converti-*.md... disponibles` → **prérequis explicites**. Le plan de tests est directement piloté par les livrables des UC précédents — chaque type de diff correspond à un type de test.

- `les programmes sans dépendances d'appel en premier` → **ordre de test fondé sur les dépendances**. Si le programme A appelle le programme B et que B a une régression, les tests de A échoueront aussi — ce qui masque l'origine réelle. Tester de bas en haut dans la pile d'appels.

- `Sensibilité (données de production à anonymiser)` → **protection des données**. Sur un IBM i de production, les tables de test peuvent être des copies de tables réelles contenant des données clients. Il faut identifier ces cas avant de les utiliser dans des tests.

> ⚠️ **Piège évité :** sans la colonne "Prérequis données de test", l'équipe commence la génération de tests et réalise en cours de session que les données nécessaires n'existent pas dans la bibliothèque de test — ce qui bloque l'exécution.

---

### Prompt 1-bis — Test de compilation du programme de test RPGUnit

> **Mode : Agent — à exécuter après le Prompt 1 et avant le Prompt 4.**
> Séparation obligatoire : compilation seule d'abord, exécution ensuite.

```
Le programme de test RPGUnit [NOM_PROGRAMME]T a été généré dans le cadre du UC 13.
La commande de compilation à utiliser est [CRTBNDRPG / RUCRTRPG selon Prompt 0].

Crée le membre source dans [NOM_LIB_TEST]/QTESTSRC, membre [NOM_PROGRAMME]T,
avec le contenu du programme de test que nous venons de générer.
(Ne pas utiliser QRPGSRC du programme source — QTESTSRC est le fichier source dédié aux tests)

Lance uniquement la compilation — ne pas exécuter RUCALLTST :
[SI CRTBNDRPG]
CRTBNDRPG PGM([NOM_LIB_TEST]/[NOM_PROGRAMME]T)
          SRCFILE([NOM_LIB_TEST]/QTESTSRC)
          SRCMBR([NOM_PROGRAMME]T)
          OPTION(*EVENTF *LIST)
          DBGVIEW(*SOURCE)
          DFTACTGRP(*NO)
          ACTGRP(RPGUNIT)

[SI RUCRTRPG — iRPGUnit]
RUCRTRPG TSTPGM([NOM_LIB_TEST]/[NOM_PROGRAMME]T)
         SRCFILE([NOM_LIB_TEST]/QTESTSRC)
         SRCMBR([NOM_PROGRAMME]T)

Analyse le résultat et produis en français :
1. Statut : COMPILATION RÉUSSIE / ERREURS DE COMPILATION
2. Si erreurs :
   | Ligne | Code erreur | Description | Cause probable |
   Causes probables à vérifier :
   - Copybook non trouvé → vérifier le nom exact identifié en Prompt 0 et la Library List
   - Prototype incorrect (DCL-PR) → vérifier les paramètres dans la spec-tech UC 6
   - Nom d'assertion invalide → utiliser uniquement les noms confirmés en Prompt 0
   - Variable hôte SQL non déclarée → ajouter la DCL-S manquante
3. Si compilation réussie : confirmer le type d'objet créé (*PGM ou *SRVPGM) et la bibliothèque
4. Rappeler que la compilation réussie ne garantit pas le comportement des assertions —
   la revue du programme de test par un développeur reste obligatoire avant le Prompt 4
```

**Analyse ligne à ligne :**

- `QTESTSRC` → **fichier source dédié aux tests, distinct de QRPGSRC**. Utiliser `CPYSRCF` depuis le source du programme testé pour initialiser un membre de test est une erreur de conception : si Bob copie le source du programme et le modifie, un glissement de la cible est possible. `QTESTSRC` isole physiquement les sources de test des sources de production.

- `[SI CRTBNDRPG] / [SI RUCRTRPG]` → **les deux branches sont présentes**. La version installée de RPGUnit détermine laquelle s'applique — le Prompt 1-bis utilise ce qui a été confirmé au Prompt 0, pas ce qui est supposé.

- `Ne pas exécuter RUCALLTST` → **séparation des étapes**. Si une table de test référencée dans le SETUP n'existe pas, l'exécution lève une exception SQL avant d'atteindre les assertions — ce qui masque une simple erreur de compilation.

> 💡 **Pour les programmes SIMPLE**, le Prompt 4 peut enchaîner compilation + exécution. Pour STANDARD et COMPLEXE, le Prompt 1-bis est toujours recommandé.

---

### Prompt 4 — Exécution des tests et rapport de résultats

```
Le programme de test RPGUnit [NOM_PROGRAMME]T est compilé dans [NOM_LIB_TEST].
[Si reprise : charger le fichier *-programme-test-*.md dans le contexte]

Étape A — Avant d'exécuter les tests, mémoriser l'état courant :
VALUES QSYS2.JOB_NAME;
-- QSYS2.JOB_NAME est une variable globale intégrée (sans parenthèses) — conserver pour la corrélation.
VALUES CURRENT_TIMESTAMP;
-- Conserver cet horodatage : il servira à filtrer les spool files créés APRÈS l'exécution.

Étape B — Exécuter les tests RPGUnit :
RUCALLTST TSTPGM([NOM_LIB_TEST]/[NOM_PROGRAMME]T)
          OUTPUT(*SYSOUT)
          DETAIL(*ALL)

Étape C — Identifier les spool files créés par CE job APRÈS l'horodatage mémorisé :
SELECT SPOOLED_FILE_NAME, FILE_NUMBER, CREATE_TIMESTAMP
FROM QSYS2.OUTPUT_QUEUE_ENTRIES
WHERE JOB_NAME = '[NOM_JOB_QUALIFIÉ]'    -- numéro/utilisateur/nom exact mémorisé à l'étape A
  AND CREATE_TIMESTAMP >= [HORODATAGE_AVANT_TEST]
ORDER BY CREATE_TIMESTAMP

Le nom du spool file dépend de la version RPGUnit installée — il n'est pas nécessairement QPRINT.
Lire tous les spool files candidats et identifier celui contenant le rapport RPGUnit
(présence de "PASSED", "FAILED", "ERROR" dans le contenu).

Étape D — Lire le contenu du spool identifié :
SELECT * FROM TABLE(SYSTOOLS.SPOOLED_FILE_DATA(
  JOB_NAME             => '[NOM_JOB_QUALIFIÉ]',
  SPOOLED_FILE_NAME    => '[NOM_SPOOL_IDENTIFIÉ]',
  SPOOLED_FILE_NUMBER  => [FILE_NUMBER]
)) AS T

Analyse le contenu et produis en français :

1. Statut global : [N] cas de test PASSED / [N] FAILED / [N] ERROR
2. Tableau des résultats :
   | Cas de test | Statut | Message d'assertion | Valeur obtenue vs attendue | Cause probable |
3. Pour chaque FAILED :
   - Valeur obtenue vs valeur attendue
   - Référence à la règle métier concernée (fichier *-regles-* ou diff UC [N])
   - Diff proposé à soumettre à revue dans le programme testé — ne pas modifier automatiquement
4. Pour chaque ERROR :
   - Distinguer erreur dans le programme de test (à corriger) / exception dans le programme testé (régression)
5. Conclusion :
   - VALIDÉ — tous les cas de test passent
   - PARTIELLEMENT VALIDÉ — échecs mineurs, aucune régression fonctionnelle critique
   - NON VALIDÉ — régression détectée, retour à UC [N_UC_SOURCE]

Ne modifier ni le programme testé ni le programme de test dans cette session.
Rappeler que PASSED ne garantit pas la non-régression sur les cas non couverts.
```

**Analyse ligne à ligne :**

- `Étape A avant Étape B` → **ordre obligatoire**. `VALUES QSYS2.JOB_NAME` et `VALUES CURRENT_TIMESTAMP` doivent être exécutés *avant* `RUCALLTST` — sinon il est impossible de distinguer les spool files de cette exécution des précédentes. L'horodatage est la clé de filtrage ; le JOB_NAME qualifié est la clé de corrélation.

- `CREATE_TIMESTAMP >= horodatage` → **filtre temporel à la place du nom fixe**. Le nom du spool file dépend de la version RPGUnit installée et du printer file utilisé par le programme de test — il n'est pas toujours `QPRINT`. Filtrer par horodatage puis identifier le spool contenant le rapport RPGUnit par son contenu est l'approche neutre vis-à-vis de la version.

- `SYSTOOLS.SPOOLED_FILE_DATA` → **lecture réelle du contenu spool**. `QSYS2.OUTPUT_QUEUE_ENTRIES` ne contient que les métadonnées (nom, numéro, date). `SYSTOOLS.SPOOLED_FILE_DATA` retourne le texte ligne à ligne — c'est la seule façon d'extraire le rapport RPGUnit via SQL sans naviguer dans WRKSPLF.

- `Diff proposé à soumettre à revue — ne pas modifier automatiquement` → **protection contre la modification involontaire**. Le Prompt 4 analyse des résultats, il ne corrige pas. Un `FAILED` déclenche un retour au UC de modernisation source, pas une modification automatique par Bob.

> ⚠️ **Piège évité :** `LIKE '%[NOM]T%' FETCH FIRST 1 ROW ONLY` sans job qualifié peut retourner le spool d'une exécution précédente, d'un autre utilisateur, ou d'un autre programme de test au nom similaire — le rapport analysé correspond à la mauvaise exécution.

---

### Prompt 5 — Automatisation de la suite de tests (script CL de régression)

```
Sur la base du plan de tests [NOM_FICHIER_PLAN_TEST]
pour l'application [NOM_APPLICATION] dans [NOM_LIB],
[Si nouvelle session : charger le fichier *-plan-test-*.md dans le contexte]

génère en français, en markdown, et en CL IBM i :

## 1. Programme CL d'exécution de la suite de tests

Génère un programme CL nommé [NOM_APPLICATION]TST (ex. APPVTETST) qui :
- Pour chaque programme de test du périmètre ([NOM_PROGRAMME1]T, [NOM_PROGRAMME2]T...) :
  RUCALLTST TSTPGM([NOM_LIB_TEST]/[NOM_PROGRAMME_N]T)
            OUTPUT(*SYSOUT)
            DETAIL(*ALL)
- Produit un fichier de log de synthèse via DSPJOBLOG OUTPUT(*PRINT)
Ne pas inclure DLTSPLF — la corrélation des spool files se fait par JOB_NAME qualifié (voir section 3).

## 2. Soumission batch et mémorisation du job

Génère la commande SBMJOB pour exécuter le programme CL de suite en batch :
SBMJOB JOB([NOM_APPLICATION]TST)
       JOBD(QBATCH)
       CMD(CALL PGM([NOM_LIB_TEST]/[NOM_APPLICATION]TST))
       OUTQ([NOM_OUTQ:File d'attente de sortie (ex. QPRINT)])
       MSGQ(*USRPRF)
       LOG(4 00 *SECLVL)

Mémoriser l'horodatage de soumission et identifier le JOB_NAME qualifié du job soumis :
VALUES CURRENT_TIMESTAMP;
-- Conserver cet horodatage — il servira à filtrer les spool files créés après l'exécution.

SELECT JOB_NAME, JOB_STATUS FROM TABLE(QSYS2.JOB_INFO())
WHERE JOB_NAME LIKE '%[NOM_APPLICATION]TST%'
ORDER BY JOB_ENTERED_SYSTEM_TIME DESC
FETCH FIRST 1 ROW ONLY
-- Conserver le JOB_NAME qualifié (numéro/utilisateur/nom) pour la corrélation des spool files.

## 3. Suivi du job et lecture déterministe du rapport

Polling jusqu'à l'état terminal (timeout recommandé : 30 min) :
SELECT JOB_STATUS, COMPLETION_STATUS FROM TABLE(QSYS2.JOB_INFO())
WHERE JOB_NAME = '[NOM_JOB_QUALIFIÉ]'
-- JOB_STATUS actif : ACTIVE, JOBQ
-- JOB_STATUS terminal : OUTQ (spool disponible)
-- COMPLETION_STATUS terminal : NORMAL (succès), ABNORMAL (échec)
-- Répéter jusqu'à JOB_STATUS = 'OUTQ' ou COMPLETION_STATUS IS NOT NULL

Identifier les spool files créés par CE job APRÈS l'horodatage mémorisé :
SELECT SPOOLED_FILE_NAME, FILE_NUMBER, CREATE_TIMESTAMP
FROM QSYS2.OUTPUT_QUEUE_ENTRIES
WHERE JOB_NAME = '[NOM_JOB_QUALIFIÉ]'
  AND CREATE_TIMESTAMP >= [HORODATAGE_AVANT_SBMJOB]
ORDER BY CREATE_TIMESTAMP

Le nom du spool file dépend de la version RPGUnit installée — ne pas supposer QPRINT.
Lire chaque candidat et identifier ceux contenant les rapports RPGUnit
(présence de "PASSED", "FAILED", "ERROR" dans le contenu).

Lire le contenu avec SYSTOOLS.SPOOLED_FILE_DATA pour chaque FILE_NUMBER candidat :
SELECT * FROM TABLE(SYSTOOLS.SPOOLED_FILE_DATA(
  JOB_NAME             => '[NOM_JOB_QUALIFIÉ]',
  SPOOLED_FILE_NAME    => '[NOM_SPOOL_IDENTIFIÉ]',
  SPOOLED_FILE_NUMBER  => [FILE_NUMBER]
)) AS T

Produire un tableau récapitulatif :
| Programme de test | PASSED | FAILED | ERROR | Conclusion |
Conclure si la suite globale est VALIDÉE (0 FAILED, 0 ERROR critiques) ou NON VALIDÉE.

## 4. Planning de régression recommandé

Sur la base du plan de tests, indiquer :
- À quelle fréquence exécuter la suite complète (après chaque session UC 7/8/1/2, ou avant UC 16)
- Quels programmes de test sont prioritaires si la suite complète dépasse 30 minutes
- Comment intégrer cette suite dans une procédure de déploiement manuelle (sans pipeline ARCAD)

Ne pas inventer de programmes de test non listés dans le plan de tests.
Signaler les programmes interactifs 5250 qui ne peuvent pas être inclus dans la suite batch.
```

**Analyse ligne à ligne :**

- `SBMJOB avec LOG(4 00 *SECLVL)` → **joblog complet**. En cas d'échec d'un test ou de message d'échappement non capturé, le joblog détaillé est la seule source d'information disponible. Sans ce paramètre, le diagnostic post-mortem d'une suite batch est aveugle.

- `QSYS2.JOB_INFO` pour suivre le job → **polling déterministe**. Interroger `JOB_STATUS` et `COMPLETION_STATUS` en boucle avec un timeout explicite évite d'attendre indéfiniment ou de lire un spool avant que le job ne soit terminé. `JOB_STATUS` prend les valeurs `ACTIVE`, `JOBQ`, `OUTQ` ; `COMPLETION_STATUS` prend `NORMAL` ou `ABNORMAL` en fin d'exécution — `COMPLETED NORMALLY` et `COMPLETED ABNORMALLY` ne sont pas des valeurs valides de `JOB_STATUS`.

- `SYSTOOLS.SPOOLED_FILE_DATA` pour lire le contenu → **même mécanisme qu'au Prompt 4**. `QSYS2.OUTPUT_QUEUE_ENTRIES` identifie l'entrée ; `SYSTOOLS.SPOOLED_FILE_DATA` lit le texte. La corrélation se fait par `JOB_NAME` qualifié (job/user/numéro), pas par "le dernier spool".

- `Comment intégrer cette suite dans une procédure de déploiement manuelle (sans pipeline ARCAD)` → **adaptation à la contrainte MCP ARCAD absent**. Le script CL généré devient la procédure de régression manuelle avant chaque déploiement — son exécution est documentée dans le manifest de traçabilité UC 13.

> 💡 **Lien avec UC 16 :** le programme CL de suite de tests généré par le Prompt 5 est le précurseur du pipeline UC 16. Si une instance ARCAD compatible est disponible pour UC 16, ce script CL devient l'étape "Run Tests" du pipeline.

> ⚠️ **Limitation importante :** les programmes interactifs 5250 (ceux avec `EXFMT`) ne peuvent pas être inclus dans une suite batch RPGUnit. Signaler ces programmes dans la section 4 du Prompt 5 — ils nécessitent des tests manuels séparés ou une refactorisation préalable (UC 8).

---

### Note — Enchaîner les prompts en conversation continue

UC 13 se pratique idéalement en **conversation continue** pour les programmes SIMPLE et STANDARD : Prompt 0 → Prompt 1 → Prompt 1-bis → Prompt 4 dans la même conversation, en restant en Ask jusqu'à la fin du Prompt 1, puis en passant en Agent pour le Prompt 1-bis et le Prompt 4.

```
[Après la réponse au Prompt 0]
"Les cas de test 1, 2 et 3 sont les plus critiques — commence par générer les TESTCASE
 pour ces trois cas dans le Prompt 1."

[Après la réponse au Prompt 1]
"Le cas de test 2 (testClientInconnu) doit aussi couvrir le cas où le client existe
 mais est bloqué — ajoute ce scénario."

[Prompt 1-bis — en Agent — exécute la compilation sur l'IBM i]
"Le programme de test est finalisé. Compile-le uniquement (Prompt 1-bis) — ne pas exécuter."

[Après compilation réussie — passer en Agent pour Prompt 4]
"La compilation est réussie. Lance maintenant l'exécution RPGUnit (Prompt 4)."
```

> 💡 Bob conserve le contexte du programme testé pendant toute la conversation. Exploiter cet historique évite de re-soumettre le source ou les fiches de qualification entre chaque prompt.

> 💡 **Sauvegarder le rapport de test** : après le Prompt 4, sauvegarder le rapport via le prompt **save-fiche** avec le type `rapport-test`. Ce fichier devient l'input de UC 16.

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Connexion IBM i, Object Browser, ouverture des membres sources |
| **IBM i Languages** | Coloration syntaxique RPGLE — indispensable pour lire et vérifier le programme de test généré |
| **Markdown All in One** | Prévisualisation des rapports de test générés en markdown |

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture du programme modernisé à tester, création du membre de test dans `QTESTSRC`, exécution de la commande de compilation confirmée en P0 (`CRTBNDRPG` ou `RUCRTRPG` selon la version installée) et `RUCALLTST`, lecture du rapport spooler via `SYSTOOLS.SPOOLED_FILE_DATA` |
| **IBM i Database MCP** | Vérification de l'existence des tables et procédures dans QSYS2, création/suppression des données de test via SQL |
| **Confluence MCP** *(si disponible)* | Publication des rapports de test sur l'espace POC Confluence |

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Confondre test de compilation et test fonctionnel | Un programme qui compile ne garantit pas qu'il produit les bons résultats — la compilation UC 7/8 a déjà été validée ; UC 13 teste le comportement | Toujours exécuter `RUCALLTST` et analyser les assertions — ne pas s'arrêter à un `CRTBNDRPG` réussi |
| Sauter le Prompt 1-bis sur un programme STANDARD ou COMPLEXE | Le Prompt 4 enchaîne compilation + exécution — si la compilation échoue à cause d'un prototype incorrect, le diagnostic est noyé dans les erreurs d'exécution | Toujours passer par le Prompt 1-bis (compilation seule) avant le Prompt 4 sur les programmes STANDARD et COMPLEXE |
| Tester avec des données de production | Risque d'effets de bord sur les tables réelles si le programme modifie des données | Toujours travailler dans une bibliothèque de test dédiée (`APPVTETEST`) — jamais dans la bibliothèque de production |
| Générer des tests sans fiche de qualification (P0) | Le programme de test couvre des cas de test arbitraires, pas les comportements critiques du programme | Toujours commencer par le Prompt 0 pour identifier les vrais cas de test à couvrir |
| Utiliser `DFTACTGRP(*YES)` lors de la compilation | Le programme de test compile mais RPGUnit ne peut pas exécuter les TESTCASE correctement — valable uniquement pour CRTBNDRPG, pas pour RUCRTRPG | Utiliser `DFTACTGRP(*NO) ACTGRP(RPGUNIT)` avec CRTBNDRPG ; avec RUCRTRPG, appliquer les paramètres confirmés en Prompt 0 |
| Regrouper plusieurs assertions dans une seule procédure TESTCASE | Si la première assertion échoue, les assertions suivantes ne sont pas exécutées — couverture masquée | Une procédure TESTCASE = un seul cas de test ; plusieurs assertions possibles si elles sont liées |
| Tester le programme appelant avant de tester ses dépendances | Une erreur dans un programme appelé fait échoue les tests du programme appelant — l'origine de la régression est masquée | Respecter l'ordre du plan de tests (Prompt 3) — tester de bas en haut dans la pile d'appels |
| Tenter de tester un programme interactif 5250 en batch | RPGUnit ne peut pas exécuter un programme avec `EXFMT` — il attend une entrée écran qui n'arrive jamais | Vérifier dans le Prompt 0 si le programme est interactif ; si oui, tester uniquement les procédures extraites (UC 8 requis en amont) |
| Accepter un résultat PASSED sans vérifier la couverture | Un test qui passe avec 2 cas de test ne prouve pas la non-régression si le programme a 20 cas | Toujours confronter le rapport (Prompt 4) au plan de tests (Prompt 3) — vérifier la couverture |
| Oublier le TEARDOWN / hook après chaque test | Les données de test s'accumulent dans la bibliothèque de test et faussent les exécutions suivantes | Toujours inclure un hook après chaque test avec la stratégie confirmée en P0 (ROLLBACK TO SAVEPOINT ou DELETE par clé de corrélation) |
| Corréler le spool par LIKE ou par position sans JOB_NAME qualifié | IBM i MCP lit le spool d'une exécution précédente ou d'un autre utilisateur — le rapport analysé est incorrect | Mémoriser le JOB_NAME qualifié avant l'exécution (étape A du Prompt 4 / section 2 du Prompt 5) et l'utiliser comme clé exacte — ne pas utiliser DLTSPLF pour masquer le problème |

---

## Check-list de validation UC 13

Avant de passer à UC 16 (DevOps), valider chaque point :

- [ ] **Version et interface RPGUnit confirmées** — copybook, assertions, hooks, commande de compilation et type d'objet identifiés depuis l'installation réelle (Prompt 0 sans aucun "À VÉRIFIER" restant)
- [ ] **Stratégie d'isolation validée** — SAVEPOINT/ROLLBACK ou DELETE par clé de corrélation, selon journalisation et périmètre de commitment control confirmés en Prompt 0
- [ ] Un plan de tests (Prompt 3) a été produit pour le périmètre complet des programmes modernisés
- [ ] Chaque programme modernisé (UC 7-8-1-2-9 au minimum) a un programme de test RPGUnit généré et compilé sans erreur dans `QTESTSRC`
- [ ] Les tests RPGUnit ont été exécutés (`RUCALLTST`) et les résultats analysés — au moins 1 rapport `*-rapport-test-*.md` par programme testé
- [ ] **Manifest de traçabilité complété** dans chaque rapport — OBJCREATED, CHANGE_TIMESTAMP, SOURCE_TIMESTAMP, SOURCE_FILE/LIBRARY/MEMBER renseignés ; corrélation spool vérifiée par JOB_NAME qualifié
- [ ] Toutes les régressions détectées (cas FAILED ou ERROR) ont fait l'objet d'une correction dans le programme source avant de passer à UC 16
- [ ] Les programmes de test RPGUnit créés dans la bibliothèque de test ont été notés pour réintégration manuelle dans ARCAD (si ACME souhaite les versionner)
- [ ] Les rapports de test sont sauvegardés avec la convention de nommage `*-rapport-test-{YYYYMMDD-HHmm}.md` dans le workspace ET publiés sur Confluence (si MCP disponible)

---

*Fiche UC 13 — Document évolutif à mettre à jour au fil du POC.*
