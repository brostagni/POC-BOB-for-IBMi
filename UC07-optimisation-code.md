# UC 7 — Amélioration, optimisation de code

> **Catégorie :** Modernisation du code
>
> **Priorité dans le POC :** 8 — suit UC 3 (accès SQL embarqué disponibles), peut être mené en parallèle avec UC 8 sur des programmes différents
>
> **Durée POC (avec Bob) :** 3 à 5 heures — optimisation d'un périmètre représentatif (3 à 5 programmes), itérations et tests de non-régression
>
> **Durée PROD (avec Bob) :** 45 min à 2 heures / programme — qualification, génération du diff d'optimisation, revue développeur, test fonctionnel
>
> **Durée PROD (sans Bob) :** 1 à 3 jours / programme — lecture complète du source, identification manuelle des variables mal nommées, des opcodes obsolètes, des procédures à extraire, refactoring itératif, tests
>
> **Gain Bob estimé :** ~5× — un programme de 500 lignes avec 30 variables à renommer et 5 procédures à extraire traité en une demi-journée au lieu de 2 à 3 jours ; gain plus fort sur les programmes avec nomenclature ancienne non documentée
>
> **Mode Bob recommandé :** IBM i Developer (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : Ask pour l'analyse/génération, Agent pour la compilation et la sauvegarde.

> ⚠️ **UC 7 et UC 8 ne se lancent jamais dans la même session Bob, ni via la même demande en mode Plan.** UC 7 d'abord, compiler et valider fonctionnellement, puis ouvrir une **nouvelle conversation** pour UC 8. Si les deux sont demandés ensemble, Bob mélange les passes de renommage (UC 7) et d'extraction de modules (UC 8) dans un seul diff — les erreurs deviennent intraçables et la compilation produit plusieurs centaines d'erreurs. Voir aussi la règle correspondante dans la fiche UC 8.

---

## Objectif

Améliorer la **lisibilité, la maintenabilité et les performances** des programmes RPG IBM i sans modifier leur logique fonctionnelle.

**Ce UC est le moins risqué de la Phase 3 — Modernisation du code.** Il ne change pas ce que fait le programme, seulement comment il le fait. Les résultats fonctionnels doivent être identiques avant et après. C'est le critère de succès principal, et le critère de succès unique qui distingue ce UC d'UC 8 (restructuration).

**Ce UC diffère de UC 3 (SQL embarqué) et de UC 8 (restructuration) :**
- UC 3 change le mécanisme d'accès aux données — les opcodes natifs deviennent du SQL. La logique reste la même, le mécanisme change.
- UC 7 change la forme interne du code — les noms, la lisibilité, la localité des responsabilités. Le mécanisme d'accès reste le même, la forme change.
- UC 8 change la structure du programme — décompose, réorganise, extrait des modules. La forme et la structure changent, avec risque plus élevé.

**L'ordre recommandé sur un même programme : UC 7 d'abord, UC 8 ensuite.** Optimiser un programme avant de le restructurer évite de porter des noms cryptiques et des redondances dans la nouvelle architecture.

**Livrable attendu :** Pour chaque programme optimisé : un fichier de diff documentant chaque modification (renommage, extraction, suppression de redondance, optimisation de performance), et optionnellement le source RPG modifié si la complexité le justifie.

**Convention de nommage des fichiers générés :**
```
{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md

Types pour cet UC :
  qualification-optim  → fiche de qualification (Prompt 0) — catégorie retenue + séquence
  diff-optim           → diff des modifications appliquées (Prompts 1, 2, 3, 4)
  plan-optim           → plan d'optimisation pour un périmètre applicatif complet (Prompt 5)
```

Exemples :
```
acme-APPVTE-GESCMD-qualification-optim-20250619-0900.md   ← qualification + stratégie
acme-APPVTE-GESCMD-diff-optim-20250619-1100.md            ← diff des modifications
acme-APPVTE-APPVTE-plan-optim-20250619-0800.md            ← plan périmètre complet
```

> 💡 Cette convention est valable en dehors du contexte POC — réutilisable en production tel quel.

---

## Démarrer par un programme que vous connaissez

> **Recommandation forte avant d'aborder les programmes critiques de ACME.**

Commencer par un **programme dont un développeur de l'équipe connaît bien la logique** — idéalement un programme avec des variables aux noms cryptiques connus de l'équipe, dont le comportement attendu est documenté ou testable rapidement.

Pourquoi ? Parce que la première optimisation sert à **calibrer deux choses** :
- La qualité des renommages proposés par Bob (est-ce que `WK01` renommé en `wkNomClient` reflète vraiment ce que contient la variable ?)
- La capacité de l'équipe à valider la non-régression (sur ce programme simple, les tests sont rapides — si Bob a introduit une régression, elle est détectée avant d'aller sur les programmes critiques)

Si la première optimisation est validée fonctionnellement, la confiance est établie pour les programmes plus complexes.

**Progression recommandée :**

> 💡 Pour chaque programme de cette progression, **commencer par le Prompt 0** — il donne la catégorie (SIMPLE / STANDARD / COMPLEXE) et la séquence exacte à suivre. Ne pas aller directement au Prompt 1.

| Étape | Programme à choisir | Catégorie attendue | Objectif |
|-------|--------------------|--------------------|---------|
| 1 | Programme court (< 200 lignes) avec renommages évidents et aucune performance critique | SIMPLE | Calibrer la qualité des renommages ; valider que le Prompt 0 classe correctement |
| 2 | Programme avec plusieurs subroutines extractibles et quelques duplications de code | STANDARD | Valider l'extraction de procédures et la détection de redondances |
| 3 | Programme avec des opcodes obsolètes (MOVE, MOVEL, SETON, indicateurs) et des performances identifiées | STANDARD à COMPLEXE | Valider la conversion des opcodes anciens et les règles de décision sur les indicateurs |
| 4 | Programme critique de production avec nomenclature opaque sur l'ensemble du source | COMPLEXE | Valider la gestion par passes et les garde-fous de non-régression sur un programme à fort enjeu |

---

## Impact de la taille du programme sur la stratégie d'optimisation

La complexité d'UC 7 ne dépend pas du nombre de lignes total du programme, mais du **nombre de points d'optimisation identifiés**, de leur **nature** (renommage sans risque vs extraction de procédure avec impact sur les appelants), et du **niveau de documentation disponible** (un programme bien documenté en UC 4 et UC 5 est plus facile à optimiser en toute sécurité).

Les vrais facteurs qui compliquent l'optimisation :
- Les **variables à noms cryptiques corrélées entre plusieurs subroutines** : renommer `WK01` en `wkNomClient` est simple si la variable n'est utilisée que dans une subroutine — c'est risqué si elle est passée en paramètre entre plusieurs sous-routines avec des utilisations différentes dans chacune
- Les **opcodes obsolètes avec effets de bord sur les indicateurs** (`SETON *IN50`, `SETOF *IN50`) : désactivés dans le code modernisé mais parfois testés dans une autre branche du même programme
- Les **subroutines extractibles qui partagent des variables globales** : une subroutine qui lit une variable globale ne peut pas être extraite en procédure indépendante sans adapter les paramètres
- Les **redondances de code dupliqué dans plusieurs programmes** : l'extraction de code commun en procédure partagée touche plusieurs programmes à la fois — dépasse le périmètre d'UC 7 sur un seul programme
- Les **optimisations de performance** qui nécessitent de comprendre les volumes de données : une boucle `READ` sur un fichier de 100 000 enregistrements mérite un curseur SQL paginé (UC 3) plutôt qu'un simple refactoring

### Programmes SIMPLE — nomenclature à améliorer, pas d'extraction, pas d'indicateurs complexes

Bob gère sans difficulté. **Séquence : Prompt 0 → Prompt 1 → Prompt 2 (renommages) → Prompt 2-bis (compilation) → Prompt 3 (opcodes obsolètes si applicable).**

> 💡 Le Prompt 0 confirme la catégorie SIMPLE et recommande directement cette séquence — pas de décision manuelle requise.

### Programmes STANDARD — renommages + extractions de procédures ou redondances

Le Prompt 0 identifie les blocs extractibles et les dépendances entre subroutines. **Séquence : Prompt 0 → Prompt 1 → Prompt 2 (renommages) → Prompt 2-bis → Prompt 3 (opcodes) → Prompt 3-bis → Prompt 4 (extractions).**

> 💡 Ne pas mélanger les renommages et les extractions dans une même passe — les renommages d'abord, les extractions ensuite. Un source avec des variables correctement nommées est plus facile à restructurer en procédures.

### Programmes COMPLEXE — indicateurs omniprésents, variables globales partagées, ou performance critique

Un programme COMPLEXE en UC 7 est souvent un programme qui a besoin d'UC 8 (restructuration) avant une optimisation de surface. Le Prompt 0 pose explicitement la question : **faut-il restructurer avant d'optimiser ?** Si le Prompt 0 conclut "restructuration recommandée", ne pas démarrer UC 7 en profondeur — ouvrir une issue UC 8 d'abord, ou limiter UC 7 aux renommages sûrs uniquement.

Si la décision est d'optimiser sans restructurer : **Séquence : Prompt 0 → Prompt 1 → Prompt 2 (renommages sûrs uniquement, passe par passe) → Prompt 2-bis après chaque passe → Prompt 3 (opcodes isolés) → Prompt 3-bis.**

> ⚠️ Sur un programme COMPLEXE, une optimisation globale en une seule passe produit un source avec des renommages partiels, des indicateurs peut-être manqués, et des extractions qui cassent des appels existants. Le Prompt 0 découpe toujours le travail par zones de risque.

---

## Démarrer une session Bob

> **À lire avant chaque session UC 7 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

Chaque session de travail sur un programme RPG doit démarrer dans une **nouvelle conversation Bob** (bouton `+` en haut du panneau Chat). Ne pas réutiliser une conversation d'un programme précédent — les renommages et décisions de l'autre programme polluent les suggestions du programme courant.

**Mode à sélectionner :** `IBM i Developer`

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Avant de lancer le Prompt 0, ouvrir dans l'éditeur Bob le programme RPG à optimiser. L'ouverture dans l'éditeur le rend accessible au MCP IBM i sans copier-coller.

**Procédure :** dans le panneau **IBM i — Object Browser** (extension Code for IBM i), naviguer jusqu'à la bibliothèque source, faire un clic droit sur le membre RPG → **Open in Editor**.

Fichiers à ouvrir pour chaque session UC 7 :
- Le programme RPG source (`[NOM_LIB]/QRPGSRC([NOM_PROGRAMME])`)
- Le fichier de compréhension du programme (`*-comprehension-*.md`) depuis UC 4
- Le fichier des règles métier du programme (`*-regles-*.md`) depuis UC 5

### 3. Fichiers de contexte à charger

Ces fichiers produits par les UC précédents doivent être disponibles dans le workspace Bob **avant** de démarrer. Utiliser **Add File to Chat** (icône trombone) ou les ouvrir dans l'éditeur.

| Fichier | Produit par | Obligatoire / Recommandé |
|---------|-------------|--------------------------|
| `{appArcad}-{fonction}-{composant}-comprehension-{date}.md` | UC 4 | **Obligatoire** — carte des subroutines et des variables, base du Prompt 1 |
| `{appArcad}-{fonction}-{composant}-regles-{date}.md` | UC 5 | **Obligatoire** — règles métier du programme, garde-fou contre les régressions |
| `{appArcad}-{fonction}-{composant}-spec-tech-{date}.md` | UC 6 | **Recommandé** — interfaces du programme, indispensable si des extractions de procédures touchent les paramètres |
| `{appArcad}-{fonction}-{fonction}-matrice-{date}.md` | UC 6 | **Recommandé** — si un renommage de variable est partagé entre programmes |
| `{appArcad}-{fonction}-{composant}-qualification-optim-{date}.md` | UC 7 (session précédente) | **Si reprise** — catégorie et stratégie déjà décidées, inventaire en cours |
| `{appArcad}-{fonction}-{composant}-sql-embarque-{date}.md` | UC 3 | **Si UC 3 appliqué** — source déjà converti en SQL embarqué, pour ne pas retoucher les accès convertis |

> 💡 Si les fichiers de compréhension (UC 4) ne sont pas disponibles, démarrer par une session rapide d'UC 4 sur le programme ciblé — 30 minutes de compréhension Bob évitent de renommer une variable qui a deux sens selon la subroutine.

> ⚠️ **Risque de réduction de contexte — sauvegarde intermédiaire recommandée :** une session UC 7 avec plusieurs prompts consécutifs peut atteindre la limite de contexte sur les programmes STANDARD ou COMPLEXE. Si Bob semble oublier une décision prise au Prompt 0 (catégorie, zones à risque, périmètre retenu), c'est un signal de compression de contexte. Sauvegarder le livrable en cours en mode Agent après chaque prompt majeur — pas seulement en fin de session. À chaque reprise de passe, commencer le prompt par : "Le fichier [NOM_FICHIER] contient les décisions prises — continuer à partir de [ÉTAPE]."

> 💡 **Reprise de session :** si l'optimisation d'un programme est interrompue, ouvrir le fichier `*-qualification-optim-*.md` déjà produit — Bob retrouve la catégorie et l'inventaire des points d'optimisation sans relancer le Prompt 0. Indiquer dans le prompt "la qualification est disponible dans [NOM_FICHIER] — reprendre à partir du Prompt [N°]".

---

## Prérequis

- Les fichiers `*-comprehension-*.md` (UC 4) des programmes à optimiser sont présents : ils fournissent la carte des subroutines, des variables et des appels — indispensable pour distinguer une variable locale (renommable sans risque) d'une variable partagée (à traiter avec précaution)
- Les fichiers `*-regles-*.md` (UC 5) sont disponibles : ils documentent les règles métier portées par le programme — garantie que les modifications ne touchent pas la logique fonctionnelle
- Les fichiers `*-spec-tech-*.md` (UC 6) sont disponibles (recommandé) : ils décrivent les interfaces et les paramètres des programmes — si une extraction de procédure touche les interfaces, la spec technique est le référentiel
- Les fichiers `*-matrice-*.md` (UC 6) sont disponibles (recommandé) : si un renommage de variable est propagé entre plusieurs programmes, la matrice identifie les impacts
- IBM i MCP actif (lecture et écriture des sources RPG pour le Prompt 2-bis)
- Accès à l'IBM i de test pour compiler et tester les programmes optimisés
- Un développeur RPG disponible pour valider la non-régression fonctionnelle

> 💡 Si les fichiers de compréhension (UC 4) ne sont pas disponibles, démarrer par une session rapide d'UC 4 sur le programme ciblé — 30 minutes de compréhension Bob évitent de renommer une variable qui a deux sens selon la subroutine.

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | **IBM i Developer** (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : **Ask** pour l'analyse/génération, **Agent** pour la compilation et la sauvegarde. |
| **Scope** | Library List → bibliothèque applicative ACME |
| **MCP actifs** | IBM i MCP (lecture des sources RPG, compilation de test) |
| **MCP différés** | IBM i Database MCP (si optimisations liées à des accès de données), Confluence MCP (publication, si token disponible) |

### Pourquoi le mode IBM i Developer pour la génération du diff d'optimisation ?

Les optimisations de code sont des modifications **précises et réversibles** — chaque renommage, chaque extraction de procédure doit être revue avant d'être appliquée. Le mode **IBM i Developer** apporte la connaissance RPG/ILE spécialisée pour toute la session. La discipline de travail repose sur **la validation humaine avant toute écriture** : Bob produit le diff complet dans le chat, l'équipe relit, questionne, corrige une dénomination proposée, puis autorise explicitement l'écriture sur l'IBM i.

> 💡 **Sans Premium Package IBM i :** remplacer IBM i Developer par le mode **Ask** pour les phases d'analyse et de génération, et le mode **Agent** pour la compilation et la sauvegarde. La discipline de validation reste identique.

| Phase | Comportement attendu | Ce que Bob fait |
|-------|----------------------|----------------|
| Qualification du programme (Prompt 0) | Génère dans le chat — pas d'écriture | Analyse le source, compte les points d'optimisation, détecte les facteurs de complexité, recommande la stratégie |
| Inventaire des optimisations (Prompt 1) | Génère dans le chat — pas d'écriture | Lit le source RPG via IBM i MCP, produit le tableau complet des points d'optimisation par catégorie |
| Renommages (Prompt 2) | Génère dans le chat — pas d'écriture | Produit le diff des renommages dans le chat — itérations possibles avant validation |
| Test de compilation après renommages (Prompt 2-bis) | **Écriture et exécution autorisées** — après validation de l'équipe | Lance `CRTBNDRPG` via IBM i MCP, rapporte les erreurs |
| Suppression des opcodes obsolètes (Prompt 3) | Génère dans le chat — pas d'écriture | Produit le diff de remplacement des opcodes dans le chat |
| Test de compilation après opcodes (Prompt 3-bis) | **Écriture et exécution autorisées** — après validation de l'équipe | Lance `CRTBNDRPG` via IBM i MCP, rapporte les erreurs |
| Extraction de procédures (Prompt 4) | Génère dans le chat — pas d'écriture | Produit la nouvelle structure procédurale dans le chat — itérations possibles |
| Test fonctionnel | **Humain uniquement** | Exécution sur IBM i de test, comparaison des résultats — non délégable à Bob |
| Sauvegarde du diff validé | **Écriture autorisée** — après validation section par section | Écrit le fichier `.md` dans le workspace |

> 💡 **Règle d'or pour UC 7 :** L'écriture et la compilation sur l'IBM i ne sont autorisées qu'après validation explicite de l'équipe pour chaque section. Pendant toute la phase de génération et d'itération (Prompts 0 à 4), Bob produit uniquement dans le chat — le mode IBM i Developer le permet, mais l'équipe ne donne pas l'instruction d'écrire.

> ⚠️ Ne jamais autoriser l'écriture pendant la phase d'analyse — une variable renommée incorrectement sera dans le source avant que l'équipe ait pu la valider.

### Intégration ARCAD

Le MCP ARCAD n'était pas disponible dans le contexte de ce POC de référence (version ARCAD non compatible avec le MCP). Si le MCP ARCAD est disponible dans votre environnement, les étapes manuelles de réintégration décrites ci-dessous peuvent être automatisées. N'hésitez pas à demander à Bob de modifier cette fiche UC en intégrant la disponibilité du MCP ARCAD.

**Impact sur UC 7 : faible.** Les optimisations de code modifient le source RPG dans les bibliothèques — IBM i MCP accède à ces bibliothèques indépendamment d'ARCAD.

| Sans MCP ARCAD (contexte de ce POC) | Avec MCP ARCAD disponible |
|--------------------------------------|---------------------------|
| Créer manuellement une tâche ARCAD pour chaque programme optimisé | Le MCP ARCAD peut créer la tâche et versionner automatiquement |
| Réintégration manuelle dans ARCAD après chaque session | IBM i MCP lit et compile les sources dans les bibliothèques ARCAD normalement dans les deux cas |
| Ajouter le placeholder `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` dans chaque diff | Le placeholder n'est plus nécessaire — la réintégration est pilotée par Bob |

> 💡 **Dans les deux cas :** avant de démarrer UC 7 sur un programme, vérifier dans ARCAD qu'il n'est pas en cours de modification par un autre développeur (promotion en cours). Charger la liste des objets verrouillés dans le contexte Bob pour éviter de travailler sur une version qui sera écrasée.

> ⚠️ **Écriture dans le fichier ARCAD ouvert — pas dans QSYS directement.** Quand le programme est ouvert depuis une version ARCAD (via Code for IBM i → Object Browser → clic droit → Open in Editor), Bob doit écrire les modifications dans **ce fichier déjà ouvert dans l'éditeur**, pas dans le membre `QSYS/QRPGSRC`. Si Bob propose d'écrire via un chemin `QSYS` absolu et affiche un WARNING, interrompre et préciser explicitement : *"Modifie le fichier actuellement ouvert dans l'éditeur — [NOM_LIB]/QRPGSRC([NOM_PROGRAMME]) — ne pas écrire dans QSYS directement."* Le mode **Ask** pendant la génération du diff empêche ce cas : Bob produit le diff dans le chat, et c'est l'équipe qui décide où et comment l'appliquer.

---

## Prompts clés

### Prompt 0 — Qualification et choix de stratégie

> **Ce prompt est le point d'entrée obligatoire de UC 7 pour chaque programme.**
> Il remplace la décision manuelle sur la stratégie (renommages seulement / renommages + extractions / optimisation complète).
> Il se lance **avant** le Prompt 1 — son résultat conditionne toute la séquence suivante.

```
Le programme [NOM_PROGRAMME] se trouve dans [NOM_LIB]/QRPGSRC.

Analyse ce programme et produis en français, en markdown, une fiche de qualification
pour l'optimisation du code :

## Qualification UC 7 — [NOM_PROGRAMME]

### 1. Inventaire rapide
| Indicateur                                        | Valeur |
|---------------------------------------------------|--------|
| Nombre de lignes de source                        | ?      |
| Nombre de subroutines (BEGSR/ENDSR)               | ?      |
| Nombre de procédures (DCL-PROC / P...B / P...E)   | ?      |
| Nombre de variables avec noms cryptiques (< 4 car ou format WKxx/Lxx/Nxx) | ? |
| Nombre d'indicateurs (*INxx utilisés)             | ?      |
| Présence d'opcodes obsolètes (MOVE, MOVEL, SETON, SETOF, GOTO, CABxx) | OUI / NON |
| Présence de code dupliqué détecté (blocs > 10 lignes répétés) | OUI / NON |
| Présence de subroutines extractibles en procédures | OUI / NON |
| Style RPG                                         | Fixe / Mixte / Free |

### 2. Facteurs de complexité détectés
Réponds par OUI / NON / À CONFIRMER pour chaque facteur :
- [ ] Variables globales partagées entre plusieurs subroutines (renommage à impact large)
- [ ] Indicateurs (*INxx) utilisés pour piloter la logique applicative (SETON/SETOF/IF *INxx)
- [ ] Opcodes GOTO ou CABxx (logique de branchement non structurée)
- [ ] Paramètres de programme (*ENTRY PLIST) — toute extraction de procédure doit préserver l'interface
- [ ] Accès natifs aux fichiers encore présents (UC 3 non complété sur ce programme)
- [ ] Programme appelé par d'autres programmes (matrice UC 6 disponible ?)
- [ ] Programme en format fixe ou mixte — les renommages doivent respecter les colonnes DDS

### 3. Catégorie et stratégie recommandée
Sur la base du comptage et des facteurs de complexité, conclure :

**Catégorie :**
- [ ] SIMPLE — renommages locaux, pas d'extraction, pas d'indicateurs pilotant la logique
- [ ] STANDARD — renommages + extraction de 1 à 3 procédures, ou opcodes obsolètes limités
- [ ] COMPLEXE — indicateurs omniprésents, variables globales inter-subroutines, ou GOTO/CABxx

> Note : UC 7 utilise SIMPLE / STANDARD / COMPLEXE (critère : nombre de points d'optimisation
> et nature des modifications). Même vocabulaire que UC 14 (critère : champs DDS) et UC 3
> (critère : opcodes d'accès) — vocabulaire unifié Phase 2 et Phase 3.
> Les échelles sont indépendantes : un programme SIMPLE en UC 7 a pu être COMPLEXE en UC 3.
> Le Prompt 0 de chaque UC calibre selon ses propres critères.

**Recommandation :**
- SIMPLE   → Séquence directe : Prompt 1 → Prompt 2 (renommages) → Prompt 2-bis → Prompt 3 (opcodes)
- STANDARD → Séquence par groupes : Prompt 1 → Prompt 2 passe par passe → Prompt 2-bis
              après chaque passe → Prompt 3 → Prompt 3-bis → Prompt 4 (extractions si applicable)
- COMPLEXE → Évaluer UC 8 (restructuration) avant UC 7 en profondeur.
              Si décision de limiter à l'essentiel : Prompt 1 → Prompt 2 (renommages sûrs seulement)
              → Prompt 2-bis. Reporter les extractions à UC 8.

**Si catégorie STANDARD ou COMPLEXE — zones de risque identifiées :**
| Zone | Nature du risque | Subroutine(s) concernée(s) | Précaution recommandée |

Signale clairement ce que tu ne peux pas déterminer sans exécuter le programme.
```

**Analyse ligne à ligne :**

- `## Qualification UC 7 — [NOM_PROGRAMME]` → titre normalisé dans la sortie. La fiche de qualification peut être sauvegardée directement — elle devient le fichier `*-qualification-optim-*.md`.

- `Inventaire rapide en tableau` → les indicateurs clés en un coup d'œil. Le comptage des variables cryptiques, indicateurs et opcodes obsolètes fournit une estimation objective du travail avant d'ouvrir le Prompt 1.

- `Facteurs de complexité OUI / NON / À CONFIRMER` → trois états. "À CONFIRMER" est délibéré — par exemple, Bob peut voir qu'une variable est utilisée dans trois subroutines, mais ne peut pas toujours déterminer si elle a la même sémantique dans les trois contextes sans exécuter le programme.

- `Accès natifs aux fichiers encore présents (UC 3 non complété)` → vérification de prérequis implicite. Si UC 3 n'a pas été appliqué sur ce programme, les opcodes natifs `CHAIN`/`READ`/`WRITE` seront présents et pourraient interférer avec les renommages de variables liées aux fichiers.

- `Programme appelé par d'autres programmes` → si une extraction de procédure crée un nouveau prototype (PR), l'interface du programme reste inchangée — mais si l'extraction modifie les paramètres exposés, les appelants doivent être recompilés. La matrice UC 6 est le filet de sécurité.

- `Catégorie SIMPLE / STANDARD / COMPLEXE` → les seuils adaptés à la réalité de UC 7 (différents de UC 3 où le critère est les opcodes d'accès, et de UC 14 où le critère est les champs DDS). Ces noms sont identiques dans tous les UC de Phase 2 et Phase 3 — vocabulaire unifié.

- `Recommandation avec séquence exacte de prompts` → la sortie du Prompt 0 est un plan d'action précis. L'équipe sait exactement quels prompts enchaîner et dans quel ordre.

> 💡 **Sauvegarder la fiche de qualification** (mode Agent) dès la fin du Prompt 0 :
> ```
> "Sauvegarde cette fiche de qualification dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-qualification-optim-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-qualification-optim-20250619-0900.md"
> ```
> Le Prompt 1 (inventaire détaillé) complète ensuite ce même contexte.

> ⚠️ **Piège évité :** sans le Prompt 0, l'équipe démarre les renommages sans savoir si le programme a des indicateurs pilotant la logique ou des variables globales inter-subroutines. Découvrir en milieu de passe qu'un `*IN50` est testé dans 8 endroits du programme oblige à tout réévaluer depuis le début.

---

### Prompt 1 — Inventaire des points d'optimisation

```
Le programme [NOM_PROGRAMME] se trouve dans [NOM_LIB]/QRPGSRC.

Sur la base de la qualification UC 7 que nous venons de faire,
produis en français, en markdown, l'inventaire complet des points d'optimisation :

## Inventaire UC 7 — [NOM_PROGRAMME]

### 1. Variables à renommer
Pour chaque variable avec un nom insuffisamment lisible :
| N° | Nom actuel | Type | Portée (locale / globale) | Utilisations (nb) | Nom proposé | Justification |
Critères : nom < 4 caractères, format WKxx / Lxx / Nxx / Axx, abréviation opaque,
nom identique à un autre dans un contexte différent.

### 2. Indicateurs (*INxx) à évaluer
| N° | Indicateur | Où il est activé (SETON/SETOF) | Où il est testé (IF/DOW) | Usage identifié | Action recommandée |
Action : renommer en variable booléenne / conserver si système / supprimer si mort |

### 3. Opcodes obsolètes à moderniser
| N° | Ligne | Opcode | Équivalent Free RPG recommandé | Risque de régression |
Opcodes à détecter : MOVE, MOVEL, SETON, SETOF, GOTO, CABxx, KLIST/KFLD,
SORTA, Z-ADD, Z-SUB, ADD, SUB, MULT, DIV, MVR, CASxx, EXSR (si extractible en CALLP)

### 4. Subroutines extractibles en procédures
Pour chaque subroutine candidate à l'extraction :
| N° | Nom BEGSR | Lignes | Variables utilisées | Variables globales dépendantes | Extractible ? |
Extractible : OUI si aucune variable globale, ou si les variables globales peuvent
              devenir des paramètres passés en valeur / référence
              NON si trop couplée à l'état global du programme

### 5. Code dupliqué détecté
Pour chaque bloc dupliqué (> 5 lignes répétées) :
| N° | Lignes occurrence 1 | Lignes occurrence 2 | Nature du bloc | Extractible en procédure ? |

Signale clairement les cas où la décision nécessite une connaissance fonctionnelle
non visible dans le source.
Ne pas générer le code optimisé dans ce prompt.
```

**Analyse ligne à ligne :**

- `Variables à renommer — Portée (locale / globale)` → la portée est la donnée de décision principale. Une variable locale à une subroutine peut être renommée sans risque avec un impact borné. Une variable globale partagée entre 5 subroutines doit être renommée partout en une seule passe — une passe partielle laisse le source dans un état incohérent.

- `Indicateurs (*INxx) à évaluer` → section dédiée aux indicateurs IBM i. Les indicateurs sont un mécanisme spécifique RPG qui n'a pas d'équivalent direct en Free RPG moderne. Un `*IN50` activé par un opcode et testé ailleurs peut piloter une logique critique — Bob doit identifier tous les sites d'activation et de test avant de recommander un renommage en variable booléenne.

- `Opcodes obsolètes à moderniser — Risque de régression` → la colonne "Risque de régression" est délibérée. Un `MOVE` vers un champ de longueur différente (troncature silencieuse) n'a pas le même risque qu'un `Z-ADD` (équivalent direct de `EVAL var = 0`). Bob doit calibrer le risque pour que l'équipe priorise les cas sûrs.

- `Subroutines extractibles — Variables globales dépendantes` → la dépendance aux variables globales est le critère discriminant. Une subroutine qui ne lit et n'écrit que des variables locales est extractible sans risque. Une subroutine qui lit et modifie une variable globale utilisée ailleurs dans le programme crée un couplage invisible — l'extraction nécessite de passer la variable en paramètre, ce qui change l'interface interne.

- `Ne pas générer le code optimisé dans ce prompt` → séparation inventaire / génération. L'inventaire est la feuille de route — l'équipe peut le relire, identifier les cas à ne pas toucher, et injecter ses décisions dans les prompts suivants.

> ⚠️ **Piège évité :** sans inventaire préalable, Bob renomme les variables au fil de la lecture et peut proposer deux noms différents pour la même variable selon le contexte où il la rencontre — créant une incohérence dans le diff.

---

### Prompt 2 — Renommage des variables et des indicateurs

```
Sur la base de l'inventaire UC 7 de [NOM_PROGRAMME] que nous venons de faire,
applique les renommages suivants :

[Lister ici les N° de variables du tableau Prompt 1, section 1, à renommer dans cette passe]
[Lister ici les N° d'indicateurs du tableau Prompt 1, section 2, à convertir]

Pour chaque variable renommée, produis le diff suivant :
// AVANT : [NOM_ACTUEL] — [JUSTIFICATION DU RENOMMAGE]
// APRÈS : [NOM_NOUVEAU]

Règles de renommage :
- Utiliser le style camelCase : wkNomVariable (préfixe selon le rôle : wk = travail,
  li = ligne/itération, fl = flag/indicateur, nb = compteur, dt = date)
- Nommer selon l'usage, pas selon le type (wkMontantHT et non wkDecimal12v2)
- **Format fixe ou mixte : limiter les noms à 10 caractères maximum** — en RPG fixe, les
  colonnes Facteur 1, Facteur 2 et Résultat ont des largeurs fixes. Un nom de plus de 10
  caractères provoque des erreurs de dépassement de colonne à la compilation. Si le programme
  est en style Fixe ou Mixte (vérifié au Prompt 0), les noms proposés doivent respecter cette
  contrainte : `wkMtHT` et non `wkMontantHorsTaxe`.
- Pour les indicateurs : créer une variable DCL-S [NOM_DESCRIPTIF] IND INZ(*OFF)
  et remplacer chaque occurrence de *INxx par cette variable
- Propager chaque renommage à **toutes** les occurrences dans le source — pas de renommage partiel
- Ne pas renommer les variables de paramètre *ENTRY PLIST sans validation explicite
- Ne pas renommer les champs de structures de données externées (EXTNAME) — ils référencent
  des colonnes de fichiers DDS dont le nom est contraint
- Signaler toute occurrence où le renommage ne peut pas être fait en toute sécurité

Produis à la fin un récapitulatif :
| Nom avant | Nom après | Nb occurrences modifiées | Risque résiduel |
```

**Analyse ligne à ligne :**

- `[Lister ici les N° de variables]` → référence explicite à l'inventaire. Travailler sur un sous-ensemble numéroté évite que Bob renomme des variables hors périmètre de la passe courante — surtout important pour les COMPLEXE où les passes sont délibérément limitées.

- `Préfixes recommandés (wk, li, fl, nb, dt)` → convention de nommage explicite. Sans convention, Bob peut choisir des noms différents pour des variables de même rôle — `workMontant` dans une subroutine et `wkMontant` dans une autre. La convention unifie le style.

- `Indicateurs : créer DCL-S [NOM] IND INZ(*OFF)` → la conversion des indicateurs RPG en variables booléennes `IND` est le remplacement recommandé en Free RPG. `IND` est un type IBM i spécifique pour les booléens — `*ON`/`*OFF` reste utilisable, mais la variable a un nom descriptif.

- `Propager à toutes les occurrences` → la cohérence est la règle absolue. Un source avec `WK01` dans certaines lignes et `wkNomClient` dans d'autres est dans un état intermédiaire invalide — il compilera peut-être, mais sera illisible.

- `Ne pas renommer les variables de paramètre *ENTRY PLIST sans validation explicite` → les paramètres de l'interface externe d'un programme ont des noms qui peuvent être documentés dans d'autres systèmes (appels CL, procédures Java, API). Le renommage d'un paramètre ne change pas l'interface externe (le nom n'est pas exposé), mais peut créer de la confusion dans la documentation existante.

- `Ne pas renommer les champs de DS externées (EXTNAME)` → garde-fou. Les champs d'une DS externée ont le même nom que les colonnes du fichier DDS/DDL — les renommer dans la DS ne change pas le nom dans le fichier, mais crée une divergence entre la DS et sa définition. Si UC 3 a été appliqué, ces DS ont peut-être déjà été converties en variables hôtes — vérifier.

> 💡 **Sauvegarder ce diff** (mode Agent) :
> ```
> "Sauvegarde ce diff de renommages dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-diff-optim-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-diff-optim-20250619-1100.md"
> ```

> ⚠️ **Piège évité :** renommer une variable globale partiellement (seulement dans la subroutine qui la lit, oublier celle qui l'écrit) laisse le source dans un état incohérent qui compilera s'il n'y a pas de conflit de noms — mais avec deux noms pour une seule variable selon l'endroit du source.

---

### Prompt 2-bis — Test de compilation par Bob (après les renommages)

> **Ce que Bob peut faire :** lancer la compilation via IBM i MCP en mode **Agent** et rapporter les erreurs.
> **Ce que Bob ne peut pas faire :** valider que les renommages n'ont pas changé la sémantique du programme — c'est une vérification humaine irréductible.
> **Quand l'utiliser :** après chaque Prompt 2 (passe de renommages), avant de passer à la passe suivante ou au Prompt 3.

```
Le source RPG de [NOM_PROGRAMME] dans [NOM_LIB]/QRPGSRC a été modifié
avec les renommages de variables que nous venons de faire.

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
   Causes probables à vérifier : renommage partiel (occurrence manquée),
   conflit de noms (nouveau nom déjà utilisé), DS externée modifiée par erreur
3. Si compilation réussie : confirmer que les [N] renommages de cette passe
   compilent sans erreur
4. Rappeler que la compilation réussie ne garantit pas la non-régression fonctionnelle —
   le test fonctionnel sur l'IBM i de test reste obligatoire (vérification humaine)
```

**Analyse ligne à ligne :**

- `CRTBNDRPG` avec `OPTION(*EVENTF *LIST)` → les options qui produisent la liste de compilation complète. `*EVENTF` active le fichier d'événements pour Code for IBM i — les erreurs sont visibles directement dans l'éditeur au-dessus de la ligne en erreur.

- `Causes probables à vérifier` → les erreurs de renommage ont des patterns connus. Un `RNF0113` (symbole inconnu) indique une occurrence manquée du renommage. Un `RNF0071` (définition dupliquée) indique un conflit de nom. Bob peut souvent diagnostiquer l'erreur depuis le code erreur sans lire le listing complet.

- `la compilation réussie ne garantit pas la non-régression fonctionnelle` → rappel obligatoire dans la sortie elle-même. La distinction compilation (automatisable par Bob) / test fonctionnel (validation humaine) doit être visible à chaque utilisation de ce prompt.

> 💡 **Ce prompt s'exécute en mode Agent** — Bob doit pouvoir lancer `CRTBNDRPG` via IBM i MCP. Vérifier que l'utilisateur IBM i associé au MCP a le droit `*USE` sur `CRTBNDRPG` et les droits d'écriture sur la bibliothèque cible de test.

> 💡 **En cas d'erreur de compilation :** revenir en mode Ask, corriger dans le chat, puis relancer ce prompt. La boucle "corriger → recompiler → valider" peut se faire entièrement dans Bob jusqu'à la compilation propre.

> ⚠️ **Ce prompt ne remplace pas le test fonctionnel.** Une compilation réussie confirme que les renommages sont syntaxiquement cohérents — pas que le programme se comporte identiquement à l'original. Les tests suivants restent obligatoires et ne peuvent pas être délégués à Bob :
> - Exécuter le programme sur l'IBM i de test avec un jeu de données réel
> - Comparer les résultats (enregistrements lus, calculés, produits) avec ceux du programme d'origine
> - Vérifier les cas limites pilotés par des indicateurs renommés

---

### Prompt 3 — Remplacement des opcodes obsolètes

```
Sur la base de l'inventaire UC 7 de [NOM_PROGRAMME], section 3 (opcodes obsolètes),
remplace les opcodes suivants par leur équivalent Free RPG :

[Lister ici les N° d'opcodes du tableau Prompt 1, section 3, à remplacer dans cette passe]

Pour chaque remplacement, produis le diff :
// AVANT : [OPCODE_OBSOLETE avec ses facteurs]
// APRÈS  : [ÉQUIVALENT FREE RPG]

Table de conversion à respecter :
- MOVE  [SRC] [DEST]        → EVAL  dest = src     (attention : MOVE tronque à droite,
                                                     EVAL affecte en entier — vérifier les longueurs)
- MOVEL [SRC] [DEST]        → EVAL  dest = src     (MOVEL aligne à gauche — même remarque)
- Z-ADD [VAL] [DEST]        → EVAL  dest = val
- Z-SUB [VAL] [DEST]        → EVAL  dest = -val
- ADD   [A] [B] [RESULT]    → EVAL  result = a + b
- SUB   [A] [B] [RESULT]    → EVAL  result = a - b
- MULT  [A] [B] [RESULT]    → EVAL  result = a * b
- DIV   [A] [B] [RESULT]    → EVAL  result = a / b  +  MVR rem → EVAL rem = %REM(a:b)
- SETON *INxx               → EVAL  [NOM_INDICATEUR] = *ON   (après renommage Prompt 2)
- SETOF *INxx               → EVAL  [NOM_INDICATEUR] = *OFF
- GOTO  [LABEL]             → Analyser le flux : IF/ELSEIF/ELSE ou DO/LEAVE/ITER
                              Ne pas remplacer un GOTO sans analyser la logique de branchement
- CABxx [A] [B] [LABEL]     → IF [A] xx [B]; [suite]; ENDIF — mêmes précautions que GOTO
- KLIST/KFLD                → Remplacer par une structure de recherche avec %KDS ou variables hôtes
- SORTA [ARRAY]             → Utiliser %SORTA si Free RPG, ou conserver si usage ponctuel

Règles :
- Signaler les cas où MOVE / MOVEL avec des longueurs différentes peut changer le comportement
- Ne pas remplacer un GOTO sans analyser le flux de contrôle complet de la subroutine
- Conserver le code d'origine en commentaire (// AVANT) pendant la session de validation
- Ne pas inventer de logique non visible dans l'opcode d'origine
```

**Analyse ligne à ligne :**

- `Table de conversion` → les équivalences sont explicites dans le prompt. Sans elles, Bob peut choisir des équivalents différents selon le contexte (parfois `EVAL`, parfois `%CHAR`, parfois une expression différente). La table force la cohérence.

- `MOVE / MOVEL — attention aux longueurs` → le point le plus risqué. `MOVE` en RPG fixe tronque silencieusement si la destination est plus courte que la source. `EVAL` en Free RPG affecte la valeur complète et peut produire un comportement différent si les longueurs diffèrent. Ce commentaire dans le prompt force Bob à signaler le cas et ne pas l'ignorer.

- `GOTO — ne pas remplacer sans analyser la logique` → les `GOTO` en RPG sont parfois utilisés pour des patterns de sortie de boucle ou de gestion d'erreur. Un `GOTO EOFRTNE` (aller à la routine de fin de fichier) ne se remplace pas par un simple `LEAVE` sans comprendre la suite. La règle explicite évite une conversion mécanique incorrecte.

- `Conserver le code d'origine en commentaire` → pendant la phase de validation, le développeur peut voir l'original et le nouveau côte à côte. Les commentaires `// AVANT` sont retirés une fois la validation terminée, dans le même fichier diff.

> 💡 **Sauvegarder ce diff** (mode Agent) — compléter ou créer un nouveau fichier selon si le Prompt 2 a déjà produit un diff pour ce programme :
> ```
> "Complète le fichier {appArcad}-{fonction}-{composant}-diff-optim-{YYYYMMDD-HHmm}.md
>  avec les remplacements d'opcodes de cette passe."
> ```

> ⚠️ **Piège évité :** remplacer `MOVE` par `EVAL` mécaniquement sans vérifier les longueurs des variables concernées est l'une des sources de régression les plus difficiles à détecter — le programme compile, les résultats sont presque corrects, mais une variable tronquée produit des montants incorrects dans 5 % des cas.

---

### Prompt 3-bis — Test de compilation par Bob (après remplacement des opcodes)

> **Ce que Bob peut faire :** lancer la compilation via IBM i MCP en mode **Agent** et rapporter les erreurs.
> **Ce que Bob ne peut pas faire :** valider que les équivalences d'opcodes sont sémantiquement correctes — vérification humaine irréductible notamment pour MOVE/MOVEL avec longueurs différentes.
> **Quand l'utiliser :** après chaque Prompt 3 (passe de remplacement d'opcodes), avant de passer à la passe suivante ou au Prompt 4.

```
Le source RPG de [NOM_PROGRAMME] dans [NOM_LIB]/QRPGSRC a été modifié
avec les remplacements d'opcodes que nous venons de faire.

Lance la compilation sur l'IBM i de test :
CRTBNDRPG PGM([NOM_LIB_TEST]/[NOM_PROGRAMME])
          SRCFILE([NOM_LIB]/QRPGSRC)
          SRCMBR([NOM_PROGRAMME])
          OPTION(*EVENTF *LIST)
          DBGVIEW(*SOURCE)

Analyse le résultat et produis en français :

1. Statut : COMPILATION RÉUSSIE / ERREURS DE COMPILATION
2. Si erreurs : liste des erreurs avec numéro de ligne, code erreur IBM i et description
   | Ligne | Code erreur | Description | Cause probable |
   Causes probables à vérifier : opcode partiellement converti (fin de bloc manquante),
   expression EVAL mal formée, indicateur non déclaré après remplacement SETON/SETOF
3. Si avertissements de troncature potentielle : les lister séparément
4. Si compilation réussie : confirmer que les [N] remplacements d'opcodes compilent
5. Rappeler que les cas MOVE/MOVEL avec des longueurs différentes doivent faire l'objet
   d'une validation fonctionnelle spécifique — ce test ne remplace pas la validation fonctionnelle

```

> 💡 **Ce prompt s'exécute en mode Agent** — mêmes prérequis de droits IBM i que le Prompt 2-bis.

> ⚠️ **Ce prompt ne remplace pas le test fonctionnel.** Les tests suivants restent obligatoires :
> - Exécuter le programme avec des valeurs qui déclenchent les opcodes remplacés
> - Vérifier en particulier les conversions MOVE/MOVEL sur des champs de longueurs différentes
> - Vérifier que les indicateurs convertis en variables booléennes sont correctement testés dans tous les contextes

---

### Prompt 4 — Extraction de procédures

```
Sur la base de l'inventaire UC 7 de [NOM_PROGRAMME], section 4 (subroutines extractibles),
extrait les subroutines suivantes en procédures :

[Lister ici les N° de subroutines du tableau Prompt 1, section 4, à extraire dans cette passe]

Pour chaque extraction, produis :

1. La déclaration de la nouvelle procédure (prototype) :
   DCL-PR [NOM_PROCEDURE] [TYPE_RETOUR] ;
     [PARAM1] [TYPE] [OPTIONS] ;
     ...
   END-PR ;

2. La définition de la procédure (à placer en fin de source) :
   DCL-PROC [NOM_PROCEDURE] ;
     DCL-PI [NOM_PROCEDURE] [TYPE_RETOUR] ;
       [PARAM1] [TYPE] [OPTIONS] ;
       ...
     END-PI ;
     [CORPS — code de l'ancienne BEGSR]
     RETURN [VALEUR_SI_APPLICABLE] ;
   END-PROC ;

3. Le remplacement de l'appel EXSR :
   // AVANT : EXSR [NOM_BEGSR]
   CALLP [NOM_PROCEDURE]([PARAMS]) ;

4. La suppression du bloc BEGSR/ENDSR d'origine (en commentaire pendant la validation)

Règles :
- Les variables globales dépendantes identifiées dans l'inventaire deviennent des paramètres
  (passés VALUE pour les entrées, CONST si non modifiées, sans qualificatif si modifiées)
- Nommer la procédure selon son rôle fonctionnel, en camelCase : calculerMontantTTC,
  validerDateCommande, chargerClientParCode
- La procédure doit être autonome : ne pas accéder à des variables globales du programme
  principal sauf si c'est inévitable et documenté dans le commentaire de la procédure
- Ne pas extraire une subroutine marquée "NON" dans l'inventaire (trop couplée à l'état global)
- Si la subroutine contient un retour conditionnel (GOTO vers la fin de la subroutine),
  remplacer par un RETURN anticipé dans la procédure
```

**Analyse ligne à ligne :**

- `DCL-PR / DCL-PI` → les deux déclarations obligatoires pour une procédure en Free RPG. `DCL-PR` est le prototype (interface visible par les appelants), `DCL-PI` est l'implémentation (interface interne). L'extraction doit produire les deux, dans les bonnes positions du source.

- `Variables globales → paramètres` → la règle de conception la plus importante de ce prompt. Une procédure qui accède à des variables globales est liée à l'état du programme — elle ne peut pas être testée isolément, ni réutilisée dans un autre programme. Passer les variables globales en paramètres est le premier pas vers une procédure autonome.

- `VALUE / CONST / sans qualificatif` → les trois modes de passage de paramètre en RPG. `VALUE` = copie en entrée (le paramètre ne peut pas être modifié dans l'appelant). `CONST` = référence en entrée non modifiable. Sans qualificatif = référence modifiable (équivalent d'un `INOUT`). Spécifier le mode dans le prompt évite que Bob choisisse le mode par défaut (sans qualificatif, donc tout modifiable) systématiquement.

- `Nommer selon le rôle fonctionnel` → cohérence avec les renommages du Prompt 2. La convention `camelCase` s'applique aussi aux procédures. Les verbes d'action (`calculer`, `valider`, `charger`) décrivent ce que fait la procédure — meilleur que des noms basés sur l'ancienne BEGSR (`CLCTOT` → `calculerTotalCommande`).

> 💡 **Sauvegarder ce diff** (mode Agent) — compléter le fichier diff existant :
> ```
> "Complète le fichier {appArcad}-{fonction}-{composant}-diff-optim-{YYYYMMDD-HHmm}.md
>  avec les extractions de procédures de cette passe."
> ```

> ⚠️ **Piège évité :** extraire une subroutine qui utilise `RETURN` ou `*INLR = *ON` pour terminer le programme principal. Dans une procédure, `RETURN` ne termine que la procédure — le `*INLR = *ON` reste valide, mais le `RETURN` qui était censé terminer le programme devient un simple retour de procédure. Toujours vérifier les instructions de fin de programme dans une subroutine avant de l'extraire.

---

### Prompt 5 — Plan d'optimisation pour un périmètre applicatif complet

```
Sur la base des fichiers de compréhension ({appArcad}-{fonction}-*-comprehension-*.md),
des règles métier ({appArcad}-{fonction}-*-regles-*.md) et des spécifications techniques
({appArcad}-{fonction}-*-spec-tech-*.md) pour l'application [NOM_APPLICATION] dans [NOM_LIB],
génère un plan d'optimisation en français, en markdown.

## Plan d'optimisation UC 7 — [NOM_APPLICATION]

### 1. Périmètre des programmes à optimiser
| Programme | Nb lignes estimées | Nb variables cryptiques | Opcodes obsolètes | Catégorie (S/St/C) | Priorité |
Catégorie : S = SIMPLE / St = STANDARD / C = COMPLEXE (critères UC 7)
Priorité : décroissante selon la combinaison risque faible + effort d'optimisation visible

### 2. Dépendances entre programmes
Les programmes qui partagent des variables ou des subroutines identiques (code dupliqué
sur le périmètre) : noter les cas où l'optimisation d'un programme affecte les autres.

### 3. Programmes à reporter à UC 8
Les programmes pour lesquels le Prompt 0 recommande une restructuration avant
optimisation — listés avec la justification.

### 4. Risques identifiés
Les 3 à 5 points d'optimisation les plus risqués de ce périmètre :
indicateurs à logique transversale, variables globales inter-programmes,
GOTO / CABxx complexes, MOVE avec troncature potentielle.

### 5. Estimation d'effort
| Programme | Prompts nécessaires | Durée estimée (avec Bob) | Gain lisibilité estimé |

Ne pas inventer de programmes ou de variables non visibles dans les sources disponibles.
```

> 💡 **Sauvegarder ce plan** (mode Agent) :
> ```
> "Sauvegarde ce plan d'optimisation dans un fichier nommé
>  {appArcad}-{fonction}-{fonction}-plan-optim-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-APPVTE-plan-optim-20250619-0800.md"
> ```

> 💡 Ce plan est le document de pilotage de UC 7. Il permet de suivre l'avancement programme par programme et d'identifier les programmes à reporter vers UC 8 avant de démarrer les optimisations.

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Ouverture des membres sources RPG, navigation Object Browser, compilation des sources optimisés et affichage des erreurs inline |
| **IBM i Languages** | Coloration syntaxique RPG Free et RPG fixe — indispensable pour lire et valider le code optimisé dans l'éditeur, en particulier les opcodes obsolètes encore présents |
| **Markdown All in One** | Prévisualisation des fichiers de diff et des plans d'optimisation sauvegardés |

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture des membres sources RPG (`QRPGSRC`) via `read_member` ; compilation des sources optimisés sur l'IBM i de test via `execute_compile_action` ou `execute_cl_command` (Prompts 2-bis, 3-bis) |
| **IBM i Database MCP** | Optionnel — utilisé si une optimisation identifiée concerne des accès aux données (vérifier qu'une boucle READ sur un gros fichier mérite plutôt un curseur SQL UC 3 qu'un simple refactoring) |
| **Confluence MCP** *(si disponible)* | Publication du plan d'optimisation et des diffs validés dans l'espace POC |

> 💡 **Requêtes QSYS2 utiles pour UC 7 (si IBM i Database MCP actif) :**
> ```sql
> -- Lister les programmes d'une bibliothèque (pour construire le plan Prompt 5)
> SELECT OBJNAME, OBJTYPE, OBJSIZE, LAST_USED_TIMESTAMP, TEXT_DESCRIPTION
> FROM QSYS2.OBJECT_STATISTICS
> WHERE OBJLIBRARY = '[NOM_LIB]' AND OBJTYPE = '*PGM'
> ORDER BY LAST_USED_TIMESTAMP DESC;
>
> -- Vérifier si un programme a été compilé récemment (indice de stabilité)
> SELECT PROGRAM_NAME, LAST_USED_TIMESTAMP, ACTIVATION_GROUP,
>        CREATION_TIMESTAMP, PROGRAM_LIBRARY
> FROM QSYS2.PROGRAM_INFO
> WHERE PROGRAM_LIBRARY = '[NOM_LIB]' AND PROGRAM_NAME = '[NOM_PROGRAMME]';
> ```
> Ces requêtes permettent de prioriser les programmes les plus actifs dans le plan d'optimisation.

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Sauter le Prompt 0 sur un programme COMPLEXE | Les indicateurs pilotant la logique applicative et les variables globales inter-subroutines ne sont pas identifiés — les renommages créent des incohérences à travers le source | Toujours lancer le Prompt 0 en premier — il prend 3 minutes et évite de perdre 2 heures |
| Renommer une variable partiellement (occurrences manquées) | Le source a deux noms pour la même variable — compile parfois, mais le résultat fonctionnel peut différer selon quelle occurrence a été lue | Le Prompt 2 exige la propagation à toutes les occurrences — utiliser le Prompt 2-bis pour détecter les incohérences compilateur |
| Remplacer MOVE par EVAL sans vérifier les longueurs | Troncature silencieuse sur des champs de longueurs différentes — montants incorrects, noms tronqués — le programme compile et produit des résultats presque corrects | Le Prompt 3 signale explicitement les MOVE à longueurs différentes — les isoler dans une passe dédiée et tester fonctionnellement |
| Extraire une subroutine avec un GOTO / RETURN de programme | Le RETURN dans la procédure ne termine plus le programme — le flux d'exécution continue après l'appel CALLP et peut produire un comportement inattendu | Le Prompt 1 (section 4) identifie les GOTO / instructions de fin de programme dans les subroutines candidates — ne pas extraire une subroutine marquée "NON extractible" |
| Optimiser un programme avant d'avoir les fichiers de compréhension UC 4 | Les renommages sont basés sur des suppositions de Bob, pas sur la connaissance fonctionnelle — une variable `WK01` renommée `wkMontantTTC` alors qu'elle stocke un compteur de lignes | Vérifier que `*-comprehension-*.md` existe pour chaque programme avant de démarrer |
| Convertir des opcodes obsolètes sur un programme avec accès natifs restants | Les opcodes d'accès natifs (CHAIN, READ…) peuvent interagir avec les indicateurs en cours de conversion — état imprévisible | Compléter UC 3 sur le programme avant de démarrer UC 7 en profondeur, ou limiter les conversions aux zones sans accès natifs |
| Travailler en mode Agent pendant la génération du diff | Bob peut sauvegarder des diffs intermédiaires avec des renommages partiels ou des opcodes à moitié convertis | Rester en mode **Ask** pendant toute la phase de génération (Prompts 0 à 4) — mode Agent uniquement pour Prompts 2-bis et 3-bis (compilation) et sauvegarde finale |
| Appliquer les optimisations sans test fonctionnel | Le programme compile et les résultats semblent corrects sur les cas nominaux — une régression sur un cas limite n'est découverte qu'en production | Tester chaque optimisation avec un jeu de données réel incluant les cas limites documentés dans `*-regles-*.md` |

---

## Check-list de validation UC 7

Avant de passer à UC 8 (voir `UC08-restructuration-code.md`) ou de déclarer un programme optimisé, valider chaque point :

- [ ] **Pour chaque programme optimisé : le Prompt 0 a été exécuté** — la fiche de qualification `{appArcad}-{fonction}-{composant}-qualification-optim-{YYYYMMDD-HHmm}.md` existe et mentionne la catégorie (SIMPLE / STANDARD / COMPLEXE) et la stratégie retenue
- [ ] Les programmes identifiés comme nécessitant UC 8 avant UC 7 ont été reportés — aucune optimisation profonde n'a été tentée sur ces programmes sans restructuration préalable
- [ ] Chaque variable renommée a été propagée à toutes ses occurrences — le Prompt 2-bis n'a pas retourné d'erreur liée à un renommage partiel
- [ ] Chaque indicateur `*INxx` pilotant la logique applicative a été converti en variable booléenne `IND` — aucun `*INxx` applicatif ne reste dans le source optimisé
- [ ] Chaque opcode obsolète a été remplacé par son équivalent Free RPG — les cas `MOVE/MOVEL` avec longueurs différentes ont fait l'objet d'une validation fonctionnelle spécifique
- [ ] Les subroutines extractibles ont été transformées en procédures (si catégorie STANDARD ou COMPLEXE avec validation explicite) — chaque procédure extraite est autonome, avec ses paramètres explicites
- [ ] Chaque programme optimisé a été **compilé sans erreur** sur l'IBM i de test (Prompts 2-bis, 3-bis)
- [ ] Chaque programme optimisé a été **testé fonctionnellement** sur l'IBM i de test avec un jeu de données réel — les résultats avant et après optimisation sont identiques
- [ ] Le plan d'optimisation (Prompt 5) est produit et sauvegardé dans `{appArcad}-{fonction}-{fonction}-plan-optim-{YYYYMMDD-HHmm}.md` — tous les programmes du périmètre sont listés avec leur catégorie et leur statut
- [ ] Les diffs d'optimisation sont sauvegardés avec la convention de nommage `{appArcad}-{fonction}-{composant}-diff-optim-{YYYYMMDD-HHmm}.md` dans le workspace ET publiés sur Confluence (si MCP disponible)
- [ ] La mention `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` est présente dans l'en-tête de chaque fichier diff

---

## Points à compléter avant passage en production

> Ces points ne bloquent pas le POC — ils concernent des cas avancés peu probables sur les programmes pilotes. Ils deviennent critiques dès que l'optimisation s'étend à l'ensemble du parc applicatif en production.

### Propagation des renommages entre programmes interdépendants

**Contexte :** lorsque deux programmes partagent une structure de données (`DS EXTNAME` commune) ou se passent des paramètres, un renommage dans l'un peut créer une divergence visuelle avec l'autre. Sur un périmètre de 50 programmes, les diffs d'optimisation de chaque programme doivent être cohérents entre eux. Dans le POC, les programmes pilotes sont traités de façon indépendante — la cohérence inter-programmes n'est pas vérifiée à ce stade.

**Pourquoi absent de la fiche POC :** les programmes pilotes ont été choisis avec des périmètres fonctionnels distincts, sans renommage de variables partagées entre programmes.

**À faire avant production :** ajouter une étape de consolidation dans le Prompt 5 (plan d'optimisation) : liste des variables partagées entre programmes et vérification que les renommages proposés sont cohérents sur l'ensemble du périmètre. Tester la recompilation de l'ensemble du périmètre après optimisation (pas seulement programme par programme).

> ⚠️ **Signal d'alerte sur le terrain :** si lors du Prompt 0 plusieurs programmes partagent les mêmes noms de variables (identifiés dans la matrice `*-matrice-*.md`), définir une convention de nommage unifiée pour le périmètre avant de démarrer — et s'y tenir pour tous les programmes du lot.

### GOTO et CABxx avec logique non structurée

**Contexte :** certains programmes legacy ont un usage extensif du `GOTO` pour implémenter des patterns de contrôle d'erreur ou des sorties anticipées de boucles. La conversion mécanique `GOTO → IF/LEAVE/ITER` n'est pas toujours directe — elle peut nécessiter une refonte de la structure de contrôle, qui dépasse le périmètre d'UC 7 et appartient à UC 8.

**Pourquoi absent de la fiche POC :** les programmes pilotes contiennent au plus quelques `GOTO` d'usage simple (sortie de subroutine ou gestion d'erreur en fin de programme). Les patterns `GOTO` complexes (sauts en avant, sauts entre subroutines) ne sont pas représentés dans les pilotes.

**À faire avant production :** ajouter dans le Prompt 0 une section dédiée à la cartographie des `GOTO` : identifier les patterns (sortie simple, gestion d'erreur, remplacement de sous-programme) et décider programme par programme si la conversion appartient à UC 7 (pattern simple) ou à UC 8 (restructuration nécessaire).

> ⚠️ **Signal d'alerte sur le terrain :** si le Prompt 1 retourne plus de 3 occurrences de `GOTO` dans un même programme, ou si un `GOTO` saute vers un label situé dans une autre subroutine — classer ce programme comme COMPLEXE et évaluer UC 8 avant de continuer UC 7.

---

*Fiche UC 7 — Document évolutif à mettre à jour au fil du POC.*
