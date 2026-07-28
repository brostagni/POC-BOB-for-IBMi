# UC 1 — Conversion COBOL → FREE RPG ILE

> **Catégorie :** Modernisation du code
>
> **Priorité dans le POC :** 10a — Track A Phase 4, parallélisable avec UC 2 sur des programmes différents
>
> **Durée POC (avec Bob) :** 3 à 6 heures — conversion d'un périmètre représentatif (2 à 3 programmes COBOL), itérations et tests de non-régression
>
> **Durée PROD (avec Bob) :** 1 à 4 heures / programme — qualification, génération du diff de conversion, revue expert COBOL + RPG, test fonctionnel
>
> **Durée PROD (sans Bob) :** 3 à 10 jours / programme — lecture complète du COBOL, mapping manuel paragraphe par paragraphe, réécriture RPG, campagne de tests de non-régression ; plus long pour les programmes avec WORKING-STORAGE complexe et niveaux 01-88
>
> **Gain Bob estimé :** ~5× — un programme COBOL de 600 lignes converti en une journée au lieu d'une semaine ; gain plus fort sur les programmes avec des WORKING-STORAGE denses et des COMPUTE/MOVE répétitifs (Bob génère les équivalences mécaniques rapidement)
>
> **Mode Bob recommandé :** IBM i Developer (mode Ask pour la qualification et la génération du diff, Agent pour la compilation de test et la sauvegarde)

---

## Objectif

Convertir un programme **COBOL IBM i** en **RPG ILE Free**, en préservant exactement la logique fonctionnelle et les interfaces d'appel existantes.

**Ce UC est le plus risqué de la Phase 4 — Track A.** COBOL et RPG ILE Free sont deux langages différents avec des modèles de données, des structures de contrôle et des mécanismes d'accès aux fichiers fondamentalement distincts. La conversion n'est pas mécanique : certains constructs COBOL n'ont pas d'équivalent direct en RPG (PERFORM THRU, ALTER, REDEFINES multi-niveaux) et nécessitent une décision de conception.

**Ce UC diffère de UC 2 (RPG colonné → Free) :**
- UC 2 change la **syntaxe** d'un même langage — les opcodes, les specs, les colonnes. La logique et les structures de données restent équivalentes ligne à ligne.
- UC 1 change de **langage** — le modèle mémoire COBOL (WORKING-STORAGE, FILE SECTION, niveaux 01-88) doit être transposé en structures RPG (DCL-DS, DCL-S, DCL-F). Il n'existe pas de bijection systématique : chaque PERFORM THRU ou REDEFINES de niveau complexe nécessite une décision de l'équipe.

**Validation obligatoire par un expert COBOL + RPG.** La conversion produite par Bob est un point de départ documenté — elle ne peut pas être livrée sans revue experte sur chaque zone marquée "À CONFIRMER" ou "décision requise".

**Livrable attendu :** pour chaque programme converti, une analyse de conversion documentant les choix de transposition, un diff (ou source complet) RPG ILE Free, et un plan de conversion pour le périmètre applicatif.

**Convention de nommage des fichiers générés :**
```
{projet}-{lib}-{programme}-{type}-{YYYYMMDD-HHmm}.md

Types pour cet UC :
  analyse-cobol   → analyse de conversion COBOL (Prompt 0) — catégorie + zones à risque
  cobol-converti  → diff de conversion ou source RPG Free (Prompts 1 à 4)
  plan-cobol      → plan de conversion pour un périmètre applicatif (Prompt 5)
```

Exemples :
```
acme-APPVTE-COBPGM1-analyse-cobol-20250622-0900.md   ← analyse + stratégie
acme-APPVTE-COBPGM1-cobol-converti-20250622-1100.md  ← diff ou source RPG Free
acme-APPVTE-APPVTE-plan-cobol-20250622-0800.md       ← plan périmètre complet
```

> 💡 Cette convention est valable en dehors du contexte POC — réutilisable en production tel quel.

---

## Démarrer par un programme que vous connaissez

> **Recommandation forte avant d'aborder les programmes COBOL critiques de ACME.**

Commencer par le **programme COBOL dont la logique est la mieux documentée** — idéalement un programme avec une compréhension UC 4 disponible, dont le comportement peut être vérifié rapidement avec un jeu de tests existant et dont un développeur connaît à la fois le COBOL d'origine et le RPG cible.

Pourquoi ? Parce que la première conversion sert à **calibrer trois choses** :
- La qualité des transpositions proposées par Bob (est-ce que `PERFORM CALCULER-MONTANT` a bien été converti en `EXSR srCalculerMontant` ou `CALLP calculerMontant(params)` selon la complexité ?)
- La pertinence des décisions de conception sur les structures COBOL sans équivalent direct (REDEFINES, niveaux 88, OCCURS DEPENDING ON)
- La capacité de l'équipe à valider la non-régression avec un expert connaissant les deux langages

> 💡 Pour chaque programme, **commencer par le Prompt 0** — il donne la catégorie (SIMPLE / STANDARD / COMPLEXE) et la séquence exacte. Ne pas aller directement au Prompt 1.

| Étape | Programme à choisir | Catégorie attendue | Objectif |
|-------|--------------------|--------------------|---------|
| 1 | Programme COBOL court (< 100 paragraphes), PROCEDURE DIVISION linéaire, pas de PERFORM THRU, pas de REDEFINES complexe | SIMPLE | Calibrer les équivalences de base COBOL → RPG ; valider que le Prompt 0 classe correctement |
| 2 | Programme avec WORKING-STORAGE dense (structures de niveau 01-05) et PERFORM vers des paragraphes multiples | STANDARD | Valider la transposition des structures de données et la gestion des PERFORM imbriqués |
| 3 | Programme avec REDEFINES multi-niveaux, niveaux 88 (conditions booléennes) et CALL vers des programmes IBM i | STANDARD à COMPLEXE | Valider les décisions de conception sur les REDEFINES et les interfaces CALL |
| 4 | Programme COBOL critique avec PERFORM THRU, ALTER, ou accès aux fichiers par FILE SECTION / FD | COMPLEXE | Valider la gestion des constructs sans équivalent RPG direct et le périmètre de conversion acceptable pour le POC |

---

## Impact de la taille du programme sur la stratégie de conversion

La complexité d'UC 1 ne se mesure pas en nombre de lignes, mais en **nombre de paragraphes COBOL dans la PROCEDURE DIVISION**, au **niveau de complexité des structures WORKING-STORAGE** (présence de REDEFINES, niveaux 88, OCCURS DEPENDING ON), et à la **nature des constructs COBOL sans équivalent RPG direct**.

Les vrais facteurs qui compliquent la conversion :
- **PERFORM THRU** : `PERFORM PARA-A THRU PARA-Z` exécute une séquence de paragraphes consécutifs comme une unité — il n'existe pas d'équivalent direct en RPG. La conversion nécessite soit de regrouper les paragraphes dans une seule subroutine EXSR, soit de les appeler séquentiellement avec plusieurs EXSR/CALLP
- **REDEFINES multi-niveaux** : une zone WORKING-STORAGE qui se redéfinit en plusieurs formats (ex. une zone de 10 octets vue tantôt comme un numérique, tantôt comme 3 sous-champs alphanumériques) se transmet en RPG via une `DCL-DS` avec `OVERLAY` — les niveaux de REDEFINES imbriqués peuvent être très complexes
- **Niveaux 88 COBOL** (valeurs de condition) : `88 CLIENT-ACTIF VALUE 'A'` définit une condition booléenne nommée que `IF CLIENT-ACTIF` teste directement. En RPG, il n'y a pas de niveau 88 — la conversion produit soit des constantes (`DCL-C C_CLIENT_ACTIF 'A'`), soit des variables `IND` peuplées par comparaison
- **ALTER GO TO** : `ALTER PARA-X TO PROCEED TO PARA-Y` modifie dynamiquement la destination d'un `GO TO` au moment de l'exécution — construct obsolète même en COBOL, sans équivalent RPG. Sa présence classe automatiquement le programme en COMPLEXE
- **WORKING-STORAGE SECTION avec OCCURS DEPENDING ON** : des tableaux à taille variable dépendant d'une variable runtime n'ont pas d'équivalent direct en RPG (les tableaux RPG ont une taille fixe ou utilisent des listes chaînées) — la conversion nécessite une décision de conception
- **FILE SECTION / FD** : les descriptions de fichiers COBOL (FD avec RECORD CONTAINS) se transposent en F-specs RPG, mais les structures d'enregistrement `01 NOM-ENREG` dans la FILE SECTION deviennent des DS externées (`DCL-DS LIKEREC`) ou des DS manuelles selon la structure DDS

### Programmes SIMPLE — PROCEDURE DIVISION linéaire, pas de PERFORM THRU ni de REDEFINES complexe

Bob gère sans difficulté. **Séquence : Prompt 0 → Prompt 1 → Prompt 2 (WORKING-STORAGE → DCL) → Prompt 3 (PROCEDURE DIVISION → sous-routines) → Prompt 3-bis (compilation) → Prompt 4 (interfaces et nettoyage).**

> 💡 Le Prompt 0 confirme la catégorie SIMPLE et recommande directement cette séquence — pas de décision manuelle requise.

### Programmes STANDARD — WORKING-STORAGE complexe, PERFORM imbriqués, niveaux 88, CALL vers d'autres programmes

Le Prompt 0 identifie les zones de REDEFINES, les niveaux 88 à transposer et les CALL à analyser. **Séquence : Prompt 0 → Prompt 1 → Prompt 2 (WORKING-STORAGE avec décisions de conception) → Prompt 3 → Prompt 3-bis → Prompt 4 (niveaux 88 + interfaces CALL).**

> 💡 Les décisions de conception sur les niveaux 88 et les REDEFINES doivent être validées par un expert avant de générer le code RPG. Une mauvaise transposition d'un REDEFINES produit un accès mémoire incorrect — difficile à détecter car le programme compile sans erreur.

### Programmes COMPLEXE — PERFORM THRU, ALTER GO TO, OCCURS DEPENDING ON, ou FILE SECTION dense

Un programme COMPLEXE en UC 1 requiert souvent une limitation de périmètre pour le POC. Le Prompt 0 pose explicitement la question : **quels paragraphes peuvent être convertis en toute sécurité dans le cadre du POC ?** La logique non convertible peut être préalablement extraite dans un **programme COBOL autonome**, puis appelée depuis RPG via `CALLP` (architecture hybride temporaire — il n'est pas possible d'appeler un paragraphe COBOL interne depuis RPG).

**Séquence : Prompt 0 (périmètre délimité) → Prompt 1 → Prompt 2 → Prompt 3 (paragraphes dans le périmètre) → Prompt 3-bis → Prompt 4 → architecture hybride si certains paragraphes restent en COBOL.**

> ⚠️ Sur un programme COMPLEXE, tenter la conversion complète en une seule session produit un source RPG avec des zones de logique incorrectes ou incomplètes. Le Prompt 0 délimite toujours le périmètre convertible en toute sécurité.

---

## Démarrer une session Bob

> **À lire avant chaque session UC 1 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

Chaque session de travail sur un programme COBOL doit démarrer dans une **nouvelle conversation Bob** (bouton `+` en haut du panneau Chat). Ne pas réutiliser une conversation d'un autre programme — les structures de données et les paragraphes COBOL d'un autre programme polluent les décisions de transposition du programme courant.

**Mode à sélectionner :** `IBM i Developer`

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Avant de lancer le Prompt 0, ouvrir dans l'éditeur Bob le programme COBOL source. L'ouverture dans l'éditeur le rend accessible au MCP IBM i sans copier-coller.

**Procédure :** dans le panneau **IBM i — Object Browser** (extension Code for IBM i), naviguer jusqu'à la bibliothèque source COBOL (`QCBLSRC` ou `QLBLSRC` selon la convention ACME), faire un clic droit sur le membre → **Open in Editor**.

Fichiers à ouvrir pour chaque session UC 1 :
- Le programme COBOL source (`[NOM_LIB]/QCBLSRC([NOM_PROGRAMME])`)
- Le fichier de compréhension du programme (`*-comprehension-*.md`) depuis UC 4
- Le fichier de spécification technique (`*-spec-tech-*.md`) depuis UC 6 — interfaces et fichiers accédés
- Si une session précédente existe : le fichier d'analyse de conversion (`*-analyse-cobol-*.md`)

### 3. Fichiers de contexte à charger

Ces fichiers produits par les UC précédents doivent être disponibles dans le workspace Bob **avant** de démarrer. Utiliser **Add File to Chat** (icône trombone) ou les ouvrir dans l'éditeur.

| Fichier | Produit par | Obligatoire / Recommandé |
|---------|-------------|--------------------------|
| `{projet}-{lib}-{programme}-comprehension-{date}.md` | UC 4 | **Obligatoire** — carte des paragraphes COBOL et des données, base du Prompt 1 |
| `{projet}-{lib}-{programme}-spec-tech-{date}.md` | UC 6 | **Obligatoire** — interfaces et fichiers accédés ; les CALL depuis COBOL doivent correspondre aux programmes RPG récepteurs |
| `{projet}-{lib}-{lib}-matrice-{date}.md` | UC 6 | **Obligatoire** — tous les programmes appelants du programme COBOL ; modifier l'interface après conversion casserait ces appelants |
| `{projet}-{lib}-{programme}-analyse-cobol-{date}.md` | UC 1 (session précédente) | **Si reprise** — catégorie, périmètre et décisions de conception déjà prises ; évite de relancer le Prompt 0 |

> ⚠️ **Prérequis critique :** ne jamais démarrer UC 1 sans les fichiers `*-comprehension-*.md` et `*-spec-tech-*.md`. Convertir un programme COBOL sans carte de ses paragraphes et sans connaissance de ses interfaces, c'est risquer de casser les appelants et de mal transposer les structures de données partagées.

> ⚠️ **Risque de réduction de contexte — règle de sauvegarde intermédiaire obligatoire :** UC 1 est l'UC le plus long du POC en termes de volume de texte généré dans une même conversation (analyse + inventaire + transposition DATA + transposition PROCEDURE). Au-delà de 4 à 5 prompts consécutifs, Bob commence à compresser les premiers tours de la conversation — les décisions prises au Prompt 0 (périmètre, REDEFINES "À CONFIRMER", niveaux 88 à transposer) peuvent disparaître de la fenêtre active. **Sauvegarder en mode Agent après chaque prompt majeur** (Prompt 0, Prompt 1, Prompt 2, Prompt 3) — pas seulement en fin de session. À chaque reprise de passe, commencer le prompt par : "L'analyse de conversion est dans [NOM_FICHIER] — les décisions prises sont : [résumer les 3 à 5 décisions clés du Prompt 0]."

> 💡 **Reprise de session :** si la conversion est interrompue, ouvrir le fichier `*-analyse-cobol-*.md` — Bob retrouve le périmètre délimité et les décisions de conception prises sans relancer le Prompt 0. Indiquer "l'analyse de conversion est dans [NOM_FICHIER] — reprendre à partir du Prompt [N°]".

> 💡 **Lien vers l'UC suivant :** les fichiers `*-cobol-converti-*.md` produits dans cet UC sont les inputs de UC 13 (tests de non-régression). Voir la Carte des livrables dans `plan-poc-bob-acme.md`.

---

## Prérequis

- Les fichiers `*-comprehension-*.md` (UC 4) des programmes COBOL à convertir sont présents : ils fournissent la carte des divisions (IDENTIFICATION, ENVIRONMENT, DATA, PROCEDURE), des paragraphes et des données — indispensable pour le Prompt 1
- Les fichiers `*-spec-tech-*.md` (UC 6) sont disponibles : ils documentent les interfaces d'appel (paramètres CALL), les fichiers accédés (FD / SELECT), et les programmes appelants — toute modification d'interface doit être tracée
- Les fichiers `*-matrice-*.md` (UC 6) sont disponibles : ils identifient tous les programmes appelants du programme COBOL — indispensable avant toute modification de l'interface d'appel après conversion
- IBM i MCP actif (lecture du source COBOL depuis `QCBLSRC`, compilation du source RPG converti)
- Accès à l'IBM i de test pour compiler et tester le programme RPG converti
- Un expert maîtrisant à la fois COBOL IBM i et RPG ILE Free disponible pour valider les décisions de conception (REDEFINES, niveaux 88, PERFORM THRU) et la non-régression fonctionnelle

> ⚠️ **Prérequis critique :** la validation par un expert COBOL + RPG est non négociable sur UC 1. Bob peut produire les équivalences mécaniques et identifier les zones à décision, mais la validation sémantique des REDEFINES, des niveaux 88 et des PERFORM THRU appartient à un humain expert dans les deux langages.

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | IBM i Developer — mode **Ask** pour la qualification et la génération du diff, **Agent** pour la compilation de test et la sauvegarde |
| **Scope** | Library List → bibliothèque applicative ACME |
| **MCP actifs** | IBM i MCP (lecture du source COBOL depuis `QCBLSRC`, écriture et compilation du source RPG dans `QRPGSRC`) |
| **MCP différés** | IBM i Database MCP (si des accès SQL embarqués COBOL sont présents — `EXEC SQL` en COBOL → embedded SQL en RPG), Confluence MCP (publication, si token disponible) |

### Pourquoi IBM i Developer — Ask pour la génération du diff de conversion ?

UC 1 est la conversion à risque le plus élevé du POC. En mode Ask, Bob produit l'analyse et chaque section du diff dans le chat — l'expert peut valider chaque décision de transposition (REDEFINES, niveau 88, PERFORM THRU) avant de sauvegarder. En mode Agent, Bob écrit directement un source RPG potentiellement incorrect sur des zones sémantiquement critiques.

| Phase | Mode | Ce que Bob fait |
|-------|------|----------------|
| Qualification du programme (Prompt 0) | **Ask** | Analyse les 4 divisions COBOL, identifie les constructs sans équivalent RPG, délimite le périmètre POC, recommande la stratégie |
| Inventaire des structures à transposer (Prompt 1) | **Ask** | Lit le source COBOL via IBM i MCP, produit le tableau complet des structures WORKING-STORAGE, paragraphes PROCEDURE DIVISION et zones à décision |
| Transposition des structures de données (Prompt 2) | **Ask** | Génère les équivalences DATA DIVISION → DCL-S / DCL-DS / DCL-F dans le chat |
| Transposition de la PROCEDURE DIVISION (Prompt 3) | **Ask** | Génère les équivalences paragraphes → sous-routines / procédures RPG dans le chat |
| Test de compilation (Prompt 3-bis) | **Agent** | Lance `CRTBNDRPG` via IBM i MCP, rapporte les erreurs |
| Interfaces et constructs spéciaux (Prompt 4) | **Ask** | Transpose les niveaux 88, les CALL vers d'autres programmes, les REDEFINES complexes |
| Test de compilation final (Prompt 4-bis) | **Agent** | Lance `CRTBNDRPG` sur le source final |
| Test fonctionnel | **Humain** | Exécution sur IBM i de test, comparaison avec le programme COBOL d'origine — non délégable à Bob |
| Sauvegarde du diff validé | **Agent** | Écrit le fichier `.md` dans le workspace — uniquement une fois chaque section validée par l'expert |

> 💡 **Règle d'or pour UC 1 :** Le mode Agent est autorisé **uniquement** pour deux opérations précises : le test de compilation via les Prompts 3-bis et 4-bis, et la sauvegarde des livrables validés. Pendant toute la phase d'analyse et de génération (Prompts 0 à 4), rester en mode Ask.

> ⚠️ Ne jamais rester en mode Agent pendant la transposition des structures de données — un REDEFINES mal transposé en DCL-DS avec OVERLAY incorrect produit un accès mémoire silencieusement erroné qui ne sera pas détecté à la compilation.

### Intégration ARCAD

Le MCP ARCAD n'était pas disponible dans le contexte de ce POC de référence (version ARCAD non compatible avec le MCP). Si le MCP ARCAD est disponible dans votre environnement, les étapes manuelles de réintégration décrites ci-dessous peuvent être automatisées. N'hésitez pas à demander à Bob de modifier cette fiche UC en intégrant la disponibilité du MCP ARCAD.

**Impact sur UC 1 : moyen.** Le programme RPG converti est un nouveau membre source (`QRPGSRC`) qui n'existait pas avant. Ce nouveau membre devra être enregistré dans ARCAD et l'ancien programme COBOL marqué comme remplacé.

| Sans MCP ARCAD (contexte de ce POC) | Avec MCP ARCAD disponible |
|--------------------------------------|---------------------------|
| Créer manuellement le nouveau membre QRPGSRC dans ARCAD après validation | IBM i MCP + MCP ARCAD peuvent créer le membre et l'enregistrer dans ARCAD directement |
| Marquer manuellement le programme COBOL comme "remplacé" dans ARCAD | Le MCP ARCAD peut automatiser la mise à jour du statut |
| Ajouter le placeholder `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` dans l'en-tête du source RPG généré | Le placeholder n'est plus nécessaire — la réintégration est pilotée par Bob |

> 💡 **Dans les deux cas :** avant de démarrer UC 1 sur un programme COBOL, vérifier dans ARCAD qu'il n'est pas en cours de modification. Documenter dans le fichier `*-analyse-cobol-*.md` le nom du programme COBOL d'origine et le nom du programme RPG cible.

---

## Prompts clés

> 💡 **Atelier Bob Industrialisation**
> La mise en place de l'atelier Bob Industrialisation permet de simplifier le travail sur cette section :
> les prompts récurrents (Prompt 0, prompts de conversion, prompt-bis) sont disponibles sous forme de **commandes slash personnalisées** (`/qualify`, `/conv-cobol-data`, `/conv-cobol-proc`, `/test-compile`, etc.).
> Au lieu de copier-coller le bloc de code, il suffit de taper la commande correspondante dans la conversation Bob.
> Voir la fiche `atelier-bob-industrialisation-prompts.md` pour la liste complète des commandes disponibles.

### Prompt 0 — Qualification et choix de stratégie

> **Ce prompt est le point d'entrée obligatoire de UC 1 pour chaque programme COBOL.**
> Il analyse les 4 divisions COBOL, identifie les constructs sans équivalent RPG direct, délimite le périmètre de conversion acceptable pour le POC, et recommande la séquence exacte.
> Il se lance **avant** le Prompt 1 — son résultat conditionne toute la séquence suivante.

```
Le programme COBOL [NOM_PROGRAMME] se trouve dans [NOM_LIB]/QCBLSRC.
[Si disponible : Le fichier de compréhension {projet}-{lib}-{programme}-comprehension-{YYYYMMDD-HHmm}.md est disponible.]
[Si disponible : La spécification technique {projet}-{lib}-{programme}-spec-tech-{YYYYMMDD-HHmm}.md est disponible.]

Analyse ce programme COBOL et produis en français, en markdown, une fiche d'analyse
de conversion COBOL → RPG ILE Free :

## Analyse de conversion UC 1 — [NOM_PROGRAMME]

### 0. Vérification des prérequis
Avant toute analyse, vérifier la disponibilité des fichiers obligatoires :
| Fichier attendu | Présent dans le contexte ? | Action si absent |
|-----------------|---------------------------|-----------------|
| `{projet}-{lib}-{programme}-comprehension-{date}.md` (UC 4) | OUI / NON | ⛔ ARRÊTER — lancer UC 4 sur ce programme avant de continuer |
| `{projet}-{lib}-{programme}-spec-tech-{date}.md` (UC 6) | OUI / NON | ⛔ ARRÊTER — sans spec technique, l'interface d'appel et les fichiers accédés ne sont pas connus ; la transposition des FD et de la LINKAGE SECTION sera incomplète |

Si l'un des deux fichiers est absent : ne pas poursuivre l'analyse — signaler le blocage
et indiquer quel UC doit être complété en premier.

### 1. Inventaire rapide des divisions COBOL
| Division / Section           | Contenu détecté                                                             |
|------------------------------|-----------------------------------------------------------------------------|
| IDENTIFICATION DIVISION      | Nom programme, auteur, date — à convertir en commentaires RPG              |
| ENVIRONMENT DIVISION         | SELECT / ASSIGN des fichiers — à transposer en DCL-F                       |
| DATA DIVISION / FILE SECTION | FD et descriptions d'enregistrements — nombre de FD détectés : ?           |
| DATA DIVISION / WORKING-STORAGE | Niveau 01 racines : ? / REDEFINES : OUI/NON / Niveaux 88 : OUI/NON      |
| DATA DIVISION / LINKAGE SECTION | Présente : OUI/NON — paramètres reçus par CALL USING                    |
| PROCEDURE DIVISION           | Nombre de paragraphes : ? / Présence PERFORM THRU : OUI/NON               |

### 2. Options de compilation à déterminer
Rechercher dans cet ordre — s'arrêter dès que l'information est trouvée :
1. Directive `PROCESS` dans le source COBOL ou dans un COPY member
2. Commande ou script de compilation disponible dans le contexte
3. Listing de compilation historique (si fourni)
4. Convention de build ARCAD connue de l'équipe

| Option | Valeur trouvée | Source de l'information |
|--------|---------------|------------------------|
| COMPASBIN / NOCOMPASBIN (comportement de COMP) | ? | ? |
| TRUNC (troncature arithmétique) | ? | ? |
| ARITHMETIC (précision étendue) | ? | ? |
| Activation group | ? | ? |
| SQL précompilé (EXEC SQL présent) | OUI / NON | analyse du source |

⚠️ Si COMPASBIN / NOCOMPASBIN est inconnu : tout champ `COMP` (sans suffixe numérique)
sera marqué `À CONFIRMER` — ne pas le convertir automatiquement dans cette session.
Note : la valeur par défaut IBM i est NOCOMPASBIN (COMP = COMP-3 = packed decimal).

### 3. Facteurs de complexité détectés
Réponds par OUI / NON / À CONFIRMER pour chaque facteur :
- [ ] PERFORM THRU (séquence de paragraphes consécutifs) — sans équivalent RPG direct
- [ ] ALTER GO TO (modification dynamique de destination GO TO) — obsolète, sans équivalent
- [ ] REDEFINES multi-niveaux (zone vue sous plusieurs formats) — transposition en DCL-DS avec OVERLAY
- [ ] Niveaux 88 COBOL avec VALUE multiples ou plages THRU — prédicat composé requis
- [ ] OCCURS DEPENDING ON (tableaux à taille variable runtime) — sans équivalent RPG direct
- [ ] CALL avec USING / RETURNING vers d'autres programmes IBM i — interfaces à préserver
- [ ] CALL BY CONTENT (passage par copie) — copie locale obligatoire côté RPG
- [ ] EXEC SQL embarqué en COBOL — compilation CRTSQLRPGI requise (pas CRTBNDRPG)
- [ ] FILE SECTION / FD complexe (RECORD CONTAINS VARYING, BLOCK CONTAINS) — transposition non triviale
- [ ] Paragraphes appelés depuis plusieurs PERFORM différents (paragraphe réutilisé) — candidat EXSR
- [ ] COPY members COBOL (COPY NOM-COPIE) — à localiser dans QCBLSRC pour inclure dans le contexte
- [ ] PERFORM ... WITH TEST AFTER — équivalent DOU, pas DOW
- [ ] ROUNDED ou ON SIZE ERROR sur des calculs — précision et dépassement à préserver
- [ ] STOP RUN / GOBACK / EXIT PROGRAM — comportements distincts, cf. table Prompt 3

### 3. Catégorie et périmètre recommandé
**Catégorie :**
- [ ] SIMPLE — PROCEDURE DIVISION linéaire, pas de PERFORM THRU ni ALTER, WORKING-STORAGE sans REDEFINES complexe
- [ ] STANDARD — WORKING-STORAGE avec REDEFINES ou niveaux 88, PERFORM imbriqués, CALL vers d'autres programmes
- [ ] COMPLEXE — PERFORM THRU, ALTER GO TO, OCCURS DEPENDING ON, ou FILE SECTION dense avec RECORD VARYING

> Note : UC 1 utilise SIMPLE / STANDARD / COMPLEXE (critère : nombre de paragraphes COBOL et
> présence de constructs sans équivalent RPG direct). Même vocabulaire que UC 14, UC 3, UC 7,
> UC 8 et UC 2 — vocabulaire unifié Phase 2, 3 et 4.
> Les échelles sont indépendantes : un programme SIMPLE en UC 1 peut être très différent
> d'un programme SIMPLE en UC 2. Le Prompt 0 de chaque UC calibre selon ses propres critères.

**Périmètre de conversion recommandé pour le POC :**
- Paragraphes convertibles directement : [liste]
- Paragraphes nécessitant une décision de conception : [liste + nature de la décision]
- Paragraphes à reporter (trop risqués pour le POC) : [liste + justification]

**Recommandation :**
- SIMPLE   → Conversion complète : Étape 0-bis → Prompt 1 → Prompt 2 → Prompt 3 → Prompt 3-bis → Prompt 4
- STANDARD → Conversion par sections : Étape 0-bis → Prompt 1 → Prompt 2 (avec points de décision) →
               Prompt 3 → Prompt 3-bis → Prompt 4
- COMPLEXE → Périmètre restreint : Étape 0-bis → traiter les paragraphes convertibles en SIMPLE/STANDARD ;
               conserver les constructs non convertibles dans un programme COBOL séparé callable depuis RPG
               (⚠️ il n'est pas possible d'appeler un paragraphe COBOL interne depuis RPG —
               la logique non convertie doit être isolée dans un programme COBOL autonome)

Signale clairement les constructs pour lesquels une décision experte est requise avant de générer le code RPG.
```

**Analyse ligne à ligne :**

- `Inventaire rapide des divisions COBOL` → la structure en 4 divisions est la première chose à documenter. Un programme COBOL sans LINKAGE SECTION n'a pas de paramètres — l'interface d'appel est différente de celle d'un programme COBOL avec LINKAGE. Cette distinction conditionne la façon dont le programme RPG converti sera appelé par les autres programmes.

- `FILE SECTION / FD` → les descriptions de fichiers COBOL (FD) sont l'équivalent des F-specs RPG. Mais la FILE SECTION COBOL contient aussi la structure `01 NOM-ENREG` qui décrit l'enregistrement — en RPG, cette structure se traduit soit par `DCL-DS LIKEREC` (si le fichier a une description DDS externe), soit par une `DCL-DS` manuelle (si le fichier est décrit dans le COBOL). Le Prompt 0 doit identifier la nature de chaque FD.

- `PERFORM THRU` → sans équivalent RPG direct, ce construct est le signal d'alerte principal pour classer un programme en COMPLEXE. Bob doit le détecter systématiquement au Prompt 0 — pas le découvrir en cours de conversion.

- `Niveaux 88 COBOL` → les conditions booléennes nommées COBOL sont un mécanisme très utilisé en COBOL IBM i legacy. `88 COMPTE-ACTIF VALUE 'A'` suivi de `IF COMPTE-ACTIF` est idiomatique COBOL — en RPG, la même logique s'écrit `IF wkStatutCompte = C_STATUT_ACTIF` après déclaration d'une constante `DCL-C C_STATUT_ACTIF 'A'`. La qualité de la transposition dépend de la compréhension du sens fonctionnel du niveau 88.

- `Périmètre de conversion recommandé pour le POC` → la restriction de périmètre est explicite dans le Prompt 0 de UC 1. Mieux vaut documenter ce qui est exclu et pourquoi, que tenter une conversion complète dont le résultat serait non validable par l'équipe.

> 💡 **Sauvegarder l'analyse de conversion** (mode Agent) dès la fin du Prompt 0 :
> ```
> "Sauvegarde cette analyse de conversion dans un fichier nommé
>  {projet}-{lib}-{programme}-analyse-cobol-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-COBPGM1-analyse-cobol-20250622-0900.md"
> ```
> Le Prompt 1 (inventaire détaillé) s'appuie sur cette analyse.

> ⚠️ **Piège évité :** sans le Prompt 0, l'équipe commence la conversion sans savoir si le programme a des PERFORM THRU ou des OCCURS DEPENDING ON. Découvrir ces constructs en cours de génération oblige à tout reprendre depuis le début — avec une analyse incorrecte déjà sauvegardée dans le workspace.

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/qualify` dans la conversation Bob — la structure complète du Prompt 0 est injectée automatiquement, pré-remplie avec le nom du programme ouvert dans l'éditeur.

---

### Prompt 1 — Inventaire des structures à transposer

```
Le programme COBOL [NOM_PROGRAMME] se trouve dans [NOM_LIB]/QCBLSRC.

Sur la base de l'analyse de conversion UC 1 que nous venons de faire,
produis en français, en markdown, l'inventaire complet des structures à transposer :

## Inventaire UC 1 — [NOM_PROGRAMME]

### 1. Structures WORKING-STORAGE à transposer
Pour chaque groupe de niveau 01 et ses sous-champs :
| N° | Niveau COBOL | Nom COBOL       | PIC / USAGE          | Équivalent RPG Free           | Décision requise |
|----|--------------|-----------------|----------------------|-------------------------------|-----------------|
Règle de transposition des types numériques :

  PIC 9(...) ou S9(...) sans USAGE   → DCL-S NomVar ZONED(n:d)  ← par défaut
     Exception : si le champ appartient exclusivement à WORKING-STORAGE ET
     - n'est pas dans un REDEFINES ou RENAMES,
     - n'est pas passé par LINKAGE ou CALL,
     - n'est pas écrit dans un fichier programme-décrit,
     - n'est pas utilisé par STRING/UNSTRING/INSPECT ni par un calcul de longueur,
     alors PACKED est autorisé comme optimisation — inscrire dans le diff :
     « Représentation DISPLAY/ZONED remplacée par PACKED — optimisation validée »

  PIC S9(7) COMP-3           → DCL-S NomVar PACKED(7:0)
  PIC S9(5)V9(2) COMP-3      → DCL-S NomVar PACKED(7:2)
  PIC S9(9) COMP-3            → DCL-S NomVar PACKED(9:0)

  PIC 9(...) COMP sans suffixe → À CONFIRMER — ne pas convertir si COMPASBIN inconnu
     Si NOCOMPASBIN confirmé (défaut IBM i) : COMP = COMP-3 → PACKED
     Si COMPASBIN confirmé                 : COMP = COMP-4 → INT ou UNS selon S9 / 9
  PIC S9(1-4) COMP-4 / BINARY  → DCL-S NomVar INT(5)
  PIC S9(5-9) COMP-4 / BINARY  → DCL-S NomVar INT(10)
  PIC S9(10-18) COMP-4 / BINARY → DCL-S NomVar INT(20)
  PIC 9(1-4) COMP-4 / BINARY   → DCL-S NomVar UNS(5)   (non signé)
  COMP-1                        → DCL-S NomVar FLOAT(4)
  COMP-2                        → DCL-S NomVar FLOAT(8)

  PIC X(10)                  → DCL-S NomVar CHAR(10)
  PIC A(10)                  → DCL-S NomVar CHAR(10)
  PIC 9(5)                   → DCL-S NomVar ZONED(5:0)  (DISPLAY par défaut)

Constantes figuratives COBOL → RPG :
  SPACES / SPACE             → *BLANKS
  ZEROS / ZEROES             → *ZEROS
  LOW-VALUES                 → *LOVAL
  HIGH-VALUES                → *HIVAL
  ALL 'x'                    → %REPEAT('x' : longueur)
                               ou DCL-C si la chaîne est une constante de longueur fixe

  01 NomDs.
    05 SousCh1 PIC X(5)      → DCL-DS NomDs ;
                                 SousCh1 CHAR(5) ;
                               END-DS ;
  01 NomRed REDEFINES NomVar → à traiter en section 2 — syntaxe OVERLAY cf. ci-dessous

### 2. REDEFINES à décider
Pour chaque REDEFINES détecté :
| N° | Zone d'origine (01) | Zone REDEFINES | Offset (octets) | Longueur | Équivalent RPG proposé | Décision |
Syntaxe correcte RPG Free pour un REDEFINES :
  DCL-DS wsZone ;
    donneesBrutes CHAR(10) ;                               ← zone d'origine
    code          CHAR(3)  OVERLAY(donneesBrutes : 1) ;    ← vue 1, offset 1
    sousCode      CHAR(2)  OVERLAY(donneesBrutes : 4) ;    ← vue 2, offset 4
    montant       PACKED(5:0) OVERLAY(donneesBrutes : 6) ; ← vue 3, offset 6
  END-DS ;
⚠️ OVERLAY s'applique à des sous-champs dans une DS, pas à une DCL-DS autonome.
⚠️ Calculer et vérifier chaque offset : %SIZE(wsZone) doit égaler la longueur de la zone d'origine.
Décision : DIRECT (offsets calculés, longueurs vérifiées) / À CONFIRMER (imbriqué ou longueur variable)

### 3. Niveaux 88 à transposer
Pour chaque niveau 88 détecté :
| N° | Variable parent | Nom niveau 88 | VALUE(s) / plage THRU | Forme | Équivalent RPG proposé | Utilisé dans |
Forme : SINGLE (valeur unique) / MULTI (valeurs multiples) / RANGE (THRU) / MIXED

Équivalences selon la forme :
  SINGLE  → DCL-C C_[NOM_88] [VALEUR]
             IF wkVar = C_[NOM_88]
  MULTI   → prédicat composé (évaluer directement dans chaque IF) :
             IF wkVar = 'A' OR wkVar = 'B' OR wkVar = 'C' ;
  RANGE   → IF wkVar >= valMin AND wkVar <= valMax ;
  MIXED   → combinaison des deux lignes ci-dessus

⚠️ Ne pas stocker dans une variable IND le résultat d'une évaluation MULTI ou RANGE
si la variable parent peut être modifiée ultérieurement — l'indicateur deviendrait périmé.
Évaluer le prédicat directement dans chaque IF / EVALUATE.

### 4. FILE SECTION / FD à transposer
Pour chaque FD :
| N° | Nom FD | Mode accès | Description 01 | Équivalent RPG DCL-F | DCL-DS associée |
Syntaxe correcte :
  DCL-F NomFich DISK(*EXT) KEYED USAGE(*INPUT) ;
  + DCL-DS NomEnreg LIKEREC(NomFormat : *INPUT) END-DS ;
    (⚠️ NomFormat = nom du FORMAT d'enregistrement RPG, pas le nom du fichier)
    (si description DDS externe disponible)
  + DCL-DS NomEnreg manuellement   (si décrit uniquement dans le COBOL)

### 5. PROCEDURE DIVISION — paragraphes à transposer
Pour chaque paragraphe ou section :
| N° | Nom paragraphe | Appelé par (PERFORM) | PERFORM THRU ? | Opérations principales | Équivalent RPG |
Équivalent RPG :
  Paragraphe appelé par PERFORM unique → BEGSR / ENDSR (subroutine)
  Paragraphe appelé depuis plusieurs PERFORM → CALLP procédure (si extractible) ou EXSR
  PERFORM THRU [PARA-A] THRU [PARA-Z] → décision de conception requise (cf. section 6)

### 6. Constructs COBOL sans équivalent RPG direct
Pour chaque construct nécessitant une décision de conception :
| N° | Construct | Lignes | Nature | Option A (simplifiée) | Option B (fidèle) | Recommandation |
Constructs à traiter ici : PERFORM THRU, ALTER GO TO, OCCURS DEPENDING ON, STRING/UNSTRING,
INSPECT TALLYING/REPLACING, CALL BY CONTENT

Signale clairement les cas où la décision nécessite une connaissance fonctionnelle
non visible dans le source COBOL.
Ne pas générer le code RPG dans ce prompt.
Sauvegarde cet inventaire (mode Agent) dès qu'il est complet :
"Sauvegarde cet inventaire dans {projet}-{lib}-{programme}-analyse-cobol-{YYYYMMDD-HHmm}.md"
```

**Analyse ligne à ligne :**

- `DISPLAY → ZONED par défaut` → la table impose `ZONED` pour tout champ sans `USAGE` explicite car c'est la représentation mémoire réelle (décimale zonée). `PACKED` est autorisé comme optimisation uniquement lorsque les cinq conditions d'exclusion sont satisfaites — l'inventaire documente le choix pour chaque champ.

- `COMP sans suffixe → À CONFIRMER` → la valeur par défaut IBM i est `NOCOMPASBIN` : `COMP` est alors traité comme `COMP-3` (packed). Avec `COMPASBIN`, `COMP` devient un binaire. Sans connaître l'option de compilation, convertir `COMP` en `INT` est une erreur silencieuse — l'inventaire bloque la conversion jusqu'à résolution.

- `REDEFINES → OVERLAY sur sous-champs d'une DS commune` → `OVERLAY` s'applique à des sous-champs à l'intérieur d'une même `DCL-DS`, pas à une `DCL-DS` autonome. Chaque offset est calculé et vérifié avec `%SIZE`.

- `Niveaux 88 SINGLE/MULTI/RANGE/MIXED` → `SINGLE` donne une `DCL-C` ; les formes avec plusieurs valeurs ou plages `THRU` produisent des prédicats composés évalués directement dans chaque `IF` — pas stockés dans une variable `IND` si la variable parent peut changer.

- `Ne pas générer le code RPG dans ce prompt` → la même discipline que UC 7 et UC 8. L'inventaire est la feuille de route — les décisions de conception de la section 6 doivent être prises par l'expert avant de démarrer la génération.

> ⚠️ **Piège évité :** sans inventaire des niveaux 88, Bob peut omettre de les transposer (car ils n'ont pas de traduction syntaxique directe en RPG) — le code RPG compile mais les conditions nommées disparaissent, remplacées par des comparaisons littérales éparpillées dans le source.

---

### Étape 0-bis — Référence comportementale minimale

> **À exécuter avant toute génération de code — sur le programme COBOL original sur l'IBM i de test.**

Avant de démarrer les passes de conversion, capturer manuellement :

- [ ] Au moins **un cas nominal** : paramètres d'entrée, résultat de sortie, fichiers créés ou modifiés
- [ ] Au moins **un cas limite** identifié au Prompt 0 (champ REDEFINES, niveau 88, PERFORM THRU...)
- [ ] Le code retour ou statut fichier pertinent après exécution
- [ ] Les paramètres retournés si le programme est callable

Consigner ces résultats dans le fichier `{projet}-{lib}-{programme}-analyse-cobol-{YYYYMMDD-HHmm}.md`
sous la section `## Référence comportementale`.

⚠️ Si le programme original ne peut pas être exécuté ou si aucun résultat de référence
n'est disponible, l'inscrire explicitement : **NON-RÉGRESSION NON DÉMONTRABLE DANS LE POC**
et en informer le responsable du POC avant de continuer.

---

### Prompt 2 — Transposition des structures de données (DATA DIVISION → DCL)

```
Sur la base de l'inventaire UC 1 de [NOM_PROGRAMME], sections 1 à 4,
transposes les structures de données COBOL en leur équivalent RPG ILE Free :

[Lister ici les N° de structures du tableau Prompt 1, sections 1 à 4, à traiter dans cette passe]

Le source RPG cible doit commencer par la directive fully free-form en colonne 1 :
**FREE
Cette directive doit être la première ligne du membre, avant toute déclaration.
Sans elle, le compilateur interprète le source en mode colonné limité.

Pour chaque structure transposée, produis le diff :
// COBOL (WORKING-STORAGE) :
[structure COBOL avec ses niveaux]
// RPG ILE Free :
[équivalent DCL-S / DCL-DS / DCL-F]

Règles de transposition :
- IDENTIFICATION DIVISION + ENVIRONMENT DIVISION → convertir en commentaires RPG d'en-tête :
  // ============================================================
  // PROGRAMME : [NOM_PROGRAMME]
  // Converti depuis COBOL [NOM_LIB]/QCBLSRC([NOM_PROGRAMME])
  // Date de conversion : [DATE]
  // Auteur original COBOL : [AUTHOR si présent]
  // Statut : À valider par expert COBOL + RPG avant tout usage
  // ⚠️ Réintégration ARCAD — à effectuer manuellement après validation
  // ============================================================
- WORKING-STORAGE 01 simples → DCL-S selon la table de types du Prompt 1 (ZONED par défaut)
- WORKING-STORAGE 01 avec sous-champs → DCL-DS NomDs ; [sous-champs] END-DS ;
- REDEFINES → utiliser la syntaxe OVERLAY dans une DS partagée (cf. Prompt 1 section 2)
  Pour les REDEFINES "À CONFIRMER" : inclure le bloc mais marquer // ⚠️ DÉCISION REQUISE
- Niveaux 88 → utiliser l'équivalent retenu dans l'inventaire section 3 (DCL-C ou prédicat)
- FD → DCL-F selon la table Prompt 1 section 4 ; DCL-DS LIKEREC(NomFormat:*INPUT) si DDS externe
- LINKAGE SECTION → générer ici la DCL-PI avec les types exacts de la LINKAGE :
  DCL-PI *N ;
    param1 [TYPE identique au PIC COBOL de la LINKAGE] ;
    ...
  END-PI ;
  (⚠️ DCL-PI générée dans cette passe — obligatoire avant le Prompt 3-bis)
  Pour les programmes sans LINKAGE SECTION : omettre la DCL-PI
- Pour chaque programme externe appelé (CALL) : générer le prototype :
  DCL-PR NomPgm EXTPGM('NOMPGM') ;
    param1 [TYPE] ;
  END-PR ;
  (obligatoire — CALLP sans prototype DCL-PR EXTPGM n'est pas valide)
- Ne pas transposer les OCCURS DEPENDING ON dans cette passe — les traiter en Prompt 4
  comme cas spéciaux
- Signaler toute structure dont la transposition n'est pas directe
Sauvegarde ce diff (mode Agent) dès qu'il est complet :
"Sauvegarde ce diff dans {projet}-{lib}-{programme}-cobol-converti-{YYYYMMDD-HHmm}.md"
```

> 💡 **Sauvegarder ce diff** (mode Agent) :
> ```
> "Sauvegarde ce diff de transposition des structures dans un fichier nommé
>  {projet}-{lib}-{programme}-cobol-converti-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-COBPGM1-cobol-converti-20250622-1100.md"
> ```

> ⚠️ **Piège évité :** transposer un `01 NomVar PIC 9(5)V9(2)` (sans USAGE, donc DISPLAY) en `PACKED(7:2)` au lieu de `ZONED(7:2)` — les deux types ont des représentations mémoire différentes (PACKED : 2 chiffres par octet / ZONED : 1 chiffre par octet). Cette confusion casse les REDEFINES et les interfaces d'appel qui reposent sur la taille physique du champ.

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/conv-cobol-data` — le prompt de transposition DATA DIVISION est injecté avec les variables du programme courant et la liste des structures à traiter (issue du Prompt 1) déjà résolue.

---

### Prompt 3 — Transposition de la PROCEDURE DIVISION

~~~
Sur la base de l'inventaire UC 1 de [NOM_PROGRAMME], section 5,
transposes les paragraphes COBOL en leur équivalent RPG ILE Free :

[Lister ici les N° de paragraphes du tableau Prompt 1, section 5, à traiter dans cette passe]

Pour chaque paragraphe transposé, produis le diff :
// COBOL (PROCEDURE DIVISION — paragraphe [NOM-PARA]) :
[code COBOL original]
// RPG ILE Free (subroutine / procédure) :
[équivalent RPG]

Table de transposition COBOL → RPG à respecter :
MOVE src TO dest                 → affectation directe uniquement si les types, longueurs,
                                   alignements et règles de remplissage sont équivalents :
                                   EVAL dest = src
                                   Si les longueurs ou catégories diffèrent, si JUSTIFIED
                                   est présent, ou si seule une partie de dest est remplacée :
                                   // ⚠️ MOVE — À CONFIRMER
                                   Générer une conversion explicite (%SUBST, %REPLACE ou
                                   conversion numérique adaptée) après analyse de l'inventaire.
                                   (⚠️ vérifier aussi si la zone est REDEFINES — accès via DS)
COMPUTE dest = expr              → EVAL dest = expr   (adapter la syntaxe arithmétique)
ADD a TO b                       → EVAL b = b + a
ADD a b GIVING c                 → EVAL c = a + b
SUBTRACT a FROM b                → EVAL b = b - a
MULTIPLY a BY b GIVING c         → EVAL c = a * b
DIVIDE a INTO b GIVING c REM r   → EVAL c = %DIV(b:a) ; EVAL r = %REM(b:a)
                                   ⚠️ Valide uniquement si a, b, c et r sont tous entiers.
                                   Si un opérande ou le résultat comporte des décimales :
                                   // ⚠️ DIVIDE DÉCIMAL — À CONFIRMER
                                   Reproduire la précision intermédiaire, la troncature,
                                   ROUNDED et ON SIZE ERROR du COBOL d'origine.
                                   %DIV/%REM tronquent vers zéro et ignorent les décimales.
IF cond THEN ... END-IF          → IF cond ; ... ENDIF ;
IF cond THEN ... ELSE ... END-IF → IF cond ; ... ELSE ; ... ENDIF ;
EVALUATE expr
  WHEN val1 ... WHEN val2 ...    → SELECT ; WHEN expr = val1 ; ... WHEN expr = val2 ; ENDSL ;
EVALUATE TRUE
  WHEN cond1 ... WHEN cond2 ...  → SELECT ; WHEN cond1 ; ... WHEN cond2 ; ENDSL ;
                                   (⚠️ EVALUATE TRUE ≠ EVALUATE expr — WHEN = conditions booléennes)
PERFORM para                     → EXSR srNomPara   (paragraphe simple)
PERFORM para VARYING ...         → FOR / DOW avec variable de contrôle explicite
PERFORM ... WITH TEST BEFORE UNTIL cond → DOW NOT cond ; ... ENDDO ;
PERFORM ... WITH TEST AFTER  UNTIL cond → DOU cond ; ... ENDDO ;
PERFORM UNTIL cond (sans TEST)   → DOW NOT cond ; ... ENDDO ;   (TEST BEFORE par défaut)
PERFORM N TIMES                  → FOR i = 1 TO n ; ... ENDFOR ;
STOP RUN                         → À CONFIRMER — dépend du rôle du programme :
                                   // Programme principal (seul dans son activation group) :
                                   *INLR = *ON ; RETURN ;   (fermeture des ressources + fin)
                                   // Programme appelé (CALL) : vérifier le comportement
                                   historique et l'activation group avant de mettre *INLR.
                                   Ne générer *INLR = *ON que si la fermeture et la
                                   réinitialisation des ressources du programme courant
                                   doivent effectivement avoir lieu.
                                   Dans un activation group partagé, *INLR = *ON ferme
                                   les fichiers et libère les ressources associées au
                                   programme RPG courant — il ne termine pas l'activation
                                   group et ne ferme pas les fichiers appartenant au
                                   programme appelant.
GOBACK                           → RETURN ;   (retour à l'appelant sans clôture du groupe)
EXIT PROGRAM                     → RETURN ;   (dans un sous-programme imbriqué)
MOVE SPACES TO dest              → dest = *BLANKS ;
MOVE ZEROS  TO dest              → dest = *ZEROS ;
MOVE LOW-VALUES TO dest          → dest = *LOVAL ;
MOVE HIGH-VALUES TO dest         → dest = *HIVAL ;
GO TO para                       → à analyser — cf. section 6 de l'inventaire
DISPLAY "message"                → pas d'équivalent direct en RPG batch — écritureFichier ou DSPLY (debug)
READ NomFich INTO NomVar
  AT END flFinFich = *ON         → Si le COBOL gérait uniquement AT END :
                                   READ NomFich NomVar ;
                                   IF %EOF(NomFich) ;
                                      flFinFich = *ON ;
                                   ENDIF ;
                                   Si le COBOL gérait aussi explicitement les erreurs d'I/O :
                                   READ(E) NomFich NomVar ;
                                   IF %ERROR ;   /* reproduire le traitement d'erreur original */
                                   ELSEIF %EOF(NomFich) ; flFinFich = *ON ;
                                   ENDIF ;
                                   (⚠️ %ERROR sans paramètre — il porte sur la dernière
                                   opération avec extender (E), pas sur un fichier nommé.
                                   ELSEIF obligatoire : %EOF peut conserver une valeur
                                   antérieure si l'opération n'a pas abouti normalement)
                                   (⚠️ NomVar : DS LIKEREC ou zone cible — ne pas omettre
                                   si le COBOL lit dans une zone distincte du tampon fichier)
READ NomFich NEXT RECORD         → Si le COBOL gérait uniquement AT END :
                                   READ NomFich ;
                                   IF %EOF(NomFich) ;
                                      /* reproduire chemin AT END original */
                                   ENDIF ;
                                   Si le COBOL gérait aussi explicitement les erreurs d'I/O :
                                   READ(E) NomFich ;
                                   IF %ERROR ;   /* reproduire le traitement d'erreur original */
                                   ELSEIF %EOF(NomFich) ; /* reproduire chemin AT END original */
                                   ENDIF ;
                                   (⚠️ %ERROR sans paramètre — voir note ci-dessus)
                                   (⚠️ séquentiel sans filtre de clé — pas READE)
WRITE NomEnreg FROM NomVar       → Si le source ne gérait pas explicitement les erreurs d'écriture :
                                   WRITE NomEnreg NomVar ;   (sans (E) — laisser remonter l'exception)
                                   Si le source utilisait un indicateur d'erreur, INFSR, ou un
                                   traitement explicite :
                                   WRITE(E) NomEnreg NomVar ;
                                   IF %ERROR ; /* reproduire comportement original */ ENDIF ;
REWRITE NomEnreg FROM NomVar     → Si le source ne gérait pas explicitement les erreurs :
                                   UPDATE NomEnreg NomVar ;   (sans (E))
                                   Si traitement d'erreur original :
                                   UPDATE(E) NomEnreg NomVar ;
                                   IF %ERROR ; /* reproduire comportement original */ ENDIF ;
DELETE NomFich RECORD            → Si le source ne gérait pas explicitement les erreurs :
                                   DELETE NomFich ;   (sans (E))
                                   Si traitement d'erreur original :
                                   DELETE(E) NomFich ;
                                   IF %ERROR ; /* reproduire comportement original */ ENDIF ;
START NomFich KEY = cle          → SETLL cle NomFich ; IF NOT %EQUAL(NomFich) ; /* INVALID KEY */ ENDIF ;
START NomFich KEY > cle          → SETGT cle NomFich ;
CALL 'NomPgm' USING BY REFERENCE param1 → CALLP NomPgm(param1) ;
                                   (prototype DCL-PR EXTPGM obligatoire — cf. Prompt 2)
CALL 'NomPgm' USING BY CONTENT  param1 → DCL-S wkCopie [TYPE] ; wkCopie = param1 ;
                                   CALLP NomPgm(wkCopie) ;
                                   (⚠️ BY CONTENT = copie — ne pas passer la variable d'origine)
STRING ... DELIMITED BY SIZE/SPACE INTO dest
                                 → aucune équivalence directe par %TRIMR ou %TRIM.
                                   Conversion par %SUBST + gestion explicite de la position courante :
                                   - DELIMITED BY SIZE : utiliser toute la longueur déclarée, espaces compris
                                   - DELIMITED BY SPACE : s'arrêter au premier espace (≠ %TRIM qui supprime
                                     tous les espaces de fin)
                                   Si WITH POINTER ou ON OVERFLOW présents : // ⚠️ DÉCISION REQUISE
                                   signaler pour chaque occurrence : // ⚠️ STRING — conversion par %SUBST requise
UNSTRING src DELIMITED BY delim
       INTO dest1 dest2          → décision requise — utiliser %SCAN + %SUBST selon la logique ;
                                   signaler // ⚠️ UNSTRING — décision de transposition requise
INSPECT src TALLYING cpt
       FOR ALL char              → cpt = 0 ;
                                   pos = %SCAN(char : src) ;
                                   DOW pos > 0 ;
                                     cpt += 1 ;
                                     pos = %SCAN(char : src : pos + %LEN(char)) ;
                                   ENDDO ;
INSPECT src REPLACING ALL old BY new → décision requise — %XLATE si remplacement caractère par caractère ;
                                   signaler // ⚠️ INSPECT REPLACING — décision de transposition requise

Règles :
- Ne pas ajouter (E) mécaniquement sur les opcodes fichier.
  Ajouter (E) uniquement si le source original comportait un traitement d'erreur
  à reproduire avec %ERROR / %STATUS.
  %EOF, %FOUND et %EQUAL ne justifient pas à eux seuls l'ajout de (E).
  (⚠️ (E) seul sans traitement d'erreur = exception fatale silencieusement ignorée)
- Nommer chaque subroutine RPG d'après le paragraphe COBOL d'origine :
  paragraphe CALCULER-MONTANT-TTC → BEGSR srCalculerMontantTtc
- Conserver le nom COBOL d'origine en commentaire au-dessus de chaque subroutine :
  // Paragraphe COBOL d'origine : CALCULER-MONTANT-TTC
- Ne pas traiter les PERFORM THRU dans cette passe — les marquer // ⚠️ PERFORM THRU — cf. Prompt 4
- Ne pas inventer de logique non visible dans le source COBOL ou dans les fichiers
  *-comprehension-*.md et *-regles-*.md
- Signaler chaque endroit où la transposition suppose une décision non tranchée
Sauvegarde ce diff (mode Agent) dès qu'il est complet :
"Complète le fichier {projet}-{lib}-{programme}-cobol-converti-{YYYYMMDD-HHmm}.md
 avec la transposition de la PROCEDURE DIVISION de cette passe."
~~~

**Analyse ligne à ligne :**

- `PERFORM para → EXSR srNomPara` → la transposition standard d'un PERFORM vers un paragraphe unique. Le préfixe `sr` (subroutine) dans le nom RPG signale clairement que c'est une sous-routine — cohérent avec la convention camelCase de UC 7. Si le paragraphe est réutilisable en dehors du programme, il peut devenir une `DCL-PROC` — Bob signale ce cas dans l'inventaire.

- `STOP RUN → À CONFIRMER` → la terminaison d'un programme COBOL n'a pas d'équivalent unique en RPG ILE. La décision dépend du rôle du programme (principal ou appelé), du modèle d'activation group et de la persistance attendue des ressources. `*INLR = *ON` libère les fichiers ouverts et réinitialise les variables statiques — comportement souhaitable pour un programme principal, potentiellement destructeur pour un sous-programme appelé dans un groupe partagé. Ne jamais générer `*INLR = *ON` sans vérification avec l'expert.

- `GO TO → à analyser` → le `GO TO` COBOL peut être un simple saut vers une fin de paragraphe (équivalent d'un `LEAVE` ou `RETURN` RPG) ou un branchement complexe vers un autre paragraphe fonctionnel. Le Prompt 3 ne le résout pas mécaniquement — il le signale pour traitement dans le Prompt 4.

- `READ ... AT END` → le mécanisme de fin de fichier COBOL. En RPG ILE Free, l'équivalent est `READ` suivi de `IF %EOF(NomFich)` — le test de fin de fichier est explicite, pas dans une clause `AT END` incorporée.

- `Ne pas inventer de logique` → garde-fou anti-hallucination central. La PROCEDURE DIVISION COBOL peut contenir des règles métier critiques dans des formules ou des conditions — Bob ne peut pas inférer le sens d'une formule complexe. Il doit la transposer mécaniquement et signaler si le sens fonctionnel n'est pas clair.

> 💡 **Sauvegarder ce diff** (mode Agent) — compléter le fichier existant :
> ```
> "Complète le fichier {projet}-{lib}-{programme}-cobol-converti-{YYYYMMDD-HHmm}.md
>  avec la transposition de la PROCEDURE DIVISION de cette passe."
> ```

> ⚠️ **Piège évité :** transposer `EVALUATE TRUE` COBOL (évaluation d'expressions booléennes) en `SELECT` RPG sans vérifier que les `WHEN` sont des conditions booléennes et non des valeurs scalaires — la sémantique de `EVALUATE TRUE WHEN cond1 WHEN cond2` est différente de `EVALUATE expr WHEN val1 WHEN val2`.

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/conv-cobol-proc` — le prompt de transposition PROCEDURE DIVISION est injecté avec la liste des paragraphes à traiter (issue du Prompt 1) déjà résolue.

---

### Prompt 3-bis — Test de compilation par Bob (après la transposition principale)

> **Ce que Bob peut faire :** écrire le source RPG sur l'IBM i via IBM i MCP, lancer la compilation, et rapporter les erreurs.
> **Ce que Bob ne peut pas faire :** valider que les équivalences COBOL → RPG sont sémantiquement correctes — notamment pour les REDEFINES, les niveaux 88 et les MOVE sur des zones redéfinies.
> **Quand l'utiliser :** après les Prompts 2 et 3, avant le Prompt 4.

```
Le diff RPG ILE Free de [NOM_PROGRAMME] que nous venons de générer doit être écrit
sur l'IBM i avant compilation. Exécute les deux étapes suivantes :

Étape 1 — Écrire le source RPG sur l'IBM i (write_member) :
Écris le source RPG ILE Free dans [NOM_LIB]/QRPGSRC([NOM_PROGRAMME])
en utilisant le contenu du diff validé dans cette conversation.
Si le membre n'existe pas encore : le créer avec le type source RPGLE.
Si le membre existe déjà (programme COBOL converti sur un RPG partiellement existant) :
remplacer le contenu intégralement.

Étape 2 — Lancer la compilation :
Lance la compilation sur l'IBM i de test.
Choisir la commande selon le contenu du source (détecté au Prompt 0) :

Si pas de EXEC SQL dans le source COBOL d'origine :
CRTBNDRPG PGM([NOM_LIB_TEST]/[NOM_PROGRAMME])
          SRCFILE([NOM_LIB]/QRPGSRC)
          SRCMBR([NOM_PROGRAMME])
          OPTION(*EVENTF *LIST)
          DBGVIEW(*SOURCE)

Si EXEC SQL présent dans le source COBOL d'origine :
CRTSQLRPGI OBJ([NOM_LIB_TEST]/[NOM_PROGRAMME])
           SRCFILE([NOM_LIB]/QRPGSRC)
           SRCMBR([NOM_PROGRAMME])
           OPTION(*EVENTF *LIST)
           DBGVIEW(*SOURCE)
           OBJTYPE(*PGM)

Analyse le résultat et produis en français :

1. Statut : COMPILATION RÉUSSIE / ERREURS DE COMPILATION
2. Si erreurs : liste des erreurs avec numéro de ligne, code erreur IBM i et description
   | Ligne | Code erreur | Description | Cause probable |
   Causes probables à vérifier : variable non déclarée (niveau COBOL non transposé),
   type incompatible dans EVAL (PIC COBOL mal converti en type RPG),
   DCL-F manquante (FD COBOL non transposé en Prompt 2),
   EXSR sans BEGSR correspondant (paragraphe non transposé dans cette passe),
   OVERLAY sur une variable de mauvaise longueur (REDEFINES mal calculé)
3. Si avertissements de troncature ou de conversion implicite : les lister séparément
4. Si compilation réussie : confirmer que le programme [NOM_PROGRAMME] est compilé dans [NOM_LIB_TEST]
5. Rappeler que la compilation réussie ne garantit pas la non-régression fonctionnelle —
   la validation par un expert COBOL + RPG et les tests fonctionnels restent obligatoires
```

> 💡 **Ce prompt s'exécute en mode Agent** — Bob doit pouvoir lancer `CRTBNDRPG` via IBM i MCP.

> ⚠️ **Ce prompt ne remplace pas la validation experte ni le test fonctionnel.** Les vérifications suivantes restent obligatoires :
> - Validation par un expert COBOL + RPG des zones marquées "⚠️ DÉCISION REQUISE" dans le diff
> - Vérification des REDEFINES transposés en OVERLAY : accéder aux deux vues de la même zone mémoire avec des jeux de tests qui déclenchent les deux chemins de lecture
> - Comparaison des résultats du programme RPG converti avec ceux du programme COBOL d'origine sur un jeu de données réel

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/test-compile` — `CRTBNDRPG` est déclenché automatiquement via MCP et le résultat est formaté avec le tableau Statut / Erreurs / Cause probable.

---

### Prompt 4 — Constructs spéciaux et nettoyage

~~~
Sur la base de l'inventaire UC 1 de [NOM_PROGRAMME], section 6
(constructs sans équivalent RPG direct), et de la compilation Prompt 3-bis :

### 4a. Vérification de la DCL-PI (générée au Prompt 2)
Vérifier que la DCL-PI produite au Prompt 2 est bien présente et conforme :
- Les types RPG correspondent exactement aux PIC de la LINKAGE SECTION COBOL
- L'ordre des paramètres est préservé
- Les programmes sans LINKAGE n'ont pas de DCL-PI
Si la DCL-PI est absente ou incomplète : la régénérer maintenant avant de continuer.
Règle : ne pas modifier les types sans validation experte et sans consulter *-matrice-*.md

### 4b. PERFORM THRU — décisions de conception
Pour chaque PERFORM THRU identifié dans l'inventaire section 6 :
Option A (simple, recommandée pour le POC) :
  Regrouper les paragraphes [PARA-A] à [PARA-Z] dans une seule subroutine EXSR,
  en préservant leur ordre d'exécution.
  // AVANT COBOL : PERFORM PARA-A THRU PARA-Z
  // RPG : EXSR srParaAaZ  ← subroutine qui contient le corps de PARA-A à PARA-Z dans l'ordre
Option B (architecturale) :
  Chaque paragraphe reste une subroutine distincte ; PERFORM THRU devient une séquence d'EXSR.
  // RPG : EXSR srParaA ; EXSR srParaB ; ... EXSR srParaZ ;
Appliquer l'option décidée par l'équipe à l'issue du Prompt 0.

### 4c. Niveaux 88 — finalisation
Vérifier que chaque niveau 88 de l'inventaire section 3 a été transposé :
- Les DCL-C sont déclarées dans la section des constantes (après CTL-OPT, avant DCL-S)
- Les variables IND sont déclarées avec les autres DCL-S
- Chaque IF / EVALUATE utilisant le nom du niveau 88 a été remplacé par la comparaison RPG

### 4d. Suppression des commentaires de validation
Retirer les commentaires // AVANT des sections validées par l'expert et compilées sans erreur.
Ne retirer que les sections dont la compilation Prompt 3-bis (ou 4-bis) n'a pas retourné d'erreur.

Règles :
- Préserver les commentaires // ⚠️ DÉCISION REQUISE jusqu'à validation experte explicite
- Ne pas inventer de logique pour les constructs ALTER GO TO ou OCCURS DEPENDING ON —
  les conserver en commentaires COBOL dans le source RPG avec un marqueur de travail restant
- Signaler chaque paragraphe du périmètre POC non couvert dans cette passe
- ⚠️ Le source RPG n'est pas « complet » tant qu'il contient des marqueurs :
  DÉCISION REQUISE / NON CONVERTI / ALTER / OCCURS DEPENDING ON non traité /
  PERFORM THRU non traité / GO TO non résolu
  Ne pas déclarer la conversion terminée dans ce cas
~~~

> 💡 **Sauvegarder le source final** (mode Agent) — compléter le fichier converti :
> ```
> "Complète le fichier {projet}-{lib}-{programme}-cobol-converti-{YYYYMMDD-HHmm}.md
>  avec les interfaces et les constructs spéciaux de cette passe."
> ```

> ⚠️ **Piège évité :** transposer une interface COBOL `PROCEDURE DIVISION USING param1` sans vérifier les types de la LINKAGE SECTION — si `param1` est `PIC X(10)` en COBOL et que le programme RPG déclare `DCL-PI *N ; param1 CHAR(20)`, l'interface est cassée. Tous les programmes qui appellent ce COBOL via `CALL 'NOM' USING` auront une incompatibilité de longueur.

---

### Prompt 4-bis — Test de compilation final

> **Ce que Bob peut faire :** écrire le source RPG final sur l'IBM i et lancer la compilation finale.
> **Ce que Bob ne peut pas faire :** valider la sémantique des REDEFINES en OVERLAY, des niveaux 88 transposés, et de l'interface d'appel.
> **Quand l'utiliser :** après le Prompt 4, avant la validation experte et les tests fonctionnels.

```
Le source RPG ILE Free de [NOM_PROGRAMME] est maintenant complet —
DATA DIVISION, PROCEDURE DIVISION, interfaces et constructs spéciaux transposés.

Étape 1 — Mettre à jour le source RPG sur l'IBM i (write_member) :
(⚠️ Vérifier avant d'écrire : le source ne doit contenir aucun marqueur DÉCISION REQUISE
ni aucun construct non converti — cf. règle Prompt 4.)
Écris le source RPG ILE Free final dans [NOM_LIB]/QRPGSRC([NOM_PROGRAMME])
avec le contenu complet issu de cette conversation (Prompts 2 + 3 + 4 assemblés).
Si le membre a déjà été écrit au Prompt 3-bis : remplacer son contenu intégralement.

Étape 2 — Lancer la compilation finale.
Choisir la commande selon le contenu du source (détecté au Prompt 0) :

Si pas de EXEC SQL dans le source COBOL d'origine :
CRTBNDRPG PGM([NOM_LIB_TEST]/[NOM_PROGRAMME])
          SRCFILE([NOM_LIB]/QRPGSRC)
          SRCMBR([NOM_PROGRAMME])
          OPTION(*EVENTF *LIST)
          DBGVIEW(*SOURCE)

Si EXEC SQL présent dans le source COBOL d'origine :
CRTSQLRPGI OBJ([NOM_LIB_TEST]/[NOM_PROGRAMME])
           SRCFILE([NOM_LIB]/QRPGSRC)
           SRCMBR([NOM_PROGRAMME])
           OPTION(*EVENTF *LIST)
           DBGVIEW(*SOURCE)
           OBJTYPE(*PGM)

Produis :
1. Statut : COMPILATION RÉUSSIE / ERREURS DE COMPILATION
2. Si erreurs : liste avec cause probable
   | Ligne | Code erreur | Description | Cause probable |
3. Si compilation réussie : confirmer que [NOM_PROGRAMME] est prêt pour la validation experte
4. Rappeler les 3 points de validation obligatoires après compilation :
   - Validation experte des zones marquées "⚠️ DÉCISION REQUISE" dans le diff
   - Test fonctionnel sur IBM i de test avec jeu de données réel incluant les cas REDEFINES
   - Comparaison résultat-par-résultat avec le programme COBOL d'origine
```

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/test-compile`.

---

### Prompt 5 — Plan de conversion pour un périmètre applicatif COBOL complet

```
Sur la base des fichiers de compréhension ({projet}-{lib}-*-comprehension-*.md)
et des analyses de conversion UC 1 déjà produites ({projet}-{lib}-*-analyse-cobol-*.md)
pour l'application [NOM_APPLICATION] dans [NOM_LIB] ([NOM_LIB_COBOL] pour les sources COBOL),
génère un plan de conversion COBOL → RPG ILE Free en français, en markdown.

## Plan de conversion UC 1 — [NOM_APPLICATION]

### 1. Périmètre des programmes COBOL à convertir
| Programme | Paragraphes | PERFORM THRU | REDEFINES | Niveaux 88 | CALL IBM i | Catégorie (S/St/C) | Priorité |
Catégorie : S = SIMPLE / St = STANDARD / C = COMPLEXE (critères UC 1)
Priorité : décroissante selon "valeur de la conversion × risque acceptable"

### 2. Programmes à reporter ou à traiter en architecture hybride
Les programmes COMPLEXE pour lesquels une conversion complète dépasse le périmètre POC —
avec la justification et la proposition d'architecture hybride COBOL + RPG (si applicable).

### 3. Dépendances entre programmes COBOL et programmes RPG existants
Les programmes COBOL qui appellent des programmes RPG (CALL) ou qui sont appelés depuis RPG —
impact sur les interfaces après conversion.

### 4. Risques identifiés
Les 3 à 5 conversions les plus risquées — REDEFINES complexes, PERFORM THRU étendus,
interfaces CALL utilisées par de nombreux appelants RPG.

### 5. Estimation d'effort
| Programme | Prompts nécessaires | Durée estimée (avec Bob) | Durée validation experte | Risque résiduel |

Ne pas inventer de programmes ou de structures non visibles dans les sources disponibles.
```

> 💡 **Sauvegarder ce plan** (mode Agent) :
> ```
> "Sauvegarde ce plan de conversion dans un fichier nommé
>  {projet}-{lib}-{lib}-plan-cobol-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-APPVTE-plan-cobol-20250622-0800.md"
> ```

> 💡 Ce plan est le document de pilotage de UC 1. Il permet d'identifier les programmes convertibles directement, ceux nécessitant une architecture hybride, et les dépendances entre les programmes COBOL et l'existant RPG.

> 🏭 **Si l'atelier d'industrialisation a été mis en place :** taper `/gen-conv-plan` — même commande que UC 2, adaptée au contexte COBOL par les variables de contexte injectées.

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Ouverture des membres sources COBOL (`QCBLSRC`), création du nouveau membre RPG (`QRPGSRC`), compilation et navigation dans les erreurs inline (Prompts 3-bis, 4-bis) |
| **IBM i Languages** | Coloration syntaxique RPG ILE Free et COBOL — indispensable pour valider côte à côte le code COBOL d'origine et l'équivalent RPG généré dans l'éditeur |
| **Markdown All in One** | Prévisualisation des analyses de conversion, des diffs et des plans sauvegardés |

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture des membres sources COBOL (`QCBLSRC`) via `read_member` ; écriture du nouveau membre RPG dans `QRPGSRC` via `write_member` ; compilation via `execute_compile_action` ou `execute_cl_command` (Prompts 3-bis, 4-bis) |
| **IBM i Database MCP** | Optionnel — vérifier que les fichiers accédés par les FD COBOL ont bien des descriptions DDS ou DDL dans les bibliothèques ACME (pour construire les `DCL-DS LIKEREC` au Prompt 2) |
| **Confluence MCP** *(si disponible)* | Publication des analyses de conversion et des diffs validés dans l'espace POC |

> 💡 **Requêtes QSYS2 utiles pour UC 1 (si IBM i Database MCP actif) :**
> ```sql
> -- Vérifier que les fichiers référencés dans les FD COBOL ont une description dans le catalogue
> SELECT TABLE_NAME, TABLE_SCHEMA, TABLE_TYPE, SYSTEM_TABLE_NAME
> FROM QSYS2.SYSTABLES
> WHERE TABLE_SCHEMA = '[NOM_LIB]'
>   AND TABLE_NAME IN ('[NOM_FICHIER_FD1]', '[NOM_FICHIER_FD2]')
> ORDER BY TABLE_NAME;
>
> -- Lister les membres COBOL d'une bibliothèque (pour construire le plan Prompt 5)
> SELECT SYSTEM_TABLE_MEMBER, SOURCE_TYPE, LAST_SOURCE_UPDATE_TIMESTAMP, TEXT_DESCRIPTION
> FROM QSYS2.SYSPARTITIONSTAT
> WHERE SYSTEM_TABLE_SCHEMA = '[NOM_LIB_COBOL]'
>   AND (SOURCE_TYPE IN ('CBL', 'CBLLE', 'SQLCBLLE', 'COB', 'COBOL')
>        OR SOURCE_TYPE IS NULL)   -- membres sans type source explicite (migrations anciennes)
> ORDER BY SYSTEM_TABLE_MEMBER;
> ```
> Ces requêtes permettent de valider que les fichiers accédés par les FD COBOL sont bien présents en DDL/DDS avant de démarrer la conversion des F-specs.

> 💡 **ARCAD MCP : NON DISPONIBLE dans ce POC.** Voir la section "Spécificité ARCAD" ci-dessus.

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Sauter le Prompt 0 sur un programme COMPLEXE | Les PERFORM THRU et les REDEFINES multi-niveaux ne sont pas détectés — la conversion est lancée sans périmètre délimité et produit un source RPG avec des zones incorrectes | Toujours lancer le Prompt 0 en premier — il prend 5 minutes et évite une journée de débogage sur des REDEFINES mal transposés |
| Transposer PIC COMP sans vérifier COMPASBIN | PIC 9(4) COMP converti en INT(5) sans connaître l'option de compilation — avec NOCOMPASBIN (défaut IBM i), COMP est traité comme COMP-3 (packed) et doit devenir PACKED, pas INT ; avec COMPASBIN seulement, déterminer INT ou UNS selon le signe et le nombre de chiffres | Le Prompt 0 recherche COMPASBIN/NOCOMPASBIN avant toute conversion de COMP ; tout COMP sans option connue est marqué À CONFIRMER dans l'inventaire |
| Transposer REDEFINES sans calculer l'offset exact | DCL-DS OVERLAY(NomVar) positionné au début de la zone au lieu de l'offset correct — accès mémoire décalé, résultats silencieusement incorrects | Le Prompt 1 section 2 calcule l'offset de chaque REDEFINES ; les REDEFINES "À CONFIRMER" sont marqués pour validation experte |
| Ignorer les niveaux 88 lors de la transposition | Les conditions nommées COBOL disparaissent du source RPG — les développeurs qui maintiennent le code RPG perdent la documentation métier portée par les noms de niveaux 88 | Le Prompt 1 section 3 inventorie tous les niveaux 88 ; le Prompt 4c vérifie que chacun a été transposé en DCL-C ou IND |
| Transposer PERFORM THRU mécaniquement en EXSR séquentiels sans vérifier l'ordre | Si les paragraphes COBOL ont des dépendances de données entre eux (un paragraphe écrit dans une zone que le suivant lit), un EXSR hors ordre produit une logique incorrecte | Le Prompt 4b explicite les deux options et demande à l'équipe de choisir — Option A (regroupement) préservant l'ordre COBOL est plus sûre |
| Modifier l'interface LINKAGE SECTION lors de la transposition | Tous les programmes qui appellent ce COBOL avec CALL 'NOM' USING ont une incompatibilité de types ou de longueurs | Le Prompt 4a impose que l'interface RPG accepte exactement les mêmes types et longueurs que le COBOL ; la matrice *-matrice-*.md liste les appelants à recompiler si malgré tout l'interface doit évoluer |
| Traiter un programme COMPLEXE sans limiter le périmètre | La session produit un source RPG incomplet (certains paragraphes marqués "À traiter") et non compilable — résultat inutilisable | Le Prompt 0 délimite explicitement le périmètre POC ; les paragraphes hors périmètre restent en commentaires COBOL dans le source RPG avec un marqueur de travail restant |
| Rester en mode Agent pendant la transposition | Bob peut écrire un source RPG avec des REDEFINES incorrects ou des interfaces cassées avant validation experte | Rester en mode **Ask** pendant Prompts 0 à 4 — mode Agent uniquement pour Prompts 3-bis et 4-bis (compilation) et sauvegarde finale |
| Valider la conversion sans expert COBOL + RPG | La compilation réussit ; les cas nominaux fonctionnent — une zone REDEFINES incorrecte ou un niveau 88 manqué ne se manifeste que sur les cas limites en production | La validation experte est un prérequis non négociable pour UC 1 ; elle est documentée dans la check-list avant passage en production |

---

## Check-list de validation UC 1

Avant de passer à UC 13 (tests de non-régression) ou de déclarer un programme converti, valider chaque point :

- [ ] **Pour chaque programme converti : le Prompt 0 a été exécuté** — le fichier d'analyse `{projet}-{lib}-{programme}-analyse-cobol-{YYYYMMDD-HHmm}.md` existe et mentionne la catégorie (SIMPLE / STANDARD / COMPLEXE), le périmètre retenu et les constructs sans équivalent RPG identifiés
- [ ] Les programmes dont le périmètre de conversion a été restreint (COMPLEXE) sont documentés dans le plan (Prompt 5) avec la justification — aucun programme COMPLEXE n'a été traité en une seule session sans décision explicite de périmètre
- [ ] Chaque zone REDEFINES "À CONFIRMER" a fait l'objet d'une validation experte documentée — aucun REDEFINES multi-niveaux n'a été transposé en OVERLAY sans vérification des offsets et des longueurs
- [ ] Chaque niveau 88 COBOL a été transposé en constante `DCL-C` ou variable `IND` — aucune condition `IF NOM-NIVEAU-88` ne reste dans le source RPG sans équivalent
- [ ] L'interface d'appel (LINKAGE SECTION / PROCEDURE DIVISION USING) a été transposée en `DCL-PI` avec les mêmes types et longueurs que le COBOL d'origine — la matrice `*-matrice-*.md` a été consultée pour identifier les appelants
- [ ] Les PERFORM THRU ont été traités selon l'option décidée par l'équipe (Prompt 4b) — aucun PERFORM THRU ne reste sans équivalent RPG dans le périmètre converti
- [ ] Chaque programme converti a été **compilé sans erreur** sur l'IBM i de test (Prompts 3-bis, 4-bis)
- [ ] **La validation par un expert COBOL + RPG a été effectuée** — toutes les zones marquées "⚠️ DÉCISION REQUISE" dans le diff ont été tranchées et documentées
- [ ] Chaque programme converti a été **testé fonctionnellement** sur l'IBM i de test avec un jeu de données réel — les résultats du programme RPG sont identiques à ceux du programme COBOL d'origine
- [ ] Le plan de conversion (Prompt 5) est produit et sauvegardé dans `{projet}-{lib}-{lib}-plan-cobol-{YYYYMMDD-HHmm}.md` — tous les programmes du périmètre COBOL sont listés avec leur catégorie et leur statut
- [ ] Les diffs de conversion sont sauvegardés avec la convention `{projet}-{lib}-{programme}-cobol-converti-{YYYYMMDD-HHmm}.md` dans le workspace ET publiés sur Confluence (si MCP disponible) — ce fichier est l'**input obligatoire de UC 13** (tests de non-régression)
- [ ] La mention `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` est présente dans l'en-tête de chaque source RPG converti

---

## Points à compléter avant passage en production

> Ces points ne bloquent pas le POC — ils concernent des cas avancés peu probables sur les programmes COBOL pilotes. Ils deviennent critiques dès que la conversion s'étend à l'ensemble du parc COBOL en production.

### Programmes COBOL avec OCCURS DEPENDING ON

**Contexte :** le construct `OCCURS n TO m TIMES DEPENDING ON variable` définit un tableau à taille variable en runtime. En RPG ILE, les tableaux ont une taille fixe définie à la compilation — il n'existe pas d'équivalent direct. La conversion nécessite soit de dimensionner le tableau RPG à sa taille maximale (`OCCURS m`), soit d'utiliser une liste chaînée ou une structure dynamique IBM i (plus complexe). Dans le POC, les programmes pilotes COBOL n'ont pas de `OCCURS DEPENDING ON`.

**Pourquoi absent de la fiche POC :** les pilotes ont été choisis sans ce construct. Les programmes avec `OCCURS DEPENDING ON` sont souvent des programmes de traitement de messages ou de données à longueur variable — plus complexes à convertir.

**À faire avant production :** définir une convention de conversion pour chaque usage de `OCCURS DEPENDING ON` dans le parc : tableau à taille maximale (si la taille max est connue et raisonnable), liste chaînée `%ALLOC` / `%REALLOC` (si la taille varie fortement), ou conservation dans un programme COBOL autonome callable depuis RPG.

> ⚠️ **Signal d'alerte sur le terrain :** si le Prompt 0 retourne `OCCURS DEPENDING ON : OUI` — arrêter et planifier une réunion de conception avec l'équipe pour décider de la stratégie de conversion avant de démarrer. Ne pas tenter la conversion sans décision explicite.

### Architecture hybride COBOL + RPG pour les programmes COMPLEXE non convertibles

**Contexte :** certains programmes COBOL legacy contiennent des constructs trop risqués à convertir dans un cadre POC (ALTER GO TO, PERFORM THRU extensifs sur 20+ paragraphes, logique transactionnelle imbriquée). Pour ces programmes, une architecture hybride est possible : la logique COBOL résiduelle est préalablement extraite dans un **programme COBOL autonome callable**, que le code RPG ILE appelle via `CALLP` — permettant une migration progressive par bloc fonctionnel plutôt qu'une conversion totale. Dans le POC, les pilotes COBOL ont été choisis pour être intégralement convertibles.

> ⚠️ Il n'est pas possible d'appeler un paragraphe COBOL **interne** depuis RPG. La logique non convertie doit d'abord être isolée dans un programme COBOL autonome avec sa propre `PROCEDURE DIVISION` et son interface `LINKAGE SECTION` avant d'être callable.

**Pourquoi absent de la fiche POC :** les pilotes n'ont pas de constructs non convertibles. L'architecture hybride est un pattern de migration progressive qui dépasse le périmètre d'un POC de démonstration.

**À faire avant production :** définir les règles d'architecture hybride : quels constructs justifient de conserver le COBOL dans un programme autonome ? Quel est le plan de migration progressive ? Comment tester l'interface RPG → COBOL séparé → RPG ?

> ⚠️ **Signal d'alerte sur le terrain :** si après le Prompt 0 plus de 20 % des paragraphes sont classés "à reporter" — envisager l'architecture hybride explicitement plutôt que de forcer la conversion. Un programme RPG avec 30 % de code non converti géré par des CALL vers COBOL est plus maintenable qu'un source RPG avec 30 % de code commenté en attente.

---

*Fiche UC 1 — Document évolutif à mettre à jour au fil du POC.*
