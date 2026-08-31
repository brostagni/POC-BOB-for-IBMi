# UC 2 — Conversion RPG Colonné / RPG III → FREE RPG ILE

> **Catégorie :** Modernisation du code
>
> **Priorité dans le POC :** 10b — suit UC 1 (COBOL → RPG), peut être mené en parallèle avec UC 1 sur des programmes différents ; à effectuer après UC 7 si l'optimisation a déjà été appliquée sur le même programme
>
> **Durée POC (avec Bob) :** 2 à 4 heures — conversion d'un périmètre représentatif (3 à 5 programmes RPG IV colonné, ou 1 à 2 programmes RPG III avec cycle), itérations et tests de non-régression
>
> **Durée PROD (avec Bob) :** 30 min à 2 heures / programme — qualification, génération du diff de conversion, revue développeur, test fonctionnel
>
> **Durée PROD (sans Bob) :** 1 à 3 jours / programme — lecture ligne à ligne du source colonné, mapping manuel des colonnes, remplacement opcode par opcode, tests de non-régression ; plus long pour RPG III (cycle RPG, MOVE/MOVEL, absence de sous-procédures)
>
> **Gain Bob estimé :** ~4× — un programme de 400 lignes RPG IV colonné converti en une demi-journée au lieu de 2 jours ; gain plus fort sur les programmes avec indicateurs omniprésents et opcodes obsolètes nombreux (RPG III)
>
> **Mode Bob recommandé :** IBM i Developer (mode Ask pour la qualification et la génération du diff, Agent pour la compilation de test et la sauvegarde)

---

## Objectif

Convertir le style de programmation RPG **colonné** (format fixe) ou **RPG III** vers du **RPG ILE Free** moderne, sans modifier la logique fonctionnelle du programme.

**Ce UC diffère de UC 7 (optimisation) et de UC 8 (restructuration) :**
- UC 7 améliore la lisibilité interne du code (renommages, extraction de procédures) sans changer la syntaxe de base. Un programme RPG IV colonné "optimisé" par UC 7 reste colonné.
- UC 8 modifie la structure du programme (extraction de modules, conversion de cycle). La syntaxe reste celle du source d'origine.
- UC 2 change la **syntaxe** du programme : les colonnes, les opcodes de format fixe et les specs C/D/F/H deviennent du Free RPG. La logique fonctionnelle, elle, doit rester identique.

**Deux sous-cas couverts dans cette fiche :**

**Sous-cas A — RPG IV colonné → FREE :** cas le plus courant. Le programme utilise les specs de format fixe (H-spec, F-spec, D-spec, C-spec colonné) mais la logique est déjà procédurale (pas de cycle RPG actif). Conversion directe par équivalences syntaxiques.

**Sous-cas B — RPG III simple ou préconverti → FREE :** cas plus délicat. Le programme n'a pas de sous-procédures `DCL-PROC`, utilise MOVE/MOVEL/SETON, et peut dépendre du cycle RPG actif.

> ⚠️ **Périmètre RPG III dans ce UC :**
> - RPG III **simple** (sans E-specs, I-specs ou O-specs programme-décrites significatives, sans cycle multi-niveaux) : couvert par cette fiche.
> - RPG III **dense** (E/I/O-specs, cycle avec ruptures L1-L9, arrays de compilation, totaux de niveaux) : `COMPLEXE — hors périmètre de conversion complète`. Appliquer `CVTRPGSRC` en préalable lorsque la commande est disponible sur l'IBM i de test ; sinon, limiter l'atelier à l'analyse et à la conversion des C-specs explicitement délimitées, sans prétendre produire un programme RPG ILE Free complet.
> - Un programme RPG III avec E/I/O-specs non converties **ne peut pas être déclaré "converti en RPG Free"** même si les C-specs compilent.

> 💡 **P-specs `PB`/`PE` :** ces specs sont des spécifications de procédures **RPG IV ILE**, pas du RPG III OPM. Leur conversion en `DCL-PROC`/`END-PROC` est couverte dans le Prompt 4b. Un vrai source RPG III ne contient pas de `PB`/`PE` — si vous en voyez, le programme est en RPG IV fixe (Sous-cas A avec procédures).

**L'ordre recommandé sur un même programme :** UC 7 d'abord (renommages, opcodes obsolètes), puis UC 2 (conversion syntaxique). Convertir un programme avec des variables cryptiques et des GOTO produit du Free RPG illisible.

**Livrable attendu :** pour chaque programme converti, un fichier de diff documentant chaque transformation (spec par spec, opcode par opcode), et optionnellement le source RPG Free complet si le programme est court.

**Convention de nommage des fichiers générés :**
```
{projet}-{lib}-{programme}-{type}-{YYYYMMDD-HHmm}.md

Types pour cet UC :
  qualification-conv  → fiche de qualification (Prompt 0) — catégorie + sous-cas + séquence
  rpg-converti        → diff de conversion ou source RPG Free complet (Prompts 1 à 4)
  plan-conv-rpg       → plan de conversion pour un périmètre applicatif complet (Prompt 5)
```

Exemples :
```
acme-APPVTE-GESCMD-qualification-conv-20250621-0900.md   ← qualification + stratégie
acme-APPVTE-GESCMD-rpg-converti-20250621-1100.md         ← diff ou source converti
acme-APPVTE-APPVTE-plan-conv-rpg-20250621-0800.md        ← plan périmètre complet
```

> 💡 Cette convention est valable en dehors du contexte POC — réutilisable en production tel quel.

---

## Démarrer par un programme que vous connaissez

> **Recommandation forte avant d'aborder les programmes critiques de ACME.**

Commencer par un **programme dont un développeur de l'équipe connaît bien la logique** — idéalement un programme batch sans cycle RPG, dont le comportement attendu peut être vérifié rapidement avec un jeu de tests existant.

Pourquoi ? Parce que la première conversion sert à **calibrer deux choses** :
- La qualité des équivalences proposées par Bob (est-ce que `MOVE src dest` a bien été converti en `EVAL dest = src` avec vérification des longueurs ?)
- La capacité de l'équipe à valider la non-régression (sur ce programme simple, les tests sont rapides — si Bob a introduit une régression, elle est détectée avant d'aller sur les programmes critiques)

> 💡 Pour chaque programme de cette progression, **commencer par le Prompt 0** — il détermine le sous-cas (A ou B) et la catégorie (SIMPLE / STANDARD / COMPLEXE), puis recommande la séquence exacte à suivre. Ne pas aller directement au Prompt 1.

| Étape | Programme à choisir | Catégorie attendue | Objectif |
|-------|--------------------|--------------------|---------|
| 1 | Programme RPG IV colonné court (< 200 lignes), pas de cycle RPG, pas d'indicateurs applicatifs | SIMPLE — Sous-cas A | Calibrer la conversion des specs et des opcodes de base ; valider que le Prompt 0 classe correctement |
| 2 | Programme RPG IV avec indicateurs (*INxx) et quelques opcodes MOVE/MOVEL | STANDARD — Sous-cas A | Valider la conversion des indicateurs en variables booléennes et les cas MOVE avec longueurs différentes |
| 3 | Programme RPG III sans cycle RPG actif — présence de MOVE, MOVEL, pas de DCL-PROC | STANDARD — Sous-cas B | Valider les équivalences RPG III → Free ; un vrai RPG III n'a pas de D-specs RPG IV — valider la conversion des déclarations produites par CVTRPGSRC ou des C-specs legacy |
| 4 | Programme RPG III avec cycle RPG actif (niveaux de contrôle L1-L9) | COMPLEXE — Sous-cas B | Valider la conversion du cycle (renvoi vers la technique UC 8 Prompt 4) et les totaux de rupture |

---

## Impact de la taille du programme sur la stratégie de conversion

La complexité d'UC 2 ne se mesure pas en nombre de lignes, mais en **nombre d'opcodes de format fixe restants**, au **niveau d'utilisation des indicateurs (*INxx)**, à la **présence du cycle RPG actif**, et au **sous-cas** (RPG IV ou RPG III).

Les vrais facteurs qui compliquent la conversion :
- Les **indicateurs (*INxx) utilisés pour piloter la logique applicative** : un indicateur activé par un opcode (`SETON *IN50`) et testé dans une condition (`IF *IN50`) doit être converti en variable booléenne `IND`. Leur inventaire et la décision de cible doivent être faits avant la conversion des C-specs (Prompt 3), mais leur remplacement effectif est réalisé au Prompt 4 — après génération des C-specs. Jusqu'au Prompt 4, les `SETON`/`SETOF` concernés restent explicitement marqués comme non convertis ; aucune compilation intermédiaire n'est lancée pour les programmes STANDARD ou COMPLEXE
- Les **colonnes d'opérande de format fixe avec des espaces significatifs** : certains opcodes RPG IV fixe utilisent les positions de colonne pour les facteurs 1 et 2 — une mauvaise lecture de colonne par Bob produit un mapping de variables incorrect
- Le **cycle RPG actif** (`*INLR`, niveaux de contrôle `L1-L9`) en RPG III : le cycle est un mécanisme implicite sans équivalent direct en Free RPG — sa conversion nécessite le Prompt 4 de UC 8 et dépasse la simple conversion syntaxique
- Les **MOVE / MOVEL avec des champs de longueurs différentes** : en RPG fixe, `MOVE` tronque silencieusement à droite et `MOVEL` aligne à gauche — en Free RPG, `EVAL` affecte en entier. Cette différence sémantique est la source de régression la plus fréquente en conversion RPG
- Les **P-specs RPG IV (PB/PE)** délimitant des procédures ILE : les P-specs `PB`/`PE` sont du RPG IV ILE (pas du RPG III), elles définissent des procédures avec interface `DCL-PI`, remplacées en Free RPG par `DCL-PROC`/`END-PROC`. Si présentes dans un source classé RPG III, le programme est en réalité un RPG IV fixe avec procédures (Sous-cas A). La conversion est directe sans paramètre, plus complexe avec PARMS.

### Programmes SIMPLE — specs directes, pas d'indicateurs applicatifs, pas de cycle RPG

Bob gère sans difficulté. **Séquence : Prompt 0 → Prompt 1 → Prompt 2 (H/F/D-specs) → Prompt 3 (C-specs) → Prompt 3-bis (compilation) → Prompt 4 (nettoyage).**

> 💡 Le Prompt 0 confirme la catégorie SIMPLE et recommande directement cette séquence — pas de décision manuelle requise. Le Prompt 3-bis est possible après Prompt 3 car un programme SIMPLE n'a pas d'indicateurs applicatifs ni de SETON/SETOF non encore convertis.

### Programmes STANDARD — indicateurs applicatifs, MOVE/MOVEL avec longueurs différentes, ou P-specs RPG IV (avec procédures)

Le Prompt 0 identifie les indicateurs à convertir en variables booléennes et les MOVE à risque. **Séquence : Prompt 0 → Prompt 1 → Prompt 2 → Prompt 3 → Prompt 4 (indicateurs + interfaces) → Prompt 4-bis (compilation).**

> ⚠️ Pour un programme STANDARD, ne pas insérer Prompt 3-bis entre Prompt 3 et Prompt 4. Le Prompt 3 laisse intentionnellement les `SETON`/`SETOF` non convertis jusqu'au Prompt 4 — `SETON` et `SETOF` sont des opcodes de format fixe qui n'ont pas d'équivalent syntaxique en Free RPG (l'affectation `*IN50 = *ON` remplace `SETON *IN50`). Le source ne compilera pas avec des opcodes `SETON`/`SETOF` en mode fully free-form. Compiler après le Prompt 4 uniquement.

> 💡 Sur les programmes STANDARD, inventorier les indicateurs et décider leur cible avant de lancer la conversion des C-specs au Prompt 3. Le remplacement effectif (déclaration `DCL-S … IND`, substitution `SETON`/`SETOF`/`IF`) est réalisé au Prompt 4, après que les C-specs ont été générées. Un indicateur non converti laisse des `SETON`/`SETOF` dans le source — des opcodes de format fixe non valides en fully free-form — et bloque la compilation.

### Programmes COMPLEXE — cycle RPG actif, indicateurs de niveau de contrôle, ou RPG III dense

Un programme COMPLEXE en UC 2 est souvent un programme RPG III avec cycle actif et totaux de niveaux. Le Prompt 0 pose explicitement la question : **la conversion du cycle RPG est-elle dans le périmètre du POC ?** Si oui, utiliser le Prompt 4 de UC 8 *en complément* de cette fiche (renvoi documenté dans le Prompt 4 ci-dessous).

Si la décision est de convertir sans traiter le cycle : **Séquence : Prompt 0 → Prompt 1 → Prompt 2 → Prompt 3 → Prompt 4 (indicateurs + *INLR explicite) → Prompt 4-bis.** Le cycle RPG est conservé avec `*INLR` explicite, mais toutes les autres structures sont converties.

> ⚠️ Sur un programme COMPLEXE, une conversion complète en une seule passe produit un source Free RPG avec des indicateurs de niveau non traités et des MOVE à sémantique ambiguë. Le Prompt 0 découpe toujours le travail par zones de risque.

---

## Démarrer une session Bob

> **À lire avant chaque session UC 2 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

Chaque session de travail sur un programme RPG doit démarrer dans une **nouvelle conversation Bob** (bouton `+` en haut du panneau Chat). Ne pas réutiliser une conversation d'un autre programme ou d'UC 7 — les renommages et décisions d'une autre session polluent les équivalences de la session courante.

**Exception :** si UC 7 vient d'être achevé sur ce même programme dans la même session (pas de fermeture de conversation), il est possible de continuer en UC 2 directement — Bob a encore le source optimisé en contexte. Cette exception ne s'applique qu'au même programme dans la même session.

**Mode à sélectionner :** `IBM i Developer`

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Avant de lancer le Prompt 0, ouvrir dans l'éditeur Bob le programme RPG à convertir. L'ouverture dans l'éditeur le rend accessible au MCP IBM i sans copier-coller.

**Procédure :** dans le panneau **IBM i — Object Browser** (extension Code for IBM i), naviguer jusqu'à la bibliothèque source, faire un clic droit sur le membre RPG → **Open in Editor**.

Fichiers à ouvrir pour chaque session UC 2 :
- Le programme RPG source colonné ou RPG III (`[NOM_LIB]/QRPGSRC([NOM_PROGRAMME])`)
- Le fichier de compréhension du programme (`*-comprehension-*.md`) depuis UC 4
- Si UC 7 a été appliqué : le fichier de diff d'optimisation (`*-diff-optim-*.md`) — source déjà partiellement modernisé

### 3. Fichiers de contexte à charger

Ces fichiers produits par les UC précédents doivent être disponibles dans le workspace Bob **avant** de démarrer. Utiliser **Add File to Chat** (icône trombone) ou les ouvrir dans l'éditeur.

| Fichier | Produit par | Obligatoire / Recommandé |
|---------|-------------|--------------------------|
| `{projet}-{lib}-{programme}-comprehension-{date}.md` | UC 4 | **Obligatoire** — style RPG détecté (IV colonné / III), carte des subroutines et des appels, base du Prompt 1 |
| `{projet}-{lib}-{programme}-diff-optim-{date}.md` | UC 7 | **Si UC 7 appliqué** — source avec variables renommées et opcodes partiellement modernisés ; point de départ de la conversion |
| `{projet}-{lib}-{programme}-spec-tech-{date}.md` | UC 6 | **Recommandé** — interfaces du programme (paramètres, fichiers accédés) ; si les F-specs à convertir touchent les interfaces, la spec technique est le référentiel |
| `{projet}-{lib}-{lib}-matrice-{date}.md` | UC 6 | **Recommandé** — si la conversion des P-specs ou de l'interface modifie le prototype d'appel, la matrice identifie les appelants |
| `{projet}-{lib}-{programme}-qualification-conv-{date}.md` | UC 2 (session précédente) | **Si reprise** — catégorie, sous-cas et stratégie déjà décidés ; évite de relancer le Prompt 0 |

> ⚠️ **Prérequis critique :** ne jamais démarrer UC 2 sans le fichier `*-comprehension-*.md`. La conversion des C-specs sans carte des subroutines peut produire des équivalences incorrectes sur les subroutines qui utilisent des variables globales — Bob ne peut pas distinguer une variable locale d'une variable globale sans cette carte.

> ⚠️ **Risque de réduction de contexte — sauvegarde intermédiaire recommandée :** une session UC 2 avec plusieurs passes de conversion (Prompt 2 → Prompt 3 → Prompt 4) peut atteindre la limite de contexte sur les programmes STANDARD ou COMPLEXE. Si Bob semble oublier qu'un MOVE était "À CONFIRMER" ou qu'un indicateur avait déjà été converti, c'est un signal de compression de contexte. Sauvegarder le diff en mode Agent après chaque passe (pas seulement en fin de session), et commencer la passe suivante par : "Le diff en cours est dans [NOM_FICHIER] — continuer à partir de la subroutine [NOM]."

> 💡 **Reprise de session :** si la conversion d'un programme est interrompue, ouvrir le fichier `*-qualification-conv-*.md` déjà produit — Bob reprend la conversion à partir du point d'arrêt documenté sans relancer le Prompt 0. Indiquer dans le prompt "la qualification est disponible dans [NOM_FICHIER] — reprendre à partir de la section [NOM_SECTION]".

> 💡 **Lien vers l'UC suivant :** les fichiers `*-rpg-converti-*.md` produits dans cet UC sont les inputs de UC 13 (tests de non-régression). Voir la Carte des livrables dans `plan-poc-bob-acme.md` pour l'enchaînement complet.

---

## Prérequis

- Les fichiers `*-comprehension-*.md` (UC 4) des programmes à convertir sont présents : ils identifient le style RPG (IV colonné / III), la présence du cycle RPG et des indicateurs applicatifs — indispensable pour choisir la bonne séquence de prompts
- Les fichiers `*-diff-optim-*.md` (UC 7) sont disponibles si UC 7 a été appliqué : la conversion Free RPG d'un source déjà optimisé (variables renommées, opcodes partiellement modernisés) est plus rapide et plus propre
- Les fichiers `*-spec-tech-*.md` (UC 6) sont disponibles (recommandé) : si la conversion modifie les interfaces du programme (conversions de P-specs RPG IV PB/PE → DCL-PROC), la spec technique est le référentiel
- IBM i MCP actif (lecture et écriture des sources RPG pour le Prompt 3-bis)
- Accès à l'IBM i de test pour compiler et tester les programmes convertis
- Un développeur RPG disponible pour valider les cas à risque (MOVE/MOVEL avec longueurs différentes, indicateurs, cycle RPG)

> 💡 Si le fichier de compréhension (UC 4) n'est pas disponible, démarrer par une session rapide d'UC 4 sur le programme ciblé — 30 minutes de compréhension Bob identifient le style RPG exact et les zones à risque avant de démarrer la conversion.

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | IBM i Developer — mode **Ask** pour la qualification et la génération du diff, **Agent** pour la compilation de test et la sauvegarde |
| **Scope** | Library List → bibliothèque applicative ACME |
| **MCP actifs** | IBM i MCP (lecture des sources RPG colonné / RPG III, compilation du source converti) |
| **MCP différés** | IBM i Database MCP (si la conversion concerne des accès aux données — vérification que les fichiers DDS accédés en natif ont des équivalents DDL), Confluence MCP (publication, si token disponible) |

### Pourquoi IBM i Developer — Ask pour la génération du diff de conversion ?

La conversion syntaxique RPG → Free est une opération à fort risque silencieux. En mode Ask, Bob produit le diff complet dans le chat — l'équipe peut valider les équivalences MOVE/MOVEL, les indicateurs convertis, et les P-specs avant de sauvegarder. En mode Agent, Bob modifie directement — une troncature non détectée sur un MOVE à longueurs différentes sera dans le source avant que l'équipe ait pu la voir.

> 💡 **Note sur `*INxx` en RPG Free :** les indicateurs `*IN01` à `*IN99` restent syntaxiquement valides en RPG ILE Free — le compilateur les accepte. Leur conversion en variables `IND` nommées est une amélioration de maintenabilité, pas une obligation syntaxique. Ne pas convertir les indicateurs appartenant à une zone INDARA (display file, subfile, printer file) — ils sont liés à la DDS écran et ne peuvent pas être remplacés par des IND locaux sans vérifier la DDS.

| Phase | Mode | Ce que Bob fait |
|-------|------|----------------|
| Qualification du programme (Prompt 0) | **Ask** | Détecte le style RPG, identifie le sous-cas (A ou B), compte les opcodes fixes, détecte le cycle et les indicateurs, recommande la stratégie |
| Inventaire des structures à convertir (Prompt 1) | **Ask** | Lit le source via IBM i MCP, produit le tableau complet des specs et opcodes à convertir, avec les zones à risque |
| Conversion des specs déclaratives (Prompt 2) | **Ask** | Produit le diff H/F/D-specs → CTL-OPT / DCL-F / DCL-S / DCL-DS dans le chat |
| Conversion des C-specs et opcodes (Prompt 3) | **Ask** | Produit le diff des C-specs et des opcodes de calcul dans le chat |
| Test de compilation intermédiaire (Prompt 3-bis) | **Agent** | SIMPLE uniquement — Lance `CRTBNDRPG` via IBM i MCP après Prompt 3. Pour STANDARD/COMPLEXE : ne pas exécuter ici — compiler après Prompt 4 uniquement (Prompt 4-bis) |
| Nettoyage, indicateurs et interfaces (Prompt 4) | **Ask** | Convertit les indicateurs résiduels, les *ENTRY PLIST, les P-specs RPG IV ; supprime les commentaires de validation ; produit le source final |
| Test de compilation final (Prompt 4-bis) | **Agent** | Lance `CRTBNDRPG` via IBM i MCP sur le source final |
| Test fonctionnel | **Humain** | Exécution sur IBM i de test, comparaison des résultats — non délégable à Bob |
| Sauvegarde du diff validé | **Agent** | Écrit le fichier `.md` dans le workspace — uniquement une fois le diff validé |

> 💡 **Règle d'or pour UC 2 :** Le mode Agent est autorisé **uniquement** pour deux opérations précises : le test de compilation via les Prompts 3-bis et 4-bis, et la sauvegarde des livrables validés. Pendant toute la phase de génération et d'itération (Prompts 0 à 4), rester en mode Ask.

> ⚠️ Ne jamais rester en mode Agent pendant la phase de conversion — Bob pourrait écrire un source converti avec des équivalences MOVE/MOVEL non validées, invisibles à la compilation mais régressives à l'exécution.

### Intégration ARCAD

Le MCP ARCAD n'était pas disponible dans le contexte de ce POC de référence (version ARCAD non compatible avec le MCP). Si le MCP ARCAD est disponible dans votre environnement, les étapes manuelles de réintégration décrites ci-dessous peuvent être automatisées. N'hésitez pas à demander à Bob de modifier cette fiche UC en intégrant la disponibilité du MCP ARCAD.

**Impact sur UC 2 : faible.** UC 2 modifie des membres sources existants — IBM i MCP accède à ces membres indépendamment d'ARCAD.

| Sans MCP ARCAD (contexte de ce POC) | Avec MCP ARCAD disponible |
|--------------------------------------|---------------------------|
| Créer manuellement une tâche ARCAD pour chaque programme converti | Le MCP ARCAD peut créer la tâche et versionner automatiquement |
| Réintégration manuelle dans ARCAD après chaque session | IBM i MCP lit et compile les sources dans les bibliothèques ARCAD normalement dans les deux cas |
| Ajouter le placeholder `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` dans chaque diff | Le placeholder n'est plus nécessaire — la réintégration est pilotée par Bob |

> 💡 **Dans les deux cas :** avant de démarrer UC 2 sur un programme, vérifier dans ARCAD qu'il n'est pas en cours de modification par un autre développeur (promotion en cours). Charger la liste des objets verrouillés dans le contexte Bob pour éviter de travailler sur une version qui sera écrasée.

---

## Prompts clés

> 💡 **Atelier Bob Industrialisation**
> La mise en place de l'atelier Bob Industrialisation permet de simplifier le travail sur cette section :
> les prompts récurrents (Prompt 0, prompts de conversion, prompt-bis) sont disponibles sous forme de **commandes slash personnalisées** (`/qualify`, `/conv-rpg-specs`, `/conv-rpg-calc`, `/test-compile`, etc.).
> Au lieu de copier-coller le bloc de code, il suffit de taper la commande correspondante dans la conversation Bob.

### Prompt 0 — Qualification et choix de stratégie

> **Ce prompt est le point d'entrée obligatoire de UC 2 pour chaque programme.**
> Il détermine le sous-cas (A : RPG IV colonné, ou B : RPG III), la catégorie de complexité, et la séquence exacte de prompts à suivre.
> Il se lance **avant** le Prompt 1 — son résultat conditionne toute la séquence suivante.

```
Le programme [NOM_PROGRAMME] se trouve dans [NOM_LIB]/QRPGSRC.
[Si disponible : Le fichier de compréhension {projet}-{lib}-{programme}-comprehension-{YYYYMMDD-HHmm}.md est disponible.]

Analyse ce programme et produis en français, en markdown, une fiche de qualification
pour la conversion RPG colonné / RPG III → FREE RPG :

## Qualification UC 2 — [NOM_PROGRAMME]

### 1. Inventaire rapide
| Indicateur                                                            | Valeur |
|-----------------------------------------------------------------------|--------|
| Style RPG détecté                                                     | RPG III / RPG IV colonné / Mixte |
| Nombre de lignes de source                                            | ?      |
| Nombre de specs H (CTL-OPT)                                           | ?      |
| Nombre de specs F (DCL-F)                                             | ?      |
| Nombre de specs D (DCL-S / DCL-DS)                                    | ?      |
| Nombre de specs C en format fixe restantes                            | ?      |
| Nombre d'indicateurs (*INxx utilisés)                                 | ?      |
| Cycle RPG actif (*INLR, niveaux L1-L9, subroutines de total)          | OUI / NON |
| Présence de P-specs RPG IV (PB / PE — procédures ILE en format fixe)  | OUI / NON |
| Présence de MOVE / MOVEL avec longueurs potentiellement différentes   | OUI / NON |
| Présence de GOTO / CABxx                                              | OUI / NON |
| UC 7 déjà appliqué sur ce programme                                   | OUI / NON |

### 2. Facteurs de complexité détectés
Réponds par OUI / NON / À CONFIRMER pour chaque facteur :
- [ ] Indicateurs (*INxx) pilotant la logique applicative (testés dans des conditions IF/DOW)
- [ ] Indicateurs de niveau de contrôle (*INL1 à *INL9) — cycle RPG de rupture
- [ ] MOVE / MOVEL avec des champs dont les longueurs diffèrent (risque de troncature sémantique)
- [ ] Cycle RPG actif — *INLR comme seul mécanisme de terminaison, ou totaux Ln
- [ ] P-specs RPG IV (PB/PE — procédures ILE en format fixe) — conversion vers DCL-PROC / END-PROC
- [ ] GOTO / CABxx (logique non structurée — branchements hors subroutine)
- [ ] Paramètres *ENTRY PLIST — l'interface d'appel ne doit pas changer après conversion
- [ ] Accès natifs aux fichiers (CHAIN / READ / WRITE) référençant des formats DDS encore en colonné
- [ ] Code partiellement en Free RPG (mode mixte) — certains specs déjà en Free, d'autres encore en fixe

### 3. Sous-cas et catégorie
**Sous-cas :**
- [ ] Sous-cas A — RPG IV colonné → FREE (specs fixe → déclarations Free RPG, C-specs → opcodes libres)
- [ ] Sous-cas B — RPG III → FREE (pas de P-specs PB/PE, pas de DCL-PROC, possibilité de cycle actif)
- [ ] Mixte — une partie du programme est en RPG IV, une autre en RPG III

**Catégorie :**
- [ ] SIMPLE — pas d'indicateurs applicatifs, pas de cycle RPG, pas de MOVE à risque, < 100 specs C
- [ ] STANDARD — indicateurs applicatifs, ou MOVE/MOVEL à longueurs différentes, ou P-specs RPG IV (PB/PE) sans cycle
- [ ] COMPLEXE — cycle RPG actif, ou GOTO/CABxx extensifs, ou RPG III dense avec totaux de niveaux

> Note : UC 2 utilise SIMPLE / STANDARD / COMPLEXE (critère : nombre d'opcodes fixes et nature
> des structures sans équivalent direct). Même vocabulaire que UC 14 (critère : champs DDS),
> UC 3 (critère : opcodes d'accès), UC 7 (critère : points d'optimisation) et UC 8 (critère :
> responsabilités) — vocabulaire unifié Phase 2, 3 et 4.
> Les échelles sont indépendantes : un programme SIMPLE en UC 2 a pu être COMPLEXE en UC 7.
> Le Prompt 0 de chaque UC calibre selon ses propres critères.

**Recommandation :**
- SIMPLE   → Conversion directe : Prompt 1 → Prompt 2 (specs) → Prompt 3 (C-specs) → Prompt 3-bis →
               Prompt 4 (nettoyage commentaires) → Prompt 4-bis
               (Prompt 3-bis valide ici : pas d'indicateurs ni SETON/SETOF non convertis)
- STANDARD → Conversion par zones : Prompt 1 → Prompt 2 → Prompt 3 →
               Prompt 4 (indicateurs + interfaces + P-specs) → Prompt 4-bis
               (⚠️ pas de Prompt 3-bis intermédiaire — le Prompt 3 laisse les SETON/SETOF
               non convertis jusqu'au Prompt 4 ; compiler après le Prompt 4 uniquement)
- COMPLEXE → Évaluer si le cycle RPG doit être converti (Prompt 4 de UC 8 en complément).
               Si non : Prompt 1 → Prompt 2 → Prompt 3 →
                        Prompt 4 (indicateurs + *INLR explicite) → Prompt 4-bis
               Si oui : appliquer Prompt 4 de UC 8 avant de finaliser avec Prompt 4 de UC 2
- Mixte    → ⛔ Ne pas démarrer le Prompt 2 avant d'avoir isolé les sections déjà en Free RPG.
               Séquence : Prompt 1 (inventaire avec exclusion explicite des DCL-S / DCL-F déjà en Free)
               → valider manuellement la liste des specs à convertir → Prompt 2 sur les specs
               encore en fixe uniquement → Prompt 3 → Prompt 4 → Prompt 4-bis.
               (⚠️ Traiter comme STANDARD : pas de Prompt 3-bis intermédiaire si des indicateurs
               ou des SETON/SETOF sont encore non convertis après Prompt 3)
               Risque si non respecté : Bob peut dupliquer des DCL-S ou des DCL-F déjà présentes
               en Free RPG, ce qui génère des erreurs de compilation "nom déjà défini".

**Si catégorie STANDARD ou COMPLEXE — zones de risque identifiées :**
| Zone | Nature du risque | Lignes concernées | Précaution recommandée |

Signale clairement ce que tu ne peux pas déterminer sans exécuter le programme.
```

**Analyse ligne à ligne :**

- `Style RPG détecté : RPG III / RPG IV colonné / Mixte` → la distinction de départ. RPG III se reconnaît au type source `RPG` (vs `RPGLE`/`SQLRPGLE`), à la présence d'E/I/O-specs, au cycle RPG actif, et à l'**absence** de D-specs et de P-specs RPG IV — un vrai RPG III n'utilise ni `D NomVar S ...` ni `P NomProc B`. RPG IV colonné utilise les specs `D`, `F`, `H` fixes, mais pas encore `CTL-OPT` — les opcodes restent en colonnes. Un programme "mixte" a été partiellement modernisé — identifier précisément quelles sections sont encore en format fixe.

- `Cycle RPG actif` → point bloquant. Si OUI, le Prompt 0 doit décider si la conversion du cycle est dans le périmètre de cette session (renvoi vers UC 8 Prompt 4) ou si le cycle est conservé avec `*INLR` explicite. Une conversion partielle (specs converties, cycle conservé) est un résultat valide pour le POC.

- `MOVE / MOVEL avec longueurs différentes` → la source de régression numéro un. La colonne "Valeur" doit être OUI / NON / À CONFIRMER — "À CONFIRMER" si Bob voit des MOVE entre des champs mais ne peut pas déterminer leurs longueurs sans inspecter les déclarations DDS/DDL.

- `Sous-cas A / Sous-cas B / Mixte` → la décision structurante. Sous-cas A avec P-specs RPG IV (PB/PE) nécessite le Prompt 4 pour convertir les procédures en `DCL-PROC`. Sous-cas B (RPG III) nécessite le Prompt 4 pour les indicateurs et le *ENTRY PLIST/PARM — sans P-specs à convertir. L'équipe doit connaître ce coût supplémentaire avant de démarrer.

- `Vocabulaire SIMPLE / STANDARD / COMPLEXE unifié` → la note rappelle explicitement que le même vocabulaire est utilisé dans tous les UC de Phase 2, 3 et 4, avec des critères propres à chaque UC. Évite la confusion entre "COMPLEXE en UC 2 (cycle RPG)" et "COMPLEXE en UC 7 (indicateurs omniprésents)".

> 💡 **Sauvegarder la fiche de qualification** (mode Agent) dès la fin du Prompt 0 :
> ```
> "Sauvegarde cette fiche de qualification dans un fichier nommé
>  {projet}-{lib}-{programme}-qualification-conv-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-qualification-conv-20250621-0900.md"
> ```
> Le Prompt 1 (inventaire) complète ensuite ce même contexte.

> ⚠️ **Piège évité :** sans le Prompt 0, l'équipe démarre la conversion sans savoir si le programme a des MOVE à longueurs différentes ou un cycle RPG actif. Découvrir en milieu de conversion que `*IN50` est testé dans 12 endroits ou que `*INLR` est le seul mécanisme de fin de programme oblige à tout réévaluer depuis le début.

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/qualify` dans la conversation Bob — la structure complète du Prompt 0 est injectée automatiquement, pré-remplie avec le nom du programme ouvert dans l'éditeur.

---

### Prompt 1 — Inventaire des structures à convertir

```
Le programme [NOM_PROGRAMME] se trouve dans [NOM_LIB]/QRPGSRC.

Sur la base de la qualification UC 2 que nous venons de faire,
produis en français, en markdown, l'inventaire complet des structures à convertir :

## Inventaire UC 2 — [NOM_PROGRAMME]

### 1. Specs déclaratives à convertir (H / F / D)
Pour chaque spec de format fixe :
| N° | Type spec | Contenu actuel (format fixe) | Équivalent Free RPG | Risque |
Équivalences attendues :
  H-spec → CTL-OPT
  F-spec → DCL-F (avec les options USAGE, KEYED, EXTFILE si présentes)
  D-spec DCL-S → DCL-S (avec type et longueur)
  D-spec DS → DCL-DS ... END-DS (avec les sous-champs)
  D-spec PR/PI → DCL-PR ... END-PR / DCL-PI ... END-PI

### 2. C-specs à convertir
Pour chaque groupe d'opcodes de format fixe (regrouper par subroutine) :
| N° | Subroutine / Section | Opcode fixe | Facteur 1 | Facteur 2 | Résultat | Indicateurs résultants (HI/LO/EQ/err) | Équivalent Free RPG |
Opcodes à inventorier :
  BEGSR / ENDSR → BEGSR SRNOM ; ... ENDSR ;   (pas END-SR — opcode inexistant)
  EXSR          → EXSR SRNOM ;   (rester EXSR — ne pas extraire en DCL-PROC dans cette passe)
  CHAIN         → CHAIN (syntaxe Free RPG)
  READ / READE / READP → READ / READE / READP (syntaxe Free RPG)
  WRITE / UPDATE / DELETE → WRITE / UPDATE / DELETE (syntaxe Free RPG)
  MOVE / MOVEL  → EVAL (⚠️ vérifier longueurs)
  Z-ADD / Z-SUB → EVAL
  ADD / SUB / MULT / DIV → EVAL
  SETGT / SETLL → SETGT / SETLL (syntaxe Free RPG)
  IF / ELSE / ENDIF → IF ... ; ELSE ; ... ENDIF ;   (syntaxe Free — point-virgule requis, ENDIF sans tiret)
  DOW / ENDDO   → DOW / ENDDO (syntaxe Free RPG)
  DOU / ENDDO   → DOU / ENDDO
  FOR / ENDFOR  → FOR / ENDFOR
  GOTO / CABxx  → à analyser (cf. section 3)
  SETON / SETOF *INxx → [NOM_INDICATEUR] = *ON / *OFF (après conversion indicateurs au Prompt 4)
  EVAL          → EVAL (déjà en Free — à conserver)
  RETURN        → RETURN ;

### 2b. Indicateurs résultants des C-specs (colonnes HI / LO / EQ / erreur)
Pour chaque opcode avec indicateurs résultants dans les colonnes 54-59 :
| N° | Opcode | Indicateur HI/LO/EQ/err | Fonction | Équivalent Free RPG |
Équivalences obligatoires — ne pas supprimer les colonnes sans générer l'équivalent :
  CHAIN ind_non_trouvé        → IF NOT %FOUND(NomFich) ;
  READ / READE ind_eof        → IF %EOF(NomFich) ;
  SETLL ind_egal              → IF %EQUAL(NomFich) ;
  opcode(E) ind_erreur        → IF %ERROR ; (+ %STATUS pour le code d'erreur)
  Comparaison HI / LO / EQ   → condition explicite dans le IF suivant
⚠️ Supprimer les colonnes d'indicateurs sans générer l'équivalent produit du code compilable
dont certains chemins conditionnels ne s'exécutent plus.

### 2c. KLIST / KFLD à inventorier (clés composites)
Pour chaque KLIST/KFLD détecté :
| N° | Nom KLIST | Champs KFLD (dans l'ordre) | Utilisé par (CHAIN/SETLL/SETGT/READE) | Clé partielle ? |
Conversion en Free RPG :
  KLIST simple (tous champs) → (champ1 : champ2 : ...) dans l'opcode
  KLIST partiel (n premiers champs) → %KDS(dsClé : n) — déclarer une DS clé
  READE *KEY après CHAIN/SETLL → READE *KEY NomFich ;   (conserver *KEY — ne pas insérer de variable)
⚠️ Convertir un KLIST partiel en clé complète change le groupe d'enregistrements lu — régression silencieuse.

### 3. MOVE / MOVEL à risque de troncature
Pour chaque MOVE / MOVEL identifié :
| N° | Ligne | MOVE/MOVEL | Source (type/longueur) | Dest (type/longueur) | Longueurs égales ? | Action recommandée |
Action : EVAL direct si longueurs égales / EVAL avec %SUBST si troncature délibérée /
         À CONFIRMER si longueurs non déterminables depuis le source seul

### 4. Indicateurs (*INxx) applicatifs
Pour chaque indicateur piloté par la logique applicative :
| N° | Indicateur | Activé par (SETON / opcode IBM i) | Testé dans (IF / DOW) | Nom proposé (IND) |
Note : certains *INxx sont positionnés automatiquement par les opcodes d'accès fichier ou par
les subroutines de cycle (DETC, DETL, TOTC, TOTL) — leur rôle réel doit être déduit depuis
les F-specs et les C-specs, pas supposé a priori. Ces indicateurs ont une conversion spécifique
(→ %EOF, %OVERFLOW, %ERROR) — les identifier séparément et ne pas les convertir en IND locaux
sans avoir identifié leur fonction dans le source.
⚠️ Un indicateur dont la fonction n'est pas déterminable depuis le source seul est à marquer
À CONFIRMER — ne pas supposer son rôle.

### 5. Procédures RPG IV (P-specs PB/PE — Sous-cas A avec procédures)
Pour chaque procédure délimitée par P-specs (PB / PE) :
| N° | Nom procédure | Paramètres PI | Équivalent DCL-PROC | Interface DCL-PI |
Note : les P-specs PB/PE sont du RPG IV ILE, pas du RPG III. Si présentes dans un source
RPG III, le programme est en réalité un RPG IV fixe avec procédures (Sous-cas A).

### 6. *ENTRY PLIST (interface d'appel du programme)
Si le programme contient un *ENTRY PLIST :
| Param N° | Nom PARM | Type/longueur déduit | Équivalent DCL-PI |
Conversion attendue dans le Prompt 2 (déclarations) :
  C     *ENTRY        PLIST
  C                   PARM                    param1     → DCL-PI *N ;
  C                   PARM                    param2          param1 [TYPE] ;
                                                              param2 [TYPE] ;
                                                         END-PI ;
⚠️ Les types des paramètres doivent être déduits des D-specs ou des usages dans le source.
⚠️ Paramètres optionnels (*NOPASS) ou omissibles (*OMIT) → signaler À CONFIRMER.
⚠️ Ne pas omettre les *ENTRY PLIST — l'interface d'appel du programme doit être préservée.

Signale clairement les cas où la décision nécessite une connaissance fonctionnelle
non visible dans le source.
Ne pas générer le code converti dans ce prompt.
Sauvegarde cet inventaire (mode Agent) dès qu'il est complet :
"Sauvegarde cet inventaire dans {projet}-{lib}-{programme}-qualification-conv-{YYYYMMDD-HHmm}.md"
```

**Analyse ligne à ligne :**

- `Regrouper par subroutine` → la structure de l'inventaire suit la structure logique du programme. Les opcodes regroupés par subroutine permettent à l'équipe de valider subroutine par subroutine avant de générer les conversions — et de décider d'exclure une subroutine si elle est trop risquée.

- `MOVE / MOVEL — Longueurs égales ?` → la colonne de décision centrale. Si les longueurs sont égales, `EVAL dest = src` est une équivalence sûre. Si elles diffèrent, Bob doit signaler le cas pour décision humaine — pas prendre la décision unilatéralement.

- `Indicateurs système séparés des indicateurs applicatifs` → la fonction de chaque indicateur doit être déduite depuis les F-specs et C-specs du programme — pas supposée a priori. `*IN99` n'est pas universellement EOF, `*IN58` n'est pas universellement overflow : ce sont des indicateurs génériques dont le rôle dépend du programme. Les indicateurs applicatifs (pilotés par la logique du programme) se convertissent en variables booléennes `IND` avec nom descriptif. Les indicateurs système se convertissent en BIF (`%EOF`, `%OVERFLOW`, `%ERROR`) uniquement après avoir identifié leur rôle réel dans le source.

- `Ne pas générer le code converti dans ce prompt` → séparation inventaire / génération. L'inventaire est la feuille de route de la conversion — l'équipe peut le corriger avant de générer.

> ⚠️ **Piège évité :** sans inventaire préalable, Bob peut proposer deux équivalences différentes pour le même opcode selon le contexte — `MOVE` converti en `EVAL` dans une subroutine et ignoré dans une autre. L'inventaire force la cohérence.

---

### Étape 0-bis — Référence comportementale minimale

> **À exécuter avant toute génération de code — sur le programme RPG original sur l'IBM i de test.**

Avant de démarrer les passes de conversion, capturer manuellement :

- [ ] Au moins **un cas nominal** : entrée (ou déclenchement batch), résultat de sortie, fichiers créés ou modifiés
- [ ] Au moins **un cas limite** identifié au Prompt 0 (MOVE à longueurs différentes, indicateur piloté, cycle RPG...)
- [ ] Le code retour ou statut fichier pertinent après exécution
- [ ] Les paramètres retournés si le programme est callable (`*ENTRY PLIST`)

Consigner ces résultats dans le fichier `{projet}-{lib}-{programme}-qualification-conv-{YYYYMMDD-HHmm}.md`
sous la section `## Référence comportementale`.

⚠️ Si le programme original ne peut pas être exécuté ou si aucun résultat de référence
n'est disponible, l'inscrire explicitement : **NON-RÉGRESSION NON DÉMONTRABLE DANS LE POC**
et en informer le responsable du POC avant de continuer.

---

### Prompt 2 — Conversion des specs déclaratives (H / F / D)

```
Sur la base de l'inventaire UC 2 de [NOM_PROGRAMME], section 1 (specs déclaratives),
convertis les specs de format fixe suivantes en leur équivalent Free RPG :

[Lister ici les N° de specs du tableau Prompt 1, section 1, à convertir dans cette passe]

Le source RPG cible doit commencer par la directive fully free-form en colonne 1 :
**FREE
Cette directive doit être la première ligne du membre, avant toute déclaration.
Sans elle, le compilateur interprète le source en mode colonné limité.

Pour chaque spec convertie, produis le diff :
// AVANT (format fixe) :
[spec originale avec positions de colonnes commentées]
// APRÈS (Free RPG) :
[équivalent Free RPG]

Table de conversion à respecter :
H DFTACTGRP(*NO) ACTGRP(*CALLER)  → CTL-OPT DFTACTGRP(*NO) ACTGRP(*CALLER) ;
FNOMDUF    IF   E           K DISK  → DCL-F NomDuf DISK(*EXT) KEYED USAGE(*INPUT) ;
FNOMUPDF   UF   E           K DISK  → DCL-F NomUpdf DISK(*EXT) KEYED USAGE(*UPDATE:*DELETE) ;
FNOMWDSPF  CF   E             WORKSTN → DCL-F NomWdspf WORKSTN ;
D NomVar          S             10A   → DCL-S NomVar CHAR(10) ;
D NomDs           DS                  → DCL-DS NomDs ;
D   SousChamp1                   5P 0 →   SousChamp1 PACKED(5:0) ;
D                               END   → END-DS ;
D NomPR           PR                  → DCL-PR NomPR ;
D   Param1                       5P 0 →   Param1 PACKED(5:0) ;
D                               END   → END-PR ;
*ENTRY PLIST (si présent dans les C-specs) → générer ici la DCL-PI :
  DCL-PI *N ;
    param1 [TYPE déduit des D-specs ou des usages] ;
    param2 [TYPE] ;
  END-PI ;
  (⚠️ obligatoire dans cette passe — la DCL-PI doit exister avant la compilation Prompt 3-bis)
  (Pour les programmes sans *ENTRY PLIST : omettre la DCL-PI)

Règles :
- Conserver les noms de variables et de fichiers tels quels (ne pas renommer dans cette passe)
  sauf si un renommage UC 7 est en cours (auquel cas utiliser le nom cible UC 7)
- Conserver les attributs EXTNAME, LIKEREC, PREFIX si présents dans les D-specs d'origine
- Signaler toute spec dont la conversion n'est pas directe (ex. D-spec avec LIKE sur un champ de DS)
- Produire à la fin un récapitulatif des specs converties :
  | Spec | Avant | Après | Risque résiduel |
Sauvegarde ce diff (mode Agent) dès qu'il est complet :
"Sauvegarde ce diff dans {projet}-{lib}-{programme}-rpg-converti-{YYYYMMDD-HHmm}.md"
```

> 💡 **Sauvegarder ce diff** (mode Agent) :
> ```
> "Sauvegarde ce diff de conversion des specs dans un fichier nommé
>  {projet}-{lib}-{programme}-rpg-converti-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-rpg-converti-20250621-1100.md"
> ```

> ⚠️ **Piège évité :** les F-specs avec plusieurs options (`RENAME`, `PREFIX`, `PRTCTL`) ont des équivalents moins directs en Free RPG. Bob doit signaler ces cas et ne pas produire un `DCL-F` incomplet.

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/conv-rpg-specs` — le prompt de conversion des specs déclaratives est injecté avec les variables du programme courant déjà résolues (nom du programme, bibliothèque, liste des specs à convertir issue du Prompt 1).

---

### Prompt 3 — Conversion des C-specs et des opcodes de calcul

~~~
Sur la base de l'inventaire UC 2 de [NOM_PROGRAMME], section 2 (C-specs),
convertis les opcodes de format fixe suivants en leur équivalent Free RPG :

[Lister ici les N° de subroutines ou sections du tableau Prompt 1, section 2, à convertir]

Pour chaque conversion, produis le diff :
// AVANT (C-spec colonné) : [opcode avec facteurs et colonnes indicateurs]
// APRÈS (Free RPG)       : [équivalent Free RPG]

Table de conversion à respecter :
C     BEGSR    SRNOM                → BEGSR SRNOM ;
C     ENDSR                        → ENDSR ;
C                   EXSR   SRNOM   → EXSR SRNOM ;
C     key       CHAIN  NomFich     → Si le source original gérait seulement « non trouvé » :
                                       CHAIN key NomFich ;
                                       IF NOT %FOUND(NomFich) ; /* reproduire chemin indicateur original */ ENDIF ;
                                     Si le source original gérait aussi les erreurs I/O :
                                       CHAIN(E) key NomFich ;
                                       IF %ERROR ; /* reproduire chemin erreur original */ ENDIF ;
                                       IF NOT %FOUND(NomFich) ; /* reproduire chemin indicateur original */ ENDIF ;
                                     (⚠️ tester %ERROR avant %FOUND — %FOUND n'est défini qu'en
                                     l'absence d'erreur I/O)
C                   READ   NomFich → Si le source original gérait seulement « fin de fichier » :
                                       READ NomFich ;
                                       IF %EOF(NomFich) ; /* reproduire chemin AT END original */ ENDIF ;
                                     Si le source original gérait aussi les erreurs I/O :
                                       READ(E) NomFich ;
                                       IF %ERROR ; /* reproduire chemin erreur original */ ENDIF ;
                                       IF %EOF(NomFich) ; /* reproduire chemin AT END original */ ENDIF ;
                                     (⚠️ tester %ERROR avant %EOF — %EOF n'est défini qu'en
                                     l'absence d'erreur I/O)
C                   READE  NomFich → Même logique que READ ci-dessus ; utiliser *KEY pour
                                     conserver la clé courante :
                                       READE *KEY NomFich ;
                                     ou avec gestion d'erreur :
                                       READE(E) *KEY NomFich ;
                                       IF %ERROR ; ... ENDIF ;
                                       IF %EOF(NomFich) ; ... ENDIF ;
                                     (⚠️ fournir une variable de clé explicite uniquement
                                     si le source original le fait)
C                   WRITE  NomFich → Si le source ne gérait pas explicitement les erreurs :
                                       WRITE NomFich ;   (sans (E) — laisser remonter l'exception)
                                     Si le source utilisait un indicateur d'erreur, INFSR,
                                     ou un traitement explicite :
                                       WRITE(E) NomFich ;
                                       IF %ERROR ; /* reproduire chemin erreur original */ ENDIF ;
C                   UPDATE NomFich → Si le source ne gérait pas explicitement les erreurs :
                                       UPDATE NomFich ;   (sans (E))
                                     Si traitement d'erreur original :
                                       UPDATE(E) NomFich ;
                                       IF %ERROR ; /* reproduire chemin erreur original */ ENDIF ;
C                   DELETE NomFich → Si le source ne gérait pas explicitement les erreurs :
                                       DELETE NomFich ;   (sans (E))
                                     Si traitement d'erreur original :
                                       DELETE(E) NomFich ;
                                       IF %ERROR ; /* reproduire chemin erreur original */ ENDIF ;
C                   SETGT  key NomFich → SETGT key NomFich ;
C                   SETLL  key NomFich → SETLL key NomFich ;
                                     IF %EQUAL(NomFich) ; /* si indicateur égal était utilisé */ ENDIF ;
C     cond      IF(E)  ...         → IF cond ; ... ENDIF ;
C                   DOW    cond    → DOW cond ; ... ENDDO ;
C                   EVAL   dest=src → dest = src ;
C     A         ADD    B   C       → C = A + B ;
C     A         SUB    B   C       → C = A - B ;
C     A         MULT   B   C       → C = A * B ;
C     A         DIV    B   C       → C = A / B ;
C     src       Z-ADD  dest        → dest = src ;
C     src       Z-SUB  dest        → dest = -src ;
C     src       MOVE   dest        → aucune équivalence générique — consulter l'inventaire section 3.
                                     MOVE est aligné à droite (justifié à droite pour les numériques,
                                     à droite dans le champ pour les alpha). Déterminer :
                                     1. types source/dest (alpha ou num), longueurs et signe
                                     2. quelle partie de dest est remplacée, quelle partie est conservée
                                     3. présence de l'extender P (MOVE P) — remplissage explicite
                                        de la partie non occupée de la destination par des blancs
                                        (alpha) ou des zéros (num) ; ne pas confondre avec un
                                        arrondi arithmétique (relevant du half-adjust, opérateur H)
                                     Équivalences possibles selon l'analyse :
                                       longueurs égales et mêmes types → dest = src ;
                                       src alpha plus long (troncature droite) →
                                         %SUBST(src : %LEN(src) - %LEN(dest) + 1 : %LEN(dest))
                                       src alpha plus court (reste de dest conservé) →
                                         %REPLACE(src : dest : %LEN(dest) - %LEN(src) + 1)
                                         // ⚠️ vérifier que la partie non remplacée doit bien
                                         // rester inchangée — pas des espaces
                                     Tout cas non couvert par l'inventaire section 3 :
                                     // ⚠️ MOVE — À CONFIRMER
C     src       MOVEL  dest        → aucune équivalence générique — consulter l'inventaire section 3.
                                     MOVEL est aligné à gauche (caractères de gauche copiés).
                                     Déterminer :
                                     1. types source/dest, longueurs
                                     2. si %LEN(src) >= %LEN(dest) : dest reçoit la partie gauche de src
                                          → %SUBST(src : 1 : %LEN(dest))
                                     3. si %LEN(src) < %LEN(dest) : seuls les %LEN(src) premiers
                                          caractères de dest sont remplacés, le reste est conservé
                                          → %REPLACE(src : dest : 1)
                                          // ⚠️ MOVEL LONGUEUR SOURCE COURTE — reste de dest conservé :
                                          // vérifier si c'est le comportement voulu
                                     Tout cas non couvert par l'inventaire section 3 :
                                     // ⚠️ MOVEL — À CONFIRMER
C     CLE       KLIST              → (cf. inventaire section 2c — convertir en clé composite ou %KDS)
C                   KFLD    CHAMP  → (intégré dans la conversion KLIST ci-dessus)
C                   RETURN         → RETURN ;
C                   CALLP  NomPgm(params) → CALLP NomPgm(params) ;

Règles :
- Préserver l'extender et le mécanisme de gestion d'erreur originaux.
  Ajouter (E) sur un opcode d'accès fichier uniquement si le chemin de gestion d'erreur
  (%ERROR / %STATUS) est explicitement généré et validé dans la même transformation.
  (⚠️ (E) seul sans IF %ERROR correspondant transforme une exception fatale en erreur
  silencieuse ignorée — ne jamais ajouter (E) sans le IF)
- Après CHAIN(E) ou READ(E), tester %ERROR avant %FOUND/%EOF — %FOUND/%EOF n'est défini
  que si aucune erreur I/O n'a eu lieu ; %ERROR précède toujours les tests fonctionnels.
- Reproduire le mécanisme original de gestion d'erreur : si le source utilisait un indicateur
  résultant, le IF généré doit reproduire exactement ce chemin conditionnel
- Pour chaque MOVE / MOVEL : appliquer l'action de l'inventaire section 3 ;
  ne jamais utiliser EVAL seul pour un MOVEL — MOVEL ne remplace qu'une partie de la destination
- Pour les SETON / SETOF : remplacer uniquement si l'indicateur a été déclaré en IND au Prompt 4
- Conserver le code d'origine en commentaire (// AVANT) pendant la session de validation
- Ne pas inventer de logique non visible dans l'opcode d'origine ou dans les fichiers
  *-comprehension-*.md et *-regles-*.md
Sauvegarde ce diff (mode Agent) dès qu'il est complet :
"Complète le fichier {projet}-{lib}-{programme}-rpg-converti-{YYYYMMDD-HHmm}.md
 avec la conversion des C-specs de cette passe."
~~~

**Analyse ligne à ligne :**

- `Extender (E) — conditionnel` → l'ajout de `(E)` sur les opcodes d'accès modifie le mécanisme d'exception original du source. N'ajouter `(E)` que lorsque le chemin `%ERROR`/`%STATUS` correspondant est généré dans la même passe et validé. Après `CHAIN(E)` ou `READ(E)`, tester `%ERROR` en premier, puis `%FOUND`/`%EOF` — ils ne sont définis que si aucune erreur I/O n'a eu lieu.

- `MOVE / MOVEL — appliquer l'action de l'inventaire section 3` → la section 3 du Prompt 1 est le référentiel de décision. Bob ne décide pas unilatéralement pour les cas "À CONFIRMER" — il laisse une marque dans le diff pour décision humaine.

- `SETON / SETOF — seulement si l'indicateur a été converti en IND` → ordre obligatoire. Si les indicateurs ne sont pas encore convertis (Prompt 4), les SETON/SETOF restent en l'état dans cette passe — on ne peut pas substituer le nom de l'indicateur avant qu'il soit déclaré.

- `Conserver le code d'origine en commentaire` → la même discipline que UC 7 et UC 8. Pendant la validation, le développeur voit l'original et l'équivalent côte à côte.

> 💡 **Sauvegarder ce diff** (mode Agent) — compléter le fichier existant :
> ```
> "Complète le fichier {projet}-{lib}-{programme}-rpg-converti-{YYYYMMDD-HHmm}.md
>  avec la conversion des C-specs de cette passe."
> ```

> ⚠️ **Piège évité :** convertir les opcodes `IF` / `ELSE` en format fixe sans ajouter les points-virgules et le `ENDIF` (Free RPG) laisse le source en erreur de compilation. Le prompt exige la syntaxe complète. Note : la syntaxe RPG Free est `ENDIF ;` sans tiret — `END-IF` est la syntaxe COBOL source.

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/conv-rpg-calc` — le prompt de conversion des C-specs est injecté avec la liste des subroutines à convertir (issue du Prompt 1) déjà résolue.

---

### Prompt 3-bis — Test de compilation par Bob (après conversion des specs et C-specs)

> **Ce que Bob peut faire :** lancer la compilation via IBM i MCP en mode **Agent** et rapporter les erreurs.
> **Ce que Bob ne peut pas faire :** valider que les équivalences MOVE/MOVEL sont sémantiquement correctes — vérification humaine irréductible, notamment pour les champs de longueurs différentes.
> **Quand l'utiliser :** uniquement pour les programmes **SIMPLE** (sans indicateurs applicatifs ni `SETON`/`SETOF` non convertis), après les Prompts 2 et 3. Pour les programmes STANDARD, COMPLEXE et Mixte : ne pas utiliser ce prompt — compiler directement après le Prompt 4 via le Prompt 4-bis.

```
Le diff de conversion de [NOM_PROGRAMME] que nous venons de générer doit être écrit
sur l'IBM i avant compilation. Exécute les deux étapes suivantes :

Étape 1 — Écrire le source converti sur l'IBM i (write_member) :
Écris le source RPG Free dans [NOM_LIB]/QRPGSRC([NOM_PROGRAMME])
en utilisant le contenu converti dans cette conversation (Prompts 2 + 3).
Le membre existe déjà en format fixe — remplacer son contenu intégralement.

Étape 2 — Lancer la compilation :
Choisir la commande selon le contenu du source (détecté au Prompt 0) :

Si pas de EXEC SQL dans le source d'origine :
CRTBNDRPG PGM([NOM_LIB_TEST]/[NOM_PROGRAMME])
          SRCFILE([NOM_LIB]/QRPGSRC)
          SRCMBR([NOM_PROGRAMME])
          OPTION(*EVENTF *LIST)
          DBGVIEW(*SOURCE)

Si EXEC SQL présent dans le source d'origine (source SQLRPGLE) :
CRTSQLRPGI OBJ([NOM_LIB_TEST]/[NOM_PROGRAMME])
           SRCFILE([NOM_LIB]/QRPGSRC)
           SRCMBR([NOM_PROGRAMME])
           OPTION(*EVENTF *LIST)
           DBGVIEW(*SOURCE)
           OBJTYPE(*PGM)

Si source NOMAIN (module de service) :
CRTRPGMOD MODULE([NOM_LIB_TEST]/[NOM_PROGRAMME])
          SRCFILE([NOM_LIB]/QRPGSRC)
          SRCMBR([NOM_PROGRAMME])
          OPTION(*EVENTF *LIST)
          DBGVIEW(*SOURCE)

Analyse le résultat et produis en français :

1. Statut : COMPILATION RÉUSSIE / ERREURS DE COMPILATION
2. Si erreurs : liste des erreurs avec numéro de ligne, code erreur IBM i et description
   | Ligne | Code erreur | Description | Cause probable |
   Causes probables à vérifier : ENDIF / ENDDO manquant (conversion incomplète d'un bloc),
   variable IND non encore déclarée (indicateur *INxx encore utilisé dans une condition),
   spec DCL-F incomplète (attribut KEYED ou USAGE manquant), SETON/SETOF non converti
3. Si avertissements sur des conversions implicites de type : les lister séparément
4. Si compilation réussie : confirmer que les [N] conversions de cette passe compilent
5. Rappeler que la compilation réussie ne garantit pas la non-régression fonctionnelle —
   les cas MOVE/MOVEL à longueurs différentes doivent faire l'objet d'une validation
   fonctionnelle spécifique
```

> 💡 **Ce prompt s'exécute en mode Agent** — Bob doit pouvoir lancer `CRTBNDRPG` via IBM i MCP. Vérifier que l'utilisateur IBM i associé au MCP a le droit `*USE` sur `CRTBNDRPG` et les droits d'écriture sur la bibliothèque cible de test.

> ⚠️ **Ce prompt ne remplace pas le test fonctionnel.** Les tests suivants restent obligatoires et ne peuvent pas être délégués à Bob :
> - Exécuter le programme converti sur l'IBM i de test avec un jeu de données réel
> - Vérifier les cas déclenchant les MOVE/MOVEL identifiés comme "À CONFIRMER" dans l'inventaire
> - Comparer les résultats du programme converti avec ceux du programme d'origine colonné

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/test-compile` — `CRTBNDRPG` est déclenché automatiquement via MCP et le résultat est formaté avec le tableau Statut / Erreurs / Cause probable.

---

### Prompt 4 — Nettoyage : indicateurs, interfaces et P-specs RPG IV (Sous-cas A et B)

```
Sur la base de l'inventaire UC 2 de [NOM_PROGRAMME],
sections 4, 5 et 6,
complète la conversion du programme :

### 4a. Vérification de la DCL-PI (*ENTRY PLIST → interface programme)
Si le programme contient un *ENTRY PLIST (inventaire section 6) :
La DCL-PI a été générée au Prompt 2 — vérifier ici qu'elle est bien présente et conforme :
- Les types RPG correspondent aux types déduits des D-specs ou des usages
- L'ordre des paramètres correspond à l'ordre des PARM dans le *ENTRY PLIST
- Les paramètres optionnels (*NOPASS) ou omissibles (*OMIT) sont signalés À CONFIRMER
Si la DCL-PI est absente ou incomplète : la régénérer maintenant.
Règle : ne pas modifier les types sans validation et sans consulter les programmes appelants.
Pour les programmes sans *ENTRY PLIST : omettre la DCL-PI.

### 4b. Conversion des indicateurs applicatifs
Pour chaque indicateur *INxx de la section 4 de l'inventaire :

⚠️ Avant de convertir un indicateur : vérifier s'il appartient à une zone INDARA
(display file, subfile, printer file). Si oui, ne pas le convertir en IND local —
signaler // ⚠️ INDICATEUR INDARA — ne pas convertir sans vérification DDS.

1. Déclarer la variable booléenne :
   DCL-S [NOM_INDICATEUR] IND INZ(*OFF) ;  // remplace *INxx
2. Remplacer chaque occurrence de *INxx par [NOM_INDICATEUR] dans tout le source :
   // AVANT : SETON *IN[xx]  → [NOM_INDICATEUR] = *ON
   // AVANT : SETOF *IN[xx]  → [NOM_INDICATEUR] = *OFF
   // AVANT : IF   *IN[xx]   → IF   [NOM_INDICATEUR]
   // AVANT : DOW  *IN[xx]   → DOW  [NOM_INDICATEUR]
3. Pour les indicateurs système — déduire leur fonction depuis les F-specs et C-specs,
   ne pas supposer leur rôle a priori :
   Indicateur EOF (positionné par READ/READE sur fin de fichier) → %EOF([NOM_FICHIER])
   Indicateur overflow (positionné par l'imprimante)            → %OVERFLOW([NOM_FICHIER])
   Indicateur erreur (positionné par un opcode fichier)         → %ERROR après opcode(E)
   ⚠️ *IN99 n'est pas universellement EOF — vérifier sa fonction dans les F-specs et C-specs
   ⚠️ *IN58 n'est pas universellement overflow — même vérification

### 4c. Conversion des P-specs RPG IV → DCL-PROC (si P-specs PB/PE présentes)
Pour chaque procédure délimitée par P-specs (inventaire section 5) :
// AVANT (P-spec RPG IV fixe) :
P NomSrPgm        B                   ← début de la procédure
D NomSrPgm        PI                  ← interface
D   Param1                  5P 0      ← paramètre
P NomSrPgm        E                   ← fin
// APRÈS (Free RPG) :
DCL-PROC NomSrPgm ;
  DCL-PI NomSrPgm ;
    Param1 PACKED(5:0) ;
  END-PI ;
  [corps de la procédure]
END-PROC ;

### 4d. Suppression des commentaires de validation
Retirer les commentaires // AVANT des conversions validées dans les Prompts 2 et 3.
Ne retirer que les commentaires dont la conversion a été compilée et validée.

Règles :
- Propager chaque remplacement d'indicateur à toutes ses occurrences dans le source
- Ne pas convertir les indicateurs système IBM i (indicateurs < *IN01 ou > *IN99 non applicatifs)
  sans vérification — certains sont positionnés par les opcodes d'accès fichier
- Ne pas inventer de logique dans les corps de DCL-PROC — reprendre exactement le corps
  de la procédure RPG IV d'origine (Sous-cas A) ou de la subroutine RPG III d'origine (Sous-cas B)
```

> 💡 **Sauvegarder le source final** (mode Agent) — compléter le fichier converti :
> ```
> "Complète le fichier {projet}-{lib}-{programme}-rpg-converti-{YYYYMMDD-HHmm}.md
>  avec les conversions d'indicateurs et de P-specs de cette passe."
> ```

> ⚠️ **Note sur le cycle RPG (Sous-cas B COMPLEXE) :** si le Prompt 0 a détecté un cycle RPG actif et que la décision est de le convertir, utiliser le **Prompt 4 de UC 8** (`UC08-restructuration-code.md`) en parallèle de cette session UC 2. Le Prompt 4 UC 8 produit le squelette de boucle `DOW/READ/ENDDO` et les variables de contrôle de niveau — puis revenir au Prompt 4 de UC 2 pour finaliser les indicateurs et P-specs.

> ⚠️ **Piège évité :** convertir un indicateur de fin de fichier RPG III (`*IN99`) en variable booléenne `IND` et ajouter `EVAL flFinFich = *ON` là où le cycle positionnait `*IN99 = *ON` automatiquement — mais oublier de supprimer la boucle READ implicite du cycle. Le programme compile mais ne lit plus les enregistrements.

---

### Prompt 4-bis — Test de compilation final

> **Ce que Bob peut faire :** lancer la compilation finale sur le source entièrement converti.
> **Ce que Bob ne peut pas faire :** valider la sémantique des indicateurs convertis et des P-specs transformées en DCL-PROC.
> **Quand l'utiliser :** après le Prompt 4, avant le test fonctionnel humain.

```
Le source RPG de [NOM_PROGRAMME] est maintenant entièrement converti
(specs, C-specs, indicateurs[, P-specs]).

Étape 1 — Mettre à jour le source sur l'IBM i (write_member) :
Écris le source RPG Free final dans [NOM_LIB]/QRPGSRC([NOM_PROGRAMME])
avec le contenu complet issu de cette conversation (Prompts 2 + 3 + 4 assemblés).
Remplacer intégralement le contenu du membre existant.

Étape 2 — Lancer la compilation finale :
Choisir la commande selon le contenu du source (détecté au Prompt 0) :

Si pas de EXEC SQL dans le source d'origine :
CRTBNDRPG PGM([NOM_LIB_TEST]/[NOM_PROGRAMME])
          SRCFILE([NOM_LIB]/QRPGSRC)
          SRCMBR([NOM_PROGRAMME])
          OPTION(*EVENTF *LIST)
          DBGVIEW(*SOURCE)

Si EXEC SQL présent dans le source d'origine (source SQLRPGLE) :
CRTSQLRPGI OBJ([NOM_LIB_TEST]/[NOM_PROGRAMME])
           SRCFILE([NOM_LIB]/QRPGSRC)
           SRCMBR([NOM_PROGRAMME])
           OPTION(*EVENTF *LIST)
           DBGVIEW(*SOURCE)
           OBJTYPE(*PGM)

Si source NOMAIN (module de service) :
CRTRPGMOD MODULE([NOM_LIB_TEST]/[NOM_PROGRAMME])
          SRCFILE([NOM_LIB]/QRPGSRC)
          SRCMBR([NOM_PROGRAMME])
          OPTION(*EVENTF *LIST)
          DBGVIEW(*SOURCE)

Produis :
1. Statut : COMPILATION RÉUSSIE / ERREURS DE COMPILATION
2. Si erreurs : liste des erreurs avec cause probable
   | Ligne | Code erreur | Description | Cause probable |
3. Si compilation réussie : confirmer que le programme [NOM_PROGRAMME] est créé dans
   [NOM_LIB_TEST] et prêt pour le test fonctionnel
4. Rappeler les 3 points de validation fonctionnelle obligatoires pour ce programme :
   - Exécuter avec un jeu de données réel incluant les cas déclenchant les MOVE/MOVEL convertis
   - Vérifier les indicateurs convertis en IND sur tous les chemins de code qui les testent
   - Comparer les résultats avec le programme d'origine colonné — les résultats doivent être identiques
```

> 💡 **Ce prompt s'exécute en mode Agent** — Bob doit pouvoir lancer `CRTBNDRPG` via IBM i MCP. Vérifier que l'utilisateur IBM i associé au MCP a le droit `*USE` sur `CRTBNDRPG` et les droits d'écriture sur la bibliothèque cible de test.

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/test-compile` — même commande que le Prompt 3-bis, applicable ici pour la compilation finale.

---

### Prompt 5 — Plan de conversion pour un périmètre applicatif complet

```
Sur la base des fichiers de compréhension ({projet}-{lib}-*-comprehension-*.md)
et des fiches de qualification UC 2 déjà produites ({projet}-{lib}-*-qualification-conv-*.md)
pour l'application [NOM_APPLICATION] dans [NOM_LIB],
génère un plan de conversion RPG → Free en français, en markdown.

## Plan de conversion UC 2 — [NOM_APPLICATION]

### 1. Périmètre des programmes à convertir
| Programme | Style RPG | Sous-cas | Cycle RPG | Indicateurs | MOVE à risque | Catégorie (S/St/C) | Priorité |
Catégorie : S = SIMPLE / St = STANDARD / C = COMPLEXE (critères UC 2)
Priorité : décroissante selon la combinaison "risque faible + nombre de specs fixes restantes"

### 2. Programmes à convertir après UC 7
Les programmes pour lesquels UC 7 doit être appliqué avant UC 2 — conversion plus propre
sur un source avec variables renommées et opcodes partiellement modernisés.

### 3. Programmes avec cycle RPG actif à traiter en coordination avec UC 8
Les programmes COMPLEXE Sous-cas B nécessitant le Prompt 4 de UC 8 —
listés avec la nature du cycle (ruptures de niveau, subroutines de total).

### 4. Risques identifiés sur le périmètre
Les 3 à 5 conversions les plus risquées — MOVE/MOVEL avec troncature probable,
indicateurs systèmes utilisés comme indicateurs applicatifs, P-specs avec paramètres complexes.

### 5. Estimation d'effort
| Programme | Prompts nécessaires | Durée estimée (avec Bob) | Gain lisibilité estimé |

Ne pas inventer de programmes ou de variables non visibles dans les sources disponibles.
```

> 💡 **Sauvegarder ce plan** (mode Agent) :
> ```
> "Sauvegarde ce plan de conversion dans un fichier nommé
>  {projet}-{lib}-{lib}-plan-conv-rpg-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-APPVTE-plan-conv-rpg-20250621-0800.md"
> ```

> 💡 Ce plan est le document de pilotage de UC 2. Il permet de suivre l'avancement programme par programme et d'identifier les programmes nécessitant UC 7 ou la conversion de cycle (UC 8) avant de démarrer la conversion syntaxique.

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/gen-conv-plan` — le prompt de plan périmètre est injecté avec le nom de l'application et la bibliothèque courants.

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Ouverture des membres sources RPG colonné et RPG III, navigation Object Browser, compilation des sources convertis et affichage des erreurs inline (Prompts 3-bis, 4-bis) |
| **IBM i Languages** | Coloration syntaxique RPG Free, RPG IV colonné et RPG III — indispensable pour valider que le code converti est bien en Free RPG et non en mode mixte résiduel |
| **Markdown All in One** | Prévisualisation des fichiers de diff et des plans de conversion sauvegardés |

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture des membres sources RPG colonné / RPG III (`QRPGSRC`) via `read_member` ; écriture du source converti via `write_member` ; compilation via `execute_compile_action` ou `execute_cl_command` (Prompts 3-bis, 4-bis) |
| **IBM i Database MCP** | Optionnel — vérifier si les fichiers accédés en natif par les F-specs ont des tables DDL equivalentes (utile si UC 14 a déjà été appliqué sur les fichiers DDS) |
| **Confluence MCP** *(si disponible)* | Publication des diffs de conversion validés et du plan périmètre dans l'espace POC |

> 💡 **Requêtes QSYS2 utiles pour UC 2 (si IBM i Database MCP actif) :**
> ```sql
> -- Lister les membres RPG d'une bibliothèque avec leur type (identifier les RPG III)
> SELECT SYSTEM_TABLE_MEMBER, SYSTEM_TABLE_SCHEMA, SOURCE_TYPE,
>        LAST_SOURCE_UPDATE_TIMESTAMP, TEXT_DESCRIPTION
> FROM QSYS2.SYSPARTITIONSTAT
> WHERE SYSTEM_TABLE_SCHEMA = '[NOM_LIB]'
>   AND SOURCE_TYPE IN ('RPG', 'RPGLE', 'SQLRPGLE')
> ORDER BY SOURCE_TYPE, SYSTEM_TABLE_MEMBER;
>
> -- Vérifier si un programme a été compilé avec une version RPG récente
> SELECT PROGRAM_NAME, PROGRAM_LIBRARY, ACTIVATION_GROUP,
>        CREATION_TIMESTAMP, COMPILER_ID
> FROM QSYS2.PROGRAM_INFO
> WHERE PROGRAM_LIBRARY = '[NOM_LIB]'
>   AND COMPILER_ID LIKE '%RPG%'
> ORDER BY CREATION_TIMESTAMP DESC;
> ```
> Ces requêtes permettent de distinguer les programmes RPG III (SOURCE_TYPE = 'RPG') des programmes RPG IV (SOURCE_TYPE = 'RPGLE' ou 'SQLRPGLE') et de prioriser le plan de conversion.

> 💡 **ARCAD MCP : NON DISPONIBLE dans ce POC.** Voir la section "Spécificité ARCAD" ci-dessus.

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Sauter le Prompt 0 sur un programme COMPLEXE | La présence d'un cycle RPG actif ou de MOVE/MOVEL à longueurs différentes n'est pas détectée — la conversion est lancée avec des équivalences incorrectes sur des zones critiques | Toujours lancer le Prompt 0 en premier — il prend 3 minutes et évite de passer une journée à déboguer des régressions silencieuses |
| Convertir MOVE par EVAL sans vérifier les longueurs | Troncature silencieuse sur des champs de longueurs différentes — montants incorrects, références tronquées — le programme compile et produit des résultats presque corrects | Le Prompt 1 section 3 inventorie explicitement les MOVE à risque — ne jamais convertir un cas "À CONFIRMER" sans validation humaine |
| Convertir SETON/SETOF sans décider du statut de l'indicateur | Convertir `SETON *IN50` en `flIndicateur50 = *ON` avant de vérifier si `*IN50` est lié à une zone INDARA (DDS écran, subfile, printer file) casse l'intégration DDS — l'indicateur doit rester `*INxx` si INDARA, ou devenir `IND` nommé si applicatif. Une conversion partielle ou incohérente (certaines occurrences converties, d'autres non) produit des résultats incorrects | Le Prompt 4b vérifie INDARA avant toute conversion d'indicateur — décision documentée pour chaque `*INxx` avant de lancer la propagation |
| Ignorer le cycle RPG actif et convertir seulement les specs | Le programme compile en Free RPG mais utilise encore `*INLR` et les niveaux de contrôle L1-L9 de façon implicite — état hybride difficile à maintenir | Le Prompt 0 détecte le cycle ; la décision de le convertir (UC 8 Prompt 4) ou de le conserver est documentée dans la fiche de qualification |
| Convertir des P-specs RPG IV (PB/PE) sans aligner les DCL-PROC sur la signature originale | Les appels CALLP existants vers ces sous-programmes ne correspondent plus à la nouvelle interface DCL-PR — erreur de compilation sur les programmes appelants | L'inventaire section 5 documente les paramètres PI de chaque P-spec — la DCL-PI générée doit être strictement identique à l'interface d'origine |
| Travailler sur un programme RPG III sans UC 7 préalable | Les variables cryptiques (Axx, Lxx) se retrouvent en Free RPG avec les mêmes noms illisibles — la conversion ne fait qu'habiller le mauvais code dans une syntaxe moderne | Appliquer UC 7 (au moins les renommages) sur les programmes RPG III avant UC 2 — un source avec des noms lisibles en Free RPG est la cible réelle |
| Rester en mode Agent pendant la génération du diff | Bob peut sauvegarder des diffs intermédiaires avec des MOVE non résolus ou des indicateurs partiellement convertis | Rester en mode **Ask** pendant toute la phase de conversion (Prompts 0 à 4) — mode Agent uniquement pour Prompts 3-bis et 4-bis (compilation) et sauvegarde finale |
| Appliquer la conversion sans test fonctionnel | Le programme compile ; les cas nominaux fonctionnent — une régression sur les cas de troncature MOVE ou les indicateurs de fin de fichier n'est découverte qu'en production | Tester chaque conversion avec un jeu de données réel incluant les cas limites documentés dans `*-regles-*.md` |

---

## Check-list de validation UC 2

Avant de passer à UC 13 (tests de non-régression), ou de déclarer un programme converti, valider chaque point :

- [ ] **Pour chaque programme converti : le Prompt 0 a été exécuté** — la fiche de qualification `{projet}-{lib}-{programme}-qualification-conv-{YYYYMMDD-HHmm}.md` existe et mentionne le sous-cas (A ou B), la catégorie (SIMPLE / STANDARD / COMPLEXE) et la stratégie retenue
- [ ] Les programmes identifiés comme nécessitant UC 7 avant UC 2 ont reçu leur optimisation préalable — aucune conversion Free RPG n'a été lancée sur un source avec des variables encore cryptiques sans décision explicite
- [ ] Chaque MOVE / MOVEL "À CONFIRMER" dans l'inventaire a fait l'objet d'une décision humaine documentée dans le diff — aucun cas à risque de troncature n'a été converti mécaniquement
- [ ] Chaque indicateur `*INxx` applicatif a été converti en variable `IND` avec un nom descriptif — aucun `*INxx` applicatif ne reste dans le source Free RPG final (exception : les indicateurs appartenant à une zone INDARA — display file, subfile, printer file — sont conservés en `*INxx` et documentés dans la fiche de qualification)
- [ ] Les indicateurs positionnés par les opcodes d'accès fichier ont été convertis en BIF (`%EOF`, `%OVERFLOW`, `%ERROR`) après identification de leur rôle réel dans les F-specs et C-specs — aucune hypothèse a priori sur leur rôle n'a été faite ; les indicateurs de niveau de cycle (`L1-L9`, `MR`, `LR`, `RT`) ne s'expriment pas directement en BIF — leur conversion nécessite une logique explicite ou le traitement UC08 (Prompt 4 de UC 8)
- [ ] Chaque P-spec RPG IV (PB/PE) a été convertie en `DCL-PROC` / `END-PROC` avec la même interface de paramètres (si Sous-cas A avec procédures)
- [ ] La décision sur le cycle RPG actif est documentée dans la fiche de qualification : converti (UC 8 Prompt 4 appliqué) ou conservé avec `*INLR` explicite
- [ ] Chaque programme converti a été **compilé sans erreur** sur l'IBM i de test (Prompts 3-bis, 4-bis)
- [ ] Chaque programme converti a été **testé fonctionnellement** sur l'IBM i de test avec un jeu de données réel — les résultats avant et après conversion sont identiques
- [ ] Le plan de conversion (Prompt 5) est produit et sauvegardé dans `{projet}-{lib}-{lib}-plan-conv-rpg-{YYYYMMDD-HHmm}.md` — tous les programmes du périmètre sont listés avec leur sous-cas, catégorie et statut
- [ ] Les diffs de conversion sont sauvegardés avec la convention `{projet}-{lib}-{programme}-rpg-converti-{YYYYMMDD-HHmm}.md` dans le workspace ET publiés sur Confluence (si MCP disponible) — ce fichier est l'**input obligatoire de UC 13** (tests de non-régression)
- [ ] La mention `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` est présente dans l'en-tête de chaque fichier diff de conversion

---

## Points à compléter avant passage en production

> Ces points ne bloquent pas le POC — ils concernent des cas avancés peu probables sur les programmes pilotes. Ils deviennent critiques dès que la conversion s'étend à l'ensemble du parc applicatif en production.

### Conversion du cycle RPG dans les programmes RPG III batch lourds

**Contexte :** les programmes batch de traitement de masse en RPG III utilisent souvent le cycle RPG avec plusieurs niveaux de contrôle (L1-L3) pour produire des récapitulatifs par clé et sous-clé (par exemple : total par client et sous-total par région). La conversion de ces cycles en boucles `DOW/READ/ENDDO` avec détection de rupture explicite modifie la structure d'exécution principale du programme — c'est la transformation la plus délicate de UC 2. Dans le POC, les programmes pilotes RPG III ont été choisis sans cycle de rupture multi-niveaux.

**Pourquoi absent de la fiche POC :** les pilotes UC 2 ont au plus un niveau de contrôle simple (L1 uniquement). Les ruptures multi-niveaux (L1-L3) avec totaux intermédiaires accumulés à chaque niveau nécessitent plusieurs variables de contrôle — la conversion dépasse la durée d'une session POC.

**À faire avant production :** définir un Prompt 4-RPG3-cycle dédié aux programmes avec cycle multi-niveaux, en s'appuyant sur le Prompt 4 de UC 8 comme base. Tester ce prompt sur un programme représentatif avant de l'appliquer au périmètre complet.

> ⚠️ **Signal d'alerte sur le terrain :** si le Prompt 0 retourne `Cycle RPG actif = OUI` avec `Niveaux L1-L9 = plusieurs niveaux détectés` — arrêter et planifier une session dédiée UC 8 Prompt 4 avant de continuer UC 2. Ne pas tenter la conversion de cycle multi-niveaux dans la même session que la conversion syntaxique.

### Programmes en mode mixte ayant déjà subi des conversions partielles

**Contexte :** certains programmes IBM i ont été partiellement modernisés au fil des années — quelques subroutines en Free RPG, le reste encore en format fixe. Ces programmes "en mode mixte" compilent sans avertissement, mais la conversion Free RPG de la partie encore en fixe peut interagir avec des déclarations Free RPG déjà présentes (noms en conflit, D-specs en double). Dans le POC, les pilotes sont soit entièrement en fixe soit entièrement déjà en Free.

**Pourquoi absent de la fiche POC :** les programmes pilotes ont un style cohérent — soit colonné, soit Free. Les programmes en mode mixte sont fréquents en production mais plus complexes à qualifier.

**À faire avant production :** ajouter dans le Prompt 0 une détection explicite du mode mixte (présence de `DCL-S` ou `DCL-F` côte à côte avec des D-specs en colonné). Générer l'inventaire de la section déjà Free séparément pour éviter les conversions redondantes.

> ⚠️ **Signal d'alerte sur le terrain :** si le Prompt 0 retourne `Style RPG détecté : Mixte` — isoler d'abord les sections déjà en Free RPG avant de lancer le Prompt 1. Ne pas inclure les sections Free existantes dans l'inventaire de conversion.

---

*Fiche UC 2 — Document évolutif à mettre à jour au fil du POC.*
