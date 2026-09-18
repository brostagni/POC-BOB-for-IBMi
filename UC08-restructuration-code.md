# UC 8 — Restructuration, transformation, conversion

> **Catégorie :** Modernisation du code
>
> **Priorité dans le POC :** 9 — suit UC 7 (optimisations appliquées sur le même périmètre), peut être mené en parallèle avec UC 7 par une sous-équipe sur des programmes différents
>
> **Durée POC (avec Bob) :** 4 à 8 heures — restructuration d'un périmètre représentatif (2 à 3 programmes monolithiques), itérations de découpage, compilation et tests de non-régression
>
> **Durée PROD (avec Bob) :** 2 à 6 heures / programme — qualification, conception de l'architecture cible, génération des modules extraits, compilation et tests
>
> **Durée PROD (sans Bob) :** 3 à 10 jours / programme — analyse complète du source, conception de la découpe, réécriture des modules, adaptation des interfaces, campagne de tests de non-régression
>
> **Gain Bob estimé :** ~5× — un programme de 800 lignes décomposé en 4 modules traité en une journée au lieu d'une semaine ; gain plus fort sur les programmes avec logique dupliquée entre plusieurs sources (Bob peut identifier et consolider)
>
> **Mode Bob recommandé :** IBM i Developer (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : Ask pour l'analyse/génération, Agent pour la compilation et la sauvegarde.

> ⚠️ **UC 8 ne se lance jamais dans la même session Bob qu'UC 7, ni via la même demande en mode Plan.** UC 7 doit être terminé, compilé sans erreur et validé fonctionnellement avant d'ouvrir une **nouvelle conversation** pour UC 8. Demander à Bob de "faire UC 7 et UC 8" en une seule passe mélange les renommages de variables et l'extraction de modules dans un même diff — les erreurs deviennent intraçables et la compilation produit plusieurs centaines d'erreurs sur un programme de 1000 lignes. Voir aussi la règle correspondante dans la fiche UC 7.

---

## Objectif

Transformer la **structure profonde** des programmes RPG IBM i : décomposer les monolithes, extraire des modules réutilisables, convertir le style de programmation (cycle RPG → procédures), réorganiser les responsabilités entre programmes.

**UC 8 est le UC le plus risqué de la Phase 3 — Modernisation du code.** Contrairement à UC 7 (optimisation de surface), UC 8 modifie l'organisation du code. La logique fonctionnelle doit rester identique, mais la frontière entre "optimiser la structure" et "changer involontairement le comportement" est mince sur des programmes legacy complexes. C'est pourquoi UC 8 s'appuie toujours sur la documentation produite en Phase 1.

**Ce UC diffère de UC 7 (optimisation) et de UC 3 (SQL embarqué) :**
- UC 7 change la forme interne du code sans déplacer de logique entre programmes. Les interfaces restent intactes.
- UC 3 change le mécanisme d'accès aux données. La structure du programme reste la même.
- UC 8 déplace de la logique entre programmes : un bloc fonctionnel qui était dans le programme A est maintenant dans un module B appelé par A. Ce déplacement crée de nouvelles interfaces, de nouvelles dépendances, et de nouveaux risques de régression.

**Ordre recommandé sur un même programme : UC 7 d'abord, UC 8 ensuite.** Restructurer un programme avec des variables cryptiques et des opcodes obsolètes revient à déplacer du code difficile à lire dans une nouvelle structure difficile à lire. UC 7 d'abord produit un source clair que UC 8 peut découper de façon lisible.

**Cas où UC 8 peut précéder UC 3 sur un programme COMPLEXE :** si le Prompt 0 de UC 3 conclut que le programme doit être restructuré avant la conversion SQL (accès natifs répartis dans un monolithe sans logique identifiable), démarrer UC 8 sur ce programme avant UC 3.

**Livrable attendu :** Pour chaque programme restructuré : un plan d'architecture décrivant la décomposition cible, le ou les nouveaux modules extraits (sources RPG), le programme principal modifié pour appeler les modules, et un fichier de traçabilité documentant chaque déplacement de logique.

**Convention de nommage des fichiers générés :**
```
{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md

Types pour cet UC :
  qualification-restr  → fiche de qualification (Prompt 0) — catégorie + stratégie
  plan-archi           → plan d'architecture de la restructuration cible (Prompt 1)
  diff-restr           → diff de restructuration appliquée (Prompts 2, 3, 4)
  spec-module          → spécification d'un nouveau module extrait (Prompt 2)
  plan-restr           → plan de restructuration périmètre applicatif complet (Prompt 5)
```

Exemples :
```
acme-APPVTE-GESCMD-qualification-restr-20250620-0900.md    ← qualification + stratégie
acme-APPVTE-GESCMD-plan-archi-20250620-1000.md             ← architecture cible
acme-APPVTE-GESCMD-spec-module-20250620-1100.md            ← spec du module extrait
acme-APPVTE-GESCMD-diff-restr-20250620-1400.md             ← diff du programme modifié
acme-APPVTE-APPVTE-plan-restr-20250620-0800.md             ← plan périmètre complet
```

> 💡 Cette convention est valable en dehors du contexte POC — réutilisable en production tel quel.

---

## Démarrer par un programme que vous connaissez

> **Recommandation forte avant d'aborder les programmes critiques de ACME.**

Commencer par un **programme dont la logique est bien documentée** — idéalement un programme avec une compréhension (UC 4) et des règles métier (UC 5) disponibles, dont un développeur senior peut valider que le comportement après restructuration est identique.

Pourquoi ? Parce que la première restructuration sert à **calibrer deux choses** :
- La qualité du plan d'architecture proposé par Bob (les modules extraits ont-ils une cohérence fonctionnelle ? Les interfaces sont-elles correctes ?)
- La capacité de l'équipe à tester la non-régression sur un programme restructuré (les jeux de tests doivent couvrir les chemins d'exécution qui traversent maintenant deux programmes au lieu d'un)

Si la première restructuration est validée fonctionnellement, la confiance est établie pour les programmes plus complexes.

**Progression recommandée :**

> 💡 Pour chaque programme de cette progression, **commencer par le Prompt 0** — il donne la catégorie (SIMPLE / STANDARD / COMPLEXE) et la séquence exacte à suivre. Ne pas aller directement au Prompt 1.

| Étape | Programme à choisir | Catégorie attendue | Objectif |
|-------|--------------------|--------------------|---------|
| 1 | Programme avec 2 responsabilités clairement séparables et peu de variables partagées entre elles | SIMPLE | Calibrer l'extraction d'un seul module ; valider que le Prompt 0 classe correctement |
| 2 | Programme avec un cycle RPG actif (niveau de contrôle, indicateurs de niveau) à convertir en procédures | STANDARD | Valider la conversion cycle RPG → structure procédurale |
| 3 | Programme monolithique avec plusieurs responsabilités imbriquées et des variables partagées entre subroutines | STANDARD à COMPLEXE | Valider la découpe en plusieurs modules et la gestion des interfaces |
| 4 | Programme critique partagé par de nombreux appelants (identifié dans la matrice UC 6) | COMPLEXE + risque élevé | Valider contre la matrice de dépendances avant toute modification d'interface |

---

## Impact de la taille du programme sur la stratégie de restructuration

La complexité d'UC 8 ne se mesure pas en nombre de lignes, mais en **nombre de responsabilités distinctes** portées par le programme, au **niveau de couplage** entre ces responsabilités (variables partagées, indicateurs transversaux), et au **nombre de programmes appelants** qui dépendent de l'interface actuelle.

Les vrais facteurs qui compliquent la restructuration :
- Le **cycle RPG actif** (`*INLR`, niveaux de contrôle `L1`-`L9`, totaux de niveaux) : le cycle RPG est un mécanisme de traitement séquentiel implicite qui n'a pas d'équivalent direct en procédures — sa conversion en logique explicite (`DOW/READ/ENDDO`) est la restructuration la plus profonde et la plus risquée
- Les **variables globales massives** : un programme avec 50 variables globales utilisées dans toutes les subroutines ne peut pas être découpé proprement sans définir d'abord une architecture de passage de données (paramètres, structures partagées, DS communes)
- Les **programmes appelants multiples** : si le programme est appelé par 15 autres programmes via `CALL`, modifier ses paramètres d'entrée/sortie casse tous les appelants — la restructuration doit préserver l'interface externe ou gérer la migration des appelants
- Les **écrans 5250** intégrés (programmes interactifs avec EXFMT) : la logique de gestion de l'écran est souvent entrelacée avec la logique métier — les séparer est une restructuration à part entière (précurseur de UC 10)
- Les **journaux IBM i et les ROLLBACK** : si le programme gère des transactions, le découpage en modules doit préserver la cohérence transactionnelle — les COMMIT/ROLLBACK doivent rester au niveau du programme appelant

### Programmes SIMPLE — 1 à 2 responsabilités séparables, peu de variables partagées

Bob gère sans difficulté. **Séquence : Prompt 0 → Prompt 1 (plan d'architecture) → Prompt 2 (nouveau module) → Prompt 3 (programme principal modifié) → Prompt 3-bis (compilation) → Prompt 4 (nettoyage et interfaces).**

> 💡 Le Prompt 0 confirme la catégorie SIMPLE et recommande directement cette séquence — pas de décision manuelle requise.

### Programmes STANDARD — 2 à 4 responsabilités, variables partagées, ou cycle RPG à convertir

Le Prompt 0 identifie les zones de découpe et les variables partagées à transformer en paramètres. **Séquence : Prompt 0 → Prompt 1 (plan complet validé avec l'équipe) → Prompt 2 répété par module → Prompt 3-bis après chaque module → Prompt 3 (programme principal) → Prompt 3-bis → Prompt 4.**

> 💡 Le plan d'architecture du Prompt 1 est la pièce centrale de UC 8 — valider ce plan avec un développeur senior avant de générer le moindre code. Une architecture mal conçue au Prompt 1 produit des modules incohérents aux Prompts 2 et 3.

### Programmes COMPLEXE — cycle RPG actif, > 4 responsabilités, ou programme interactif avec écrans 5250

Un programme COMPLEXE en UC 8 est souvent un programme qui a évolué pendant 20 ans et qui porte une partie critique du système. Le Prompt 0 pose explicitement la question : **le risque de cette restructuration est-il acceptable dans le cadre du POC ?** Si le Prompt 0 conclut "risque élevé — périmètre à restreindre", limiter UC 8 à une seule extraction bien délimitée et reporter le reste.

Si la décision est de procéder : **Séquence : Prompt 0 → Prompt 1 (architecture cible validée) → découpage en plusieurs lots → Prompt 2 lot par lot → Prompt 3-bis après chaque lot → Prompt 3 (programme principal) → Prompt 3-bis → Prompt 4.**

> ⚠️ Sur un programme COMPLEXE, une restructuration globale en une seule session produit du code incomplet et des interfaces incorrectes. Le Prompt 0 découpe toujours le travail en lots cohérents. Ne commencer le lot suivant que lorsque le lot précédent compile sans erreur.

---

## Démarrer une session Bob

> **À lire avant chaque session UC 8 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

Chaque session de travail sur un programme RPG doit démarrer dans une **nouvelle conversation Bob** (bouton `+` en haut du panneau Chat). Ne pas réutiliser une conversation d'un autre programme ou d'UC 7 — l'architecture d'un autre programme bruite les décisions de découpe du programme courant.

**Mode à sélectionner :** `IBM i Developer`

**Scope à sélectionner :** `Library List` → choisir la bibliothèque applicative ACME.

> 💡 **Ce que le scope Library List change :** Bob dirige ses outils IBM i vers les bibliothèques
> configurées dans Code for IBM i. Les rules `.bob/rules/`, skills et fichiers markdown locaux
> **restent accessibles** en parallèle. Les 5 workflows IBM i (bouton ▶) deviennent disponibles
> dès qu'une connexion IBM i est active. (Source : PPi Onboarding Usage Guide)

> 💡 **Avant la première session UC 8 :** vérifier que la Library List inclut la bibliothèque
> source, les bibliothèques contenant les **copybooks et les prototypes** utilisés par le programme
> à restructurer. UC 8 lit les copybooks pour analyser les interfaces — un copybook manquant
> force Bob à scanner tout `*LIBL`.

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Avant de lancer le Prompt 0, ouvrir dans l'éditeur Bob le programme RPG à restructurer. L'ouverture dans l'éditeur le rend accessible au MCP IBM i sans copier-coller.

**Procédure :** dans le panneau **IBM i — Object Browser** (extension Code for IBM i), naviguer jusqu'à la bibliothèque source, faire un clic droit sur le membre RPG → **Open in Editor**.

Fichiers à ouvrir pour chaque session UC 8 :
- Le programme RPG source (`[NOM_LIB]/QRPGSRC([NOM_PROGRAMME])`)
- Le fichier de compréhension du programme (`*-comprehension-*.md`) depuis UC 4
- Le fichier des règles métier (`*-regles-*.md`) depuis UC 5
- Le fichier de spécification technique (`*-spec-tech-*.md`) depuis UC 6
- Le plan d'architecture si une session précédente l'a produit (`*-plan-archi-*.md`)

### 3. Fichiers de contexte à charger

Ces fichiers produits par les UC précédents doivent être disponibles dans le workspace Bob **avant** de démarrer. Utiliser **Add File to Chat** (icône trombone) ou les ouvrir dans l'éditeur.

| Fichier | Produit par | Obligatoire / Recommandé |
|---------|-------------|--------------------------|
| `{appArcad}-{fonction}-{composant}-comprehension-{date}.md` | UC 4 | **Obligatoire** — carte des subroutines et des appels, base du Prompt 1 |
| `{appArcad}-{fonction}-{composant}-regles-{date}.md` | UC 5 | **Obligatoire** — règles métier portées par chaque partie du programme |
| `{appArcad}-{fonction}-{composant}-spec-tech-{date}.md` | UC 6 | **Obligatoire** — interfaces actuelles, paramètres et fichiers accédés |
| `{appArcad}-{fonction}-{fonction}-matrice-{date}.md` | UC 6 | **Obligatoire** — tous les programmes appelants du programme à restructurer |
| `{appArcad}-{fonction}-{composant}-diff-optim-{date}.md` | UC 7 | **Si UC 7 appliqué** — source optimisé (variables renommées, opcodes remplacés) |
| `{appArcad}-{fonction}-{composant}-sql-embarque-{date}.md` | UC 3 | **Si UC 3 appliqué** — accès SQL embarqués déjà en place, à préserver dans les modules extraits |
| `{appArcad}-{fonction}-{composant}-qualification-restr-{date}.md` | UC 8 (session précédente) | **Si reprise** — catégorie et périmètre déjà décidés |
| `{appArcad}-{fonction}-{composant}-plan-archi-{date}.md` | UC 8 (session précédente) | **Si reprise** — plan d'architecture validé, évite de relancer le Prompt 1 |

> ⚠️ **Prérequis critique :** ne jamais démarrer UC 8 sans les fichiers `*-comprehension-*.md`, `*-regles-*.md` et `*-matrice-*.md`. Restructurer sans ces documents revient à déplacer du code dont on ne connaît pas la sémantique ni les impacts sur les appelants.

> ⚠️ **Risque de réduction de contexte — sauvegarde intermédiaire recommandée :** une session UC 8 avec plusieurs prompts consécutifs peut atteindre la limite de contexte sur les programmes STANDARD ou COMPLEXE. Si Bob semble oublier une décision prise au Prompt 0 (catégorie, périmètre retenu, modules identifiés), c'est un signal de compression de contexte. Sauvegarder le livrable en cours en mode Agent après chaque prompt majeur — pas seulement en fin de session. À chaque reprise de passe, commencer le prompt par : "Le fichier [NOM_FICHIER] contient les décisions prises — continuer à partir de [ÉTAPE]."

> 💡 **Reprise de session :** si la restructuration d'un programme est interrompue après le Prompt 1, ouvrir le fichier `*-plan-archi-*.md` validé — Bob reprend la génération des modules à partir de l'architecture convenue sans repartir de zéro. Indiquer dans le prompt "le plan d'architecture est disponible dans [NOM_FICHIER] — reprendre à partir du module [NOM_MODULE]".

---

## Prérequis

- Les fichiers `*-comprehension-*.md` (UC 4) des programmes à restructurer sont présents : ils fournissent la carte complète des subroutines, des appels et des flux de données — la base du plan d'architecture du Prompt 1
- Les fichiers `*-regles-*.md` (UC 5) sont présents : ils documentent les règles métier portées par chaque partie du programme — garantie que le découpage préserve l'intégrité fonctionnelle
- Les fichiers `*-spec-tech-*.md` (UC 6) sont disponibles : ils décrivent les interfaces actuelles du programme (paramètres, fichiers accédés) — toute modification d'interface doit être tracée contre ce document
- Les fichiers `*-matrice-*.md` (UC 6) sont disponibles : ils identifient tous les programmes appelants — indispensable avant toute modification d'interface externe
- UC 7 complété sur le programme si applicable : les variables sont nommées de façon lisible, les opcodes obsolètes remplacés — le source est propre avant restructuration
- Les fichiers `*-sql-embarque-*.md` (UC 3) disponibles si la conversion SQL embarqué a été faite : le programme principal modifié doit préserver les accès SQL existants dans les modules extraits
- IBM i MCP actif (lecture et écriture des sources RPG, compilation de test)
- Accès à l'IBM i de test pour compiler et tester les programmes restructurés
- Un développeur RPG senior disponible pour valider le plan d'architecture (Prompt 1) et la non-régression fonctionnelle

> ⚠️ **Prérequis critique :** ne jamais démarrer UC 8 sans les fichiers `*-comprehension-*.md` et `*-regles-*.md`. Restructurer sans carte des subroutines et sans connaissance des règles métier, c'est déplacer du code dont on ne connaît pas la sémantique — la régression fonctionnelle est quasi certaine sur un programme COMPLEXE.

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | **IBM i Developer** (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : **Ask** pour l'analyse/génération, **Agent** pour la compilation et la sauvegarde. |
| **Scope** | Library List → bibliothèque applicative ACME |
| **MCP actifs** | IBM i MCP (lecture des sources RPG, écriture des nouveaux modules, compilation de test) |
| **MCP différés** | IBM i Database MCP (si restructuration implique des accès de données à vérifier), Confluence MCP (publication, si token disponible) |

> 💡 **Workflow disponible pour UC 8 :** le workflow **RPG Modernization**
> (bouton ▶ → *RPG Modernization*), en mode **IBM i Developer**, peut prendre en charge
> la conversion Free-format des modules extraits ou du programme principal, **après** que
> le plan d'architecture (Prompt 1) a été validé.
> Pipeline : sélection membre → détection OPM/ILE → conversion Free → boucle self-heal → rapport.
> À utiliser en complément du Prompt 4 quand l'objectif est la conversion Free complète,
> pas seulement l'extraction de procédures.
> (Source : Bob IBM i L3 Course — Seismic)

### Pourquoi le mode IBM i Developer pour la conception et la génération ?

UC 8 est l'UC où le risque de dégradation silencieuse est le plus élevé. Le mode **IBM i Developer** apporte la connaissance RPG/ILE spécialisée pour toute la session. La discipline de travail repose sur **la validation humaine avant toute écriture** : Bob produit le plan d'architecture et chaque module dans le chat — l'équipe valide chaque étape avant d'autoriser explicitement l'écriture sur l'IBM i.

> 💡 **Sans Premium Package IBM i :** remplacer IBM i Developer par le mode **Ask** pour les phases d'analyse et de génération, et le mode **Agent** pour la compilation et la sauvegarde. La discipline de validation reste identique.

| Phase | Comportement attendu | Ce que Bob fait |
|-------|----------------------|----------------|
| Qualification du programme (Prompt 0) | Génère dans le chat — pas d'écriture | Analyse le source, identifie les responsabilités, détecte le niveau de couplage, recommande la stratégie et les limites du périmètre POC |
| Plan d'architecture (Prompt 1) | Génère dans le chat — pas d'écriture | Produit le plan de découpe complet — modules, interfaces, ordre d'extraction — dans le chat pour validation avant toute génération de code |
| Génération des nouveaux modules (Prompt 2) | Génère dans le chat — pas d'écriture | Génère chaque module extrait dans le chat — itérations possibles sur les interfaces et le contenu |
| Test de compilation des modules (Prompt 3-bis) | **Écriture et exécution autorisées** — après validation de l'équipe | Lance `CRTBNDRPG` / `CRTRPGMOD` via IBM i MCP, rapporte les erreurs |
| Modification du programme principal (Prompt 3) | Génère dans le chat — pas d'écriture | Génère le programme principal modifié (appels `CALLP`, interfaces adaptées) dans le chat |
| Test de compilation du programme principal (Prompt 3-bis) | **Écriture et exécution autorisées** — après validation de l'équipe | Lance `CRTBNDRPG` via IBM i MCP après modification du programme principal |
| Nettoyage et interfaces (Prompt 4) | Génère dans le chat — pas d'écriture | Produit les prototypes, vérifie la cohérence des interfaces, génère le programme de service (si applicable) |
| Test fonctionnel | **Humain uniquement** | Exécution sur IBM i de test, comparaison des résultats — non délégable à Bob |
| Sauvegarde des livrables validés | **Écriture autorisée** — après validation de chaque livrable | Écrit les fichiers `.md` dans le workspace |

> 💡 **Règle d'or pour UC 8 :** L'écriture et la compilation sur l'IBM i ne sont autorisées qu'après validation explicite de l'équipe. Pendant toute la phase d'analyse, de conception et de génération (Prompts 0 à 4), Bob produit uniquement dans le chat — le mode IBM i Developer le permet, mais l'équipe ne donne pas l'instruction d'écrire.

> ⚠️ Ne jamais autoriser l'écriture pendant la phase de conception du plan d'architecture — une erreur dans le Prompt 1 se propage dans tous les modules générés ensuite.

### Spécificité ARCAD — MCP non disponible

ACME utilise ARCAD pour la gestion du code source IBM i. Le MCP ARCAD n'est **pas actif** dans ce POC (incompatibilité de version).

**Impact sur UC 8 : moyen.** UC 8 crée de nouveaux membres sources (les modules extraits) qui n'existent pas encore dans ARCAD. Ces nouveaux membres devront être enregistrés manuellement dans ARCAD après validation.

| Ce que l'absence du MCP ARCAD change | Ce qui fonctionne quand même |
|--------------------------------------|------------------------------|
| Impossible de créer automatiquement les nouveaux membres dans ARCAD lors de l'extraction | IBM i MCP peut créer les membres dans les bibliothèques source directement — ARCAD les verra ensuite lors de la synchro manuelle |
| Impossible de vérifier si le programme en cours de restructuration est verrouillé par une promotion ARCAD en cours | Vérification manuelle dans l'interface ARCAD avant de démarrer chaque session UC 8 |
| Les nouveaux modules extraits ne sont pas automatiquement intégrés dans les packages de déploiement ARCAD | Réintégration manuelle dans ARCAD — documenter la liste des nouveaux membres créés dans le fichier `*-plan-archi-*.md` |
| L'historique de version du programme original n'est pas accessible depuis Bob | Charger un export ARCAD si l'historique est nécessaire pour comprendre l'évolution du programme |

> 💡 **Contournement :** au début de chaque session UC 8, exporter depuis ARCAD la liste des programmes du périmètre et leur statut (en promotion ou non). Créer un checkpoint ARCAD sur les membres à modifier avant de démarrer — facilite la comparaison avant/après en cas de problème.

> ⚠️ **Écriture dans le fichier ARCAD ouvert — pas dans QSYS directement.** Quand le programme est ouvert depuis une version ARCAD (via Code for IBM i → Object Browser → clic droit → Open in Editor), toutes les écritures de Bob (programme principal modifié, nouveaux modules extraits) doivent cibler les membres dans la **bibliothèque source ARCAD ouverte**, pas un chemin `QSYS` absolu. Si Bob affiche un WARNING et propose un chemin `QSYS`, interrompre et préciser : *"Écris dans [NOM_LIB]/QRPGSRC([NOM_PROGRAMME]) — ne pas écrire dans QSYS directement."* La validation dans le chat avant tout `write_member` est le rempart contre ce cas.

---

## Prompts clés

### Prompt 0 — Qualification et choix de stratégie

> **Ce prompt est le point d'entrée obligatoire de UC 8 pour chaque programme.**
> Il remplace la décision manuelle sur la stratégie (extraction simple / découpe multi-modules / conversion de cycle).
> Il se lance **avant** le Prompt 1 — son résultat conditionne toute la séquence suivante.

```
Le programme [NOM_PROGRAMME] se trouve dans [NOM_LIB]/QRPGSRC.
Les fichiers de compréhension suivants sont disponibles :
  - {appArcad}-{fonction}-{composant}-comprehension-{YYYYMMDD-HHmm}.md
  - {appArcad}-{fonction}-{composant}-regles-{YYYYMMDD-HHmm}.md
  - {appArcad}-{fonction}-{composant}-spec-tech-{YYYYMMDD-HHmm}.md (si disponible)

Analyse ce programme et produis en français, en markdown, une fiche de qualification
pour la restructuration :

## Qualification UC 8 — [NOM_PROGRAMME]

### 1. Inventaire rapide
| Indicateur                                               | Valeur |
|----------------------------------------------------------|--------|
| Nombre de lignes de source                               | ?      |
| Nombre de subroutines (BEGSR/ENDSR)                      | ?      |
| Nombre de procédures déjà présentes (DCL-PROC)           | ?      |
| Cycle RPG actif (*INLR, niveaux de contrôle L1-L9)       | OUI / NON |
| Nombre de responsabilités fonctionnelles distinctes identifiées | ?  |
| Nombre de variables globales (portée tout le programme)  | ?      |
| Nombre de programmes appelants (matrice UC 6 disponible ?)| ?     |
| Programmes appelés (CALL / CALLP dans le source)         | ?      |
| Présence d'écrans 5250 (EXFMT / WRITE vers WDSPF)        | OUI / NON |
| Présence de gestion transactionnelle (COMMIT / ROLLBACK / ROLBK) | OUI / NON |

### 2. Identification des responsabilités fonctionnelles
Pour chaque responsabilité identifiée dans le programme :
| N° | Responsabilité | Subroutines associées | Variables exclusives | Variables partagées | Extractible ? |
Extractible : OUI si la responsabilité peut être isolée avec des interfaces claires
              NON si le couplage aux variables globales est trop fort pour l'isoler
              PARTIELLE si extractible uniquement après refactoring interne préalable

### 3. Facteurs de complexité détectés
Réponds par OUI / NON / À CONFIRMER pour chaque facteur :
- [ ] Cycle RPG actif avec niveaux de contrôle (L1-L9, *INLR comme seul pilotage)
- [ ] Variables globales > 20 (couplage fort entre responsabilités)
- [ ] Programmes appelants multiples (> 3) — modification d'interface à impact large
- [ ] Logique transactionnelle imbriquée (COMMIT/ROLLBACK au milieu d'une subroutine)
- [ ] Écrans 5250 imbriqués dans la logique métier (EXFMT dans une subroutine de calcul)
- [ ] Accès natifs encore présents (UC 3 non appliqué sur ce programme)
- [ ] Code dupliqué entre ce programme et d'autres programmes du périmètre
- [ ] Dépendances sur des bibliothèques ou formats de fichiers spécifiques à l'objet

### 4. Catégorie et stratégie recommandée
Sur la base de l'inventaire et des facteurs de complexité, conclure :

**Catégorie :**
- [ ] SIMPLE — 1 à 2 responsabilités séparables, peu de variables partagées, pas de cycle RPG
- [ ] STANDARD — 2 à 4 responsabilités, variables partagées gérables, ou cycle RPG simple
- [ ] COMPLEXE — > 4 responsabilités imbriquées, ou cycle RPG actif complexe,
                  ou programme interactif, ou > 3 programmes appelants

> Note : UC 8 utilise SIMPLE / STANDARD / COMPLEXE (critère : nombre de responsabilités et
> niveau de couplage). Même vocabulaire que UC 14 (critère : champs DDS), UC 3 (critère :
> opcodes d'accès) et UC 7 (critère : points d'optimisation) — vocabulaire unifié Phase 2
> et Phase 3. Les échelles sont indépendantes : un programme SIMPLE en UC 8 a pu être
> COMPLEXE en UC 7. Le Prompt 0 de chaque UC calibre selon ses propres critères.

**Recommandation :**
- SIMPLE   → Extraction directe d'un module :
              Prompt 1 → Prompt 2 (module unique) → Prompt 3-bis → Prompt 3 → Prompt 3-bis → Prompt 4
- STANDARD → Découpe par responsabilités avec plan validé :
              Prompt 1 (plan complet, validation humaine obligatoire) →
              Prompt 2 lot par lot → Prompt 3-bis après chaque lot →
              Prompt 3 (programme principal) → Prompt 3-bis → Prompt 4
- COMPLEXE → Périmètre restreint recommandé pour le POC :
              Identifier la responsabilité la plus isolable → traiter en SIMPLE/STANDARD
              Reporter les extractions à fort couplage après le POC
              Si cycle RPG : Prompt 1 (plan conversion cycle → procédures) → traitement dédié

**Périmètre recommandé pour le POC (si COMPLEXE) :**
Limiter à [décrire la partie la plus isolable] — reporter [décrire ce qui est trop risqué]

Signale clairement les facteurs qui rendent cette restructuration risquée dans le cadre du POC.
```

**Analyse ligne à ligne :**

- `Les fichiers de compréhension suivants sont disponibles` → ancrage explicite sur les livrables Phase 1. Le Prompt 0 de UC 8 doit être alimenté par ces fichiers — sans eux, Bob analyse uniquement le source et peut manquer des règles métier critiques portées par des commentaires ou de la documentation interne.

- `Nombre de responsabilités fonctionnelles distinctes identifiées` → le critère central de UC 8. Une "responsabilité" est un bloc fonctionnel cohérent (calculer le montant TTC, valider une commande, charger les données client) — pas une subroutine technique (gérer une erreur, lire une ligne). Bob doit identifier les responsabilités, pas les compter mécaniquement.

- `Extractible : OUI / NON / PARTIELLE` → trois états. "PARTIELLE" est délibéré — une responsabilité peut être extractible uniquement après un refactoring interne (par exemple, séparer deux blocs actuellement entrelacés dans la même subroutine). Ce cas est fréquent en legacy IBM i et doit être identifié au Prompt 0, pas découvert en cours de génération.

- `Cycle RPG actif` → point bloquant potentiel. Le cycle RPG est un mécanisme de traitement implicite qui exécute automatiquement des phases (`*GETIN`, `*DETC`, `*TOTC`, `*DETL`...) sans appel explicite. Un programme qui en dépend fortement ne peut pas être restructuré en procédures sans convertir d'abord ce cycle en boucle `DOW/READ/ENDDO` explicite — une opération qui modifie le flux d'exécution principal.

- `Programmes appelants multiples` → facteur de risque d'interface. Chaque modification du prototype d'appel du programme (`*ENTRY PLIST`) doit être répercutée sur tous les appelants. La matrice UC 6 est la seule source fiable pour les identifier tous.

- `Catégorie SIMPLE / STANDARD / COMPLEXE` → vocabulaire unifié Phase 2 et Phase 3, avec critères propres à UC 8 (différents de UC 3, UC 7 et UC 14). La note explicite dans le prompt rappelle cette convention.

- `Périmètre recommandé pour le POC (si COMPLEXE)` → restriction explicite pour le POC. Mieux vaut documenter ce qui est exclu du POC que tenter une restructuration dont le risque dépasse ce qu'une équipe en phase d'apprentissage peut gérer.

> 💡 **Sauvegarder la fiche de qualification** (mode Agent) dès la fin du Prompt 0 :
> ```
> "Sauvegarde cette fiche de qualification dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-qualification-restr-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-qualification-restr-20250620-0900.md"
> ```
> Le Prompt 1 (plan d'architecture) s'appuie sur cette qualification.

> ⚠️ **Piège évité :** sans le Prompt 0, l'équipe démarre le plan d'architecture sans savoir si le programme a un cycle RPG actif ou des programmes appelants multiples. Découvrir en cours de génération qu'un programme a 8 appelants dont les interfaces seraient cassées oblige à tout réexaminer depuis le début.

---

### Prompt 1 — Plan d'architecture de la restructuration

```
Sur la base de la qualification UC 8 de [NOM_PROGRAMME] et du fichier de compréhension
{appArcad}-{fonction}-{composant}-comprehension-{YYYYMMDD-HHmm}.md :

Produis en français, en markdown, le plan d'architecture complet de la restructuration :

## Plan d'architecture UC 8 — [NOM_PROGRAMME]

### 1. Architecture actuelle
Description synthétique de l'organisation du programme actuel :
- Nombre de subroutines et leur rôle
- Variables globales principales et leur usage
- Interface d'appel actuelle (*ENTRY PLIST ou prototype DCL-PR)
- Fichiers accédés et mode d'accès (natif / SQL embarqué)

### 2. Architecture cible
Pour chaque module à créer ou à modifier :
| Module | Type (programme / module de service) | Responsabilité | Source des données (paramètres / DS partagée) | Ordre de création |
Préciser pour chaque module :
- Nom proposé (convention : [NOM_ORIGINAL]_[SUFFIXE_ROLE], ex : GESCMD_CALC)
- Type IBM i : *PGM (programme appelable par CALL) ou *SRVPGM (module de service,
  appelable par CALLP avec prototype)
- Interface : liste des paramètres IN / OUT / INOUT avec types
- Responsabilité exclusive : règles métier portées par ce module (référencer *-regles-*.md)

### 3. Interface du programme principal après restructuration
L'interface externe du programme appelant doit-elle changer ?
- OUI → lister les appelants identifiés dans *-matrice-*.md et le plan de migration
- NON → confirmer que l'interface *ENTRY PLIST est préservée

### 4. Variables partagées entre modules
Pour chaque variable globale qui sera partagée entre le programme principal et
les modules extraits :
| Variable | Type | Sens (IN / OUT / INOUT) | Module émetteur | Module(s) récepteur(s) | Devient paramètre ou DS partagée |

### 5. Ordre d'extraction recommandé
Les modules doivent être créés dans un ordre qui permet de compiler et tester
progressivement — ne jamais créer tous les modules en une seule passe :
1. [Module le plus indépendant] → compiler → tester
2. [Module suivant avec dépendance sur 1] → compiler → tester
3. [Programme principal modifié] → compiler → tester complet

### 6. Risques identifiés et mesures d'atténuation
Les 3 à 5 points les plus risqués de cette restructuration spécifique.

Ne pas générer de code dans ce prompt.
Signale clairement les décisions qui doivent être prises par l'équipe
avant de lancer la génération de code.
```

**Analyse ligne à ligne :**

- `Architecture actuelle` → point de départ documenté. Un plan d'architecture qui commence par la description de l'état actuel permet à l'équipe de valider que Bob a bien compris la structure avant de proposer des modifications. Si la description de l'état actuel est incorrecte, le plan cible sera faux.

- `Type IBM i : *PGM ou *SRVPGM` → décision d'architecture fondamentale. Un `*PGM` est appelable par `CALL` depuis CL ou d'autres programmes — interface classique IBM i. Un `*SRVPGM` est un programme de service lié au programme appelant via `BNDDIR` — interface de procédure, plus efficace en performance mais nécessite un `CRTSRVPGM` et une binding directory. Sur IBM i legacy, les `*SRVPGM` sont moins courants — les introduire demande une décision consciente de l'équipe.

- `Interface : liste des paramètres IN / OUT / INOUT` → les directions des paramètres doivent être explicites dans le plan. Un paramètre `INOUT` crée un couplage bi-directionnel — le module lit et modifie la valeur, ce qui est plus risqué qu'un paramètre `IN` (lecture seule) ou `OUT` (écriture uniquement).

- `L'interface externe du programme appelant doit-elle changer ?` → question binaire obligatoire dans le plan. Si la réponse est OUI, la liste des appelants (matrice UC 6) devient un livrable de migration à part entière. Si la réponse est NON (souhaitée dans le POC), c'est une contrainte de conception que Bob doit respecter dans toute la suite.

- `Variables partagées entre modules — Devient paramètre ou DS partagée` → le choix entre paramètre et DS partagée est la décision de conception la plus importante de UC 8. Un paramètre est explicite et testable isolément. Une DS partagée est plus compacte mais crée un couplage implicite entre les modules — deux modules qui modifient la même DS peuvent interférer.

- `Ne pas générer de code dans ce prompt` → séparation conception / génération. Le plan doit être relu et validé par un développeur senior **avant** de lancer les Prompts 2 et 3. Une architecture mal conçue au Prompt 1 se retrouvera dans tous les modules générés ensuite.

> 💡 **Sauvegarder le plan d'architecture** (mode Agent) après validation par l'équipe :
> ```
> "Sauvegarde ce plan d'architecture dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-plan-archi-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-plan-archi-20250620-1000.md"
> ```
> Ce fichier est la référence de toute la suite — Prompts 2, 3 et 4 en dépendent.

> ⚠️ **Piège évité :** générer les modules sans plan validé produit des interfaces incohérentes entre eux. Le module A passe un paramètre que le module B n'attend pas, ou deux modules modifient la même variable sans se coordonner. Corriger des incohérences d'interfaces en cours de génération est beaucoup plus coûteux que de les prévenir au Prompt 1.

---

### Prompt 2 — Génération d'un module extrait

```
Sur la base du plan d'architecture UC 8 de [NOM_PROGRAMME]
({appArcad}-{fonction}-{composant}-plan-archi-{YYYYMMDD-HHmm}.md),
génère le module [NOM_MODULE] correspondant à la responsabilité
"[DESCRIPTION_RESPONSABILITE]".

Structure attendue du nouveau source RPG Free :

**F-specs / DCL-F :** uniquement les fichiers accédés exclusivement par ce module
**D-specs / DCL-S, DCL-DS :** uniquement les variables nécessaires à ce module
                               (pas de copie des variables globales du programme principal)
**Interface (DCL-PI) :**
  [PARAM1] [TYPE] [VALUE/CONST/sans qualificatif] ;  // IN : ...
  [PARAM2] [TYPE] ;                                   // OUT : ...
  ...

**Corps du module :**
  [Logique extraite depuis [NOM_PROGRAMME] — subroutines [BEGSR1], [BEGSR2]...]
  Adaptée pour :
  - Utiliser les paramètres au lieu des variables globales d'origine
  - Remplacer les EXSR internes par des appels CALLP si la subroutine reste dans
    le programme principal, ou inclure le corps si la subroutine est déplacée dans ce module

**Règles :**
- Nom du module : [NOM_MODULE] (convention : [NOM_ORIGINAL]_[SUFFIXE_ROLE])
- Type cible : [*PGM / *SRVPGM] selon le plan d'architecture
- En-tête obligatoire :
  // ============================================================
  // MODULE : [NOM_MODULE]
  // Extrait de : [NOM_LIB]/QRPGSRC([NOM_PROGRAMME])
  // Responsabilité : [DESCRIPTION_RESPONSABILITE]
  // Généré le : [DATE]
  // Statut : À valider et tester sur IBM i de test avant tout usage
  // ⚠️ Réintégration ARCAD — à effectuer manuellement après validation
  // ============================================================
- Ne pas copier de logique hors périmètre de la responsabilité de ce module
- Ne pas inventer de variables ou de règles non visibles dans le source d'origine
  ou dans les fichiers *-regles-*.md
- Signaler chaque endroit où la logique extraite suppose une décision non tranchée
```

**Analyse ligne à ligne :**

- `DCL-F uniquement les fichiers accédés exclusivement par ce module` → règle de partition des ressources. Si un fichier est accédé par plusieurs modules, il ne peut pas être déclaré en DCL-F dans chacun — il doit être accédé via le programme principal (qui le déclare) ou via un paramètre de structure. Cette règle évite les conflits d'ouverture de fichier IBM i.

- `Interface (DCL-PI) — avec commentaires IN / OUT` → les commentaires de direction dans le code lui-même. Quand un développeur lit le source du module 6 mois après l'extraction, il peut voir immédiatement quels paramètres sont en entrée, en sortie, ou en entrée-sortie — sans aller chercher le plan d'architecture.

- `Adaptée pour utiliser les paramètres au lieu des variables globales d'origine` → la règle d'extraction. Chaque fois qu'une subroutine utilisait `wkMontantHT` (variable globale), le module extrait utilise le paramètre `pMontantHT` passé à l'appel. Ce mapping entre l'ancien nom global et le nouveau nom de paramètre doit être explicite dans le diff.

- `Ne pas copier de logique hors périmètre` → garde-fou anti-sur-extraction. Bob peut être tenté d'inclure dans un module des subroutines "qui semblent liées" mais qui ne font pas partie de la responsabilité définie. Un module qui en fait trop est aussi difficile à tester qu'un monolithe.

- `Ne pas inventer de variables ou de règles non visibles` → garde-fou anti-hallucination central pour UC 8. La logique extraite doit provenir du source d'origine ou des fichiers de règles métier — jamais d'une inférence de Bob sur ce que "devrait" faire le module.

> 💡 **Sauvegarder la spec du module** (mode Agent) :
> ```
> "Sauvegarde ce nouveau module dans un fichier nommé
>  {appArcad}-{fonction}-{NOM_MODULE}-spec-module-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD_CALC-spec-module-20250620-1100.md"
> ```

> ⚠️ **Piège évité :** générer un module qui accède directement à un fichier déclaré en DCL-F dans le programme principal crée un conflit d'ouverture sur IBM i — le fichier est déjà ouvert par le programme principal et le module ne peut pas l'ouvrir à nouveau. Ce cas ne produit pas d'erreur de compilation mais une erreur à l'exécution, difficile à diagnostiquer.

---

### Prompt 3 — Modification du programme principal

```
Sur la base du plan d'architecture UC 8 de [NOM_PROGRAMME] et des modules
déjà générés et validés :
  [NOM_MODULE_1] — spec dans {appArcad}-{fonction}-{NOM_MODULE_1}-spec-module-*.md
  [NOM_MODULE_2] — spec dans {appArcad}-{fonction}-{NOM_MODULE_2}-spec-module-*.md
  ...

Produis le programme principal [NOM_PROGRAMME] modifié après extraction des modules :

### Modifications à apporter :
1. Ajout des prototypes (DCL-PR) pour chaque module extrait :
   DCL-PR [NOM_MODULE] [TYPE_RETOUR] ;
     [PARAMS selon les DCL-PI générés au Prompt 2]
   END-PR ;

2. Remplacement des EXSR [NOM_BEGSR] par des CALLP [NOM_MODULE]([PARAMS]) :
   Pour chaque subroutine extraite, produire le diff :
   // AVANT : EXSR [NOM_BEGSR]
   CALLP [NOM_MODULE]([LISTE_PARAMS]) ;

3. Suppression des blocs BEGSR/ENDSR extraits :
   - Conserver en commentaire pendant la session de validation
   - Lister les subroutines supprimées avec leur module de destination

4. Adaptation des variables globales devenues paramètres :
   - Conserver les DCL-S des variables qui sont encore utilisées dans le programme principal
   - Supprimer les DCL-S des variables exclusivement utilisées dans les modules extraits
     (elles sont maintenant déclarées dans les modules)

5. Si le type cible est *SRVPGM : ajouter la référence à la binding directory
   H BNDDIR('[NOM_BNDDIR]')

### Règles :
- L'interface externe du programme (*ENTRY PLIST ou DCL-PI) ne doit pas changer
  sauf si le plan d'architecture section 3 indique OUI et liste les appelants
- Conserver toutes les F-specs des fichiers accédés directement par le programme principal
- Conserver la logique qui n'a pas été extraite dans un module — ne pas supprimer
  de code hors périmètre de la restructuration
- Produire un récapitulatif des modifications :
  | Subroutine extraite | Module destination | Appel CALLP généré | Paramètres passés |
```

**Analyse ligne à ligne :**

- `Ajout des prototypes DCL-PR` → le premier élément à générer dans le programme principal modifié. Sans prototype, `CALLP [NOM_MODULE]` provoque une erreur de compilation. Les prototypes doivent correspondre exactement aux `DCL-PI` générés au Prompt 2 — Bob peut vérifier cette cohérence en interne.

- `// AVANT : EXSR [NOM_BEGSR]` → conserver le code d'origine en commentaire. Pendant la validation, le développeur peut voir l'ancien appel `EXSR` et le nouveau `CALLP` côte à côte. Ces commentaires sont retirés une fois la validation terminée.

- `Adapter les variables globales` → nettoyage symétrique de celui du Prompt 2. Les variables qui sont maintenant des paramètres de modules ne doivent plus être des variables globales du programme principal — les deux déclarations coexistantes créent de la confusion et potentiellement des conflits de noms.

- `L'interface externe ne doit pas changer` → contrainte absolue dans le POC sauf indication contraire explicite. Une modification de `*ENTRY PLIST` est un changement d'interface public — tous les appelants doivent être mis à jour. Cette contrainte doit être vérifiée explicitement par Bob dans ce prompt.

- `Récapitulatif des modifications` → la traçabilité de la restructuration au niveau du programme principal. Ce tableau est l'équivalent du diff de UC 7 — il documente chaque transformation pour la revue de code et la réintégration ARCAD.

> 💡 **Sauvegarder le diff du programme principal** (mode Agent) :
> ```
> "Sauvegarde ce diff du programme principal dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-diff-restr-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-diff-restr-20250620-1400.md"
> ```

> ⚠️ **Piège évité :** oublier de supprimer les BEGSR/ENDSR extraits du programme principal laisse du code mort dans le source. Ce code ne provoque pas d'erreur de compilation mais peut induire en erreur un développeur qui modifie la subroutine morte en croyant modifier le module extrait.

---

### Prompt 3-bis — Test de compilation par Bob (après chaque module ou après le programme principal)

> **Ce que Bob peut faire :** lancer la compilation via IBM i MCP en mode **Agent** et rapporter les erreurs.
> **Ce que Bob ne peut pas faire :** valider que la logique extraite se comporte fonctionnellement comme l'original — c'est une vérification humaine irréductible.
> **Quand l'utiliser :** après chaque Prompt 2 (nouveau module) et après le Prompt 3 (programme principal modifié), avant de passer à la suite.

```
Le source RPG de [NOM_MODULE ou NOM_PROGRAMME] dans [NOM_LIB]/QRPGSRC a été
créé ou modifié selon le plan d'architecture UC 8.

Lance la compilation sur l'IBM i de test :

Si c'est un module *PGM (programme complet) :
CRTBNDRPG PGM([NOM_LIB_TEST]/[NOM_PROGRAMME_OU_MODULE])
          SRCFILE([NOM_LIB]/QRPGSRC)
          SRCMBR([NOM_PROGRAMME_OU_MODULE])
          OPTION(*EVENTF *LIST)
          DBGVIEW(*SOURCE)

Si c'est un module *SRVPGM (programme de service) :
CRTRPGMOD MODULE([NOM_LIB_TEST]/[NOM_MODULE])
           SRCFILE([NOM_LIB]/QRPGSRC)
           SRCMBR([NOM_MODULE])
           OPTION(*EVENTF *LIST)
           DBGVIEW(*SOURCE)
-- puis, après compilation de tous les modules :
CRTSRVPGM SRVPGM([NOM_LIB_TEST]/[NOM_SRVPGM])
           MODULE([NOM_MODULE1] [NOM_MODULE2])
           EXPORT(*ALL)

Analyse le résultat et produis en français :

1. Statut : COMPILATION RÉUSSIE / ERREURS DE COMPILATION
2. Si erreurs : liste des erreurs avec numéro de ligne, code erreur IBM i et description
   | Ligne | Code erreur | Description | Cause probable |
   Causes probables à vérifier : paramètre manquant dans DCL-PR (prototype incohérent
   avec DCL-PI), F-spec de fichier déjà ouvert par le programme appelant, variable
   globale manquante (devenue paramètre mais pas encore adaptée dans ce source),
   BEGSR restant référencé par un EXSR non encore remplacé par CALLP
3. Si compilation réussie : confirmer que [NOM_MODULE] est créé dans [NOM_LIB_TEST]
   et prêt pour les tests suivants
4. Rappeler que la compilation réussie ne garantit pas que la logique extraite se
   comporte fonctionnellement comme l'original — le test fonctionnel reste obligatoire
```

**Analyse ligne à ligne :**

- `CRTBNDRPG` vs `CRTRPGMOD` + `CRTSRVPGM` → les deux chemins de compilation selon le type IBM i cible. Un `*PGM` se compile en une seule commande (`CRTBNDRPG`). Un `*SRVPGM` nécessite deux étapes : compiler le module (`CRTRPGMOD`) puis lier le programme de service (`CRTSRVPGM`). Bob doit savoir quelle commande utiliser selon l'architecture choisie au Prompt 1.

- `EXPORT(*ALL)` dans `CRTSRVPGM` → exporte toutes les procédures du programme de service. Dans un POC, c'est le paramètre le plus simple. En production, on utilise un fichier d'export sélectif pour ne pas exposer les procédures internes.

- `Causes probables spécifiques à UC 8` → les erreurs de compilation liées à la restructuration ont des patterns propres : `RNF3312` (symbole non défini dans le module extrait car c'était une variable globale du programme principal), `MCH3401` (conflit d'ouverture de fichier, détecté à l'exécution pas à la compilation), `RNF0113` (prototype incohérent avec l'implémentation).

> 💡 **Ce prompt s'exécute en mode Agent** — mêmes prérequis de droits IBM i que les Prompts 2-bis et 3-bis de UC 7.

> ⚠️ **Ce prompt ne remplace pas le test fonctionnel.** Les tests suivants restent obligatoires et ne peuvent pas être délégués à Bob :
> - Exécuter le programme principal sur l'IBM i de test avec un jeu de données réel
> - Vérifier que le programme modifié appelle bien les modules extraits dans le bon ordre
> - Comparer les résultats (enregistrements produits, valeurs calculées) avec le programme d'origine non modifié
> - Tester les cas limites des règles métier portées par les modules extraits (référencer `*-regles-*.md`)

---

### Prompt 4 — Conversion du cycle RPG en structure procédurale

> **Ce prompt est spécifique aux programmes qui utilisent le cycle RPG actif.**
> Ne l'utiliser que si le Prompt 0 a détecté `Cycle RPG actif = OUI`.
> C'est la transformation la plus risquée de UC 8 — réserver au personnel RPG senior.

~~~
Le programme [NOM_PROGRAMME] utilise le cycle RPG actif avec les éléments suivants
identifiés au Prompt 0 :
[Copier ici le résultat de la section "Cycle RPG actif" du Prompt 0]

Produis en français, en markdown, le plan de conversion du cycle RPG en structure
procédurale explicite :

1. Inventaire du cycle actuel
Pour chaque élément du cycle présent dans le programme :
| Élément cycle             | Usage actuel                              | Équivalent procédural                                                  |
|---------------------------|-------------------------------------------|------------------------------------------------------------------------|
| *INLR                     | Terminaison du programme                  | RETURN (main proc) / *INLR = *ON explicite                             |
| Niveaux de contrôle L1-L9 | Totaux de rupture                         | Variables de contrôle + IF/WHEN sur changement de clé                  |
| *GETIN implicite          | Lecture automatique du fichier primaire   | DOW NOT %EOF / EXEC SQL FETCH (si UC 3 appliqué)                       |
| Subroutines de détail     | Exécutées à chaque enregistrement         | Corps de la boucle principale                                          |
| Subroutines de total (Ln) | Exécutées à la rupture de niveau          | IF [clé_actuelle] <> [clé_précédente] ; EXSR [TOTAL] ; ENDIF           |

2. Structure procédurale cible
Squelette du programme après conversion :

  // Boucle principale remplaçant le cycle
  DOW NOT %EOF([NOM_FICHIER_PRIMAIRE]) ;  // ou DOW SQLCODE = 0 si SQL embarqué
    // Détection des ruptures de niveau (remplace les niveaux L1-L9)
    IF [CLE_L1_ACTUELLE] <> [CLE_L1_PRECEDENTE] ;
      EXSR TOTAL_L1 ;  // → sera extrait en module si plan d'architecture le prévoit
      [CLE_L1_PRECEDENTE] = [CLE_L1_ACTUELLE] ;
    ENDIF ;
    // Corps de traitement (remplace la subroutine de détail)
    EXSR TRAITER_DETAIL ;
    // Lecture suivant (si natif — sinon FETCH SQL)
    READ [NOM_FICHIER_PRIMAIRE] ;
  ENDDO ;
  // Traitement des derniers totaux
  EXSR TOTAL_L1 ;
  *INLR = *ON ;
  RETURN ;

3. Règles de conversion
- *INLR = *ON au lieu du cycle implicite : RETURN en fin de programme principal
- Niveau de contrôle Ln → variable [CLE_PRECEDENTE] pour la détection de rupture
- Lectures implicites → READ (ou FETCH SQL si UC 3 appliqué) explicite en fin de boucle
- Subroutines de total sans paramètre → EXSR ou CALLP selon le plan d'architecture
- Ne pas modifier la logique des subroutines — seulement leur mode de déclenchement
- Signaler tout cas où la sémantique du cycle ne peut pas être reproduite exactement

4. Impact sur les modules extraits au Prompt 2
Les modules extraits au Prompt 2 doivent-ils être adaptés suite à la conversion du cycle ?
[Analyse des dépendances entre cycle et modules]
~~~

**Analyse ligne à ligne :**

- `Ne l'utiliser que si le Prompt 0 a détecté Cycle RPG actif = OUI` → le Prompt 4 est conditionnel. Il ne doit pas être utilisé sur les programmes sans cycle — appliqué à tort, il introduit une boucle `DOW/READ` là où il n'y en avait pas besoin.

- `*GETIN implicite → DOW NOT %EOF / FETCH SQL` → l'équivalent de la lecture automatique du fichier primaire par le cycle RPG. Si UC 3 a été appliqué sur ce programme, la lecture est déjà un `FETCH` SQL — la boucle `DOW SQLCODE = 0` remplace le `DOW NOT %EOF`.

- `Détection des ruptures de niveau (L1-L9)` → la conversion la plus délicate. En cycle RPG, les niveaux de contrôle déclenchent automatiquement les subroutines de total lors d'un changement de valeur de la clé de contrôle. En procédures, cette détection doit être explicite : comparer la valeur courante de la clé avec la valeur précédente, et déclencher la subroutine de total si elles diffèrent.

- `Ne pas modifier la logique des subroutines` → la conversion du cycle ne touche pas le contenu des subroutines — seulement leur mode de déclenchement (implicite par le cycle → explicite par un `IF` ou un `EXSR`). C'est la frontière entre UC 8 (conversion de cycle = structure) et une modification fonctionnelle interdite.

> 💡 **Sauvegarder le plan de conversion du cycle** (mode Agent) :

```
"Sauvegarde ce plan de conversion du cycle RPG dans un fichier nommé
 {appArcad}-{fonction}-{composant}-diff-restr-{YYYYMMDD-HHmm}.md
 (ou complète le fichier diff-restr existant)"
```

> ⚠️ **Piège évité :** en cycle RPG, les subroutines de total s'exécutent **après** la lecture du premier enregistrement du groupe suivant — ce n'est pas "avant la rupture" mais "après la détection". En procédures, la détection et l'exécution du total doivent reproduire fidèlement cette sémantique. Un total déclenché au mauvais moment produit des cumuls incorrects silencieusement.

---

### Prompt 5 — Plan de restructuration pour un périmètre applicatif complet

```
Sur la base des fichiers de compréhension ({appArcad}-{fonction}-*-comprehension-*.md),
des règles métier ({appArcad}-{fonction}-*-regles-*.md), des spécifications techniques
({appArcad}-{fonction}-*-spec-tech-*.md) et de la matrice de références croisées
({appArcad}-{fonction}-*-matrice-*.md) pour l'application [NOM_APPLICATION] dans [NOM_LIB],
génère un plan de restructuration en français, en markdown.

## Plan de restructuration UC 8 — [NOM_APPLICATION]

### 1. Périmètre des programmes à restructurer
| Programme | Nb lignes | Nb responsabilités identifiées | Cycle RPG | Appelants | Catégorie (S/St/C) | Priorité |
Catégorie : S = SIMPLE / St = STANDARD / C = COMPLEXE (critères UC 8)
Priorité : décroissante selon la combinaison "valeur de la restructuration × risque acceptable"

### 2. Programmes à exclure du périmètre de restructuration
Les programmes pour lesquels le risque dépasse la valeur dans le cadre du POC —
avec la justification et la condition qui permettrait de les traiter en production.

### 3. Architecture applicative cible
Vue synthétique : quels programmes deviennent des modules de service (*SRVPGM) ?
Quels programmes restent des programmes *PGM ? Y a-t-il des opportunités de
consolidation (plusieurs programmes similaires pouvant partager un module commun) ?

### 4. Dépendances entre restructurations
Les programmes qui ont des dépendances d'interface — l'ordre dans lequel ils
doivent être restructurés pour ne pas casser les appelants en cours de parcours.

### 5. Risques identifiés sur le périmètre
Les 3 à 5 restructurations les plus risquées — avec la raison et la recommandation
(reporter à UC 8 phase 2, ou traiter en priorité avec renforts).

### 6. Estimation d'effort
| Programme | Prompts nécessaires | Durée estimée (avec Bob) | Modules créés | Risque résiduel |

Ne pas inventer de programmes, modules ou interfaces non visibles dans les sources disponibles.
Signaler les cas où les fichiers de compréhension sont insuffisants pour planifier
la restructuration en toute sécurité.
```

> 💡 **Sauvegarder ce plan** (mode Agent) :

```
"Sauvegarde ce plan de restructuration dans un fichier nommé
 {appArcad}-{fonction}-{fonction}-plan-restr-{YYYYMMDD-HHmm}.md
 Exemple : acme-APPVTE-APPVTE-plan-restr-20250620-0800.md"
```

> 💡 Ce plan est le document de pilotage de UC 8 sur le périmètre. Il permet d'identifier les restructurations à fort ROI, celles à reporter, et les dépendances d'ordre entre programmes.

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Ouverture des membres sources RPG, création de nouveaux membres pour les modules extraits, compilation et navigation dans les erreurs inline |
| **IBM i Languages** | Coloration syntaxique RPG Free — indispensable pour lire et valider les modules extraits et les interfaces dans l'éditeur |
| **Markdown All in One** | Prévisualisation des plans d'architecture, des specs de modules et des diffs sauvegardés |

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture des membres sources RPG (`QRPGSRC`) via `read_member` ; création des nouveaux membres pour les modules extraits via `write_member` ; compilation (`CRTBNDRPG`, `CRTRPGMOD`, `CRTSRVPGM`) via `execute_cl_command` ou `execute_compile_action` (Prompt 3-bis) |
| **IBM i Database MCP** | Optionnel — vérification que les accès SQL embarqués (UC 3) présents dans un module extrait référencent des tables existantes, via `QSYS2.SYSTABLES` |
| **Confluence MCP** *(si disponible)* | Publication des plans d'architecture et des specs de modules dans l'espace POC |

> 💡 **Requêtes QSYS2 utiles pour UC 8 :**
> ```sql
> -- Lister les programmes qui appellent un programme donné (candidats à l'impact d'interface)
> SELECT OBJNAME, OBJTYPE, OBJLIBRARY, TEXT_DESCRIPTION
> FROM QSYS2.OBJECT_STATISTICS
> WHERE OBJLIBRARY = '[NOM_LIB]' AND OBJTYPE = '*PGM'
> ORDER BY OBJNAME;
>
> -- Vérifier l'existence d'un *SRVPGM créé lors de la restructuration
> SELECT OBJNAME, OBJTYPE, OBJSIZE, CREATION_TIMESTAMP
> FROM QSYS2.OBJECT_STATISTICS
> WHERE OBJLIBRARY = '[NOM_LIB_TEST]'
>   AND OBJTYPE IN ('*PGM', '*SRVPGM', '*MODULE')
>   AND OBJNAME LIKE '[NOM_ORIGINAL]%'
> ORDER BY CREATION_TIMESTAMP DESC;
>
> -- Vérifier les procédures exportées d'un *SRVPGM
> SELECT ROUTINE_NAME, ROUTINE_SCHEMA, EXTERNAL_NAME,
>        PARAMETER_COUNT
> FROM QSYS2.SYSROUTINES
> WHERE EXTERNAL_NAME LIKE '[NOM_SRVPGM]/%'
>   AND ROUTINE_SCHEMA = '[NOM_LIB_TEST]';
> ```
> Ces requêtes permettent de vérifier les objets créés par la restructuration sans sortir de Bob.

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Sauter le Prompt 0 sur un programme COMPLEXE | La présence d'un cycle RPG actif ou de 8 programmes appelants n'est pas détectée — la restructuration est lancée avec un périmètre trop large et des interfaces qui cassent les appelants | Toujours lancer le Prompt 0 en premier — il prend 5 minutes et évite de perdre une journée de débogage |
| Démarrer le Prompt 2 sans valider le Prompt 1 avec un développeur senior | Le plan d'architecture est incorrect (mauvaise partition des responsabilités, interfaces mal définies) — tous les modules générés sont fondés sur un plan faux | Le Prompt 1 ne génère pas de code — il produit un plan. La validation humaine du plan est obligatoire avant tout `write_member` |
| Modifier l'interface externe (*ENTRY PLIST) sans inventaire des appelants | Tous les programmes appelants cassent à la compilation ou à l'exécution — sur un programme appelé par 15 autres, le diagnostic est long | Le Prompt 0 identifie le nombre d'appelants ; le Prompt 3 rappelle explicitement la contrainte d'interface ; la matrice `*-matrice-*.md` liste les appelants |
| Déclarer en DCL-F un fichier dans un module extrait alors qu'il est déjà ouvert par le programme principal | Le fichier est ouvert deux fois — erreur MCH3401 à l'exécution sur l'IBM i (pas à la compilation) | Le Prompt 2 applique la règle : DCL-F uniquement pour les fichiers accédés **exclusivement** par le module |
| Convertir le cycle RPG sans Prompt 4 dédié | Le cycle est partiellement supprimé mais les niveaux de contrôle et les totaux de rupture ne sont pas reproduits — le programme produit des cumuls incorrects | Le Prompt 0 détecte le cycle RPG et conditionne l'utilisation du Prompt 4 — ne jamais supprimer *INLR ou des niveaux de contrôle sans ce prompt |
| Effectuer une restructuration et une optimisation (UC 7) dans la même passe | Les diffs se mélangent — impossible de savoir si une erreur fonctionnelle vient du renommage (UC 7) ou de l'extraction de module (UC 8) | UC 7 d'abord, compiler et valider, puis UC 8 sur le source propre — jamais les deux en simultané sur le même programme |
| Générer tous les modules en une seule passe sans compilation intermédiaire | Les erreurs d'interface entre modules s'accumulent — il y a 5 modules à corriger en même temps | Le Prompt 5 (plan périmètre) et le Prompt 1 (plan programme) définissent un ordre d'extraction — compiler et tester après chaque module, pas après tous |
| Travailler en mode Agent pendant la conception du plan | Bob peut générer des sources intermédiaires basés sur un plan non validé | Rester en mode **Ask** pendant les Prompts 0 à 4 — mode Agent uniquement pour Prompt 3-bis (compilation) et sauvegarde finale |
| Démarrer UC 8 sans les fichiers *-comprehension-*.md et *-regles-*.md | Les modules extraits peuvent porter des noms incorrects, des responsabilités floues, ou manquer des règles métier critiques | Vérifier l'existence de ces fichiers avant de démarrer — 30 minutes d'UC 4 sur le programme ciblé évitent une restructuration à refaire |

---

## Check-list de validation UC 8

Avant de déclarer un programme restructuré (ou de passer à UC 13 — Tests, voir `plan-poc-bob-acme.md` section Carte des livrables), valider chaque point :

- [ ] **Pour chaque programme restructuré : le Prompt 0 a été exécuté** — la fiche de qualification `{appArcad}-{fonction}-{composant}-qualification-restr-{YYYYMMDD-HHmm}.md` existe et mentionne la catégorie (SIMPLE / STANDARD / COMPLEXE) et le périmètre retenu pour le POC
- [ ] Les programmes exclus du périmètre de restructuration sont documentés dans le plan (Prompt 5) avec la justification — aucun programme COMPLEXE n'a été traité sans décision explicite
- [ ] Le plan d'architecture (Prompt 1) de chaque programme restructuré est sauvegardé dans `{appArcad}-{fonction}-{composant}-plan-archi-{YYYYMMDD-HHmm}.md` et a été **validé par un développeur senior** avant la génération de code
- [ ] Chaque module extrait est sauvegardé dans `{appArcad}-{fonction}-{NOM_MODULE}-spec-module-{YYYYMMDD-HHmm}.md` avec son interface complète (DCL-PI, paramètres, direction)
- [ ] L'interface externe de chaque programme principal restructuré est **inchangée** sauf si la migration des appelants est documentée et planifiée
- [ ] Les subroutines BEGSR/ENDSR extraites ont été supprimées du programme principal (ou commentées pendant la phase de validation)
- [ ] Chaque programme de service (*SRVPGM) a été créé via `CRTSRVPGM` sur l'IBM i de test et les procédures exportées sont visibles dans `QSYS2.SYSROUTINES`
- [ ] Chaque module extrait et le programme principal modifié ont été **compilés sans erreur** sur l'IBM i de test (Prompt 3-bis)
- [ ] Chaque programme restructuré a été **testé fonctionnellement** sur l'IBM i de test avec un jeu de données réel — les résultats avant et après restructuration sont identiques
- [ ] Les programmes identifiés comme ayant un cycle RPG actif ont été traités avec le Prompt 4 (si applicable) — aucun *INLR ou niveau de contrôle Ln n'a été supprimé sans conversion explicite
- [ ] Le plan de restructuration (Prompt 5) est produit et sauvegardé dans `{appArcad}-{fonction}-{fonction}-plan-restr-{YYYYMMDD-HHmm}.md`
- [ ] Les diffs sont sauvegardés avec la convention de nommage `{appArcad}-{fonction}-{composant}-diff-restr-{YYYYMMDD-HHmm}.md` dans le workspace ET publiés sur Confluence (si MCP disponible)
- [ ] La mention `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` est présente dans l'en-tête de chaque module extrait et dans chaque diff de programme principal

---

## Points à compléter avant passage en production

> Ces points ne bloquent pas le POC — ils concernent des cas avancés peu probables sur les programmes pilotes. Ils deviennent critiques dès que la restructuration s'étend à des programmes de gestion complexes en production.

### Programmes interactifs avec écrans 5250 (EXFMT)

**Contexte :** les programmes interactifs IBM i (WDSPF, `EXFMT`) entrelacent la logique métier et la gestion des écrans dans les mêmes subroutines. UC 8 peut extraire la logique métier en module, mais la logique de navigation écran reste dans le programme principal. Si le programme est un candidat à la conversion 5250 → Web (UC 10), restructurer d'abord avec UC 8 (séparer métier et présentation) avant UC 10 (remplacer la présentation) est la bonne séquence. Dans le POC, les programmes pilotes de UC 8 sont des programmes batch sans écrans.

**Pourquoi absent de la fiche POC :** les pilotes UC 8 sont des programmes batch — aucun EXFMT dans le périmètre POC.

**À faire avant production :** définir la convention de partition Présentation / Métier avant de démarrer UC 8 sur un programme interactif. Ajouter dans le Prompt 1 une section dédiée à la séparation écran/métier. Coordonner avec UC 10 si la conversion 5250 → Web est dans la feuille de route.

> ⚠️ **Signal d'alerte sur le terrain :** si le Prompt 0 détecte `EXFMT` ou `WRITE vers WDSPF` dans les subroutines listées comme candidates à l'extraction — marquer ce programme comme "interactif" et reporter à l'analyse UC 10 avant de continuer UC 8.

### Gestion de la cohérence transactionnelle après extraction de modules

**Contexte :** si le programme original gère une transaction (COMMIT/ROLLBACK englobant plusieurs WRITE/UPDATE/DELETE), et qu'un des blocs transactionnels est extrait dans un module, la limite de transaction peut être scindée entre le programme principal et le module. Le module peut réussir son WRITE sans que le programme principal ait émis son COMMIT — laissant une transaction partielle ouverte.

**Pourquoi absent de la fiche POC :** les programmes pilotes ont été choisis parmi des programmes sans gestion transactionnelle explicite. Les COMMIT/ROLLBACK apparaissent principalement dans les programmes de gestion complexes traités en fin de parcours.

**À faire avant production :** ajouter dans le Prompt 0 (section "Facteurs de complexité") la détection explicite de `COMMIT`, `ROLLBACK` et `ROLBK`. Si détectés, ajouter un Prompt 4-ter dédié à la cartographie des limites de transaction et à leur préservation dans l'architecture cible (le COMMIT doit rester dans le programme principal appelant, jamais dans un module extrait).

> ⚠️ **Signal d'alerte sur le terrain :** si après restructuration, des enregistrements partiellement créés ou des mises à jour orphelines apparaissent dans les données de test — vérifier en priorité si une limite de transaction a été scindée lors de l'extraction d'un module.

---

*Fiche UC 8 — Document évolutif à mettre à jour au fil du POC.*
