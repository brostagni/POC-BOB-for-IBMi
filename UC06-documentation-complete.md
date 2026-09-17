# UC 6 — Documentation complète

> **Catégorie :** Documentation
>
> **Priorité dans le POC :** 5 — capitalise les livrables UC 4 et UC 5
>
> **Durée POC (avec Bob) :** 3 à 6 heures — génération des 5 livrables sur une application pilote, itérations et validations intermédiaires
>
> **Durée PROD (avec Bob) :** 4 à 8 heures / application — équipe rodée, 5 prompts enchaînés + revue développeur et expert métier
>
> **Durée PROD (sans Bob) :** 2 à 4 semaines / application — rédaction manuelle par un analyste senior : spécifications fonctionnelles, spécifications techniques, modèle de données, matrice de références croisées
>
> **Gain Bob estimé :** ~20× — dossier documentaire complet produit en 1 journée au lieu de 2 à 4 semaines ; livrable le plus impactant de la Phase 1 pour justifier le ROI du POC
>
> **Mode Bob recommandé :** IBM i Developer (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : Ask.

---

## Objectif

Produire une **documentation structurée, durable et actionnable** de l'application ACME à partir de tout ce qui a été compris en UC 4 et UC 5 : modèles d'architecture, spécifications fonctionnelles et techniques, diagrammes des relations entre programmes et données, matrices de références croisées.

**Ce UC est le point de convergence de la Phase 1.** UC 4 a fourni la compréhension programme par programme. UC 5 a extrait les règles métier. UC 6 les assemble en un corpus documentaire cohérent, navigable, et directement utilisable comme input pour les phases de modernisation.

**Sans UC 6, les phases suivantes partent à l'aveugle.** Un développeur qui attaque UC 7 (optimisation) ou UC 8 (restructuration) sans documentation de référence n'a aucun filet pour valider qu'il n'a pas cassé la logique.

**Livrable attendu :** Un dossier documentaire par application comprenant : un modèle d'architecture (diagramme de composants + flux de données), une spécification fonctionnelle, une spécification technique, un diagramme des relations données (modèle physique), et une matrice de références croisées programmes-fichiers-règles.

**Convention de nommage des fichiers générés :**
```
{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md
```

| Valeur de `{type}` | Livrable correspondant | Prompt |
|--------------------|------------------------|--------|
| `architecture` | Diagramme de composants + flux de traitement | Prompt 1 |
| `spec-fonc` | Spécification fonctionnelle | Prompt 2 |
| `spec-tech` | Spécification technique + modèle ER | Prompt 3 |
| `matrice` | Matrice de références croisées | Prompt 4 |
| `synthese` | Fiche applicative de synthèse | Prompt 5 |

Quand le livrable couvre l'application entière plutôt qu'un composant spécifique, utiliser `{fonction}` comme valeur de `{composant}` :
```
Exemples :
acme-APPVTE-APPVTE-architecture-20250616-0900.md   ← vue application complète
acme-APPVTE-GESCMD-spec-fonc-20250616-1045.md      ← spec d'un programme spécifique
acme-APPVTE-APPVTE-matrice-20250616-1130.md        ← matrice couvrant toute la lib
acme-APPVTE-GESCMD-spec-tech-20250616-1200.md
acme-APPVTE-APPVTE-synthese-20250616-1400.md
```

Ces fichiers sont les **inputs nommés** des phases de modernisation (UC 7 à UC 16) — un prompt de modernisation peut référencer un fichier par son nom exact pour que Bob charge le bon contexte.

---

## Démarrer par une application dont vous connaissez déjà les livrables UC 4 et UC 5

UC 6 ne produit rien de bon sur une application non préalablement analysée. La règle est ferme : **ne démarrer UC 6 que sur les applications pour lesquelles les fichiers de compréhension (type `comprehension`) et les catalogues de règles (type `regles`) sont complets et validés**.

Pourquoi ? Parce que UC 6 assemble des informations existantes — il ne découvre pas. Si les informations sources sont incomplètes ou non validées par un expert métier, la documentation produite sera incomplète et propagera les erreurs aux UC suivants.

**Progression recommandée :**

| Étape | Point de départ | Document à produire | Objectif |
|-------|----------------|---------------------|---------|
| 1 | Fichiers `*-comprehension-*.md` de l'application | Architecture générale (Prompt 1) → fichier `*-architecture-*.md` | Valider le diagramme avec l'équipe — s'assurer que Bob a bien cartographié tous les composants |
| 2 | Fichiers `*-regles-*.md` validés par l'expert métier | Spécification fonctionnelle (Prompt 2) → fichier `*-spec-fonc-*.md` | Vérifier la fidélité de la transcription des règles en specs — la validation métier est le filet de sécurité |
| 3 | Fichiers `*-comprehension-*.md` (F-specs + dépendances) | Modèle de données physique (Prompt 3) → fichier `*-spec-tech-*.md` | Confronter le modèle généré aux fichiers réels — identifier les écarts DDS vs données réelles |
| 4 | Fichiers `*-comprehension-*.md` (liste complète des programmes) | Matrice références croisées (Prompt 4) → fichier `*-matrice-*.md` | Valider la complétude : chaque programme listé dans les fichiers de compréhension doit apparaître dans la matrice |

---

## Impact de la taille du programme — Renvoi à UC 4

Les mêmes seuils de taille définis en UC 4 s'appliquent ici pour les programmes sources. Deux implications spécifiques à UC 6 :

**1. La documentation d'architecture est moins sensible à la taille individuelle des programmes que l'extraction de règles (type `regles`).**
UC 6 travaille à un niveau d'abstraction plus élevé — on documente les interfaces, les flux et les relations, pas la logique interne ligne à ligne. Un programme de 8 000 lignes peut être représenté en 3 lignes dans le diagramme d'architecture. Ce qui compte ici, c'est la **couverture** (tous les programmes de l'application sont-ils représentés ?) plutôt que la profondeur.

**2. La matrice de références croisées est d'autant plus longue que l'application est volumineuse.**  
Sur une application de 50+ programmes, la matrice (Prompt 4) doit être construite par domaine fonctionnel et non en une seule passe. Utiliser les domaines fonctionnels identifiés dans les fichiers `*-comprehension-*.md` (cartographie macro) pour segmenter le travail :

```
"Construis la matrice de références croisées pour le domaine fonctionnel 
 [NOM_DOMAINE] uniquement — programmes : [PROG1], [PROG2], [PROG3], [PROG4]."
 
[Après validation]
"Maintenant le domaine [NOM_DOMAINE_2] — programmes : [PROG5], [PROG6]."
```

> ⚠️ Sur une application de 100+ programmes, ne jamais lancer la matrice globale en une seule passe — Bob produit une matrice incomplète sans le signaler.

---

## Démarrer une session Bob

> **À lire avant chaque session UC 6 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

UC 6 travaille sur l'ensemble d'une application — il assemble les livrables UC 4 et UC 5 plutôt que d'analyser un programme à la fois. Démarrer dans une **nouvelle conversation Bob** (bouton `+`) dédiée à l'application à documenter.

Ne pas réutiliser une conversation UC 4 ou UC 5 d'un programme spécifique — le contexte d'un seul programme ne représente pas l'application entière.

**Mode à sélectionner :** `IBM i Developer`

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

UC 6 travaille principalement depuis les fichiers `.md` déjà produits. Ouvrir dans l'éditeur les livrables UC 4 et UC 5 avant de démarrer.

**Procédure :** dans l'explorateur de fichiers Bob (panneau Explorer), ouvrir chaque fichier de contexte → il apparaît dans l'éditeur et devient accessible via **Add File to Chat**.

Fichiers à ouvrir pour chaque session UC 6 :
- Tous les fichiers `*-comprehension-*.md` des programmes de l'application
- Tous les fichiers `*-regles-*.md` validés par les experts métier
- Les livrables UC 6 partiellement produits si reprise de session (`*-architecture-*.md`, `*-spec-fonc-*.md`…)

### 3. Fichiers de contexte à charger

Ces fichiers sont les inputs directs de chaque prompt UC 6. Les charger dans le chat via **Add File to Chat** avant de lancer le prompt correspondant.

| Fichier | Produit par | Utilisé dans |
|---------|-------------|-------------|
| `{appArcad}-{fonction}-{composant}-comprehension-{date}.md` | UC 4 | **Obligatoire** — Prompts 1, 3, 4 (architecture, spec-tech, matrice) |
| `{appArcad}-{fonction}-{composant}-regles-{date}.md` | UC 5 | **Obligatoire** — Prompt 2 (spec-fonc) |
| `{appArcad}-{fonction}-{fonction}-architecture-{date}.md` | UC 6 Prompt 1 | **Si reprise** — disponible pour les Prompts 2 à 5 |
| `{appArcad}-{fonction}-{fonction}-spec-fonc-{date}.md` | UC 6 Prompt 2 | **Si reprise** — pour compléter ou affiner |
| `{appArcad}-{fonction}-{fonction}-spec-tech-{date}.md` | UC 6 Prompt 3 | Input obligatoire de **UC 14** (DDS→DDL) et **UC 3** (SQL embarqué) |
| `{appArcad}-{fonction}-{fonction}-matrice-{date}.md` | UC 6 Prompt 4 | Input obligatoire de **UC 7** (Optimisation) et **UC 8** (Restructuration) |

> ⚠️ **Prérequis bloquant :** ne pas démarrer UC 6 sans au moins un fichier `*-comprehension-*.md` validé et un fichier `*-regles-*.md` validé par un expert métier pour l'application ciblée.

> ⚠️ **Risque de réduction de contexte — sauvegarde intermédiaire recommandée :** une session UC 6 avec plusieurs prompts consécutifs peut atteindre la limite de contexte sur les applications STANDARD ou volumineuses. Si Bob semble oublier les décisions d'architecture prises au Prompt 1 (composants retenus, périmètre délimité, nœuds exclus), c'est un signal de compression de contexte. Sauvegarder le livrable en cours en mode Agent après chaque prompt majeur — pas seulement en fin de session. À chaque reprise de passe, commencer le prompt par : "Le fichier [NOM_FICHIER] contient les décisions prises — continuer à partir de [ÉTAPE]."

> 💡 **Reprise de session :** UC 6 peut se faire en plusieurs sessions (un prompt par session si l'application est volumineuse). Ouvrir les livrables partiels déjà produits au début de chaque session — Bob reprend là où la session précédente s'est arrêtée.

> 💡 **Lien UC 6 → Phase 2 :** les fichiers `*-spec-tech-*.md` et `*-matrice-*.md` produits ici sont les inputs nommés des sections "Démarrer une session Bob" de UC 14 (`UC14-dds-ddl.md`) et UC 3 (`UC03-sql-embarque.md`). Vérifier leur présence dans le workspace avant de démarrer Phase 2.

---

## Prérequis

- Les fichiers `*-comprehension-*.md` de tous les programmes de l'application à documenter sont présents dans le workspace
- Les fichiers `*-regles-*.md` sont complets et marqués comme validés par un expert métier
- IBM i MCP actif (lecture des sources pour les détails techniques)
- IBM i Database MCP actif (interrogation QSYS2 pour les métadonnées de fichiers)
- Les fichiers de compréhension et de règles sont chargés dans le workspace ou référencés en contexte dans la session Bob

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | **IBM i Developer** (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : **Ask**. |
| **Scope** | Library List → bibliothèque applicative ACME |
| **MCP actifs** | IBM i MCP + IBM i Database MCP |
| **MCP de publication** | Confluence MCP (publication de l'espace documentaire, si token disponible) |

### Pourquoi le mode IBM i Developer pour la documentation complète ?

Le mode **IBM i Developer** apporte la connaissance RPG/CL/DDS spécialisée pour toute la session — vocabulaire des opcodes, connaissance des structures ILE, vues QSYS2, comportements des MCP IBM i. Sans ce mode, ces notions doivent être réexpliquées dans chaque prompt de génération documentaire.

UC 6 est un UC **lecture seule** pendant toute la phase de génération : Bob lit les sources et métadonnées via MCP, puis produit les livrables dans le chat. La discipline de validation repose sur la **relecture humaine avant sauvegarde** : Bob génère chaque livrable dans le chat de façon itérative, l'équipe valide et corrige dans le chat, puis autorise explicitement l'écriture en fin de session.

> 💡 **Sans Premium Package IBM i :** utiliser le mode **Ask** pour toute la session — la discipline de validation reste identique.

| Phase | Comportement attendu | Ce que Bob fait |
|-------|----------------------|----------------|
| Génération des livrables (Prompts 1 à 5) | Génère dans le chat — pas d'écriture | Lit les sources et métadonnées via MCP, génère les documents dans le chat — itérations possibles |
| Sauvegarde des fichiers `.md` | **Écriture autorisée** — après validation de chaque livrable | Écrit les fichiers de documentation dans le workspace |
| Publication Confluence | **Écriture MCP autorisée** — après validation du livrable | Publie via Confluence MCP si token disponible |

> 💡 **Règle d'or pour UC 6 :** IBM i Developer pour toute la session. Bob génère chaque livrable dans le chat de façon itérative — l'équipe valide et corrige sans risque d'écriture intermédiaire, puis autorise la sauvegarde une fois le livrable finalisé. Aucune écriture IBM i dans cet UC.

> ⚠️ Ne jamais autoriser d'écriture pendant la génération des diagrammes Mermaid ou des matrices — Bob pourrait écrire des fichiers intermédiaires non validés dans le workspace.

> 💡 Si le Confluence MCP est actif, publier chaque livrable directement dans l'espace POC à l'issue de sa production. La documentation publiée au fil de l'eau est plus utile qu'un export final en fin de phase.

### Intégration ARCAD

Le MCP ARCAD n'était pas disponible dans le contexte de ce POC de référence (version ARCAD non compatible avec le MCP). Si le MCP ARCAD est disponible dans votre environnement, les étapes manuelles de réintégration décrites ci-dessous peuvent être automatisées. N'hésitez pas à demander à Bob de modifier cette fiche UC en intégrant la disponibilité du MCP ARCAD.

**Impact sur les UC 4, 5 et 6 : limité.** ARCAD gère les versions et les déploiements — pas la compréhension du code. Les sources sont accessibles via IBM i MCP indépendamment d'ARCAD.

| Sans MCP ARCAD (contexte de ce POC) | Avec MCP ARCAD disponible |
|--------------------------------------|---------------------------|
| Lire manuellement l'historique des versions ARCAD | Le MCP ARCAD peut exposer l'historique directement dans le contexte Bob |
| Exporter manuellement l'historique ARCAD (CSV/texte) et le charger dans Bob | L'historique est intégrable automatiquement dans la spec technique (Prompt 3) |
| Ajouter le placeholder `⚠️ Historique ARCAD — à compléter manuellement depuis l'interface ARCAD` | Le placeholder n'est plus nécessaire |

> 💡 **Dans les deux cas :** IBM i MCP lit les sources courants dans les bibliothèques ARCAD normalement. L'analyse du code et la génération de documentation sont intégralement fonctionnelles.

> 💡 **Pour UC 16 (DevOps/CI-CD avec ARCAD) :** UC 16 tire pleinement parti du MCP ARCAD — Bob peut piloter les pipelines directement si le MCP est disponible. Voir la note UC 16 dans le plan.

---

## Prompts clés

### Prompt 1 — Architecture générale de l'application

```
Sur la base des fichiers de compréhension ({appArcad}-{fonction}-*-comprehension-*.md)
pour l'application [NOM_APPLICATION] dans [NOM_LIB],
génère la documentation d'architecture en français, en markdown.

[Si nouvelle session : charger les fichiers *-comprehension-*.md dans le contexte avant ce prompt]

Produis les sections suivantes :

## 1. Vue d'ensemble de l'application
- Rôle métier de l'application en 3 phrases (pour un non-technicien)
- Périmètre fonctionnel : quels processus métier cette application couvre-t-elle ?
- Volumétrie : nombre de programmes, fichiers physiques, fichiers logiques, écrans

## 2. Diagramme de composants (Mermaid)
Génère un diagramme Mermaid de type `graph TD` représentant :
- Les programmes principaux comme nœuds (PGM)
- Les fichiers physiques partagés comme nœuds (PF)
- Les interfaces avec d'autres applications ou systèmes comme nœuds (EXT)
- Les appels CALL entre programmes comme flèches →
- Les accès en lecture aux PF comme flèches en pointillés -.->
- Les accès en écriture/mise à jour comme flèches =>
Limiter à 15 nœuds maximum — si l'application est plus grande, 
représenter uniquement les composants centraux (les plus appelés ou les plus partagés).

## 3. Flux de traitement principal
Décris en 5 à 8 étapes le flux de traitement de bout en bout :
depuis l'entrée utilisateur (écran 5250 ou déclencheur batch) 
jusqu'à la production de la sortie (mise à jour base, impression, appel externe).

## 4. Points d'intégration
Liste les interfaces avec l'extérieur :
- Autres applications IBM i appelées ou appelantes
- Fichiers d'interface (entrée / sortie) avec des systèmes tiers
- Déclencheurs batch (JOBSCD, CL de soumission)

Ne pas inventer de composants non visibles dans les fichiers de compréhension.
Signaler explicitement les zones où le périmètre est incomplet ou incertain.
```

> 💡 **Sauvegarder ce livrable** (mode Agent) :
> ```
> "Sauvegarde ce document d'architecture dans un fichier nommé
>  {appArcad}-{fonction}-{fonction}-architecture-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-APPVTE-architecture-20250616-0900.md"
> ```

**Analyse ligne à ligne :**

- `Sur la base des fichiers de compréhension (*-comprehension-*.md)` → ancre explicite sur les fichiers nommés produits lors de l'analyse. Sans cette ancre, Bob génère une architecture plausible mais non fondée sur l'analyse réelle du code ACME.

- `[Si nouvelle session : charger les fichiers *-comprehension-*.md]` → même garde-fou que dans l'étape règles métier. En pratique, la documentation se déroule souvent dans une nouvelle session — la note de rappel dans le prompt lui-même évite l'oubli.

- `Mermaid de type graph TD` → type de diagramme précisé. Bob connaît plusieurs variantes Mermaid (`flowchart`, `graph`, `C4Context`) — spécifier `graph TD` produit un rendu cohérent avec les autres diagrammes générés en UC 4 Prompt 2.

- `Limiter à 15 nœuds maximum` → contrainte de lisibilité. Un diagramme de 50 nœuds est illisible dans Confluence et dans le chat Bob. La limitation force Bob à hiérarchiser les composants importants — ce qui est plus utile qu'un graphe exhaustif illisible.

- `les plus appelés ou les plus partagés` → critère de priorisation explicite. Sur IBM i, les programmes centraux se reconnaissent au nombre de CALL entrants et au nombre de fichiers physiques partagés — deux métriques que Bob peut évaluer depuis les fichiers de compréhension.

- `Points d'intégration` → section critique souvent oubliée. Sur IBM i, les intégrations passent souvent par des fichiers d'interface en batch plutôt que par des APIs — les identifier ici évite de les manquer lors de la modernisation.

- `Ne pas inventer de composants non visibles dans les fichiers de compréhension` → garde-fou anti-hallucination de niveau architecture. Une relation inventée dans un diagramme d'architecture propage une fausse information dans toute la documentation suivante.

> ⚠️ **Piège évité :** sans la contrainte sur le nombre de nœuds, Bob génère des diagrammes Mermaid qui dépassent la capacité du rendu et produisent une erreur dans le preview. Toujours limiter à 15 nœuds, quitte à faire un second diagramme pour les composants secondaires.

---

### Prompt 2 — Spécification fonctionnelle

```
Sur la base des fichiers de compréhension ({appArcad}-{fonction}-*-comprehension-*.md)
et du catalogue de règles validé ({appArcad}-{fonction}-*-regles-*.md) pour
[NOM_APPLICATION] / [NOM_DOMAINE_FONCTIONNEL], rédige une spécification
fonctionnelle en français, en markdown.

[Si nouvelle session : charger les fichiers *-comprehension-*.md ET *-regles-*.md
 dans le contexte avant ce prompt — les deux sources sont nécessaires]

Structure attendue :

## Spécification fonctionnelle — [NOM_APPLICATION] / [NOM_DOMAINE]

### 1. Contexte et périmètre
- Processus métier couvert
- Acteurs : qui déclenche le traitement ? qui en consomme le résultat ?
- Volume et fréquence : traitement unitaire ou batch ? quotidien / hebdomadaire / à la demande ?

### 2. Cas d'utilisation
Pour chaque cas d'utilisation identifié :
- Nom et résumé en 2 phrases
- Préconditions : qu'est-ce qui doit être vrai pour que le traitement se déclenche ?
- Flux nominal : les étapes du traitement dans le cas normal
- Flux alternatifs : les cas particuliers identifiés dans le code (conditions, exceptions)
- Post-conditions : quel est l'état du système à la fin du traitement ?

### 3. Règles de gestion
Reporter ici les règles du catalogue de règles (*-regles-*.md), organisées par cas d'utilisation.
Pour chaque règle :
- Identifiant et intitulé (reprendre la numérotation du catalogue *-regles-*.md)
- Description en langage naturel
- Cas d'utilisation auquel elle s'applique

### 4. Messages et comportements utilisateur
Liste des messages affichés à l'écran (codes message IBM i, textes MSGF),
avec leur déclencheur et leur sévérité (information / avertissement / erreur bloquante).

### 5. Interfaces et données échangées
- Données en entrée : champs, formats, sources
- Données en sortie : champs, formats, destinations
- Règles de correspondance entrée → sortie

Ne pas reproduire le code RPG dans ce document.
Signaler en note les points qui n'ont pas pu être déterminés depuis les sources disponibles.
```

> 💡 **Sauvegarder ce livrable** (mode Agent) :
> ```
> "Sauvegarde cette spécification fonctionnelle dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-spec-fonc-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-spec-fonc-20250616-1045.md"
> ```
> Si la spec couvre l'application entière : `acme-APPVTE-APPVTE-spec-fonc-20250616-1045.md`

**Analyse ligne à ligne :**

- `fichiers *-comprehension-*.md ET *-regles-*.md` → **double source obligatoire** pour ce prompt, contrairement aux Prompts 1, 3 et 4 qui n'utilisent que les fichiers de compréhension. Les cas d'utilisation viennent de la compréhension du code (structure, flux) ; les règles de gestion viennent du catalogue de règles validé. Les deux sont indispensables — un seul des deux produit une spec incomplète.

- `catalogue de règles validé (*-regles-*.md)` → le qualificatif "validé" est délibéré. Une spécification fonctionnelle construite sur des règles non validées par l'expert métier est une spécification fausse. La check-list de l'étape règles métier doit être complète avant ce prompt.

- `Acteurs : qui déclenche le traitement ?` → dimension souvent absente des analyses de code IBM i. Le code ne dit pas qui appuie sur Entrée — l'information vient du contexte métier et des écrans 5250. Bob peut la déduire des display files et des paramètres de soumission.

- `Flux nominal / Flux alternatifs` → structure de cas d'utilisation standard. Les flux alternatifs sur IBM i correspondent souvent aux conditions `*IN50 = *ON` ou aux branches `WHEN` des `SELECT` — ce que Bob a déjà cartographié dans les fichiers `*-regles-*.md` (arbres de décision, Prompt 4 de l'extraction de règles).

- `Identifiant et intitulé (reprendre la numérotation du catalogue *-regles-*.md)` → traçabilité directe entre la spec fonctionnelle et le catalogue de règles. Un auditeur ou un développeur peut remonter de la spec à la règle, puis de la règle à la ligne de code.

- `Messages et comportements utilisateur` → section spécifique IBM i. Les codes message IBM i (`CPF`, `MCH`, `USR`) et les fichiers de messages (`MSGF`) sont la documentation de l'expérience utilisateur — souvent absents des analyses de code classiques.

- `Ne pas reproduire le code RPG dans ce document` → la spec fonctionnelle est destinée aux analystes et au management, pas aux développeurs. Le code a sa place dans la spec technique (Prompt 3), pas ici.

> 💡 **Ce prompt est le plus long à générer mais aussi le plus utile.** La spécification fonctionnelle est le document le plus demandé par les équipes projet lors des phases de modernisation et de recette. La produire via Bob à partir des fichiers de compréhension et du catalogue de règles est un gain de temps considérable.

---

### Prompt 3 — Spécification technique et modèle de données physique

```
Sur la base des fichiers de compréhension ({appArcad}-{fonction}-*-comprehension-*.md)
pour [NOM_APPLICATION] dans [NOM_LIB], génère une spécification technique
en français, en markdown.

[Si nouvelle session : charger les fichiers *-comprehension-*.md dans le contexte avant ce prompt]

## Spécification technique — [NOM_APPLICATION]

### 1. Inventaire des objets
Produis un tableau :
| Nom objet | Type (PGM / PF / LF / DSPF / PRTF / CL) | Bibliothèque | Rôle | Programmes utilisant cet objet |

### 2. Modèle de données physique
Pour chaque fichier physique (PF) de l'application :
- Nom du fichier et bibliothèque
- Rôle fonctionnel (ce que représente chaque enregistrement)
- Champs principaux : nom, type, longueur, rôle supposé
- Clé primaire (champ UNIQUE ou premier champ de la clé)
- Fichiers logiques associés (LF) et critère de sélection/tri de chaque LF

Génère ensuite un diagramme Mermaid `erDiagram` représentant les relations 
entre les fichiers physiques de l'application (clés étrangères déduites 
des champs communs entre fichiers).

### 3. Interfaces de programmes (signatures)
Pour chaque programme de l'application appelé depuis l'extérieur (CALL entrant) :
- Nom du programme
- Liste des paramètres avec type, longueur et sens (entrée / sortie / entrée-sortie)
- Valeurs de retour ou codes de statut

### 4. Dépendances techniques
- Copybooks partagés (/COPY) et leur contenu (structures, constantes, prototypes)
- Data areas (*DTAARA) utilisées
- Data queues (*DTAQ) utilisées
- User spaces (*USRSPC) utilisés

### 5. Contraintes techniques identifiées
- Indicateurs partagés entre programmes
- Variables globales ou structures de données partagées
- Appels dynamiques (CALL avec variable de nom — non résolvables statiquement)
- Programmes sans source disponible (objets compilés uniquement)

Signaler explicitement les informations non déterminables depuis les sources disponibles.
```

> 💡 **Sauvegarder ce livrable** (mode Agent) :
> ```
> "Sauvegarde cette spécification technique dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-spec-tech-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-GESCMD-spec-tech-20250616-1200.md"
> ```
> Si la spec couvre l'application entière : `acme-APPVTE-APPVTE-spec-tech-20250616-1200.md`

**Analyse ligne à ligne :**

- `Inventaire des objets` → tableau exhaustif indispensable avant toute modernisation. Sur IBM i, les objets ne sont pas dans un système de fichiers visible — il n'y a pas de `ls` pour voir ce qui existe. Ce tableau est la carte du territoire.

- `Type (PGM / PF / LF / DSPF / PRTF / CL)` → les types d'objets IBM i listés explicitement. Sans cette liste, Bob peut utiliser une nomenclature générique qui perd la distinction entre fichiers physiques et logiques — distinction critique pour UC 14 (DDS → DDL).

- `Fichiers logiques associés (LF) et critère de sélection/tri` → les LF sont la couche d'indexation d'IBM i — l'équivalent des index SQL. Les documenter ici prépare directement UC 14 (conversion DDS → DDL) où chaque LF doit être transformé en index ou vue SQL.

- `erDiagram` → format Mermaid pour les diagrammes entité-relation. Plus lisible qu'un `graph TD` pour les données, directement reconnu par Confluence et les outils de documentation.

- `clés étrangères déduites des champs communs entre fichiers` → sur IBM i DDS, les clés étrangères ne sont pas déclarées — elles sont implicites et visibles dans les noms de champs communs entre fichiers. Bob peut les déduire, mais il doit être explicitement invité à le faire.

- `Dépendances techniques — Data areas, Data queues, User spaces` → artefacts IBM i souvent invisibles dans une analyse de code standard mais qui jouent un rôle d'état global. Un data area partagé entre programmes est l'équivalent d'une variable globale d'application — critique à documenter.

- `Appels dynamiques (CALL avec variable)` → signaler les appels non résolvables statiquement est plus utile que de les ignorer. Ces appels sont des risques résiduels dans la documentation — les nommer explicitement permet à l'équipe de les investiguer manuellement.

> ⚠️ **Piège évité :** ne pas confondre la spécification technique (ce prompt) et la spécification fonctionnelle (Prompt 2). La spec technique est destinée aux développeurs et à l'équipe de modernisation. Elle peut contenir des noms de champs, des types SQL, des structures DDS — tout ce qu'on exclut de la spec fonctionnelle.

---

### Prompt 4 — Matrice de références croisées

```
Sur la base des fichiers de compréhension ({appArcad}-{fonction}-*-comprehension-*.md)
et du catalogue de règles ({appArcad}-{fonction}-*-regles-*.md) pour l'application
[NOM_APPLICATION] dans [NOM_LIB], génère une matrice de références croisées
en markdown.

[Si nouvelle session : charger les fichiers *-comprehension-*.md ET *-regles-*.md
 dans le contexte avant ce prompt]

## Matrice de références croisées — [NOM_APPLICATION]

### 1. Matrice Programmes × Fichiers
Produis un tableau markdown :
- Lignes : tous les programmes de l'application (un programme par ligne)
- Colonnes : tous les fichiers physiques de l'application (un PF par colonne)
- Cellule : L (lecture seule) / E (écriture seule) / LE (lecture+écriture) / - (pas d'accès)

### 2. Matrice Programmes × Règles métier
Produis un tableau :
- Lignes : tous les programmes de l'application
- Colonnes : les règles métier numérotées du catalogue de règles (*-regles-*.md)
- Cellule : ✓ (cette règle est implémentée dans ce programme) / - (pas concerné)

### 3. Matrice Programmes × Programmes (appels)
Produis un tableau :
- Lignes : programmes appelants
- Colonnes : programmes appelés
- Cellule : CALL / CALLP / EXSR / - (pas d'appel direct)

### 4. Index des objets partagés
Liste les fichiers physiques utilisés par plus d'un programme, 
avec pour chaque PF partagé :
- Nombre de programmes en lecture
- Nombre de programmes en écriture
- Programmes en écriture (risque de contention)
- Niveau de criticité : Élevé (écrit par 3+ programmes) / Moyen / Faible

Pour les tableaux avec plus de 8 colonnes, utiliser des abréviations pour les noms
de colonnes et fournir un index d'abréviations en dessous du tableau.
```

> 💡 **Sauvegarder ce livrable** (mode Agent) :
> ```
> "Sauvegarde cette matrice de références croisées dans un fichier nommé
>  {appArcad}-{fonction}-{fonction}-matrice-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-APPVTE-matrice-20250616-1130.md"
> ```

**Analyse ligne à ligne :**

- `L / E / LE / -` → nomenclature simple et lisible dans un tableau markdown. Plus expressif que `R/W/RW` pour un document destiné à des non-anglophones, cohérent avec la langue française du document.

- `Matrice Programmes × Règles métier` → lien entre le code et les specs fonctionnelles. Permet de répondre à la question "si je modifie le programme GESCMD, quelles règles métier sont impactées ?" — question centrale lors de toute modernisation.

- `Matrice Programmes × Programmes` → différencie CALL (sous-programme OPM), CALLP (procédure ILE) et EXSR (sous-programme interne appelé depuis l'extérieur, si exporté). Sur IBM i, la distinction est importante pour comprendre le type de couplage.

- `Index des objets partagés` → la section la plus utile pour les phases de modernisation. Un fichier physique écrit par 3+ programmes est un point de risque majeur — le modifier ou l'encapsuler dans un service SQL peut casser plusieurs programmes simultanément.

- `Pour les tableaux avec plus de 8 colonnes, utiliser des abréviations` → contrainte de lisibilité markdown. Les tables markdown avec 15+ colonnes sont illisibles dans tout rendu (Confluence, GitHub, VS Code). L'instruction prévient le problème avant qu'il ne se pose.

> 💡 **Mise à jour incrémentale :** au fil des UC suivants (UC 7, UC 8, UC 14…), des programmes ou des fichiers seront modifiés. Revenir à cette matrice et la mettre à jour est plus rapide que de la régénérer depuis zéro — Bob peut mettre à jour une ligne ou une colonne sur instruction.

> 💡 **Enchaînement avec UC 8 (Restructuration) :** la colonne "Niveau de criticité" de l'Index des objets partagés identifie les fichiers à encapsuler en priorité dans des procédures de service lors de la restructuration.

---

### Prompt 5 — Fiche de synthèse applicative (livrable final UC 6)

```
Sur la base de toute la documentation générée pour [NOM_APPLICATION]
(fichiers *-architecture-*.md, *-spec-fonc-*.md, *-spec-tech-*.md, *-matrice-*.md),
génère une fiche de synthèse applicative en français, en markdown.

[Si nouvelle session : charger tous les fichiers de documentation de l'application
 dans le contexte avant ce prompt]

Cette fiche est destinée à être :
- Publiée en page d'accueil de l'espace Confluence du POC
- Distribuée à tout nouveau membre de l'équipe arrivant sur ce périmètre
- Utilisée comme référence lors des réunions de pilotage

Structure :

## Fiche applicative — [NOM_APPLICATION]
**Date :** [date du jour]  
**Statut :** Documentée dans le cadre du POC Bob × ACME  
**Auteur :** Généré par Bob — à valider par l'équipe ACME  

### Résumé exécutif (5 lignes max)
Ce que fait l'application, son importance dans le SI, son état technique, 
et la recommandation de modernisation principale issue de l'analyse.

### Indicateurs clés
| Indicateur | Valeur |
|------------|--------|
| Nombre de programmes | ? |
| Nombre de fichiers physiques | ? |
| Nombre de règles métier documentées | ? |
| Langage principal | ? |
| Complexité globale | Faible / Moyenne / Élevée |
| Priorité de modernisation | Haute / Normale / Basse |

### Points de vigilance
Les 3 à 5 points qui nécessitent une attention particulière 
lors des phases de modernisation (UC 7 à UC 16).

### Liens vers les livrables détaillés
- [ ] Architecture : {appArcad}-{fonction}-{fonction}-architecture-{YYYYMMDD-HHmm}.md
- [ ] Spécification fonctionnelle : {appArcad}-{fonction}-{composant}-spec-fonc-{YYYYMMDD-HHmm}.md
- [ ] Spécification technique : {appArcad}-{fonction}-{composant}-spec-tech-{YYYYMMDD-HHmm}.md
- [ ] Matrice de références croisées : {appArcad}-{fonction}-{fonction}-matrice-{YYYYMMDD-HHmm}.md
- [ ] Catalogue de règles métier : {appArcad}-{fonction}-{composant}-regles-{YYYYMMDD-HHmm}.md
- [ ] Fiches de compréhension : {appArcad}-{fonction}-{composant}-comprehension-{YYYYMMDD-HHmm}.md
```

> 💡 **Sauvegarder ce livrable** (mode Agent) :
> ```
> "Sauvegarde cette fiche de synthèse dans un fichier nommé
>  {appArcad}-{fonction}-{fonction}-synthese-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-APPVTE-synthese-20250616-1400.md"
> ```

**Analyse ligne à ligne :**

- `publiée en page d'accueil de l'espace Confluence` → audience et destination déclarées dans le prompt. Bob calibre la forme et le vocabulaire en conséquence.

- `Date / Statut / Auteur` → en-tête de document formalisé. Le mention `Généré par Bob — à valider par l'équipe ACME` est volontaire : toute documentation générée par IA doit être clairement identifiée comme telle jusqu'à sa validation humaine.

- `Résumé exécutif (5 lignes max)` → contrainte ferme pour forcer la synthèse. Ce résumé est la seule partie lue par le management lors des réunions de pilotage — 5 lignes lisibles valent mieux qu'une page ignorée.

- `Indicateurs clés` → tableau structuré avec des valeurs objectives. Bob remplit ce tableau depuis les livrables des prompts précédents — si une valeur n'est pas déterminable, il laisse `?` plutôt que d'inventer.

- `Priorité de modernisation : Haute / Normale / Basse` → évaluation synthétique que Bob peut produire à partir des indicateurs de complexité, du nombre de règles métier, et des niveaux de criticité identifiés dans la matrice.

- `Liens vers les livrables détaillés` → les noms de fichiers normalisés remplacent les placeholders génériques. L'équipe peut les retrouver directement dans le workspace par leur nom, sans avoir à chercher.

> 💡 **Ce prompt est le dernier de UC 6 et le livrable de présentation de la Phase 1.** La fiche applicative est ce qu'on montre lors du point de mi-parcours du POC avec le management de ACME.

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Lecture des sources et des métadonnées d'objets (DDS, QSYS2) pour les spécifications techniques |
| **IBM i Languages** | Coloration syntaxique lors des allers-retours sur le code source pour vérifier les informations documentées |
| **Mermaid Preview** | Rendu visuel des diagrammes de composants (Prompt 1) et des diagrammes ER (Prompt 3) |
| **Markdown All in One** | Prévisualisation en temps réel des documents générés — indispensable pour vérifier les tableaux et la structure |

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture des sources pour compléter ou vérifier les informations issues des fichiers de compréhension |
| **IBM i Database MCP** | Interrogation `QSYS2.SYSCOLUMNS`, `QSYS2.SYSINDEXES`, `QSYS2.SYSPROGRAMSTAT` pour enrichir les specs techniques avec des métadonnées en temps réel |
| **Confluence MCP** *(si disponible)* | Publication directe de chaque livrable (architecture, specs, matrice, fiche de synthèse) sur l'espace POC |

> 💡 **Requêtes QSYS2 utiles pour le Prompt 3 :**
> ```sql
> -- Colonnes d'un fichier physique (équivalent DESC TABLE)
> SELECT COLUMN_NAME, DATA_TYPE, LENGTH, NUMERIC_SCALE, IS_NULLABLE, COLUMN_TEXT
> FROM QSYS2.SYSCOLUMNS
> WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_PF]'
> ORDER BY ORDINAL_POSITION;
> 
> -- Fichiers logiques associés à un PF
> SELECT TABLE_NAME, TABLE_SCHEMA, TABLE_TYPE, SYSTEM_TABLE_NAME
> FROM QSYS2.SYSTABLES
> WHERE BASE_TABLE_NAME = '[NOM_PF]' AND TABLE_SCHEMA = '[NOM_LIB]'
>   AND TABLE_TYPE = 'L';
> ```
> Ces deux requêtes permettent de vérifier en temps réel les informations de structure déduites depuis le code DDS.

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Démarrer UC 6 sans avoir les fichiers `*-comprehension-*.md` et `*-regles-*.md` | La documentation est générée sans base factuelle — elle reflète ce que Bob suppose, pas ce qui existe | Vérifier que tous les fichiers de compréhension et de règles sont présents et validés dans le workspace avant de démarrer |
| Utiliser des fichiers `*-regles-*.md` non validés dans la spec fonctionnelle | Des règles erronées sont formalisées comme spécifications — elles alimenteront les UC de modernisation avec de mauvaises entrées | Ne sourcer la spec fonctionnelle que depuis le catalogue de règles marqué "validé par [nom de l'expert]" dans son en-tête |
| Produire un diagramme Mermaid de 30+ nœuds | Le diagramme génère une erreur de rendu ou est illisible | Limiter à 15 nœuds (Prompt 1) — produire plusieurs diagrammes par couche si nécessaire |
| Confondre fichier logique (LF) et fichier physique (PF) dans le modèle ER | La matrice et le diagramme ER représentent des index comme des entités — le modèle de données est faux | Séparer explicitement PF et LF dans les prompts — les LF vont dans la section "Fichiers logiques associés", pas dans le diagramme ER |
| Ne pas mettre à jour la matrice après les UC suivants | La matrice devient obsolète dès la première modernisation — elle induit en erreur les développeurs | Définir dès UC 6 un "propriétaire" de la matrice qui est responsable de la tenir à jour au fil du POC |
| Publier la documentation sans mention "à valider" | Le management la considère comme définitive et l'utilise pour des décisions avant la validation de l'équipe | Toujours inclure le bandeau `Statut : à valider par l'équipe ACME` jusqu'à la revue formelle |
| Générer tous les livrables en une seule session sans revue intermédiaire | Les erreurs du Prompt 1 se propagent dans tous les prompts suivants — la documentation entière est cohérente mais fausse | Valider chaque livrable avec un développeur de l'équipe avant de passer au prompt suivant |

---

## Check-list de validation UC 6

Avant de passer aux UC de la Phase 2 — UC 14 (voir `UC14-dds-ddl.md`) puis UC 3 (voir `UC03-sql-embarque.md`) — valider chaque point :

- [ ] Le diagramme d'architecture (Prompt 1) a été revu et validé par un développeur qui connaît l'application — tous les composants importants sont présents, aucun nœud fictif
- [ ] La spécification fonctionnelle (Prompt 2) a été relue par l'expert métier ayant validé les fichiers `*-regles-*.md` — les règles sont correctement référencées et organisées par cas d'utilisation
- [ ] La spécification technique (Prompt 3) liste tous les fichiers physiques et logiques de l'application — vérifier contre `QSYS2.SYSTABLES` via le MCP Database
- [ ] Le modèle ER (Prompt 3) représente correctement les relations entre PF — les clés étrangères implicites ont été confirmées par un développeur
- [ ] La matrice de références croisées (Prompt 4) couvre tous les programmes listés dans les fichiers `*-comprehension-*.md` — aucun programme orphelin
- [ ] L'Index des objets partagés (Prompt 4) identifie les PF à risque (écrits par plusieurs programmes) — ces PF sont notés comme points de vigilance pour UC 7/8/14
- [ ] La fiche de synthèse (Prompt 5) a été produite et relue par le chef de projet POC
- [ ] Tous les livrables sont sauvegardés dans le workspace avec la convention de nommage `{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md` ET publiés sur Confluence (si MCP disponible)
- [ ] La mention `à valider par l'équipe ACME` est présente sur les documents non encore revus formellement

---

*Fiche UC 6 — Document évolutif à mettre à jour au fil du POC.*
