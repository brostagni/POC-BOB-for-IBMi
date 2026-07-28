# Apport du MCP ARCAD sur les use cases 1 à 14

> **Contexte :** cette fiche complète le POC anonymisé décrit dans [`plan-poc-bob-acme.md`](plan-poc-bob-acme.md). Le POC de référence a été mené **sans** MCP ARCAD. L'objectif ici est d'estimer, pour un futur POC, **ce que changerait réellement la disponibilité du MCP ARCAD** sur les UC 1 à 14.
>
> **Sources d'analyse :** documentation ARCAD disponible au moment de l'étude, éléments de démonstration et de transcripts associés, dépôt public [`ArcadeAI/arcade-mcp`](https://github.com/ArcadeAI/arcade-mcp) et documentation publique [`docs.arcade.dev`](https://docs.arcade.dev/en/home).
>
> **Positionnement important :** le dépôt `arcade-mcp` documente surtout le **framework MCP et la plateforme Arcade**, pas nécessairement le catalogue exact des outils IBM i exposés par le MCP ARCAD dans un environnement client donné. En conséquence, cette fiche distingue :
> - les apports **confirmés** par la documentation ARCAD ;
> - les mécanismes **techniquement réalistes** prouvés par la documentation Arcade ;
> - les points à **valider en atelier** dans un futur POC.
>
> **Usage recommandé :** cette fiche sert de document de cadrage pour challenger un futur POC avec MCP ARCAD disponible, et de base de reprise pour adapter ensuite les fiches UC existantes.

---

## 1. Thèse de synthèse

Le MCP ARCAD n'apporte pas d'abord une meilleure génération de code que Bob seul.

Son apport principal est ailleurs :
- **contexte applicatif déterministe** issu du référentiel ARCAD ;
- **réduction des hallucinations** et des approximations sur les dépendances ;
- **versioning, promotions, statuts et traçabilité** accessibles dans le flux de travail Bob ;
- **réintégration gouvernée** des changements au lieu d'un simple copier-coller hors outil ;
- **observabilité** des actions, builds, imports, déploiements et audits ;
- **contrôle d'accès** à deux niveaux : accès au serveur MCP et accès aux opérations sensibles.

En pratique, le MCP ARCAD transforme Bob d'un excellent assistant de lecture/génération en un assistant **mieux ancré dans la réalité du patrimoine applicatif ARCAD**.

---

## 2. Ce que l'on peut considérer comme établi

### Confirmé par la documentation ARCAD

Le MCP ARCAD permet d'exposer à un assistant IA des informations de type :
- dépendances entre composants ;
- architecture applicative ;
- flux ;
- analyses d'impact ;
- historique/versioning ARCAD ;
- informations utiles à la maintenance et à la modernisation.

Les transcripts fournis montrent aussi un usage sur :
- cross-reference ;
- build validation ;
- import et déploiement ;
- statut d'exécution ;
- respect des autorisations ARCAD ;
- limitation des opérations selon les rôles.

### Confirmé par la doc Arcade / le framework MCP

La plateforme documente des mécanismes particulièrement utiles pour un futur POC ARCAD :
- auth au niveau du serveur HTTP et auth au niveau des tools ;
- gestion sécurisée des secrets et des tokens ;
- tasks longues avec statut/polling/résultat différé ;
- logs, audit, statut de déploiement ;
- metadata de tools pour classifier les opérations ;
- intégration VS Code / Cursor / Claude / Bob-like workflows ;
- déploiement cloud et gestion centralisée.

### Point de prudence

Cette fiche **n'affirme pas** que le MCP ARCAD réalise lui-même les conversions métier de type COBOL → RPG ou DDS → DDL. L'apport attendu est surtout de :
- mieux **qualifier** ;
- mieux **cadrer** ;
- mieux **sécuriser** ;
- mieux **tracer** ;
- mieux **réintégrer**.

---

## 3. Matrice de synthèse

| UC | Intitulé | Impact probable du MCP ARCAD | Pourquoi |
|---|---|---|---|
| 1 | COBOL → FREE RPG | **Moyen** | meilleur contexte d'impact, versioning, statut de remplacement, réintégration |
| 2 | RPG colonné → FREE | **Moyen** | même logique que UC 1, avec meilleur cadrage des dépendances et promotions |
| 3 | Accès natifs → SQL embarqué | **Moyen à fort** | dépendances programme/fichier, analyse d'impact, traçabilité des conversions |
| 4 | Compréhension de code | **Fort** | cas d'usage le plus naturellement aligné avec le référentiel ARCAD |
| 5 | Extraction logique métier | **Fort** | meilleur ancrage des règles dans les flux, fichiers, champs et composants réels |
| 6 | Documentation complète | **Fort** | historique, architecture, dépendances, références croisées plus fiables |
| 7 | Optimisation de code | **Moyen à fort** | borne mieux le périmètre et la réintégration des changements |
| 8 | Restructuration | **Fort** | réduction du risque par meilleure vision du couplage et des objets touchés |
| 9 | Génération de code | **Moyen** | aide surtout à générer dans le bon contexte applicatif et de gouvernance |
| 10 | Génération d'applications | **Moyen** | surtout utile côté back-end IBM i, traçabilité et dépendances |
| 11 | Génération d'objets SQL | **Moyen à fort** | dépendances BD/applicatives, versioning, intégration outillée |
| 12 | Serveurs MCP | **Très fort** | le MCP ARCAD devient lui-même un enabler transverse majeur |
| 13 | Tests unitaires | **Moyen à fort** | traçabilité test/version/changement + intégration pipeline |
| 14 | DDS → DDL | **Moyen à fort** | meilleure analyse des impacts PF/LF/programmes et meilleure réintégration |

---

## 4. Fiches par use case

## UC 1 — COBOL → FREE RPG
**Impact proposé : Moyen**

**Apports probables :**
- qualification plus fiable des dépendances autour du programme COBOL source ;
- meilleure identification des objets appelants/appelés avant conversion ;
- traçabilité du remplacement COBOL → RPG dans le référentiel ARCAD ;
- réduction du travail manuel de création/réintégration du nouveau membre.

**Ce que cela change par rapport au POC sans MCP :**
- moins de post-traitement ARCAD manuel après génération ;
- meilleur suivi du statut du composant remplacé ;
- meilleure sécurisation si une promotion concurrente existe déjà.

**Limite :** le MCP ARCAD ne remplace pas l'expertise de conversion COBOL/RPG ; il réduit surtout le risque de contexte faux.

## UC 2 — RPG colonné → FREE
**Impact proposé : Moyen**

**Apports probables :**
- visibilité sur promotions en cours et version active avant transformation ;
- exposition des dépendances réelles du programme converti ;
- meilleur workflow de versioning/réintégration après validation.

**Bénéfice principal :** Bob convertit, ARCAD sécurise le cadre de modification.

## UC 3 — Accès natifs → SQL embarqué
**Impact proposé : Moyen à fort**

**Apports probables :**
- meilleure cartographie entre programme RPG, fichiers accédés, impacts indirects ;
- aide à identifier quels composants seront affectés par une conversion d'accès ;
- meilleure traçabilité des sources convertis et de leur statut ;
- possibilité future de lier plus facilement conversion, build, tests et promotion.

**Pourquoi plus fort qu'UC 1/2 :** cette UC touche directement la relation code ↔ données ↔ autres composants.

## UC 4 — Compréhension de code
**Impact proposé : Fort**

**Apports probables :**
- cas d'usage le plus naturel du MCP ARCAD ;
- réponses appuyées sur le référentiel ARCAD plutôt que sur le seul source ouvert ;
- meilleure restitution du rôle du composant, de ses relations et de son périmètre ;
- onboarding plus rapide de nouveaux profils.

**C'est probablement l'un des meilleurs UC de démonstration d'un futur POC.**

## UC 5 — Extraction logique métier
**Impact proposé : Fort**

**Apports probables :**
- meilleure extraction des règles dans leur vrai contexte applicatif ;
- lien plus fiable entre règles métier, fichiers, champs, flux et composants ;
- réduction des faux positifs où le LLM “raconte une histoire plausible”.

**Valeur métier :** meilleure confiance dans la logique métier restituée.

## UC 6 — Documentation complète
**Impact proposé : Fort**

**Apports probables :**
- meilleure qualité des specs techniques ;
- références croisées plus fiables ;
- meilleure capacité à intégrer historique, versions et dépendances ;
- architecture et cartographie plus proches de la réalité ARCAD.

**Bénéfice concret :** les livrables de documentation deviennent beaucoup plus crédibles pour préparer les UC suivants.

## UC 7 — Optimisation de code
**Impact proposé : Moyen à fort**

**Apports probables :**
- meilleure délimitation du périmètre de changement ;
- meilleure anticipation des régressions indirectes ;
- meilleure intégration des diffs dans le workflow ARCAD ;
- éventuelle validation plus fluide avec build/versioning.

**Bénéfice principal :** réduction du risque de “petite optimisation locale” ayant un effet plus large que prévu.

## UC 8 — Restructuration
**Impact proposé : Fort**

**Apports probables :**
- meilleure lecture du couplage réel avant extraction de modules ;
- meilleure identification des composants impactés ;
- création/réintégration mieux gouvernée des nouveaux modules ;
- meilleure préparation aux étapes de build/test/promotion.

**Pourquoi fort :** c'est une UC à risque structurel élevé ; tout gain de précision sur le périmètre a beaucoup de valeur.

## UC 9 — Génération de code
**Impact proposé : Moyen**

**Apports probables :**
- aide à générer dans un contexte applicatif existant, pas “hors sol” ;
- réduction des collisions de nommage ou de doublons déjà gérés ;
- meilleur rattachement aux conventions de versioning/réintégration.

**Limite :** le MCP ARCAD n'est pas le moteur de génération lui-même ; son rôle est surtout de contextualiser et gouverner.

## UC 10 — Génération d'applications
**Impact proposé : Moyen**

**Apports probables :**
- meilleur ancrage côté composants IBM i existants ;
- meilleure vision des dépendances back-end ;
- meilleure traçabilité des objets générés devant rejoindre un workflow ARCAD.

**Moins fort sur la partie UI pure**, plus utile sur l'intégration applicative et le cycle de vie.

## UC 11 — Génération d'objets SQL
**Impact proposé : Moyen à fort**

**Apports probables :**
- meilleure visibilité sur l'impact applicatif des objets SQL créés ;
- meilleure réintégration/versioning des scripts et objets ;
- meilleure cohérence avec les flux de promotion et de déploiement.

**Bénéfice :** moins de scripts “juste générés”, plus d'objets réellement intégrés au patrimoine gouverné.

## UC 12 — Utilisation des serveurs MCP
**Impact proposé : Très fort**

**Apports probables :**
- le MCP ARCAD devient un enabler transversal pour presque tous les autres UC ;
- donne à Bob un accès structuré à l'intelligence applicative ARCAD ;
- complète les MCP IBM i et IBM i Database par une couche de gouvernance, d'historique et de dépendances.

**Pour un futur POC, UC 12 + UC 4 est probablement le couple de démonstration le plus convaincant.**

## UC 13 — Tests unitaires
**Impact proposé : Moyen à fort**

**Apports probables :**
- meilleure traçabilité entre source modifié, version ARCAD, tests produits et résultats ;
- meilleure intégration du post-test dans un pipeline gouverné ;
- meilleure exploitation des artefacts de test dans un workflow de promotion.

**Bénéfice principal :** les tests cessent d'être seulement un livrable Bob, ils deviennent plus facilement un élément du cycle de vie ARCAD.

## UC 14 — DDS → DDL
**Impact proposé : Moyen à fort**

**Apports probables :**
- meilleure vision des PF/LF et des programmes impactés ;
- meilleure prise en compte de la criticité des objets touchés ;
- meilleure réintégration des scripts DDL produits ;
- meilleure préparation d'une migration structurelle sans casser l'existant.

**Bénéfice principal :** le MCP ARCAD sécurise la migration par la connaissance de l'écosystème autour du DDS, pas seulement du DDS lui-même.

---

## 5. Propositions de cadrage pour un futur POC

### Proposition A — POC “compréhension + impact”
Priorité : **UC 12 + UC 4 + UC 6 + UC 14**

Objectif : démontrer que le MCP ARCAD améliore fortement la compréhension, la documentation et l'analyse d'impact avant toute modernisation profonde.

### Proposition B — POC “modernisation sécurisée”
Priorité : **UC 12 + UC 3 + UC 7 + UC 8 + UC 13**

Objectif : démontrer que le MCP ARCAD sécurise les transformations de code par meilleure analyse de périmètre, meilleure traçabilité et meilleure intégration dans les validations.

### Proposition C — POC “chaîne complète”
Priorité : **UC 12 + UC 4 + UC 8 + UC 13 + UC 16**

Objectif : montrer la chaîne la plus différenciante : compréhension → restructuration → tests → pipeline.

> Si l'objectif est de convaincre vite, **Proposition A** est la plus démonstrative.
> Si l'objectif est de démontrer un ROI industriel, **Proposition B** est probablement la plus crédible.

---

## 6. Points à challenger en atelier

Voici les questions à valider avant de figer le futur POC :
- Quel est le **catalogue réel des outils MCP ARCAD** disponibles sur la version cible ?
- Le MCP expose-t-il seulement la **lecture du référentiel**, ou aussi certaines **actions** de versioning/promotion/build/deploy ?
- Quelle granularité de **droits** peut être appliquée par rôle ?
- Quels artefacts du cycle ARCAD sont réellement **auditables** depuis le MCP ?
- Quels UC veulent un apport surtout de **compréhension**, et lesquels visent une **orchestration actionnable** ?

---

## 7. Prompt de reprise des fiches UC 1 à 14 avec MCP ARCAD disponible

> À utiliser dans Bob lorsque le MCP ARCAD est réellement disponible dans l'environnement cible.
> Objectif : demander à Bob de **revoir une fiche UC existante** pour mieux exploiter le MCP ARCAD dans les prérequis, prompts, validations, livrables et points de vigilance.
>
> **Important :** appuie-toi sur les sources publiques en ligne les plus à jour, en priorité le dépôt GitHub [`ArcadeAI/arcade-mcp`](https://github.com/ArcadeAI/arcade-mcp) et la documentation [`docs.arcade.dev`](https://docs.arcade.dev/en/home).

```md
Le MCP ARCAD est disponible dans cet environnement, en complément des MCP IBM i et IBM i Database.

Je veux que tu revoies la fiche `[NOM_FICHIER_UC].md` pour l'adapter à un fonctionnement **avec MCP ARCAD actif**.

Travaille uniquement sur cette fiche UC, sans modifier les autres documents.

Appuie-toi sur les sources publiques en ligne les plus à jour, en priorité :
- le dépôt GitHub `https://github.com/ArcadeAI/arcade-mcp`
- la documentation `https://docs.arcade.dev/en/home`

Objectif : identifier où le MCP ARCAD apporte une valeur concrète et réécrire la fiche pour mieux l'exploiter, sans changer l'intention fonctionnelle de l'UC.

Merci de produire ta réponse en 4 sections :

## 1. Analyse d'impact du MCP ARCAD sur cette fiche
- Quelles parties de la fiche changent réellement si le MCP ARCAD est disponible ?
- Quelles étapes actuellement manuelles deviennent assistées, pilotées ou traçables ?
- Quels gains sont attendus : contexte, dépendances, analyse d'impact, historique, versioning, promotion, build, déploiement, audit, traçabilité, gouvernance ?
- Distingue bien :
  - ce qui est **certain**,
  - ce qui est **probable mais à valider** selon le catalogue réel du MCP ARCAD.

## 2. Liste précise des modifications à apporter à la fiche
Pour chaque zone de la fiche, indique les changements proposés :
- Objectif
- Prérequis
- Mode Bob et MCP à utiliser
- Prompts clés
- Livrables attendus
- Validation / checklist
- Section ARCAD / spécificité ARCAD
- Limites et risques

Présente cela sous forme de tableau :
| Section de la fiche | Modification proposée | Pourquoi |

## 3. Nouveau texte prêt à insérer dans la fiche
Rédige les paragraphes réécrits ou ajoutés pour intégrer le MCP ARCAD dans cette UC.
Le texte doit être directement réutilisable dans la fiche finale, en français, avec un style homogène au document d'origine.

## 4. Ajustements des prompts de l'UC
Pour chaque prompt de l'UC qui devrait mieux exploiter le MCP ARCAD, propose :
- le prompt actuel à enrichir ;
- l'information MCP ARCAD à demander explicitement ;
- le nouveau texte recommandé.

Quand c'est pertinent, fais exploiter au MCP ARCAD des informations comme :
- dépendances entre composants ;
- objets impactés ;
- historique des versions ;
- statut de promotion / verrouillage ;
- composants déjà managés ;
- traçabilité build / test / déploiement ;
- informations de gouvernance ARCAD.

Contraintes :
- ne pas inventer des capacités non prouvées du MCP ARCAD ;
- si une capacité dépend du catalogue réel d'outils MCP ARCAD, le signaler explicitement ;
- ne pas transformer l'UC en pipeline DevOps complet si ce n'est pas son objet ;
- conserver l'équilibre entre Bob, IBM i MCP, IBM i Database MCP et MCP ARCAD.
```

---

## 8. Conclusion

Sur les UC 1 à 14, l'apport du MCP ARCAD ne doit pas être vendu comme “Bob générera mieux du code”.

La vraie proposition de valeur est plus forte et plus crédible :
- Bob comprend mieux l'application ;
- Bob travaille avec un contexte plus vrai ;
- Bob s'insère mieux dans la gouvernance ARCAD ;
- les changements sont mieux tracés, mieux validés et potentiellement mieux promus.

Autrement dit, **le MCP ARCAD augmente surtout la fiabilité opérationnelle et la valeur industrielle de Bob sur IBM i**.
