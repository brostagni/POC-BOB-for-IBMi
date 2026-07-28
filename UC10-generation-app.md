# UC 10 — Génération d'applications

> **Catégorie :** Développement
>
> **Priorité dans le POC :** 11c — Track B Phase 4, dernier UC de la track ; après UC 9 et UC 11 ; dépend de UC 4 et UC 6 pour les sous-cas B/C
>
> **Durée POC (avec Bob) :** 4 à 8 heures — cadrage du sous-cas (Prompt 0), analyse de 1 à 2 écrans pilotes par sous-cas, 1 livrable par sous-cas
>
> **Durée PROD (avec Bob) :** sous-cas A : 1 à 3 heures / écran ; sous-cas B/C : 2 à 6 heures / écran (SUBFILE compte double)
>
> **Durée PROD (sans Bob) :** sous-cas A : 1 à 2 jours / écran (DDS display file + programme RPG) ; sous-cas B/C : 1 à 3 jours / écran selon complexité de la navigation 5250 et présence de SUBFILE
>
> **Gain Bob estimé :** ~6× en moyenne, très variable selon le sous-cas — gain fort sur la génération de la structure DDS (sous-cas A) et des composants Web (sous-cas B/C) ; gain faible sur la réflexion UX et le CSS (non automatisables)
>
> **Mode initial :** Ask ou ACME IBM i Review.
> **Mode d'exécution :** Agent ou ACME IBM i Execute.

---

## Objectif

Générer des **interfaces applicatives** IBM i — nouvelles applications 5250, composants Web modernes, ou conversions d'écrans 5250 existants vers le Web.

**Ce UC couvre 3 sous-cas à traiter comme 3 niveaux d'ambition de modernisation de l'interface** — pas comme 3 UC indépendants. Le Prompt 0 détermine le sous-cas retenu pour chaque écran ou groupe d'écrans.

**Sous-cas A — Nouvelle application 5250 :** génération d'un display file DDS (`*DSPF`) et du programme RPG de gestion d'écran, *ex nihilo*, à partir d'une description fonctionnelle ou de maquettes.

**Sous-cas B — Conversion 5250 → Web :** analyse d'un display file DDS existant et génération des composants Web équivalents (React / Vue / Angular selon le choix ACME). Requiert une réflexion UX — la conversion ne doit pas être un copier-coller de l'écran 5250.

**Sous-cas C — Web UI moderne sur back-end existant :** variante du sous-cas B avec la contrainte supplémentaire que le programme RPG back-end ne doit **pas** être modifié. Toute la logique de présentation reste côté Web.

**Démarcation avec UC 9 :** UC 9 génère la **logique RPG** (programme batch, procédures, CRUD). UC 10 génère l'**interface** (DDS display file ou composant Web). Pour une application interactive complète, les deux UCs sont complémentaires : UC 9 pour le programme RPG de traitement, UC 10 pour le display file ou le composant Web. Ne pas utiliser UC 9 seul pour un programme interactif — utiliser UC 9 pour la partie RPG et UC 10 pour la partie interface.

**Démarcation avec UC 11 :** UC 11 génère des objets SQL (tables, procédures, triggers). Les composants Web produits en UC 10 (sous-cas B/C) peuvent interroger des procédures stockées générées en UC 11 — les deux UCs sont complémentaires sur ce point.

> ⚠️ **UC 10 est l'UC le plus consommateur en Bob Coins et en effort de cadrage de la Track B.** Ne pas démarrer sans avoir défini le périmètre exact : quel sous-cas, combien d'écrans, quel écran pilote. Envoyer toute une application au Prompt 0 sans délimitation produit un résultat générique sans valeur.

**Convention de nommage des fichiers générés :**
```
{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md

Types pour cet UC :
  analyse-ecrans  → analyse des écrans 5250 (Prompt 0 / Prompt 1) — inventaire, catégorie, points d'attention
  dspf-genere     → source DDS display file généré (sous-cas A)
  web-genere      → source composant Web généré (sous-cas B/C)
  plan-ecrans     → plan de conversion du périmètre applicatif complet (Prompt 4)
```

Exemples :
```
acme-APPVTE-ECRCDE-analyse-ecrans-20250625-0900.md   ← analyse écran de saisie de commande
acme-APPVTE-ECRCDE-dspf-genere-20250625-1100.md      ← display file DDS généré (sous-cas A)
acme-APPVTE-ECRCDE-web-genere-20250625-1400.md       ← composant React généré (sous-cas B/C)
acme-APPVTE-APPVTE-plan-ecrans-20250625-1600.md      ← plan périmètre complet
```

> 💡 Cette convention est valable en dehors du contexte POC — réutilisable en production tel quel.

---

## Démarrer par un écran connu

> **Recommandation forte avant d'aborder les écrans critiques de ACME.**

Commencer par un **écran simple dont l'équipe connaît le comportement attendu** — idéalement un écran de consultation ou de saisie, sans SUBFILE, sans navigation multi-niveaux, dont le rendu 5250 peut être vérifié immédiatement sur un terminal ou via le DDS Previewer de Code for IBM i.

Pourquoi ? Parce que la première génération sert à **calibrer deux choses** :
- La qualité de la génération DDS ou Web de Bob sur le style d'écran ACME (libellés, conventions de nommage des champs, validations)
- La capacité de l'équipe à valider le résultat avant de passer aux écrans avec SUBFILE ou navigation complexe

> 💡 Pour chaque écran, **commencer par le Prompt 0** — il détermine le sous-cas (A/B/C) et la catégorie (SIMPLE/STANDARD/COMPLEXE). Ne pas aller directement au Prompt 1. Le sous-cas peut changer d'un écran à l'autre dans la même application.

**Tableau de progression — Sous-cas A (nouvelle application 5250) :**

| Étape | Écran à choisir | Catégorie attendue | Objectif |
|-------|----------------|--------------------|---------|
| 1 | 1 écran de consultation, < 15 champs, pas de SUBFILE, pas de navigation | SIMPLE | Calibrer la génération DDS : types de champs, CHECK/VALUES, layout |
| 2 | Écran de saisie avec validation (VALUES, RANGE), 1 ou 2 touches de fonction (PF3, PF12) | SIMPLE à STANDARD | Valider la génération des indicateurs de contrôle et la logique de validation côté DDS |
| 3 | Application de 2 à 5 écrans liés avec navigation PF, menu de sélection | STANDARD | Valider la cohérence de la navigation inter-écrans et des indicateurs partagés |
| 4 | Écran avec SUBFILE (liste de sélection ou grille de saisie) | COMPLEXE | Appliquer la stratégie en deux passes (analyse SUBFILE, puis génération) |

**Tableau de progression — Sous-cas B/C (conversion 5250 → Web) :**

| Étape | Écran à choisir | Catégorie attendue | Objectif |
|-------|----------------|--------------------|---------|
| 1 | 1 écran simple, sans SUBFILE, validations DDS directes, programme RPG documenté (UC 4 complété) | SIMPLE | Calibrer la génération du composant Web : structure, champs, validations côté client |
| 2 | Écran avec validation DDS (CHECK, VALUES, RANGE) et touches de fonction PF | SIMPLE à STANDARD | Valider la traduction des validations DDS en validation côté client |
| 3 | Écran avec 1 SUBFILE — liste de sélection simple | STANDARD | Appliquer la stratégie deux passes sur un SUBFILE simple avant d'aborder un SUBFILE complexe |
| 4 | Application multi-écrans, navigation dynamique, SUBFILE imbriqués ou indicateurs RPG pilotant l'affichage | COMPLEXE | Valider les limites de Bob et identifier les écrans nécessitant un atelier UX préalable |

---

## Impact de la complexité sur la stratégie

La complexité d'UC 10 ne se mesure pas en nombre de lignes de DDS, mais en **facteurs structurels** qui déterminent si Bob peut générer en une passe ou si une stratégie en plusieurs passes est nécessaire.

**Facteurs réels qui compliquent UC 10 :**
- **Présence de SUBFILE** — la construction 5250 la plus complexe. La logique de chargement (page par page ou complet) est souvent dans le programme RPG, pas dans le DDS. Aucun équivalent direct en HTML/React — la conversion nécessite une passe d'analyse avant la génération
- **Logique de navigation implicite dans le programme RPG** — les touches de fonction PF et les indicateurs de niveau contrôlent quelle page s'affiche. Cette logique doit être lue dans le programme RPG, pas seulement dans le DDS
- **Validations DDS embarquées** (CHECK, VALUES, RANGE) — à recréer côté client Web avec une sémantique différente (validation JavaScript vs validation DDS côté IBM i)
- **Appels programme imbriqués depuis un écran** — un écran qui appelle un autre programme interactif (chaîne d'écrans dans plusieurs programmes)
- **Contrainte sous-cas C : le programme RPG back-end ne peut pas être modifié** — toute adaptation doit se faire côté interface ; Bob ne doit pas proposer de modifier le programme RPG pour faciliter la génération Web

### Sous-cas A — Nouvelle application 5250

| Catégorie | Critères | Stratégie |
|-----------|---------|-----------|
| **SIMPLE** | 1 écran, < 15 champs, pas de SUBFILE, logique de validation simple (quelques CHECK/VALUES) | Prompt 0 → Prompt 1a (DDS) → Prompt 1a-bis (CRTDSPF) → Prompt 2a (programme RPG) |
| **STANDARD** | 2 à 5 écrans liés, navigation par PF, quelques validations, pas de SUBFILE | Prompt 0 → Prompt 1a par écran → Prompt 1a-bis → Prompt 2a avec logique de navigation |
| **COMPLEXE** | SUBFILE, navigation multi-niveaux, logique de programme RPG couplée à l'affichage | Prompt 0 (délimiter le périmètre) → Prompt 1a par écran sans SUBFILE → **Prompt 2A-SFL** (génération DDS SUBFILE en trois passes) → Prompt 2a |

### Sous-cas B/C — Conversion 5250 → Web

| Catégorie | Critères | Stratégie |
|-----------|---------|-----------|
| **SIMPLE** | 1 à 2 écrans sans SUBFILE, validations directes, programme RPG bien documenté (UC 4 complété) | Prompt 0 → Prompt 1b (analyse DDS) → Prompt 3b (composant Web) |
| **STANDARD** | 3 à 6 écrans, 1 SUBFILE, navigation par touches de fonction | Prompt 0 → Prompt 1b → Prompt 2b (SUBFILE deux passes) → Prompt 3b par écran |
| **COMPLEXE** | SUBFILE imbriqués, navigation dynamique, état d'écran piloté par indicateurs RPG, appels imbriqués | Prompt 0 (périmètre + atelier UX requis avant génération) → stratégie à définir écran par écran |

> 💡 Sur un COMPLEXE, le Prompt 0 peut conclure qu'un atelier UX avec l'équipe ACME est nécessaire avant toute génération. Dans ce cas, ne pas passer au Prompt 1b — attendre les maquettes ou wireframes de l'équipe Front-End.

---

## Démarrer une session Bob

> **À lire avant chaque session UC 10 — nouvelle conversation ou reprise.**

> 💰 **Avertissement Bob Coins — UC 10 est le plus consommateur de la Track B.** Définir le périmètre avant de démarrer : 1 écran pilote, pas toute l'application. Ne pas envoyer 10 écrans DDS au Prompt 0 — saturation de contexte et résultat générique. La règle : 1 écran (ou groupe d'écrans fonctionnellement liés) par session. Valider, puis continuer sur les suivants.

### 1. Nouvelle conversation Bob

Démarrer une **nouvelle conversation Bob** (bouton `+` en haut du panneau Chat) pour chaque écran ou groupe d'écrans fonctionnellement liés.

**Exception :** enchaîner la génération de plusieurs écrans dans la même session si la navigation entre eux est documentée et que les structures de données sont partagées — cela économise les Bob Coins sur le rechargement du contexte DDS et du programme RPG de navigation.

**Mode à sélectionner :** Ask ou ACME IBM i Review (génération), Agent ou ACME IBM i Execute (compilation CRTDSPF et sauvegarde).

Ne pas réutiliser une conversation d'un autre écran — les indicateurs, les noms de champs et la logique de navigation d'un autre écran polluent la génération de l'écran courant.

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Avant de lancer le Prompt 0, ouvrir dans l'éditeur Bob les sources pertinents. L'ouverture dans l'éditeur les rend accessibles au MCP IBM i sans copier-coller.

**Procédure :** dans le panneau **IBM i — Object Browser** (extension Code for IBM i), naviguer jusqu'à la bibliothèque source (`QDDSSRC` pour les display files, `QRPGSRC` pour les programmes), faire un clic droit sur le membre → **Open in Editor**.

Fichiers à ouvrir selon le sous-cas :

| Sous-cas | Fichiers à ouvrir |
|----------|------------------|
| **A** | Les maquettes ou description fonctionnelle des écrans (fichier texte ou PDF dans le workspace) |
| **B/C** | Le display file DDS source (`[NOM_LIB]/QDDSSRC([NOM_DSPF])`) — via Object Browser |
| **B/C** | Le programme RPG de gestion d'écran (`[NOM_LIB]/QRPGSRC([NOM_PROGRAMME])`) — pour que Bob comprenne la logique de navigation et les indicateurs |
| **B/C** | Les copybooks DDS si le display file en référence (`[NOM_LIB]/QDDSSRC([NOM_COPYBK])`) |

### 3. Fichiers de contexte à charger

Ces fichiers produits par les UC précédents doivent être disponibles dans le workspace Bob **avant** de démarrer. Utiliser **Add File to Chat** (icône trombone) ou les ouvrir dans l'éditeur.

| Fichier | Produit par | Obligatoire / Recommandé |
|---------|-------------|--------------------------|
| `{projet}-{lib}-{programme}-comprehension-{date}.md` | UC 4 | **Obligatoire pour B/C** — carte du programme interactif : écrans, indicateurs, logique de navigation |
| `{projet}-{lib}-{programme}-spec-fonc-{date}.md` | UC 6 | **Recommandé** — spécification fonctionnelle des écrans à convertir ou à créer |
| Maquettes / wireframes fournis par ACME | Équipe Front-End | **Recommandé pour B/C** — sans maquette, Bob génère une mise en page par défaut sans valeur UX |
| `{projet}-{lib}-{programme}-analyse-ecrans-{date}.md` | UC 10 (session précédente) | **Si reprise** — catégorie, périmètre et points d'attention déjà identifiés ; évite de relancer le Prompt 0 |

> ⚠️ **Prérequis critique pour B/C :** ne jamais démarrer la génération Web sans le fichier `*-comprehension-*.md` du programme interactif. Convertir un écran 5250 sans carte de la logique de navigation (indicateurs PF, indicateurs d'option, logique de chargement SUBFILE) produit un composant Web qui affiche les champs mais ne gère pas les interactions.

> ⚠️ **Risque de réduction de contexte — règle de sauvegarde intermédiaire obligatoire :** UC 10 est l'UC le plus long de la Track B. Au-delà de 4 à 5 prompts consécutifs (Prompt 0 + Prompt 1b + Prompt 2b Passe 1 + Passe 2 + Prompt 3b), Bob commence à compresser les premiers tours — les points d'attention détectés au Prompt 0 (SUBFILE identifiés, indicateurs de navigation, contrainte no-modify sous-cas C) peuvent disparaître de la fenêtre active. **Sauvegarder en mode Agent après chaque prompt majeur** (Prompt 0, Prompt 1b, Prompt 2b) — pas seulement en fin de session. À chaque reprise, commencer le prompt par : "L'analyse des écrans est dans [NOM_FICHIER] — le sous-cas retenu est [A/B/C], la catégorie est [SIMPLE/STANDARD/COMPLEXE]."

> 💡 **Reprise de session :** si la session est interrompue, ouvrir le fichier `*-analyse-ecrans-*.md` — Bob retrouve le sous-cas, la catégorie et les points d'attention identifiés sans relancer le Prompt 0. Indiquer "l'analyse des écrans est dans [NOM_FICHIER] — reprendre à partir du Prompt [N°]".

> 💡 **Lien vers l'UC suivant :** les fichiers `*-dspf-genere-*.md` et `*-web-genere-*.md` produits dans cet UC sont les inputs de UC 13 (tests fonctionnels des interfaces générées). Voir la Carte des livrables dans `plan-poc-bob-acme.md`.

---

## Prérequis

- UC 15 et UC 12 complétés — maîtrise de Bob et MCPs actifs
- **Pour sous-cas B/C :** UC 4 complété sur le programme interactif source — les fichiers `*-comprehension-*.md` sont disponibles et couvrent la logique de navigation et les indicateurs
- **UC 6 recommandé** — les fichiers `*-spec-fonc-*.md` documentent les écrans à convertir ou à créer ; sans spec fonctionnelle, le Prompt 0 sera sous-optimal
- UC 9 et UC 11 complétés (si les composants Web appellent des programmes RPG ou des procédures SQL générés dans ces UCs)
- **Pour sous-cas A :** maquettes ou description fonctionnelle des écrans à générer (champs, validations, navigation) — sans maquette, le Prompt 1a produit un écran générique
- **Pour sous-cas B/C :** display file DDS source accessible via IBM i MCP ; programme RPG de gestion d'écran accessible
- **Pour sous-cas B/C :** le **framework Web retenu par ACME** (React / Vue / Angular) doit être décidé et documenté **avant de démarrer UC 10** — sans ce choix, les Prompts 2b (Passe 2) et 3b génèrent du React par défaut, incompatible avec l'équipe Front-End si elle travaille dans un autre framework
- **Pour sous-cas C :** confirmation explicite et écrite de l'équipe ACME que le programme RPG back-end ne sera pas modifié — à consigner dans le fichier `*-analyse-ecrans-*.md` avant de démarrer

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | Mode **Ask** (ou ACME IBM i Review) pour l'analyse et la génération, **Agent** (ou ACME IBM i Execute) uniquement pour la compilation CRTDSPF (sous-cas A) et la sauvegarde. Ne pas présenter Ask/Agent comme des sous-modes d'IBM i Developer — ce sont des modes pairs. |
| **Scope** | Library List → bibliothèque applicative ACME |
| **MCP actifs** | IBM i MCP (lecture du display file DDS source pour B/C, écriture et compilation du display file pour A via `CRTDSPF`) |
| **MCP complémentaires** | IBM i Database MCP (si le composant Web interroge des vues SQL ou des procédures stockées générées en UC 11), Confluence MCP (publication du plan de conversion Prompt 4, si token disponible) |

### Tableau Phase / Mode / Ce que Bob fait

| Phase | Mode | Ce que Bob fait |
|-------|------|----------------|
| Qualification (Prompt 0) | **Ask** | Analyse le besoin, détermine le sous-cas (A/B/C), la catégorie, les points d'attention (SUBFILE ?, validations complexes ?), recommande la séquence |
| Analyse des écrans 5250 (Prompt 1b) | **Ask** | Lit le display file DDS via IBM i MCP, produit le tableau des écrans (RECORD, champs, SUBFILE, validations, indicateurs PF) |
| Génération DDS display file (Prompt 1a) | **Ask** | Génère les R-specs DDS, les champs avec types et validations, dans le chat |
| Analyse SUBFILE (Prompt 2b — Passe 1) | **Ask** | Décrit la logique du SUBFILE (chargement, indicateurs de contrôle, touches PF de navigation) |
| Génération composant SUBFILE (Prompt 2b — Passe 2) | **Ask** | Génère le composant React/Vue/Angular équivalent dans le chat |
| Génération composant Web par écran (Prompt 3b) | **Ask** | Génère la structure du composant Web (formulaire, champs, validations, mapping PF → boutons) dans le chat |
| Génération programme RPG (Prompt 2a — sous-cas A) | **Ask** | Génère le programme RPG Free de gestion d'écran dans le chat |
| Test de compilation CRTDSPF (Prompt 1a-bis) | **Agent** | Lance `CRTDSPF` via IBM i MCP, rapporte les erreurs, propose les corrections |
| Sauvegarde des livrables | **Agent** | Écrit les fichiers `.md` dans le workspace — uniquement une fois chaque section validée |

> 💡 **Règle d'or pour UC 10 :** Le mode Agent est autorisé **uniquement** pour deux opérations : la compilation via `CRTDSPF` (sous-cas A) et la sauvegarde des livrables validés. Pendant toute la phase d'analyse et de génération, rester en mode Ask.

> ⚠️ **Pas de Prompt-bis CRTDSPF pour les sous-cas B/C.** Le code Web n'est pas compilé sur IBM i — la validation est fonctionnelle (revue développeur Web + test dans le navigateur). Ne pas utiliser le mode Agent pour "valider" un composant Web.

> ⚠️ **Ne jamais rester en mode Agent pendant l'analyse DDS.** Un mode Agent actif pendant l'analyse du display file source peut modifier le membre DDS existant sur l'IBM i de test.

### Intégration ARCAD

Le MCP ARCAD n'était pas disponible dans le contexte de ce POC de référence (version ARCAD non compatible avec le MCP). Si le MCP ARCAD est disponible dans votre environnement, les étapes manuelles de réintégration décrites ci-dessous peuvent être automatisées. N'hésitez pas à demander à Bob de modifier cette fiche UC en intégrant la disponibilité du MCP ARCAD.

**Impact sur UC 10 : faible à moyen.** Les display files DDS générés (sous-cas A) et les sources Web générés (sous-cas B/C) sont des objets nouveaux — ils doivent être enregistrés dans ARCAD après validation.

| Sans MCP ARCAD (contexte de ce POC) | Avec MCP ARCAD disponible |
|--------------------------------------|---------------------------|
| Créer manuellement le nouveau membre QDDSSRC dans ARCAD après validation | IBM i MCP + MCP ARCAD peuvent créer le membre et l'enregistrer dans ARCAD directement |
| Marquer manuellement le display file d'origine comme "remplacé" dans ARCAD (sous-cas B/C) | Le MCP ARCAD peut automatiser la mise à jour du statut |
| Les sources Web sont sauvegardés dans le workspace Bob et versionnés via Git | Ce fonctionnement reste identique — Git est indépendant d'ARCAD |

> 💡 **Dans les deux cas :** ajouter un placeholder de traçabilité dans l'en-tête de chaque source DDS généré pour faciliter le suivi ARCAD.

---

## Prompts clés

> 💡 **Atelier Bob Industrialisation**
> Les prompts récurrents (Prompt 0, prompts d'analyse DDS, prompts de génération Web) peuvent être disponibles sous forme de **commandes slash personnalisées** dans le mode "ACME Developer" construit en UC 9.
> Au lieu de copier-coller le bloc de code, taper la commande correspondante dans la conversation Bob.

### Prompt 0 — Qualification : quel sous-cas, quelle catégorie ?

> **Ce prompt est le point d'entrée obligatoire de UC 10 pour chaque écran ou groupe d'écrans.**
> Il détermine le sous-cas (A/B/C), la catégorie (SIMPLE/STANDARD/COMPLEXE), les points d'attention, et recommande la séquence exacte de prompts.
> Il se lance **avant** le Prompt 1 — son résultat conditionne toute la séquence suivante.
> Ne jamais sauter ce prompt sur un écran COMPLEXE avec SUBFILE — le Prompt 0 prend 10 minutes et évite 4 heures de génération dans le mauvais sens.

```
[Si conversion B/C : Le display file DDS [NOM_DSPF] se trouve dans [NOM_LIB]/QDDSSRC.]
[Si disponible : Le fichier de compréhension {projet}-{lib}-{programme}-comprehension-{date}.md est disponible.]
[Si disponible : La spécification fonctionnelle {projet}-{lib}-{programme}-spec-fonc-{date}.md est disponible.]
[Si nouvelle app A : Description du besoin : [décrire l'objectif UX — ex. "écran de saisie d'une commande client avec validation du code article et de la quantité, touches PF3/PF12"]]

Analyse ce besoin et produis en français, en markdown, une fiche de qualification d'écran UC 10 :

## Qualification UC 10 — [NOM_ECRAN ou NOM_DSPF]

### 1. Sous-cas retenu
Indiquer : A (nouvelle 5250), B (conversion 5250 → Web), ou C (Web UI sur back-end existant sans modification RPG)
Justification en 1 à 2 phrases.

### 2. Catégorie
SIMPLE / STANDARD / COMPLEXE — justifier selon les critères de la fiche UC10 (SUBFILE, nombre d'écrans, validations, navigation)

### 3. Points d'attention détectés
- SUBFILE présent ? (OUI/NON — si OUI : type de chargement détecté ?)
- Validations DDS complexes ? (CHECK, VALUES, RANGE)
- Logique de navigation implicite dans le programme RPG ? (indicateurs PF, indicateurs d'option)
- Appels programme imbriqués depuis l'écran ?
- [Sous-cas C uniquement] Le programme RPG back-end est-il réellement appelable hors 5250 (pas de fichier WORKSTN, pas d'EXFMT, pas d'état interactif non exposé) ?

### 3bis. Conclusion d'éligibilité (sous-cas C uniquement)
```
ÉLIGIBLE AU SOUS-CAS C — le programme [NOM_PROGRAMME] peut être appelé depuis un back-end Web sans modification.
```
ou
```
NON ÉLIGIBLE AU SOUS-CAS C — le programme dépend de [raison : EXFMT / fichier WORKSTN / état interactif]. Stratégie alternative à définir avec l'équipe ACME avant de poursuivre.
```
Ne pas passer au Prompt 1b si la conclusion est NON ÉLIGIBLE.

### 4. Séquence de prompts recommandée
Liste ordonnée : Prompt X → Prompt Y → ...

### 5. Estimation Bob Coins
Faible / Moyen / Élevé — et recommandation si Élevé (réduire le périmètre à 1 écran pilote)
```

**Analyse ligne à ligne :**
- *Pourquoi décrire d'abord l'objectif UX avant la technique :* Bob ne peut pas deviner si l'écran doit ressembler à un formulaire Web classique ou à une grille de saisie — la description fonctionnelle oriente la génération vers la bonne structure de composant
- *Comment Bob détecte la présence de SUBFILE dans le DDS :* il cherche les mots-clés `SFL` (sous-fichier), `SFLCTL` (contrôle de sous-fichier) et `SFLPAG` (taille de page) dans les R-specs — si ces mots-clés sont présents, le SUBFILE est confirmé et la stratégie deux passes s'applique automatiquement
- *Pourquoi ne pas sauter le Prompt 0 même sur un écran qui semble simple :* un écran avec peu de champs peut avoir une logique de navigation complexe pilotée par des indicateurs RPG — le Prompt 0 détecte ces cas que l'analyse visuelle du DDS ne révèle pas

---

## — SOUS-CAS A : Nouvelle application 5250 —

### Prompt 1a — Génération du display file DDS

> **À exécuter après le Prompt 0 (catégorie confirmée : SIMPLE ou STANDARD).**

```
[Description des écrans : champs, types, longueurs, validations attendues, touches de fonction PF]
[Si maquette disponible : La maquette de l'écran [NOM_ECRAN] est disponible dans [NOM_FICHIER].]

Génère en français, en markdown, un display file DDS pour l'écran [NOM_ECRAN] conforme aux normes ACME :

## Display file DDS — [NOM_ECRAN]

### 1. Header DDS
Inclure les lignes de commentaires standard ACME (nom écran, date, auteur, programme appelant).

### 2. Spécifications de fichier (A-specs)
DSPSIZ, INDARA, et paramètres globaux nécessaires.

### 3. Enregistrements (R-specs) et champs
Pour chaque RECORD :
- Nom du RECORD (convention de nommage ACME)
- Champs : nom, type (A/S/Y/L/T/Z), longueur, position (ligne/colonne), libellé
  (A = alphanumérique, S = zoned decimal, Y = numérique keyboard-shift, L = date, T = heure, Z = timestamp — D n'est pas un type DDS display file valide)
- Validations DDS : CHECK(ER/LC/RZ), VALUES, RANGE selon le besoin
- Indicateurs de contrôle : DSPATR(HI/BL/RI) et COLOR si nécessaire

### 4. Touches de fonction
Touches de fonction actives par écran :
- CAxx (Command Attention) : la touche déclenche un retour au programme sans retransmettre les saisies (ex. CA03 pour F3=Quitter)
- CFxx (Command Function) : la touche déclenche un retour au programme EN retransmettant les saisies (ex. CF06 pour F6=Ajouter, CF12 pour F12=Annuler)
VLDCMDKEY valide qu'une touche de commande valide a été utilisée — il ne définit pas les touches PF elles-mêmes.

[Pour COMPLEXE avec SUBFILE : voir Prompt 2A-SFL avant de générer les specs SUBFILE]
```

**Analyse ligne à ligne :**
- *Types DDS et leur correspondance :* `A` = alphanumérique, `S` = zoned decimal (numérique signé), `Y` = numérique avec attribut keyboard-shift numérique (l'affichage des séparateurs dépend de `EDTCDE` ou `EDTWRD`, pas du type Y lui-même), `L` = date, `T` = heure, `Z` = timestamp — choisir selon le type SQL correspondant (VARCHAR → A, DECIMAL → S, DATE → L, TIME → T, TIMESTAMP → Z). **`D` n'est PAS un type de champ DDS display file — erreur de compilation garantie.** **`Y` n'affiche pas automatiquement des séparateurs** — utiliser `EDTCDE` pour contrôler le format d'affichage.
- *Indicateurs de condition sur les champs :* `DSPATR(HI)` met le champ en surbrillance, `DSPATR(PR)` le rend protégé (non modifiable), `DSPATR(ND)` masque l'affichage (mot de passe) — les indicateurs sont pilotés par les numéros d'indicateurs RPG (01-99) ; définir la plage d'indicateurs ACME en début de session
- *Convention de nommage ACME pour les champs DDS :* appliquer le même préfixe que dans les programmes RPG générés en UC 9 (mode "ACME Developer") pour garantir la cohérence avec le programme de gestion d'écran généré en Prompt 2a

### Prompt 1a-bis — Test de compilation CRTDSPF

> **Ce prompt s'exécute en mode Agent. Passer en mode Agent avant d'envoyer ce prompt.**

```
Étape 0 — Vérifier que le membre DDS n'existe pas encore :
SELECT COUNT(*) AS MEMBER_COUNT
FROM QSYS2.SYSMEMBERSTAT
WHERE SYSTEM_TABLE_SCHEMA = '[NOM_LIB]'
  AND SYSTEM_TABLE_NAME   = 'QDDSSRC'
  AND SYSTEM_TABLE_MEMBER = '[NOM_DSPF]';
Si MEMBER_COUNT > 0 : arrêter et signaler — ne pas écraser sans validation explicite.

Étape 1 — Écrire le source DDS validé dans le membre :
Écrire uniquement le source DDS (sans balises Markdown) dans [NOM_LIB]/QDDSSRC([NOM_DSPF]).

Étape 2 — Compiler avec CRTDSPF :
CRTDSPF FILE([NOM_LIB]/[NOM_DSPF]) SRCFILE([NOM_LIB]/QDDSSRC) SRCMBR([NOM_DSPF]) REPLACE(*NO)

Si la commande retourne CPF5813 (objet déjà existant) :
- Afficher le message et s'arrêter. NE PAS relancer avec REPLACE(*YES) automatiquement.
- Demander une décision explicite : conserver l'objet existant ou autoriser le remplacement.
- Si remplacement autorisé : relancer avec REPLACE(*YES) et confirmer dans le compte-rendu.

Si la compilation échoue (erreurs syntaxiques) :
1. Lister les messages d'erreur avec leur numéro de ligne source
2. Pour chaque erreur, proposer la correction DDS
3. Corriger le source dans QDDSSRC et relancer CRTDSPF (REPLACE selon la décision ci-dessus)
```

**Analyse ligne à ligne :**
- *Pourquoi `REPLACE(*NO)` en premier essai :* le premier essai avec `REPLACE(*NO)` protège un objet existant d'un écrasement accidentel. Si CPF5813 est retourné (objet déjà existant), une décision explicite est demandée avant tout remplacement. `REPLACE(*YES)` n'est utilisé qu'après validation humaine.
- *Limites de Bob sur la validation visuelle :* Bob peut corriger les erreurs de syntaxe DDS (champs mal positionnés, types invalides, mots-clés mal orthographiés) mais **ne peut pas valider le rendu visuel ni la navigation interactive** — tester obligatoirement sur un terminal 5250 réel ou via le DDS Previewer de Code for IBM i (extension Code for IBM i)

### Prompt 2a — Génération du programme RPG de gestion d'écran

> **À exécuter après le Prompt 1a-bis (display file compilé sans erreur).**

```
Le display file [NOM_DSPF] est compilé et disponible dans [NOM_LIB].
[Si disponible : Le mode "ACME Developer" définit les normes de nommage et de structure.]

Génère en français, en markdown, le programme RPG ILE Free de gestion d'écran pour [NOM_ECRAN] :

## Programme RPG — [NOM_PROGRAMME]

### 1. Header et CTL-OPT
Normes ACME (header, DFTACTGRP, OPTION).

### 2. Déclarations (DCL-F, DCL-S, DCL-DS)
DCL-F pour le display file [NOM_DSPF], structures de données pour les champs d'écran.

### 3. Boucle d'affichage principale
EXFMT pour afficher/lire l'écran.
Le DDS utilise INDARA — les indicateurs NE SONT PAS lus via IN(03)/*IN12 directement sur la DCL-F.
Utiliser INDDS(dsIndicateurs) sur la DCL-F et une DCL-DS décrivant chaque indicateur :
  DCL-F [NOM_DSPF] WORKSTN INDDS(dsIndicateurs) ;
  DCL-DS dsIndicateurs ;
    quitter  IND POS(3) ;   -- CA03 / F3
    ajouter  IND POS(6) ;   -- CF06 / F6
    annuler  IND POS(12) ;  -- CF12 / F12
  END-DS ;
Tester dsIndicateurs.quitter, dsIndicateurs.ajouter, etc. après chaque EXFMT.

### 4. Logique de validation côté programme
Validation des champs obligatoires, appels aux procédures de service (générées en UC 9 si disponibles).

### 5. Appels aux procédures métier
CALLP vers les programmes ou procédures de traitement — ne pas dupliquer la logique métier dans le programme d'écran.
```

**Analyse ligne à ligne :**
- *Démarcation UC 10 vs UC 9 :* ce prompt génère le programme RPG qui **pilote l'interface** (boucle d'affichage, lecture PF, validation de saisie). La logique métier (CRUD, calculs, règles de gestion) appartient aux procédures générées en UC 9. Ne pas demander à Bob de mélanger interface et logique métier dans le même programme
- *Renvoyer vers UC 9 si la logique est l'objectif principal :* si la demande est "génère la logique de traitement de commande", utiliser UC 9 ; UC 10 reste focalisé sur l'interface

> 💡 **Sauvegarder ce source** (mode Agent) :
> ```
> "Sauvegarde ce programme RPG dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-dspf-genere-{YYYYMMDD-HHmm}.md
>  (section Programme RPG — [NOM_PROGRAMME])
>  Exemple : acme-APPVTE-ECRCDE-dspf-genere-20250625-1300.md"
> ```

### Prompt 2a-bis — Test de compilation CRTBNDRPG

> **Ce prompt s'exécute en mode Agent.** Bob compile le programme RPG de gestion d'écran sur l'IBM i de test via IBM i MCP.
> **À exécuter après le Prompt 2a** — le display file `[NOM_DSPF]` doit déjà être compilé (Prompt 1a-bis réussi) avant de compiler le programme RPG qui le référence.

```
Le programme RPG [NOM_PROGRAMME] est prêt à être sauvegardé.
Le display file [NOM_DSPF] est compilé et disponible dans [NOM_LIB].

Étape 1 — Vérifier que le membre RPG n'existe pas encore :
SELECT COUNT(*) AS MEMBER_COUNT
FROM QSYS2.SYSMEMBERSTAT
WHERE SYSTEM_TABLE_SCHEMA = '[NOM_LIB]'
  AND SYSTEM_TABLE_NAME   = '[SRCFILE_RPG]'
  AND SYSTEM_TABLE_MEMBER = '[NOM_PROGRAMME]';
Si MEMBER_COUNT > 0 : arrêter et signaler — ne pas écraser sans validation explicite.

Étape 2 — Écrire le source RPG validé dans le membre :
Écrire uniquement le source RPG (sans balises Markdown) dans [NOM_LIB]/[SRCFILE_RPG]([NOM_PROGRAMME]).

Étape 3 — Compiler avec CRTBNDRPG et rapporter le résultat :
1. Si la compilation réussit sans erreur ni avertissement : confirmer
2. Si des erreurs de compilation sont présentes :
   - Lister chaque erreur (code, numéro de ligne, description)
   - Proposer la correction dans le source pour chaque erreur
   - Préciser si la correction est mécanique (syntaxe) ou nécessite une décision du développeur
3. Si des avertissements sont présents : les lister avec leur niveau de sévérité
4. Rappeler que la compilation réussie ne valide pas le comportement fonctionnel de l'écran —
   le test sur un terminal 5250 réel ou via le DDS Previewer reste obligatoire
```

> 💡 **Ce prompt s'exécute en mode Agent** — droits `*CHANGE` sur la bibliothèque cible requis.

> ⚠️ **Ordre obligatoire : Prompt 1a-bis (CRTDSPF) avant Prompt 2a-bis (CRTBNDRPG).** Le programme RPG contient une DCL-F qui référence le display file — si le `*DSPF` n'existe pas encore sur l'IBM i de test, la compilation du programme RPG échoue avec un message `RNF7031` (objet introuvable).

---

### Prompt 2A-SFL — Génération DDS SUBFILE 5250 (cas COMPLEXE sous-cas A)

> **Ce prompt est réservé au sous-cas A COMPLEXE.** Il se déroule en trois passes distinctes.
> **Ne jamais générer un SUBFILE en une seule passe** — le DDS SUBFILE, le SFLCTL et la logique RPG de chargement sont des entités séparées qui doivent être validées indépendamment.

---

**Passe 1 — Qualification SUBFILE (mode Ask) :**

```
Je dois créer un SUBFILE 5250 pour [NOM_ECRAN].
Le display file [NOM_DSPF] existe dans [NOM_LIB]/QDDSSRC.

Produis en français, en markdown, la qualification de ce SUBFILE :

## Qualification SUBFILE — [NOM_SFL]

### 1. Structure du SUBFILE
- Colonnes affichées (nom, type DDS, longueur, position) — utiliser A/S/Y/L/T/Z (pas D)
- Taille de page (SFLPAG) : nombre de lignes visibles par page
- Taille totale (SFLSIZ) : préciser la stratégie retenue :
  - **Page-at-a-time** : `SFLSIZ = SFLPAG` — le SUBFILE ne contient qu'une page ; le programme gère lui-même les touches de pagination et recharge la page à chaque navigation
  - **Load-all (expanding)** : `SFLSIZ > SFLPAG` — plusieurs pages sont chargées dans le SUBFILE ; IBM i peut assurer la pagination parmi les enregistrements déjà présents
- Champ RRN : nom du champ SFLRCDNBR dans le SFLCTL — permet de positionner la page affichée sur un enregistrement spécifique (avec l'option CURSOR, positionne aussi le curseur)

### 2. Structure du SFLCTL (enregistrement de contrôle)
- Indicateurs de contrôle requis : SFLDSP (afficher le SFL), SFLDSPCTL (afficher le SFLCTL),
  SFLCLR (effacer le SFL avant chargement), SFLEND (indicateur fin de liste)
- Choisir un pattern cohérent et documenté pour les numéros d'indicateurs — les indicateurs SFLDSP, SFLDSPCTL, SFLCLR et SFLEND peuvent utiliser des numéros distincts selon les besoins. Aucun n'est obligatoirement identique ou inverse d'un autre.
- Touches de fonction associées : CA ou CF selon la règle CAxx/CFxx

### 3. Stratégie de chargement (programme RPG appelant)
- Page-at-a-time : boucle de chargement dans le RPG, un WRITE par enregistrement visible, PF7/PF8 pour la navigation
- Load-all : les enregistrements sont chargés dans le SUBFILE avant l'affichage. Lorsque SFLSIZ est différent de SFLPAG, IBM i peut assurer la pagination parmi les enregistrements chargés. SFLRCDNBR est utilisé uniquement lorsqu'un repositionnement explicite de la page ou du curseur est nécessaire.
- Lecture de la ligne sélectionnée : READC (lit le prochain enregistrement modifié — champ option saisi par l'utilisateur) ; SFLRCDNBR (contrôle la page affichée). Ces deux mécanismes ont des rôles différents et ne sont pas interchangeables.

### 4. Risques et décisions
- Nombre max d'enregistrements prévu : [N] → SFLSIZ à fixer selon la stratégie retenue
- Pagination page par page ou chargement complet ?
- Champ de sélection (option) : présent / absent ?
```

**Analyse :**
- *Stratégie SFLPAG/SFLSIZ :* les deux valeurs sont liées à la stratégie de chargement choisie. `SFLSIZ = SFLPAG` est le pattern habituel du **page-at-a-time** — le sous-fichier ne contient qu'une page, le programme recharge explicitement à chaque navigation. `SFLSIZ > SFLPAG` correspond au **load-all** — plusieurs pages sont chargées, IBM i peut paginer parmi les enregistrements présents. `SFLRCDNBR` est un mécanisme de **positionnement** (quelle page afficher, où placer le curseur) — il ne détermine pas la stratégie de chargement.
- *SFLRCDNBR :* le mot-clé `SFLRCDNBR` dans le SFLCTL contrôle la page affichée et, avec l'option `CURSOR`, la position du curseur. Sans ce mot-clé, IBM i affiche la première page par défaut — ce n'est pas "aléatoire", c'est le comportement attendu sans positionnement explicite.
- *READC vs SFLRCDNBR :* `READC` lit le prochain enregistrement de SUBFILE modifié (champ option saisi) ; `SFLRCDNBR` contrôle la page ou la position du curseur. Ce sont deux mécanismes distincts avec des objectifs différents.

---

**Passe 2 — Génération du DDS SUBFILE (mode Ask) :**

```
La qualification du SUBFILE [NOM_SFL] est disponible.

Génère en français, en markdown, les A-specs et R-specs DDS complètes pour ce SUBFILE :

## DDS SUBFILE — [NOM_SFL]

### SFL (enregistrement de données du SUBFILE)
Format A-specs et R-specs pour le record [NOM_SFL] :
- Mot-clé SFL sur le RECORD
- Un champ par colonne affichée (type A/S/Y/L/T/Z uniquement — pas D)
- Champ option de sélection si nécessaire (type A, longueur 1)

### SFLCTL (enregistrement de contrôle du SUBFILE)
Format A-specs et R-specs pour le record [NOM_SFLCTL] :
- Mot-clé SFLCTL([NOM_SFL])
- SFLPAG([N]) SFLSIZ([M]) — M selon la stratégie qualifiée (M = N pour page-at-a-time, M > N pour load-all/expanding)
- SFLRCDNBR(CURSOR) sur le champ RRN si positionnement du curseur requis
- Indicateurs de contrôle SFLDSP, SFLDSPCTL, SFLCLR, SFLEND : définir un pattern cohérent et documenté ; justifier les numéros d'indicateurs choisis

### Touches de fonction associées (CAxx / CFxx)
Selon la règle CAxx / CFxx définie au Prompt 1a.
```

---

**Passe 3 — Test de compilation CRTDSPF (mode Agent) :**

```
Le DDS SUBFILE [NOM_SFL] / [NOM_SFLCTL] a été ajouté dans [NOM_LIB]/QDDSSRC([NOM_DSPF]).

Compile le display file complet (écran principal + SUBFILE) :
CRTDSPF FILE([NOM_LIB]/[NOM_DSPF]) SRCFILE([NOM_LIB]/QDDSSRC) SRCMBR([NOM_DSPF]) REPLACE(*NO)

Appliquer la même règle que le Prompt 1a-bis pour REPLACE(*NO) / CPF5813 / décision humaine avant REPLACE(*YES).

Si des erreurs concernent le SUBFILE (mots-clés SFL mal ordonnés, SFLSIZ invalide, type D utilisé) :
1. Identifier la ligne source et le mot-clé incriminé
2. Proposer la correction DDS précise
3. Corriger et relancer
```

**Analyse :**
- *Ordre des mots-clés DDS dans un SUBFILE :* les mots-clés SFL, SFLCTL, SFLPAG, SFLSIZ, SFLRCDNBR doivent apparaître dans l'ordre IBM i documenté — une inversion provoque une erreur de compilation.
- *Indicateurs SFLDSP / SFLDSPCTL / SFLCLR :* choisir un pattern cohérent et documenté. Les trois indicateurs ne doivent pas nécessairement être identiques ou inverses l'un de l'autre — IBM fournit des exemples avec des indicateurs distincts selon les besoins.

> ⚠️ **Après la Passe 3 réussie : passer au Prompt 2a** — la logique de chargement du SUBFILE (WRITE en boucle, activation SFLDSP, READC pour la sélection) doit être incluse dans le Prompt 2a.

---



## — SOUS-CAS B/C : Conversion 5250 → Web (et Web UI sur back-end existant) —

### Prompt 1b — Analyse des écrans 5250 existants

> **À exécuter après le Prompt 0 (catégorie confirmée). Le display file DDS source doit être ouvert dans l'éditeur.**

```
Le display file DDS [NOM_DSPF] se trouve dans [NOM_LIB]/QDDSSRC.
[Si disponible : Le fichier de compréhension {projet}-{lib}-{programme}-comprehension-{date}.md est disponible.]

Analyse ce display file via IBM i MCP et produis en français, en markdown, l'inventaire complet des écrans :

## Analyse des écrans 5250 — [NOM_DSPF]

### Tableau des enregistrements (RECORD)
| Nom RECORD | Nb champs | Type (écran / SUBFILE / SFLCTL) | Validations DDS | SUBFILE (O/N) | Touches PF utilisées |
|------------|-----------|--------------------------------|----------------|--------------|---------------------|
| [une ligne par RECORD] |

### Points d'attention par écran
Pour chaque RECORD complexe : décrire les indicateurs de condition, les DSPATR conditionnels, les champs affichés/masqués selon les indicateurs RPG.

### Recommandation de séquence de génération Web
Ordre conseillé : écrans sans SUBFILE en premier, SUBFILE en dernier.
```

**Analyse ligne à ligne :**
- *Pourquoi obtenir ce tableau avant toute génération Web :* ce tableau est la base de chaque composant à créer — sans inventaire, Bob génère un composant pour le premier RECORD qu'il voit, en oubliant les enregistrements de message d'erreur, les enregistrements cachés ou les SFLCTL associés
- *Les indicateurs de condition (DSPATR conditionnels) :* un champ affiché uniquement quand l'indicateur 30 est activé (`30 DSPATR(ND)`) pilote son affichage dans le composant Web via une propriété booléenne — Bob doit documenter ces conditions pour que le développeur Front-End les implémente correctement

### Prompt 2b — Analyse et conversion d'un SUBFILE

> **Ce prompt est obligatoire si un SUBFILE est présent. Il s'exécute en deux passes distinctes.**
> **Ne jamais convertir un SUBFILE en une seule passe** — la logique de chargement est partiellement dans le programme RPG, pas dans le DDS. Une conversion en passe unique produit un composant Web qui affiche les colonnes mais ne gère pas la pagination ni la sélection de ligne.

**Passe 1 — Analyse du SUBFILE (mode Ask) :**

```
Le display file [NOM_DSPF] contient un SUBFILE [NOM_SFL] avec son SFLCTL [NOM_SFLCTL].
[Si disponible : Le programme RPG [NOM_PROGRAMME] pilote ce SUBFILE — le source est ouvert dans l'éditeur.]

Analyse la structure de ce SUBFILE et produis en français, en markdown :

## Analyse SUBFILE — [NOM_SFL]

### 1. Structure DDS du SUBFILE
Colonnes (champs dans le SFL), taille de page (SFLPAG), taille totale (SFLSIZ).

### 2. Logique de chargement
- Chargement page par page ou complet (SFLRCDNBR vs boucle de chargement dans le programme RPG) ?
- Indicateurs de contrôle : SFLDSP, SFLDSPCTL, SFLCLR, SFLEND — quand sont-ils activés ?
- Touches PF de navigation dans le SUBFILE (PF7/PF8 — page précédente/suivante, etc.)

### 3. Logique de sélection de ligne
Option de sélection (champ option dans le SFL) — que se passe-t-il quand l'utilisateur saisit '1', '2', '4' dans le champ option ?

### 4. Partie de la logique dans le programme RPG (pas dans le DDS)
Identifier les sections du programme RPG qui chargent, effacent ou naviguent dans le SUBFILE — ces sections doivent être documentées avant la génération Web.
```

**Passe 2 — Génération du composant équivalent (mode Ask) :**

```
L'analyse du SUBFILE [NOM_SFL] est disponible dans [NOM_FICHIER].
Framework Web retenu par ACME : [React / Vue / Angular].

Génère en français, en markdown, le composant Web équivalent au SUBFILE :

## Composant Web — [NOM_SFL]

### 1. Structure du composant
Props, state, interface de données (DTO équivalent aux colonnes du SFL).

### 2. Pagination
Logique de chargement paginé — appel API back-end avec paramètres de page, affichage de l'indicateur "fin de liste" (équivalent SFLEND).

### 3. Sélection de ligne
Gestion des actions par ligne (équivalent du champ option 1/2/4) — boutons d'action ou menu contextuel.

### 4. Intégration avec le back-end RPG
Interface d'appel du programme RPG (paramètres IN/OUT) — [Sous-cas C : rappel que le programme RPG back-end ne doit PAS être modifié ; l'API d'appel doit préserver l'interface existante].
```

**Analyse ligne à ligne :**
- *Pourquoi les SUBFILE ne peuvent jamais être convertis en une seule passe :* le DDS définit la **structure** du SUBFILE (colonnes, taille de page), mais la **logique de chargement** (combien d'enregistrements charger, quand effacer, comment gérer la fin de fichier) est dans le programme RPG. Bob ne peut pas générer un composant Web fonctionnel sans avoir lu le programme RPG — la Passe 1 est donc indispensable
- *Logique de pagination :* `SFLPAG` définit le nombre de lignes affichées par page dans l'écran 5250 — l'équivalent Web est une pagination côté serveur (la requête API retourne une page d'enregistrements). Ne pas faire de la pagination côté client sur tous les enregistrements si le SUBFILE d'origine était chargé page par page dans le programme RPG


> ⚠️ **Avant de lancer le Prompt 3b : définir le niveau de prototype**
>
> | Type de prototype | Exigences de sécurité pour ce POC |
> |---|---|
> | **Données mockées, aucune API IBM i réelle** | Sécurité détaillée différable — aucun secret ni donnée de production ne transite. Mentionner ce choix explicitement dans le fichier `*-analyse-ecrans-*.md`. |
> | **API IBM i réelle** | Authentification, autorisation côté serveur, validation des données reçues par l'API, gestion des secrets (pas de credentials en dur dans le composant), profil IBM i applicatif dédié — **obligatoires avant tout test avec des données réelles**. Les validations JavaScript côté client ne constituent pas des contrôles de sécurité. |
>
> Si le prototype passe de mock à API réelle en cours de session, appliquer les exigences de la ligne API réelle avant de continuer.


### Prompt 3b — Génération d'un composant Web par écran 5250

> **Un prompt par écran — ne pas regrouper plusieurs écrans dans le même prompt sauf s'ils partagent le même composant de base.**

```
L'analyse des écrans 5250 est disponible dans [NOM_FICHIER].
[Si SUBFILE : Le composant SUBFILE [NOM_SFL] a été généré en Prompt 2b.]
Framework Web retenu par ACME : [React / Vue / Angular].
[Sous-cas C : Rappel — le programme RPG back-end [NOM_PROGRAMME] ne doit PAS être modifié. Préserver l'interface d'appel existante (paramètres, structure de données échangées).]

Génère en français, en markdown, le composant Web pour l'écran [NOM_RECORD] :

## Composant Web — [NOM_RECORD]

### 1. Structure du composant
Props, state, events — structure [React/Vue/Angular].

### 2. Formulaire et champs
Un champ par champ DDS : type HTML équivalent, label (libellé DDS), contraintes de validation côté client.

### 3. Validations côté client
Traduction des validations DDS :
- CHECK(ER) → required
- VALUES('A' 'B' 'C') → select ou radio avec les valeurs autorisées
- RANGE(1 999) → min/max sur input number

### 4. Gestion des touches de fonction
Mapping PF → boutons ou raccourcis clavier (ex. PF3 = "Quitter" → bouton "Annuler" ou touche Escape).

### 5. Appel du back-end
Couche d'exposition à préciser obligatoirement (un composant navigateur ne peut pas appeler directement un `*PGM` IBM i) :
- IWS (IBM i Web Services) — service REST ou SOAP exposé depuis IBM i
- Adaptateur Node.js / Java côté serveur appelant le programme via IBM i Toolkit
- CGI IBM i
- API existante (documenter l'URL et les paramètres d'appel)
[Sous-cas C : rappel que le programme RPG back-end ne doit PAS être modifié. Définir ici le contrat d'API : URL, méthode HTTP, sérialisation des paramètres IN/OUT, authentification, gestion de session — à valider avec l'équipe ACME avant de générer le composant.]

Note : Bob génère la structure et la logique du composant — la mise en forme CSS et le design final sont à la charge de l'équipe Front-End.
```

**Analyse ligne à ligne :**
- *Quel framework choisir et pourquoi demander à Bob de respecter le choix ACME :* Bob peut générer dans React, Vue ou Angular — mais il faut lui préciser le choix de l'équipe dans le prompt, sinon il choisit par défaut (souvent React). Un composant Vue envoyé à une équipe Angular produit du travail de conversion inutile
- *Comment les validations DDS se traduisent en validation côté client :* les validations DDS sont exécutées côté IBM i (le programme RPG est notifié par le display file) — leur équivalent Web est côté client (JavaScript). La sémantique est différente : une validation `VALUES` en DDS bloque la touche Entrée si la valeur n'est pas dans la liste ; en Web, la validation est déclenchée à la soumission du formulaire (ou onChange selon l'implémentation)
- *Limites de Bob sur le CSS :* Bob génère des composants avec des classes CSS génériques (ex. `className="form-field"`) — il ne génère pas de CSS personnalisé conforme à la charte graphique ACME. Le design final est toujours à la charge de l'équipe Front-End

### Prompt 4 — Plan de conversion des écrans (périmètre applicatif complet)

> **Ce prompt est en mode Ask. La sauvegarde se fait en mode Agent.**

```
L'analyse des écrans 5250 est disponible dans [NOM_FICHIER].
Les composants Web pilotes générés jusqu'ici sont : [liste des composants/écrans traités].

Produis en français, en markdown, le plan de conversion des écrans pour l'application [NOM_APP] :

## Plan de conversion des écrans — [NOM_APP]

| Écran (RECORD) | Sous-cas | Catégorie | SUBFILE (O/N) | Réflexion UX requise (O/N) | Effort estimé | Priorité |
|----------------|----------|-----------|--------------|--------------------------|---------------|----------|
| [une ligne par écran] |

### Points de synchronisation
Écrans interdépendants (navigation l'un vers l'autre) — à traiter dans la même session Bob.

### Écrans nécessitant un atelier UX préalable
[Liste des écrans COMPLEXE dont la conversion ne doit pas démarrer sans maquettes ou wireframes]
```

**Analyse ligne à ligne :**
- *Pourquoi une colonne "Réflexion UX préalable requise" :* un écran COMPLEXE avec SUBFILE ou navigation riche 5250 ne peut pas être converti directement en Web sans décision UX — le 5250 et le Web ont des paradigmes d'interaction fondamentalement différents (touche Entrée vs événements, navigation séquentielle vs navigation libre). Cette colonne signale à l'équipe quel écran nécessite un atelier UX avant de démarrer le Prompt 1b
- *Ce plan est l'input obligatoire de UC 13 :* les tests fonctionnels des interfaces générées (UC 13) s'appuient sur la liste priorisée des écrans — sans ce plan, l'équipe de test ne sait pas quels composants valider en priorité

---

## Add-ons Bob

| Extension | Utilité dans UC 10 |
|-----------|-------------------|
| **Code for IBM i** | DDS Previewer — rendu visuel du display file généré (sous-cas A) sans passer par un terminal 5250 ; Object Browser pour accéder aux sources DDS et RPG |
| **IBM i Languages** | Coloration syntaxique DDS et RPG — indispensable pour valider le source DDS généré dans l'éditeur avant compilation |
| **Markdown All in One** | Prévisualisation des analyses d'écrans et des plans de conversion sauvegardés |

> 💡 Pour les sous-cas B/C, les extensions de framework Web (ESLint, Prettier, React/Vue/Angular snippets) sont hors périmètre Bob — elles s'installent dans l'environnement de développement Front-End de l'équipe, pas dans Bob.

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture du display file DDS source via `QDDSSRC` (sous-cas B/C) ; lecture du programme RPG de navigation pour l'analyse SUBFILE ; écriture et compilation du display file généré via `CRTDSPF` (sous-cas A) |
| **IBM i Database MCP** | Si le composant Web interroge des vues SQL ou des procédures stockées générées en UC 11 — introspection `QSYS2.SYSROUTINES` pour valider les interfaces d'appel |
| **Confluence MCP** *(si disponible)* | Publication du plan de conversion des écrans (Prompt 4) dans l'espace POC ACME |

> ⚠️ **ARCAD MCP : NON DISPONIBLE dans ce POC.** Voir la section "Spécificité ARCAD" ci-dessus.

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| **Sauter le Prompt 0 sur un COMPLEXE avec SUBFILE** | Bob attaque le Prompt 1b directement et génère un composant Web incomplet — la logique de chargement SUBFILE et les indicateurs de navigation ne sont pas documentés, le composant généré ne fonctionne pas | Le Prompt 0 prend 10 minutes et évite 4 heures de génération dans le mauvais sens. Toujours exécuter le Prompt 0 pour chaque écran |
| **Envoyer toute l'application au Prompt 0 sans définir le périmètre** | Saturation de contexte — Bob produit un inventaire générique de tous les écrans sans analyser la logique de navigation. Les Bob Coins sont consommés sans valeur | Limiter le Prompt 0 à 1 écran (ou groupe d'écrans liés) à la fois ; utiliser le Prompt 4 pour établir le plan de conversion du périmètre complet |
| **Convertir un SUBFILE en une seule passe** | Le composant Web généré affiche les colonnes mais ne gère pas la pagination (logique de chargement dans le programme RPG non lue) ni la sélection de ligne | Toujours appliquer le Prompt 2b en deux passes : Passe 1 (analyse logique SUBFILE + programme RPG) obligatoire avant Passe 2 (génération composant) |
| **Sous-cas C : Bob propose de modifier le programme RPG back-end** | Bob peut suggérer d'ajuster le programme RPG pour simplifier l'interface — violation de la contrainte ; le programme en production ne sera pas le même que celui analysé, risque de régression | Vérifier après chaque prompt que le programme RPG `[NOM_PROGRAMME]` n'apparaît pas dans les modifications proposées. Rappeler dans **chaque** prompt 2b et 3b : "le programme RPG `[NOM_PROGRAMME]` ne doit PAS être modifié" |
| **Sous-cas C : demander à Bob de modifier le programme RPG back-end (action directe)** | Violation de la contrainte sous-cas C — le programme RPG produit en production ne sera pas le même que celui analysé ; risque de régression fonctionnelle | Rappeler explicitement dans chaque prompt B/C la contrainte "le programme RPG [NOM_PROGRAMME] ne doit PAS être modifié" ; vérifier que les paramètres d'appel dans le composant Web généré correspondent à l'interface existante |
| **Valider le rendu visuel uniquement via la sortie texte de Bob** | Un display file DDS valide en texte peut avoir des positions de champs incorrectes (chevauchements) ou des libellés tronqués — non détectables sans rendu visuel | Toujours tester le display file généré (sous-cas A) sur un terminal 5250 réel ou via le DDS Previewer de Code for IBM i après le Prompt 1a-bis |
| **Enchaîner la génération de 5 écrans d'un coup** | Consommation élevée de Bob Coins, contexte surchargé, qualité dégradée sur les derniers écrans de la session | 1 écran pilote → validation → puis les suivants. Exception : écrans fonctionnellement liés avec structures de données partagées peuvent être enchaînés dans la même session |
| **Démarrer sous-cas B/C sans avoir choisi le framework Web** | Bob génère par défaut en React — si l'équipe Front-End travaille en Vue ou Angular, tout le code généré est à réécrire | Décider et documenter le framework (React / Vue / Angular) dans le fichier `*-analyse-ecrans-*.md` au Prompt 0 ; rappeler le choix dans chaque prompt de génération Web |

---

## Check-list de validation UC 10

Avant de passer à UC 13 (voir `UC13-tests.md`), valider chaque point :

- [ ] **Pour chaque écran : le Prompt 0 a été exécuté** — le sous-cas (A/B/C) et la catégorie (SIMPLE/STANDARD/COMPLEXE) sont documentés dans le fichier `{appArcad}-{fonction}-{composant}-analyse-ecrans-{date}.md` (produit par UC 10 — à distinguer du fichier `*-comprehension-*.md` produit par UC 4 qui est consommé en entrée)
- [ ] **[Sous-cas A]** Le source DDS validé a été **écrit dans `[LIBSRC]/[SRCFILE_DDS]([MEMBRE_DDS])`** (après vérification via `QSYS2.SYSMEMBERSTAT` qu'il n'existait pas) avant CRTDSPF
- [ ] **[Sous-cas A]** Le display file DDS généré (`*-dspf-genere-*.md`) **compile sans erreur** (CRTDSPF — Prompt 1a-bis) et a été **testé manuellement** sur un terminal 5250 réel ou via le DDS Previewer de Code for IBM i
- [ ] **[Sous-cas A]** Le source RPG validé a été **écrit dans `[LIBSRC]/[SRCFILE_RPG]([MEMBRE_RPG])`** (après vérification via `QSYS2.SYSMEMBERSTAT` qu'il n'existait pas) avant CRTBNDRPG
- [ ] **[Sous-cas A]** Le programme RPG de gestion d'écran (Prompt 2a) a été **compilé sans erreur (`CRTBNDRPG` — Prompt 2a-bis)** et testé fonctionnellement sur l'IBM i de test — ordre obligatoire : CRTDSPF (Prompt 1a-bis) avant CRTBNDRPG (Prompt 2a-bis)
- [ ] **[Sous-cas C]** La conclusion d'éligibilité est documentée dans le fichier `*-analyse-ecrans-*.md` (ÉLIGIBLE / NON ÉLIGIBLE) et la couche d'exposition Web → IBM i est définie avant toute génération de composant
- [ ] **[Sous-cas B/C]** Chaque SUBFILE a été traité en **deux passes distinctes** (Prompt 2b Passe 1 : analyse + Passe 2 : génération) — ne pas livrer un composant Web généré en une passe sur un SUBFILE
- [ ] **[Sous-cas C]** Confirmé **par écrit** (dans le fichier `*-analyse-ecrans-*.md`) que le programme RPG back-end n'a pas été modifié — les paramètres d'appel dans le composant Web correspondent à l'interface RPG existante
- [ ] **[Sous-cas B/C]** Le framework Web retenu (React / Vue / Angular) est documenté dans le fichier `*-analyse-ecrans-*.md` et a été rappelé dans chaque prompt de génération
- [ ] **[Sous-cas B/C]** Les validations DDS (CHECK, VALUES, RANGE) ont été traduites en validations côté client dans chaque composant Web généré — revue par un développeur Front-End
- [ ] Le plan de conversion des écrans (Prompt 4, type `plan-ecrans`) est **produit et validé avec l'équipe ACME** — c'est l'**input obligatoire de UC 13** pour les tests fonctionnels des interfaces générées
- [ ] Les sources générés (DDS et Web) sont sauvegardés avec la convention de nommage correcte (`*-analyse-ecrans-*`, `*-dspf-genere-*`, `*-web-genere-*`, `*-plan-ecrans-*`) et publiés sur Confluence (si MCP disponible)
- [ ] Les sources DDS générés (sous-cas A) sont réintégrés dans ARCAD manuellement après validation (cf. contrainte ARCAD, `plan-poc-bob-acme.md` §5)

---

## Points à compléter avant passage en production

> Ces points ne bloquent pas le POC — ils concernent des cas avancés à traiter avant d'industrialiser la génération d'interfaces sur l'ensemble du parc en production.

### Écrans COMPLEXE avec navigation dynamique

**Contexte :** certains écrans 5250 affichent ou masquent des champs selon des indicateurs RPG pilotés dynamiquement par la logique du programme (ex. champ "Motif de refus" visible uniquement si l'indicateur 45 est activé par une condition métier). La cartographie complète de ces indicateurs conditionnels n'est pas dans le DDS — elle est dans le programme RPG.

**Pourquoi absent de la fiche POC :** nécessite une analyse poussée de la logique des indicateurs RPG d'affichage conditionnel — va au-delà du périmètre d'un POC.

**À faire avant production :** cartographier tous les indicateurs DDS d'affichage conditionnel (`DSPATR` et `COLOR` conditionnels) avant de générer le composant Web ; les traduire en propriétés réactives dans le composant (ex. `v-if` en Vue, conditional rendering en React) avec les conditions métier correspondantes.

### Accessibilité et conformité RGAA / WCAG

**Contexte :** les composants Web générés par Bob ne sont pas conformes RGAA (Référentiel Général d'Amélioration de l'Accessibilité) ni WCAG 2.1 par défaut — les labels HTML peuvent être absents, les contrastes non vérifiés, la navigation au clavier non optimisée.

**À faire avant production :** revue accessibilité par l'équipe Front-End après chaque génération ; vérifier a minima : attributs `aria-label` sur les champs sans label visible, contrastes de couleur (ratio ≥ 4,5:1), navigation au clavier (Tab order, focus visible).

### Internationalisation et support multilingue

**Contexte :** les libellés des champs DDS sont souvent écrits en dur dans le source (`TEXT('Numéro de commande')`) en français fixe. Les composants Web générés reprennent ces libellés directement — pas de gestion i18n.

**À faire avant production :** externaliser les libellés dans des fichiers de traduction (ex. `fr.json`, `en.json`) lors de la génération Web ; éviter les libellés hardcodés dans les templates de composants. Définir avec l'équipe ACME si le support multilingue est requis avant de démarrer la génération en masse.
