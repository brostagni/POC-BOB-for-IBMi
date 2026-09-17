# UC 5 — Explication de code et extraction de la logique métier

> **Catégorie :** Documentation
>
> **Priorité dans le POC :** 4 — s'appuie sur les fiches produites en UC 4
>
> **Durée POC (avec Bob) :** 2 à 4 heures — extraction et validation sur 1 domaine métier représentatif
>
> **Durée PROD (avec Bob) :** 3 à 5 heures / domaine métier — génération Bob + réunion de validation avec expert métier
>
> **Durée PROD (sans Bob) :** 2 à 5 jours / domaine métier — ateliers d'extraction avec les développeurs seniors et le métier, rédaction et consolidation manuelle
>
> **Gain Bob estimé :** ~8× — les règles métier d'un domaine formalisées en une demi-journée au lieu d'une semaine d'ateliers
>
> **Mode Bob recommandé :** IBM i Developer (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : Ask.

---

## Objectif

Extraire la **logique métier enfouie dans le code** : règles de calcul, conditions, seuils, formules, validations, décisions — tout ce qui représente la connaissance fonctionnelle du client codée en dur dans le RPG, COBOL ou CL. Ces règles sont souvent non documentées ailleurs et constituent un risque majeur en cas de remplacement ou de modernisation.

**Ce UC va plus loin que UC 4** : UC 4 décrit *ce que fait* un programme, UC 5 extrait *pourquoi* et *comment* — les règles de gestion précises, utilisables par un analyste métier ou pour alimenter un cahier des charges.

**Livrable attendu :** Un catalogue de règles métier par domaine fonctionnel (calculs, validations, flux de décision), formulé en langage naturel, vérifiable par un expert métier, et directement réutilisable pour les UC de modernisation.

**Convention de nommage des fichiers générés :**
```
{appArcad}-{fonction}-{composant}-regles-{YYYYMMDD-HHmm}.md

Exemple : acme-APPVTE-GESCMD-regles-20250615-1600.md
```
- `{appArcad}` : Application ARCAD — périmètre fonctionnel global (ex. `acme`)
- `{fonction}` : Fonction ARCAD — sous-ensemble cohérent de l'application (ex. `APPVTE`)
- `{composant}` : nom du composant IBM i traité — programme, fichier DDS ou objet SQL (ex. `GESCMD`)
- `regles` : type fixe pour les livrables de cet UC (catalogue de règles métier)
- `{YYYYMMDD-HHmm}` : date et heure de génération

> 💡 Si le catalogue couvre un domaine fonctionnel entier (plusieurs programmes), remplacer `{composant}` par le nom du domaine : `acme-APPVTE-DOMAINE-VENTE-regles-20250615-1600.md`

---

## Démarrer par un programme dont vous connaissez les règles métier

Même principe que UC 4 : **commencer par un programme dont un expert métier de l'équipe ACME connaît les règles**. L'objectif est de vérifier que Bob extrait correctement les règles réelles — pas des règles plausibles mais fausses.

Idéalement, avoir en salle lors de cette session :
- Un développeur qui connaît le code
- Un référent métier qui peut valider que les règles extraites correspondent à la réalité opérationnelle

Si une règle extraite par Bob est fausse ou incomplète, corriger le prompt immédiatement (plus de contexte, copybooks fournis, paramétrage explicité) avant de passer aux programmes moins connus.

---

## Impact de la taille du programme — Renvoi à UC 4

Les stratégies d'analyse par taille définies en UC 4 s'appliquent intégralement ici — consulter la section **"Impact de la taille du programme sur la stratégie d'analyse"** de la fiche UC 4 pour les seuils et les prompts adaptés (< 1 000 lignes, 1 000–3 000, 3 000–8 000, > 8 000 lignes).

**Deux différences importantes par rapport à UC 4 :**

**1. L'extraction de règles est plus sensible à la taille que la compréhension générale.**
UC 4 peut produire une vue d'ensemble acceptable même sur un programme volumineux avec une seule passe. UC 5 nécessite une lecture fine du code — une règle mal lue est une règle fausse dans le catalogue. Sur les programmes de plus de 1 000 lignes, **toujours appliquer la stratégie de découpage** même si UC 4 avait pu se faire en une passe directe.

**2. Extraire les règles subroutine par subroutine est plus fiable qu'une extraction globale.**
Quelle que soit la taille du programme, demander à Bob d'extraire les règles **subroutine par subroutine** (en utilisant la cartographie produite en UC 4 comme guide) produit des résultats plus précis et plus traçables qu'une extraction en une seule fois sur tout le programme. Exemple d'enchaînement :

```
"Sur la base de la cartographie de [NOM_PROGRAMME], extrait les règles métier
 de la subroutine [SR_CALCUL_PRIX] uniquement."

[Après validation]
"Maintenant extrait les règles de la subroutine [SR_REMISES]."
```

> 💡 Cette approche subroutine par subroutine exploite la cartographie produite en UC 4 — c'est la raison pour laquelle UC 4 doit être complété avant UC 5.

> ⚠️ Sur un programme > 3 000 lignes, ne jamais lancer le Prompt 1 sur l'intégralité du programme en une seule fois — Bob produira une liste de règles incomplète sans le signaler.

---

## Démarrer une session Bob

> **À lire avant chaque session UC 5 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

UC 5 peut se dérouler dans la **même conversation** que UC 4 si l'analyse de compréhension vient d'être faite (Bob a encore le programme en contexte). Dans ce cas, enchaîner directement avec les prompts UC 5 sans ouvrir une nouvelle session.

Si UC 4 a été fait dans une session **précédente** (autre jour, autre session) : démarrer une nouvelle conversation Bob (bouton `+`) et charger le fichier de compréhension en contexte avant de commencer.

**Mode à sélectionner :** `IBM i Developer`

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Si UC 5 démarre dans une nouvelle session, ouvrir le programme RPG source dans l'éditeur — Bob en a besoin pour affiner les règles extraites avec le code précis.

**Procédure :** dans le panneau **IBM i — Object Browser**, clic droit sur le membre → **Open in Editor**.

Fichiers à ouvrir pour chaque session UC 5 :
- Le programme RPG source (`[NOM_LIB]/QRPGSRC([NOM_PROGRAMME])`)
- Les copybooks référencés (si présents dans le scope)
- Le display file DDS si le programme est interactif (les validations de saisie y sont parfois codées)

### 3. Fichiers de contexte à charger

Ces fichiers produits par UC 4 doivent être chargés **avant de démarrer** si la session est nouvelle. Utiliser **Add File to Chat** (icône trombone) ou les ouvrir dans l'éditeur.

| Fichier | Produit par | Obligatoire / Recommandé |
|---------|-------------|--------------------------|
| `{appArcad}-{fonction}-{composant}-comprehension-{date}.md` | UC 4 | **Obligatoire** — carte des subroutines, guide de navigation pour l'extraction règle par règle |
| `{appArcad}-{fonction}-{composant}-regles-{date}.md` | UC 5 (session précédente) | **Si reprise** — catalogue partiel déjà produit, pour compléter sans recommencer |

> 💡 Si le fichier `*-comprehension-*.md` n'est pas disponible, démarrer par une session rapide UC 4 sur le programme ciblé — 20 minutes de cartographie des subroutines évitent d'extraire des règles dans le désordre.

> ⚠️ **Risque de réduction de contexte — sauvegarde intermédiaire recommandée :** une session UC 5 avec plusieurs passes d'extraction (subroutine par subroutine) peut atteindre la limite de contexte sur les programmes STANDARD ou COMPLEXE. Si Bob semble oublier une décision prise au Prompt 1 (statut des règles, formules identifiées, tables de paramétrage signalées), c'est un signal de compression de contexte. Sauvegarder le livrable en cours en mode Agent après chaque prompt majeur — pas seulement en fin de session. À chaque reprise de passe, commencer le prompt par : "Le fichier [NOM_FICHIER] contient les décisions prises — continuer à partir de [ÉTAPE]."

> 💡 **Reprise de session :** ouvrir le fichier `*-regles-*.md` déjà produit et indiquer dans le prompt "le catalogue de règles de [NOM_PROGRAMME] est en cours — voici les règles déjà extraites [NOM_FICHIER]. Continuer avec la subroutine [NOM_SR]".

> 💡 **Lien direct UC 5 → UC 6 :** le catalogue de règles validé (`*-regles-*.md`) est l'input principal du Prompt 2 de UC 6 (spécification fonctionnelle). Ne pas démarrer UC 6 Prompt 2 sans ce fichier — voir `UC06-documentation-complete.md`.

---

## Prérequis

- Les fichiers `*-comprehension-*.md` des programmes à analyser sont présents dans le workspace (la compréhension générale est le socle)
- Ces fichiers sont chargés dans le contexte Bob en début de session (ou la session de compréhension est en cours)
- IBM i MCP actif (lecture des sources)
- IBM i Database MCP actif (interrogation des tables de paramétrage via SQL)
- Idéalement : un expert métier disponible pour valider les règles extraites

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | **IBM i Developer** (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : **Ask**. |
| **Scope** | Library List → bibliothèque applicative ACME |
| **MCP actifs** | IBM i MCP + IBM i Database MCP |
| **MCP différés** | Confluence MCP (publication du catalogue de règles, si token disponible) |

### Pourquoi le mode IBM i Developer pour l'extraction de logique métier ?

Le mode **IBM i Developer** apporte la connaissance RPG/CL/DDS spécialisée pour toute la session — vocabulaire des opcodes, connaissance des structures ILE, vues QSYS2, comportements des MCP IBM i. Sans ce mode, ces notions doivent être réexpliquées dans chaque prompt d'extraction de règles.

UC 5 est un UC **lecture seule** : Bob analyse le code et les tables de paramétrage, puis produit le catalogue de règles dans le chat. La discipline de validation repose sur la **relecture humaine avant sauvegarde** : Bob génère le catalogue dans le chat, l'expert métier valide les règles extraites, puis autorise explicitement l'écriture du fichier `.md` dans le workspace.

Le MCP IBM i Database est particulièrement utile dans cet UC pour **deux usages distincts** :
- Interroger les tables de paramétrage référencées dans les règles (taux, codes, seuils) — les valeurs réelles s'affichent dans le chat
- Vérifier l'existence et la structure des fichiers physiques cités dans le code (via `QSYS2.SYSCOLUMNS`)

> 💡 **Sans Premium Package IBM i :** utiliser le mode **Ask** pour toute la session — la discipline de validation reste identique.

> 💡 **Règle d'or pour UC 5 :** IBM i Developer pour toute la session. Bob génère le catalogue de règles dans le chat — l'expert métier valide les règles avant d'autoriser la sauvegarde du fichier `.md`. Aucune écriture IBM i dans cet UC.

> ⚠️ Ne jamais autoriser d'écriture sur l'IBM i pendant UC 5 — cet UC est exclusivement lecture et extraction. Si Bob propose une action `write_member`, refuser.

---

## Prompts clés

### Prompt 1 — Extraction des règles métier d'un programme

```
Le programme [NOM_PROGRAMME] dans [NOM_LIB]/QRPGSRC a fait l'objet d'une fiche de compréhension.
[Si nouvelle session : charger le fichier {appArcad}-{fonction}-{composant}-comprehension-*.md dans le contexte avant ce prompt]

Extrait maintenant toutes les règles métier implicites contenues dans ce programme.
Une règle métier est une décision, un calcul, une validation ou une condition 
qui reflète une politique ou une pratique de l'entreprise — pas une contrainte technique.

Pour chaque règle identifiée, fournis en markdown :
- Numéro et intitulé court de la règle
- Description en langage naturel (compréhensible par un non-développeur)
- Déclencheur : dans quelle situation cette règle s'applique-t-elle ?
- Formule ou logique précise : reproduis le calcul ou la condition en pseudo-code clair
- Données impliquées : champs, fichiers ou paramètres utilisés dans la règle
- Origine : numéro de ligne ou nom de subroutine dans le source
- Statut : certain (visible dans le code) / probable (déduit) / à confirmer (ambigu)

Si une règle dépend de valeurs stockées en table de paramétrage (non codées en dur),
signale-le et indique la table concernée.
```

**Analyse ligne à ligne :**

- `a fait l'objet d'une fiche de compréhension` → ancre Bob dans la continuité de la démarche. Si on est dans la **même session** que l'analyse de compréhension, Bob utilise l'historique directement. Si on démarre une **nouvelle session**, la mention `[charger le fichier *-comprehension-*.md]` est incluse dans le prompt lui-même comme rappel — ne pas l'oublier, sinon Bob analyse le programme sans le contexte des dépendances et de la cartographie déjà établies.

- `Une règle métier est une décision, un calcul...` → **définition explicite** dans le prompt. Sans elle, Bob extrait aussi des règles techniques (gestion des erreurs, boucles, I/O) qui ne sont pas des règles métier. Cette ligne filtre le bruit.

- `pas une contrainte technique` → complète la définition. Évite que Bob liste `IF SQLCODE <> 0 THEN...` comme règle métier.

- `Numéro et intitulé court` → force une liste structurée numérotée. Directement exploitable comme référentiel de règles, importable dans Confluence ou un outil de gestion des exigences.

- `Formule ou logique précise : reproduis en pseudo-code clair` → le pseudo-code est plus lisible qu'une paraphrase pour un analyste. Évite les descriptions vagues du type "le programme calcule le prix" sans donner la formule.

- `Origine : numéro de ligne ou nom de subroutine` → **traçabilité** de chaque règle jusqu'au code source. Indispensable pour la validation et pour les UC de modernisation qui devront modifier ces règles.

- `Statut : certain / probable / à confirmer` → **ingrédient anti-hallucination** adapté à l'extraction de règles. Bob distingue ce qui est explicite dans le code de ce qu'il déduit. Les règles "à confirmer" sont les plus risquées — elles doivent être validées par un expert métier avant d'être utilisées.

- `Si une règle dépend de valeurs en table de paramétrage` → flag critique. Sur IBM i, beaucoup de règles métier ne sont pas codées en dur — elles lisent des tables de codes ou de taux dans la base. Bob le signale plutôt que d'inventer une valeur.

> ⚠️ **Piège évité :** sans le statut "certain / probable / à confirmer", Bob présente toutes les règles avec le même niveau de confiance. Les règles déduites d'un contexte partiel (copybook manquant, table de paramétrage non lue) passent pour certaines.

---

### Prompt 2 — Extraction des règles de calcul et formules

```
Dans le programme [NOM_PROGRAMME] de [NOM_LIB], identifie tous les calculs 
et formules métier.

Pour chaque calcul :
- Nom du calcul (déduis un nom fonctionnel si le code n'en a pas)
- Formule mathématique exacte, avec les noms de champs tels qu'ils apparaissent dans le code
- Formule reformulée avec des noms lisibles (si les noms de champs sont cryptiques)
- Unités et types : est-ce un montant ? un pourcentage ? une quantité ? une durée ?
- Précision et arrondis : comment le résultat est-il arrondi ou tronqué ?
- Conditions d'application : dans quel cas ce calcul est-il exécuté ?

Signale tout calcul dont le résultat dépend d'une valeur lue en base de données 
(taux, coefficient, seuil) plutôt que codée en dur.
```

**Analyse ligne à ligne :**

- `Nom du calcul (déduis un nom fonctionnel si le code n'en a pas)` → sur IBM i legacy, les variables s'appellent `MNTHT`, `TXREM`, `QTDSP` — aucun nom lisible. Demander à Bob de nommer les calculs force une traduction vers un vocabulaire métier utile.

- `Formule mathématique exacte, avec les noms de champs tels qu'ils apparaissent dans le code` → deux formules en une : l'exacte (traçable dans le code) et la lisible (communicable au métier). Le fait de demander les deux évite de perdre le lien avec le source.

- `Unités et types` → sur IBM i, un champ numérique peut être un montant en centimes, un pourcentage × 100, ou une quantité — rien dans le type SQL ou RPG ne le dit. Bob déduit l'unité depuis le contexte et le signale.

- `Précision et arrondis` → critique sur les calculs financiers. Une différence d'arrondi entre l'ancien et le nouveau programme est une régression silencieuse. Demander à Bob de l'identifier explicitement.

- `Signale tout calcul dont le résultat dépend d'une valeur lue en base` → même principe que Prompt 1 : les taux de remise, TVA, coefficients lus dans une table de paramétrage doivent être identifiés comme tels — ce sont des règles dynamiques, pas des constantes.

> 💡 **Variante — interroger la table de paramétrage directement :** si Bob identifie une table de taux ou de seuils, enchaîner immédiatement avec :
> ```
> "Interroge la table [NOM_TABLE] dans [NOM_LIB] et affiche les valeurs 
>  actuelles des taux / seuils utilisés dans ce calcul."
> ```
> Le MCP IBM i Database exécute la requête SQL en temps réel — les valeurs réelles s'affichent dans le chat.

---

### Prompt 3 — Extraction des règles de validation et contrôles de saisie

```
Dans le programme [NOM_PROGRAMME] de [NOM_LIB], identifie toutes les règles 
de validation et de contrôle appliquées aux données entrantes.

Pour chaque validation :
- Ce qui est contrôlé (quel champ, quelle valeur, quelle combinaison)
- La condition de rejet : quand la validation échoue-t-elle ?
- Le message ou le comportement en cas d'échec (message d'erreur, blocage, valeur par défaut)
- La sévérité : bloquant (l'utilisateur ne peut pas continuer) ou avertissement ?
- L'origine dans le code (subroutine, ligne)

Distingue les validations de format (type, longueur, obligatoire) 
des validations métier (règle de gestion, cohérence entre champs, existence en référentiel).
```

**Analyse ligne à ligne :**

- `Ce qui est contrôlé... La condition de rejet... Le message... La sévérité` → structure complète d'une règle de validation. Sans cette structure, Bob liste des conditions IF sans contexte — inutilisable pour un analyste ou pour recoder la validation dans un nouveau système.

- `Distingue les validations de format des validations métier` → distinction fondamentale pour la modernisation. Les validations de format (longueur, type, obligatoire) sont triviales à recoder. Les validations métier (cohérence entre champs, existence dans un référentiel) représentent la vraie connaissance à capturer.

> ⚠️ **Piège évité :** les validations sur écran 5250 sont souvent dans le fichier DDS display (`CHECK`, `RANGE`, `VALUES`) **et** dans le programme RPG. Si on n'analyse que le programme RPG, on rate les validations côté DDS. Toujours inclure le display file dans le scope pour cet UC.

---

### Prompt 4 — Extraction des flux de décision (arbres de décision)

```
Dans le programme [NOM_PROGRAMME] de [NOM_LIB], identifie les points de décision 
métier principaux — les endroits où le programme choisit entre plusieurs chemins 
en fonction d'une condition métier.

Pour les 3 à 5 décisions les plus importantes :
- La condition de décision en langage naturel
- Les branches possibles et leur effet métier
- Les données qui influencent la décision
- Représente la décision sous forme d'arbre en markdown (listes imbriquées)

Ne pas inclure les décisions purement techniques (gestion des erreurs SQL, 
boucles de lecture de fichier, fin de programme).
```

**Analyse ligne à ligne :**

- `les 3 à 5 décisions les plus importantes` → limite explicite. Sans elle, Bob liste toutes les conditions IF/ELSE du programme — des dizaines sur un programme legacy. On veut les décisions à valeur métier, pas l'exhaustivité.

- `Représente la décision sous forme d'arbre en markdown (listes imbriquées)` → format structuré lisible sans outil spécial. Plus actionnable qu'un diagramme Mermaid pour les arbres de décision simples, plus rapide à valider par un expert métier.

- `Ne pas inclure les décisions purement techniques` → filtre identique à Prompt 1. Sur IBM i, les programmes sont truffés de conditions techniques (`IF SQLCODE <> 0`, `IF %EOF(FICHIER)`) — les exclure explicitement allège la sortie.

> 💡 **Enchaînement avec UC 6 :** les arbres de décision extraits ici sont directement utilisables dans les spécifications fonctionnelles produites en UC 6.

---

### Prompt 5 — Synthèse : traduire les règles en langage métier pour validation

```
Sur la base des règles métier extraites pour [NOM_PROGRAMME], rédige un document 
de synthèse en français destiné à être validé par un expert métier ACME.

Le document doit être rédigé sans aucun terme technique RPG ou IBM i.
Structure :
1. Contexte : à quel processus métier ce programme participe-t-il ?
2. Règles de gestion : liste numérotée des règles extraites, en langage naturel pur
3. Calculs clés : les formules reformulées en termes métier (ex: "Le prix net = prix brut × (1 - taux de remise)")
4. Points à valider : règles marquées "probable" ou "à confirmer" qui nécessitent 
   une confirmation par un expert métier
5. Règles dépendant du paramétrage : règles dont les valeurs sont en base de données 
   (à faire vérifier avec les valeurs actuelles)

En fin de document, ajoute une section "Questions pour l'expert métier" avec 
3 à 5 questions précises sur les points ambigus identifiés.
```

**Analyse ligne à ligne :**

- `destiné à être validé par un expert métier ACME` → calibre l'audience et donc le niveau de langage. Bob adapte son style à l'audience déclarée.

- `sans aucun terme technique RPG ou IBM i` → règle absolue pour un livrable métier. `CHAIN`, `SETLL`, `*IN50` n'ont aucun sens pour un responsable de domaine.

- `Points à valider : règles marquées "probable" ou "à confirmer"` → exploite le statut défini en Prompt 1. Le document de validation est pré-segmenté : les règles certaines ne nécessitent pas de réunion, les probables oui.

- `Questions pour l'expert métier` → le livrable le plus précieux de cet UC pour la réunion de validation. 3 à 5 questions ciblées sont bien plus efficaces qu'une réunion de revue de 50 règles.

> 💡 **Ce document est le livrable de sortie de UC 5.** Il structure la réunion de validation avec l'équipe métier ACME et nourrit directement UC 6 (spécifications fonctionnelles).

> 💡 **Sauvegarder le catalogue de règles** : une fois la synthèse finalisée et validée par l'expert métier, basculer en mode **Agent** et demander à Bob de sauvegarder le document :
> ```
> "Sauvegarde ce catalogue de règles dans un fichier markdown
>  nommé {appArcad}-{fonction}-{composant}-regles-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-regles-20250615-1600.md"
> ```
> Ce fichier sera référencé directement dans les prompts de génération des spécifications fonctionnelles (Prompt 2 de l'étape Documentation complète).

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Lecture des membres sources et des display files DDS (validations côté écran) |
| **IBM i Languages** | Lecture du code RPG dans l'éditeur pour contextualiser les règles extraites |
| **Markdown All in One** | Prévisualisation du catalogue de règles et du document de validation |

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture des membres sources RPG, CL, DDS (display files pour les validations) |
| **IBM i Database MCP** | Interrogation des tables de paramétrage référencées dans les règles (taux, codes, seuils) |
| **Confluence MCP** *(si disponible)* | Publication du catalogue de règles et du document de validation sur l'espace POC |

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Analyser le programme RPG sans le display file DDS associé | Les validations de saisie côté écran sont manquantes — le catalogue de règles est incomplet | Inclure le display file dans le scope, ou l'ouvrir dans l'éditeur avant le prompt |
| Ne pas distinguer règles certaines et règles déduites | Toutes les règles semblent fiables — les règles fausses passent inaperçues | Toujours inclure le statut "certain / probable / à confirmer" (Prompt 1) |
| Oublier les tables de paramétrage | Les règles qui lisent des taux ou des seuils en base sont extraites avec des valeurs nulles ou inventées | Demander explicitement à Bob de signaler les dépendances aux tables, puis les interroger avec le MCP Database |
| Présenter les règles extraites directement au management sans validation métier | Des règles fausses ou incomplètes sont validées comme spécifications — risque majeur sur les UC suivants | Toujours passer par le Prompt 5 (document de validation) et une session de revue avec un expert métier |
| Confondre UC 5 et UC 4 | On produit une compréhension générale au lieu d'un catalogue de règles précis | UC 5 = règles métier actionnables avec formules et conditions précises. UC 4 = vue d'ensemble. Les deux sont distincts. |
| Extraire les règles métier de programmes CL | Les programmes CL contiennent rarement des règles métier — ils orchestrent des appels | Se concentrer sur les programmes RPG et COBOL pour cet UC ; les programmes CL relèvent de l'architecture (UC 6) |

---

## Check-list de validation UC 5

Avant de passer à UC 6 (voir `UC06-documentation-complete.md`), valider chaque point :

- [ ] Au moins un programme métier central de ACME a été analysé avec les Prompts 1 et 2
- [ ] Les calculs clés (prix, remises, taxes, quantités…) ont été extraits avec formules et unités
- [ ] Les validations métier (pas seulement les validations de format) ont été identifiées
- [ ] Les dépendances aux tables de paramétrage ont été identifiées et interrogées via le MCP Database
- [ ] Le document de synthèse (Prompt 5) a été produit pour au moins un domaine métier
- [ ] Le document a été soumis à un expert métier ACME pour validation
- [ ] Les règles "à confirmer" ont été revues et clarifiées (ou explicitement laissées ouvertes)
- [ ] Le catalogue de règles validé est sauvegardé avec la convention de nommage `{appArcad}-{fonction}-{composant}-regles-{YYYYMMDD-HHmm}.md` dans le workspace ET publié sur Confluence (si MCP disponible)

---

*Fiche UC 5 — Document évolutif à mettre à jour au fil du POC.*
