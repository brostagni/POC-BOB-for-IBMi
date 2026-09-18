# UC 4 — Compréhension de code et d'applications

> **Catégorie :** Documentation
>
> **Priorité dans le POC :** 3 — premier UC "terrain" sur les données ACME
>
> **Durée POC (avec Bob) :** 2 à 4 heures — calibration des prompts sur 3 à 5 programmes représentatifs
>
> **Durée PROD (avec Bob) :** 20 à 30 min / programme — équipe rodée, prompts calibrés, génération + relecture
>
> **Durée PROD (sans Bob) :** 1 jour / programme — lecture manuelle, cartographie des dépendances, rédaction par un développeur senior
>
> **Gain Bob estimé :** ~15× — une application de 10 programmes documentée en 1 journée au lieu de 2 semaines
>
> **Mode Bob recommandé :** IBM i Developer (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : Ask.

---

## Objectif

Comprendre rapidement du code legacy IBM i inconnu ou peu documenté : rôle fonctionnel d'un programme, ses entrées/sorties, ses dépendances, sa logique de traitement. C'est le point d'entrée obligatoire avant toute modernisation.

**Livrable attendu :** Une fiche de compréhension par programme analysé — rôle, flux de données, dépendances, points d'attention — prête à servir de base aux UC 5, 6, 7 et 8.

**Convention de nommage des fichiers générés :**
```
{appArcad}-{fonction}-{composant}-comprehension-{YYYYMMDD-HHmm}.md

Exemple : acme-APPVTE-GESCMD-comprehension-20250615-1430.md
```
- `{appArcad}` : Application ARCAD — périmètre fonctionnel global (ex. `acme`)
- `{fonction}` : Fonction ARCAD — sous-ensemble cohérent de l'application (ex. `APPVTE`)
- `{composant}` : nom du composant IBM i traité — programme, fichier DDS ou objet SQL (ex. `GESCMD`)
- `comprehension` : type fixe pour les livrables de cet UC
- `{YYYYMMDD-HHmm}` : date et heure de génération — permet de conserver l'historique des analyses successives

> 💡 Cette convention est valable en dehors du contexte POC — réutilisable en production tel quel.

---

## Démarrer par un programme que vous connaissez

> **Recommandation forte avant de démarrer sur les programmes ACME.**

Avant d'analyser des programmes inconnus ou complexes, **commencer par un programme que l'équipe connaît bien** — idéalement un programme simple, fonctionnellement clair, dont au moins une personne dans la salle peut valider la sortie de Bob.

Pourquoi ? Parce que la première analyse sert à **calibrer la confiance** :
- Est-ce que Bob identifie correctement le rôle fonctionnel ?
- Est-ce qu'il manque des dépendances évidentes ?
- Est-ce que les "Questions ouvertes" sont pertinentes ou trop nombreuses ?
- Est-ce que le niveau de détail est utile pour l'équipe ?

Si la sortie sur un programme connu est juste, l'équipe peut ensuite aborder les programmes moins connus avec confiance. Si elle est approximative, il faut ajuster le prompt (plus de contexte, copybooks fournis, scope restreint) **avant** d'aller sur des programmes critiques.

**Progression recommandée :**

| Étape | Programme à choisir | Stratégie d'analyse | Objectif |
|-------|--------------------|--------------------|---------|
| 1 | Programme simple, bien connu de l'équipe (< 1 000 lignes) | Analyse directe — Prompt 1 tel quel | Calibrer Bob et valider la qualité de la sortie |
| 2 | Programme métier central, connu mais plus dense (1 000–3 000 lignes) | Analyse avec focalisation sur la structure | Vérifier que Bob tient sur des programmes avec logique métier complexe |
| 3 | Programme critique ou legacy peu connu (3 000–8 000 lignes) | Analyse en deux passes (cartographie puis détail ciblé) | Première analyse "réelle" avec filet de sécurité |
| 4 | Programme monolithique ou très volumineux (> 8 000 lignes) | Analyse par modules fonctionnels | Délimiter les zones pour les UC suivants (UC 7, UC 8) |

---

## Impact de la taille du programme sur la stratégie d'analyse

Avec le MCP IBM i actif, Bob accède aux sources via des appels outils ciblés — il ne charge pas l'intégralité du source en mémoire d'un coup comme un copier-coller. Cela lui permet de naviguer dans des programmes volumineux comme un développeur dans son éditeur. **La taille brute n'est donc pas le seul facteur limitant.**

Les vrais facteurs qui dégradent la qualité de l'analyse sont :
- La **densité de logique** : 800 lignes de calculs financiers imbriqués sont plus difficiles que 2 000 lignes d'I/O répétitifs
- Les **copybooks non résolus** : si les `/COPY` ne sont pas dans le scope, Bob travaille sur un programme incomplet
- Les **appels dynamiques** : `CALL` avec variable de nom de programme — invisibles statiquement
- Les **indicateurs partagés** entre subroutines (`*IN50`, `*IN51`…) — la logique devient globale et difficile à isoler

### Programmes < 1 000 lignes — Analyse directe

Bob gère sans difficulté. Le Prompt 1 s'applique tel quel, analyse complète et fiable en un seul échange.

```
→ Utiliser le Prompt 1 sans modification.
→ S'assurer que les copybooks /COPY sont dans le scope Library List.
```

### Programmes 1 000 à 3 000 lignes — Analyse avec focalisation sur la structure

Bob peut analyser l'ensemble mais une instruction de focalisation améliore la précision, en particulier sur les subroutines et les flux de contrôle :

```
Le programme [NOM_PROGRAMME] dans [NOM_LIB]/QRPGSRC fait environ [N] lignes.

Analyse ce programme et réponds en français en markdown :
1. Rôle fonctionnel global en 3 phrases (pour un non-technicien)
2. Langage et style (RPG III / RPG IV fixe / RPG Free / COBOL / CL)
3. Entrées et sorties : paramètres, fichiers, écrans
4. Liste des subroutines / procédures avec leur rôle supposé et leur déclencheur
5. Dépendances : CALL / CALLP externes, /COPY, fichiers partagés
6. Points d'attention : indicateurs partagés entre subroutines, logique conditionnelle complexe
7. Questions ouvertes : ce que tu ne peux pas déterminer sans contexte supplémentaire

```

> 💡 Sur un programme de cette taille, les subroutines (`EXSR`) sont souvent la clé de lecture. Bob les liste et les résume — ce qui donne une carte de navigation avant d'aller dans le détail.

### Programmes 3 000 à 8 000 lignes — Analyse en deux passes

Un programme de cette taille contient typiquement plusieurs domaines fonctionnels. La stratégie en deux passes produit de meilleurs résultats qu'un prompt unique :

**Passe 1 — Cartographie structurelle** (vue d'ensemble, sans le détail) :

```
Le programme [NOM_PROGRAMME] dans [NOM_LIB]/QRPGSRC fait environ [N] lignes.

Passe 1 — cartographie uniquement, ne pas entrer dans le détail du code :
1. Liste toutes les subroutines (EXSR / SR) avec leur nom et rôle supposé
2. Liste tous les fichiers déclarés en spécifications F (ou DCL-F en free)
3. Liste tous les CALL / CALLP vers des programmes externes
4. Identifie les structures de données principales (DS) et leur rôle probable
5. Décris le flux principal d'exécution en 5 à 10 étapes

Ne pas détailler le code de chaque subroutine dans cette passe.
```

**Passe 2 — Analyse ciblée sur les sections d'intérêt** :

```
Sur la base de la cartographie que nous venons de faire pour [NOM_PROGRAMME],
analyse en détail les subroutines suivantes : [SR1], [SR2], [SR3].

Pour chaque subroutine :
- Rôle fonctionnel précis
- Données lues et modifiées
- Logique conditionnelle et indicateurs utilisés
- Appels externes éventuels
```

> ⚠️ Même si Bob peut techniquement tout analyser en une passe, la stratégie deux passes produit une analyse plus précise et plus utile : la Passe 1 donne une carte, la Passe 2 explore les zones critiques identifiées.

### Programmes > 8 000 lignes — Analyse par module fonctionnel

Un programme de plus de 8 000 lignes est un **programme monolithique**. La stratégie change d'objectif : on ne cherche plus à tout comprendre en détail, on cherche à **délimiter les modules fonctionnels** pour les traiter indépendamment dans les UC suivants.

```
Le programme [NOM_PROGRAMME] dans [NOM_LIB]/QRPGSRC est très volumineux (environ [N] lignes).

Objectif : identifier les grandes zones fonctionnelles, pas analyser le détail.

1. Combien de subroutines / procédures / sections distinctes contient ce programme ?
2. Regroupe ces subroutines en 3 à 6 domaines fonctionnels cohérents
   (ex. : initialisation, saisie écran, calculs métier, accès DB, gestion erreurs, clôture)
3. Pour chaque domaine : liste les subroutines concernées, les fichiers utilisés,
   et une estimation de complexité (faible / moyenne / élevée)
4. Identifie les blocs de code candidats à l'extraction en programmes ou procédures autonomes

Produis un tableau : Domaine fonctionnel | Subroutines | Fichiers | Complexité | Extractable ?
```

> 💡 Ce prompt prépare directement UC 8 (Restructuration). Un programme de 8 000+ lignes est un candidat prioritaire à la décomposition — l'analyse exhaustive en une seule session n'est ni utile ni réaliste.

> ⚠️ Sur un programme de cette taille, prévoir plusieurs sessions. La première session produit la cartographie macro ; les sessions suivantes approfondissent chaque domaine fonctionnel identifié.

---

## Démarrer une session Bob

> **À lire avant chaque session UC 4 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

Chaque session d'analyse d'un programme doit démarrer dans une **nouvelle conversation Bob** (bouton `+` en haut du panneau Chat). Ne pas analyser deux programmes différents dans la même conversation — les informations du premier programme polluent la compréhension du second.

**Exception :** si on enchaîne l'analyse de plusieurs programmes **liés entre eux** (Prompt 2 — vue applicative), rester dans la même conversation exploite l'historique des dépendances.

**Mode à sélectionner :** `IBM i Developer`

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Avant de lancer le Prompt 1, ouvrir dans l'éditeur Bob le programme RPG à analyser. L'ouverture dans l'éditeur le rend accessible au MCP IBM i sans copier-coller.

**Procédure :** dans le panneau **IBM i — Object Browser** (extension Code for IBM i), naviguer jusqu'à la bibliothèque source, faire un clic droit sur le membre → **Open in Editor**.

Fichiers à ouvrir pour chaque session UC 4 :
- Le programme principal à analyser (`[NOM_LIB]/QRPGSRC([NOM_PROGRAMME])`)
- Les copybooks référencés via `/COPY` si présents dans le scope (sinon Bob travaille sur un programme incomplet)
- Les display files DDS si le programme est interactif (`[NOM_LIB]/QDDSSRCD([NOM_DSPF])`)

### 3. Fichiers de contexte à charger

UC 4 est le **premier UC terrain** — il n'y a pas de fichiers produits par des UC précédents à charger. Les seuls prérequis sont la configuration Bob (UC 15) et les MCP actifs (UC 12).

| Fichier | Produit par | Obligatoire / Recommandé |
|---------|-------------|--------------------------|
| *(aucun fichier de contexte préalable requis pour UC 4)* | — | — |
| `{appArcad}-{fonction}-{composant}-comprehension-{date}.md` | UC 4 (session précédente) | **Si reprise** — ouvrir la fiche existante pour compléter ou affiner sans relancer l'analyse depuis zéro |

> ⚠️ **Risque de réduction de contexte — sauvegarde intermédiaire recommandée :** une session UC 4 avec plusieurs prompts consécutifs peut atteindre la limite de contexte sur les programmes STANDARD ou COMPLEXE. Si Bob semble oublier une décision prise au Prompt 1 (catégorie, dépendances identifiées, questions ouvertes), c'est un signal de compression de contexte. Sauvegarder le livrable en cours en mode Agent après chaque prompt majeur — pas seulement en fin de session. À chaque reprise de passe, commencer le prompt par : "Le fichier [NOM_FICHIER] contient les décisions prises — continuer à partir de [ÉTAPE]."

> 💡 Si la session UC 4 est interrompue et reprise, ouvrir le fichier `*-comprehension-*.md` déjà produit dans l'éditeur. Indiquer dans le prompt "la fiche de compréhension de [NOM_PROGRAMME] est disponible dans [NOM_FICHIER] — compléter la section [SECTION]".

> 💡 **Lien direct UC 4 → UC 5 :** quand Bob liste les subroutines dans sa réponse au Prompt 1 (point 4 — subroutines), **noter cette liste immédiatement** — elle devient le guide de navigation pour l'extraction de règles métier en UC 5 (`UC05-logique-metier.md`).

---

## Prérequis

- UC 15 complété (maîtrise de Bob)
- UC 12 Phase 0 complété (IBM i MCP + IBM i Database MCP actifs)
- Accès aux bibliothèques sources de ACME dans Code for IBM i
- La bibliothèque applicative de ACME ajoutée à la Library List dans les paramètres Code for IBM i

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | **IBM i Developer** (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : **Ask**. |
| **Scope** | Library List → bibliothèque applicative ACME |
| **MCP actifs** | IBM i MCP (lecture membres sources) + IBM i Database MCP (vues QSYS2) |
| **MCP différés** | Confluence MCP (publication, si token disponible) |

### Pourquoi le mode IBM i Developer pour la compréhension de code ?

Le mode **IBM i Developer** apporte la connaissance RPG/CL/DDS spécialisée pour toute la session — vocabulaire des opcodes, connaissance des structures ILE, vues QSYS2, comportements des MCP IBM i. Sans ce mode, ces notions doivent être réexpliquées dans chaque prompt.

UC 4 est un UC **lecture seule** : Bob analyse le code et produit la documentation dans le chat. La discipline de validation repose sur la **relecture humaine avant sauvegarde** : Bob génère la fiche de compréhension dans le chat, le développeur valide le contenu, puis autorise explicitement l'écriture du fichier `.md` dans le workspace.

> 💡 **Sans Premium Package IBM i :** utiliser le mode **Ask** pour toute la session — la discipline de validation reste identique.

> 💡 **Règle d'or pour UC 4 :** IBM i Developer pour toute la session. Bob génère la documentation dans le chat — l'expert valide le contenu avant d'autoriser la sauvegarde du fichier `.md`. Aucune écriture IBM i dans cet UC.

> ⚠️ Ne jamais autoriser d'écriture sur l'IBM i pendant UC 4 — cet UC est exclusivement lecture et documentation. Si Bob propose une action `write_member`, refuser.

---

## Prompts clés

### Prompt 1 — Compréhension générale d'un programme

```
Le programme [NOM_PROGRAMME] se trouve dans la bibliothèque [NOM_LIB]/QRPGSRC.

Analyse ce programme et réponds en français aux points suivants, en markdown :
1. Rôle fonctionnel en 3 phrases maximum (pour un non-technicien)
2. Langage et style (RPG III / RPG IV fixe / RPG Free / COBOL / CL)
3. Entrées : paramètres reçus, fichiers lus, écrans utilisés
4. Sorties : paramètres retournés, fichiers écrits/mis à jour, impressions
5. Dépendances : programmes appelés (CALL), sous-programmes (EXSR), copybooks (/COPY)
6. Points d'attention : code complexe, indicateurs, logique conditionnelle lourde
7. Questions ouvertes : ce que tu ne peux pas déterminer sans contexte supplémentaire
```

**Analyse ligne à ligne :**

- `Le programme [NOM_PROGRAMME] se trouve dans [NOM_LIB]/QRPGSRC` → **ancrage objet explicite**. Sans cette ligne, Bob peut travailler sur le dernier fichier ouvert ou sur un programme homonyme dans une autre lib. Toujours nommer la bibliothèque complète.

- `réponds en français` → force la langue de travail. Sans ça, Bob répond parfois en anglais selon le contexte du mode actif.

- `en markdown` → le résultat est directement exploitable : copiable dans Confluence, dans un fichier `.md`, ou dans un ticket JIRA. Sans format imposé, Bob produit du texte brut difficile à réutiliser.

- `1. Rôle fonctionnel en 3 phrases maximum (pour un non-technicien)` → contraint la longueur et le niveau. Évite un résumé de 2 pages incompréhensible pour le métier. Ce point est le livrable immédiat pour le management.

- `2. Langage et style` → critique sur IBM i où plusieurs générations de RPG coexistent. Savoir si on a du RPG III ou du RPG IV colonné change complètement la stratégie de modernisation.

- `5. Dépendances` → le point le plus souvent oublié dans une analyse manuelle. Bob interroge les CALL, EXSR et /COPY dans le source. Sans ça, la modernisation casse les programmes appelants.

- `7. Questions ouvertes` → **ingrédient anti-hallucination**. Force Bob à signaler ce qu'il ne sait pas plutôt que d'inventer. Critique sur IBM i où les copybooks externes, les fichiers de données réels et les tables de paramétrage sont souvent invisibles dans le seul source.

> ⚠️ **Piège évité :** sans le point 7, Bob complète les zones d'ombre silencieusement avec des hypothèses plausibles mais fausses.

---

### Prompt 2 — Comprendre les dépendances entre programmes (vue application)

```
Dans la bibliothèque [NOM_LIB], analyse les relations entre les programmes 
suivants : [PROG1], [PROG2], [PROG3].

Pour chaque programme, identifie :
- Les programmes qu'il appelle (CALL, CALLP, EXSR vers sous-programmes externes)
- Les fichiers physiques et logiques qu'il lit ou modifie
- Les paramètres échangés entre programmes

Produis ensuite un diagramme Mermaid flowchart montrant les appels entre programmes,
avec les fichiers partagés comme nœuds intermédiaires.
Ne pas inventer de relations non visibles dans le code source.
```

**Analyse ligne à ligne :**

- `Dans la bibliothèque [NOM_LIB]` → scope restreint à la lib du client. Sans ça, Bob peut chercher dans d'autres bibliothèques du scope et polluer le résultat.

- `les programmes suivants : [PROG1], [PROG2], [PROG3]` → liste explicite. Ne pas dire "tous les programmes de la lib" — sur une bibliothèque réelle avec 200+ programmes, Bob sature la fenêtre de contexte et produit un résultat incomplet.

- `Les fichiers physiques et logiques qu'il lit ou modifie` → sépare lecture et écriture. Un programme qui lit `FCOMMANDES` et un autre qui l'écrit sont deux risques différents en modernisation.

- `un diagramme Mermaid flowchart` → format visuel directement rendu dans Bob (avec l'extension Mermaid Preview) et dans Confluence. Plus utile qu'une liste textuelle pour comprendre les flux.

- `Ne pas inventer de relations non visibles dans le code source` → **garde-fou anti-hallucination** explicite. Sur IBM i, les appels dynamiques (variable de nom de programme dans CALL) sont fréquents et invisibles statiquement — Bob doit les signaler, pas les inventer.

> ⚠️ **Piège évité :** sans la dernière ligne, Bob peut déduire des relations plausibles d'après les noms de programmes (ex. `GESCMD` appelle probablement `GESCMDL`) sans que ce soit visible dans le code — c'est une hallucination structurelle.

> 💡 **Variante pour une seule application :** remplacer la liste de programmes par `tous les programmes du menu [NOM_MENU]` si le point d'entrée applicatif est connu.

---

### Prompt 3 — Comprendre un programme sans en avoir le source (objet compilé uniquement)

```
Le programme [NOM_PROGRAMME] dans [NOM_LIB] n'a pas de source disponible
(objet compilé uniquement).

Utilise les informations disponibles pour me donner :
1. Date de création et dernière utilisation : QSYS2.OBJECT_STATISTICS
2. Langage de création : QSYS2.PROGRAM_INFO (attribut PROGRAM_TYPE)
3. Références à d'autres objets : exécute DSPPGMREF OUTPUT(*OUTFILE) sur ce programme
   et interroge le fichier de sortie pour lister les fichiers et programmes référencés
4. Si programme ILE : modules liés via QSYS2.BOUND_MODULE_INFO

DSPPGMREF est préférable aux vues QSYS2 pour les programmes OPM (RPG III / RPG400)
car ces programmes ne remontent rien dans BOUND_MODULE_INFO.

Signale clairement ce que tu ne peux pas déterminer sans le source.
```

**Analyse ligne à ligne :**

- `n'a pas de source disponible` → contextualise la contrainte dès le départ. Sans ça, Bob cherche le source et signale une erreur plutôt que de basculer sur une analyse par les objets compilés.

- `QSYS2.OBJECT_STATISTICS` / `QSYS2.PROGRAM_INFO` → les vues exactes à interroger. Les nommer évite que Bob utilise des vues dépréciées ou incorrectes.

- `DSPPGMREF OUTPUT(*OUTFILE)` → **commande CL clé pour les programmes OPM**. Contrairement à `QSYS2.BOUND_MODULE_INFO` qui ne couvre que les programmes ILE, `DSPPGMREF` fonctionne sur tous les types de programmes IBM i et liste les références aux fichiers, programmes et autres objets. C'est la source la plus fiable pour les vieux programmes RPG III / RPG400 — souvent les plus nombreux sur un IBM i de production.

- `DSPPGMREF est préférable pour les programmes OPM` → note explicative incluse dans le prompt lui-même. Évite que Bob utilise uniquement `BOUND_MODULE_INFO` et retourne un résultat vide sur un programme OPM sans signaler pourquoi.

- `Signale clairement ce que tu ne peux pas déterminer sans le source` → garde-fou anti-hallucination, ici encore plus critique car la source d'information est incomplète par nature.

> ⚠️ **Piège évité :** `QSYS2.BOUND_MODULE_INFO` et `QSYS2.PROGRAM_EXPORT_IMPORT_INFO` ne remontent **rien** pour les programmes OPM (RPG III, RPG400, COBOL OPM). Sur un IBM i de production avec des programmes des années 80-90, c'est la majorité du parc. `DSPPGMREF` est la seule commande fiable sur ces objets.

> ⚠️ **Piège évité :** ne jamais demander à Bob de "décompiler" un programme IBM i — ce n'est pas possible et Bob pourrait générer un source fictif qui ressemble au vrai sans l'être.

---

### Prompt 3-bis — Inventaire des programmes obsolètes (non recompilés depuis N ans)

> **Complément au Prompt 3** — ne nécessite pas l'absence de source. Utilisable sur toute bibliothèque applicative.

```
Dans la bibliothèque [NOM_LIB], identifie les programmes qui n'ont pas été recompilés
depuis [N] ans (à partir de la date d'aujourd'hui).

Utilise QSYS2.OBJECT_STATISTICS avec les filtres suivants :
- OBJECT_LIBRARY = '[NOM_LIB]'
- OBJECT_TYPE = '*PGM'
- OBJCREATED < CURRENT_DATE - [N] YEARS

Pour chaque programme identifié, produis un tableau avec :
| Nom du programme | Date de création (OBJCREATED) | Langage (PROGRAM_TYPE) | Âge estimé (années) |

Trier par OBJCREATED ascendant (les plus anciens en premier).
Signaler le nombre total de programmes dans la bibliothèque et le pourcentage considéré obsolète.
Ne pas conclure sur la pertinence de moderniser sans information métier supplémentaire.
```

**Pourquoi ce prompt est utile :**

- **Inventaire de modernisation rapide** — un IBM i de production héberge souvent des programmes compilés dans les années 80-90 qui n'ont jamais été retouchés. Ce prompt produit en 30 secondes un backlog priorisé pour UC 7/8/2.
- `QSYS2.OBJECT_STATISTICS` → vue la plus fiable sur les attributs des objets IBM i. `OBJCREATED` correspond à la date de compilation (ou de dernière recompilation) — pas à la date de création du source.
- `OBJECT_TYPE = '*PGM'` → restreint aux programmes compilés. Pour inclure les modules ILE, ajouter `OR OBJECT_TYPE = '*MODULE'`.
- `Signaler le nombre total et le pourcentage` → donne un ratio d'obsolescence immédiatement exploitable en réunion de bilan.

> 💡 **Variante** : remplacer `[N] ans` par une date fixe (ex. `OBJCREATED < DATE('2010-01-01')`) pour caler l'inventaire sur une date de référence connue (migration système, passage à OS400 V7, etc.).

> ⚠️ **Piège** : `OBJCREATED` est la date de **recompilation**, pas la date d'écriture du source. Un programme peut avoir été recompilé en 2015 avec un source des années 90 — il n'apparaîtra pas dans cet inventaire. Croiser avec `LAST_USED_TIMESTAMP` pour détecter les programmes qui n'ont jamais été utilisés récemment.

---

### Prompt 4 — Résumé exécutif pour le management

```
Sur la base de l'analyse du programme [NOM_PROGRAMME] que nous venons de faire,
rédige un résumé exécutif en français de 10 lignes maximum, destiné à un 
responsable informatique non-développeur.

Le résumé doit inclure :
- Ce que fait le programme en termes métier
- Son importance dans le système (critique / secondaire / autonome)
- Son état technique (moderne / legacy / à moderniser en priorité)
- Le risque si ce programme est modifié sans précaution

Ne pas utiliser de termes techniques RPG ou IBM i sans les expliquer.
```

**Analyse ligne à ligne :**

- `Sur la base de l'analyse que nous venons de faire` → s'appuie sur le contexte de la conversation en cours. Évite de re-soumettre le code — Bob utilise ce qui est déjà dans sa fenêtre de contexte.

- `10 lignes maximum` → contrainte de longueur ferme. Les résumés exécutifs non contraints produits par Bob sont souvent trop longs pour être lus par un manager.

- `destiné à un responsable informatique non-développeur` → calibre le vocabulaire. Bob adapte son niveau de langage à l'audience spécifiée.

- `Son importance dans le système (critique / secondaire / autonome)` → trois catégories simples que Bob peut évaluer depuis l'analyse précédente (nombre d'appelants, fréquence d'utilisation, dépendances).

- `Ne pas utiliser de termes techniques RPG ou IBM i sans les expliquer` → évite le jargon IBM i dans un livrable destiné au management. Critique pour les réunions de présentation du POC.

> 💡 **Quand l'utiliser :** ce prompt est idéal pour préparer la réunion de bilan de fin de Phase 1 avec le management de ACME. Produire un résumé exécutif par application analysée.

---

### Note — Enchaîner les prompts en conversation continue

UC 4 se pratique en **conversation continue**, pas en prompts isolés. Une fois le Prompt 1 envoyé et la réponse obtenue, on peut approfondir sans tout re-soumettre :

```
[Après la réponse au Prompt 1]
"Tu as mentionné que le programme appelle CALCREMISE.
 Peux-tu analyser ce programme de la même façon ?"

"Le point 6 (points d'attention) mentionne des indicateurs *IN50 et *IN51.
 Explique leur rôle précis dans la logique de ce programme."

"Reformule le point 1 (rôle fonctionnel) en insistant sur l'impact
 pour le processus de commande client."
```

> 💡 Bob conserve le contexte du programme analysé pendant toute la conversation. Exploiter cet historique évite de re-soumettre le code à chaque question — et économise des Bob Coins.

> 💡 **Sauvegarder la fiche de compréhension** : une fois la conversation finalisée, basculer en mode **Agent** et demander à Bob de sauvegarder le résumé dans un fichier en respectant la convention de nommage :
> ```
> "Sauvegarde la fiche de compréhension de ce programme dans un fichier markdown
>  nommé {appArcad}-{fonction}-{composant}-comprehension-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-comprehension-20250615-1430.md"
> ```

> ⚠️ Si la conversation dure plus de 20-30 échanges, ouvrir une nouvelle session et re-soumettre le programme pour repartir avec un contexte propre (voir UC 15, § 6.3).

> 💡 **Lien avec UC 5 :** quand Bob liste les subroutines en réponse au Prompt 1 (point 5 — Dépendances) ou dans la Passe 1 des programmes volumineux, **sauvegarder cette liste** — elle devient le guide de navigation pour l'extraction de règles métier en UC 5.

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Navigation Object Browser, ouverture des membres sources, DDS Previewer |
| **IBM i Languages** | Coloration syntaxique RPG/CL/DDS — indispensable pour lire le code dans l'éditeur |
| **Mermaid Preview** | Rendu visuel des diagrammes de dépendances générés par le Prompt 2 |
| **Markdown All in One** | Prévisualisation des fiches de compréhension générées en markdown |

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture des membres sources (QRPGSRC, QCLSRC, QDDSSRCD…) sans copier-coller |
| **IBM i Database MCP** | Interrogation QSYS2 pour les programmes sans source (Prompt 3), analyse d'impact |
| **Confluence MCP** *(si disponible)* | Publication directe des fiches de compréhension sur l'espace POC Confluence |

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Analyser sans définir le scope Library List | Bob analyse le fichier ouvert mais rate les copybooks et fichiers liés dans d'autres membres | Toujours définir le scope → Library List avant le premier prompt |
| Demander l'analyse de toute une bibliothèque en un prompt | Saturation de la fenêtre de contexte, résultat tronqué ou générique | Analyser programme par programme, ou par groupe fonctionnel de 3 à 5 max |
| Ne pas fournir les copybooks inclus via `/COPY` | Bob analyse le programme incomplet — les structures de données et constantes définies dans les copybooks sont invisibles | Ouvrir aussi les copybooks dans l'éditeur, ou les mentionner explicitement dans le prompt |
| Accepter un résultat sans le point "Questions ouvertes" | Bob comble les zones d'ombre par des hypothèses — risque d'erreur sur les UC suivants | Toujours inclure la demande de signalement des inconnues (voir Prompt 1, point 7) |
| Demander à Bob de "décompiler" un programme sans source | Bob peut générer un source fictif plausible mais faux | Utiliser le Prompt 3 (analyse via QSYS2) pour les programmes sans source |
| Travailler en mode Code ou Agent pour cet UC | Risque de modification accidentelle de sources | Rester en mode **Ask** pour tout UC de lecture/compréhension |
| Confondre fichiers logiques et fichiers physiques dans l'analyse | L'analyse des dépendances est faussée — un fichier logique n'est pas une table | Demander explicitement la distinction PF/LF dans les prompts d'analyse |

---

## Accélérateur Premium Package — Workflow "Business Rules Extraction"

Le Premium Package for i propose un workflow Bob dédié à l'extraction des règles métier :

> **Workflow : "Business Rules Extraction"**
> Disponible depuis le bouton `Start Workflow` en haut du panneau Chat Bob, en mode **Agent** ou **IBM i Developer**.

Ce workflow guide Bob à travers des étapes structurées pour analyser un membre QSYS et produire automatiquement un rapport de règles métier au format markdown, sauvegardé dans l'IFS. Il est plus reproductible qu'un prompt libre car il garantit la couverture des mêmes points sur chaque programme.

**Quand l'utiliser :** en complément du Prompt 1 (compréhension programme), sur les programmes avec une logique métier dense (calculs, validations, branchements conditionnels importants). Le workflow produit un fichier `business-rules-{NOM_PROGRAMME}-{date}.md` directement utilisable comme input de UC 5 (logique métier) et UC 6 (documentation).

**Configuration type :**
- `LIBRARY` : la bibliothèque QSYS contenant le source (ex. `APPVTE`)
- `SOURCE FILE` : fichier source (ex. `QRPGLESRC`, `QRPGSRC`)
- `MEMBER` : nom du membre à analyser (ex. `GESCMD`)

> 💡 Le workflow "Business Rules Extraction" est **complémentaire** aux prompts UC 4 — il ne remplace pas le Prompt 1 (compréhension structurelle) mais approfondit l'extraction des règles métier sur les programmes le justifiant. Sur les programmes simples, le Prompt 1 suffit.

---

## Référence complémentaire — Labs IBM

Ce UC est documenté et mis en pratique dans deux labs officiels IBM :

> **Lab 101 — Document SAMCO with Bob (Steps 1 à 4)**
> https://github.com/bmarolleau/IBM-i-Application-Modernization-with-Bob/blob/main/lab101-premium-discover-samco.md

Ce lab illustre UC 4 sur l'application SAMCO : exploration de la bibliothèque QSYS avec `search_qsys`, documentation programme-level avec `read_member`, génération de documentation fonctionnelle multi-programmes, et utilisation du workflow "Business Rules Extraction" (Step 4). Il montre concrètement la progression d'une analyse simple (Step 2 — programme seul) à une analyse transversale (Step 3 — fonction métier complète).

**Différence de contexte :** le lab utilise `SAMSRCn` (bibliothèque de démonstration). Pour ACME, remplacer les noms de bibliothèques et de membres par les équivalents QSYS de l'application cible — la logique de prompt est identique.

> **Lab FLIGHT400 — Exercise 1 & Exercise 5 (Code Explanation + System Queries)**
> https://github.com/bmarolleau/flight400-demo

L'Exercise 1 illustre UC 4 en deux temps : exploration de l'Object Browser et DDS Previewer (1a), puis génération d'une architecture overview avec diagramme Mermaid et ERD (1b — switch en IBM i Database mode pour la commande `/erd`). L'Exercise 5 montre l'usage des requêtes système en langage naturel, dont le Prompt 3-bis ("programmes non recompilés depuis N ans" via `QSYS2.OBJECT_STATISTICS`).

**Différence de contexte :** le lab utilise la bibliothèque `FLGHT4nn`. Pour ACME, remplacer les noms de bibliothèques par les équivalents QSYS de l'application cible — la logique des prompts est identique.

---

## Check-list de validation UC 4

Avant de passer à UC 5 (voir `UC05-logique-metier.md`), valider chaque point :

- [ ] Au moins 3 programmes représentatifs de ACME ont été analysés avec le Prompt 1 — choisir : 1 programme critique (beaucoup d'appelants ou fichiers partagés), 1 programme métier central, 1 programme technique ou utilitaire
- [ ] Un diagramme de dépendances (Prompt 2) a été produit pour au moins une application
- [ ] Les copybooks utilisés par les programmes analysés ont été identifiés
- [ ] Les programmes sans source ont été traités avec le Prompt 3 (si applicable)
- [ ] Un résumé exécutif (Prompt 4) a été produit pour au moins une application
- [ ] Les fiches de compréhension sont sauvegardées avec la convention de nommage `{appArcad}-{fonction}-{composant}-comprehension-{YYYYMMDD-HHmm}.md` dans le workspace ET publiées sur Confluence (si MCP disponible)
- [ ] Les "Questions ouvertes" identifiées par Bob ont été notées — elles alimenteront UC 5

---

*Fiche UC 4 — Document évolutif à mettre à jour au fil du POC.*
