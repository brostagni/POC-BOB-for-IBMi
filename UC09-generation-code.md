# UC 9 — Génération de code

> **Catégorie :** Développement
>
> **Priorité dans le POC :** 11a — Track B Phase 4, parallélisable avec UC 11 sur des objets différents, après UC 7/8 sur le même périmètre
>
> **Durée POC (avec Bob) :** 2 à 4 heures — 1 mode personnalisé construit + 3 à 5 programmes générés et compilés
>
> **Durée PROD (avec Bob) :** 15 à 45 min / programme selon complexité
>
> **Durée PROD (sans Bob) :** 2 à 4 heures / programme — rédaction manuelle du source, respect des normes, header, tests
>
> **Gain Bob estimé :** ~8× — un programme batch STANDARD avec header, structures de données et logique CRUD généré en 20 min au lieu de 3 heures
>
> **Mode Bob recommandé :** IBM i Developer (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : Ask pour la qualification/génération, Agent pour la compilation et la sauvegarde.

---

## Objectif

Générer des **programmes RPG ILE Free et des procédures** conformes aux normes de développement ACME, *ex nihilo* — sans programme source à migrer ni à moderniser.

**Ce UC est fondamentalement différent de tous les UC précédents.** UC 1 à UC 8 modernisent, convertissent ou documentent du code existant. UC 9 crée du nouveau code à partir d'une description fonctionnelle et des normes de l'entreprise. Il n'y a pas de comportement antérieur à préserver — le risque de non-régression par rapport à un source remplacé est nul. En revanche, le risque fonctionnel, transactionnel et d'intégrité des données reste entier : un programme nouveau peut corrompre des données, introduire des verrous ou créer une incompatibilité d'interface.

**Livrable principal :** pour chaque programme généré, un fichier source RPG (ou diff commenté) respectant les normes ACME.

**Livrable secondaire (UC 9 uniquement) :** le mode personnalisé "ACME Developer" — fichier `.bob/custom_modes.yaml` exportable et distribuable à l'équipe Bob.

**Convention de nommage des fichiers générés :**
```
{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md

Types pour cet UC :
  programme-genere  → source RPG généré ou diff commenté (Prompts 1, 2)
  plan-generation   → backlog priorisé de programmes à générer (Prompt 4)
```

Exemples :
```
acme-APPVTE-PGMUTIL-programme-genere-20250622-1000.md   ← source généré
acme-APPVTE-APPVTE-plan-generation-20250622-0800.md     ← plan périmètre complet
```

> 💡 Cette convention est valable en dehors du contexte POC — réutilisable en production tel quel.

---

## Démarrer par un programme connu

> **Recommandation forte avant d'aborder les programmes de production ACME.**

Commencer par un **programme utilitaire simple** (< 100 lignes, un seul objectif) dont un développeur ACME peut valider immédiatement que les normes sont respectées — header conforme, types de données corrects, conventions de nommage respectées.

Pourquoi ? Parce que la première génération sert à **calibrer deux choses** :
- La fidélité du mode personnalisé "ACME Developer" (Bob applique-t-il vraiment les conventions de nommage, le header standard, les types de données préférés ?)
- La capacité de l'équipe à valider le code généré avant de l'envoyer en compilation

| Étape | Programme à choisir | Catégorie attendue | Objectif |
|-------|--------------------|--------------------|---------|
| 1 | Programme utilitaire ou service, < 100 lignes, 1 à 3 paramètres, pas de fichier | SIMPLE | Calibrer le header, les DCL-S et les conventions de nommage |
| 2 | Programme de service avec paramètres et logique de calcul ou validation | STANDARD | Valider les structures de données et la gestion de paramètres |
| 3 | Programme batch avec 1 à 3 fichiers et logique CRUD | STANDARD | Valider la génération des DCL-F et des procédures d'accès |
| 4 | Programme interactif 5250 avec display file | COMPLEXE → préférer UC 10 | UC 9 gère le programme RPG ; UC 10 gère le display file |

---

## Impact de la complexité sur la stratégie de génération

La complexité d'UC 9 ne se mesure pas en lignes de code, mais en **type de programme et en nombre d'interfaces** — fichiers accédés, paramètres reçus, appels externes.

| Catégorie | Critères | Recommandation |
|-----------|----------|----------------|
| **SIMPLE** | Programme utilitaire ou service sans état, 1 à 3 paramètres, aucun fichier | Génération en une session — Prompt 0 → Prompt 1 → Prompt 2 → Prompt 2-bis |
| **STANDARD** | Programme batch avec 1 à 3 fichiers, logique CRUD, validation de paramètres | Génération en deux passes — header/DCL d'abord, logique métier ensuite |
| **COMPLEXE** | Programme interactif, plusieurs fichiers liés, logique métier significative, appels externes | UC 9 pour la partie RPG — **UC 10 est plus adapté si l'interface 5250 est au cœur du besoin** |

> 💡 Le Prompt 0 donne la catégorie et la séquence exacte — toujours démarrer par le Prompt 0, pas directement par Prompt 1.

---

## Démarrer une session Bob

> **À lire avant chaque session UC 9 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

Chaque session de génération d'un programme doit démarrer dans une **nouvelle conversation Bob** (bouton `+` en haut du panneau Chat). Ne pas réutiliser une conversation d'un autre programme — les normes et structures de données d'un autre programme peuvent contaminer les conventions du programme en cours.

**Exception :** enchaîner dans la même session si deux programmes appartiennent au même domaine fonctionnel et partagent les mêmes structures de données — économie de Bob Coins sur le chargement des normes.

**Mode à sélectionner :** Ask ou ACME IBM i Review (génération), Agent ou ACME IBM i Execute (compilation et sauvegarde).

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Si le programme généré remplace ou s'intègre à un programme existant, ouvrir ce programme dans l'éditeur Bob avant de lancer le Prompt 0. L'ouverture le rend accessible au MCP IBM i sans copier-coller.

**Procédure :** dans le panneau **IBM i — Object Browser** (extension Code for IBM i), naviguer jusqu'à la bibliothèque source, faire un clic droit sur le membre RPG → **Open in Editor**.

Fichiers à ouvrir pour chaque session UC 9 :
- Le programme existant de référence, si le programme généré s'inscrit dans une application existante
- Les fichiers de normes ACME (si disponibles sous forme de fichier texte dans le workspace)
- Le mode "ACME Developer" (si déjà créé lors d'une session précédente)

### 3. Fichiers de contexte à charger

Ces fichiers doivent être disponibles dans le workspace Bob **avant** de démarrer. Utiliser **Add File to Chat** (icône trombone) ou les ouvrir dans l'éditeur.

| Fichier | Produit par | Obligatoire / Recommandé |
|---------|-------------|--------------------------|
| `*-spec-fonc-{date}.md` | UC 6 | **Recommandé** — si le programme s'intègre dans une application documentée |
| `*-spec-tech-{date}.md` | UC 6 | **Recommandé** — si le programme accède des fichiers existants |
| `*-ddl-complet-{date}.md` | UC 14 | **Recommandé** — si le programme accède des tables DDL (génération des DCL-F cohérentes) |
| Mode "ACME Developer" (`.bob/custom_modes.yaml`) | UC 9 (session précédente) | **Recommandé** — si le mode a été créé, le charger pour éviter de recharger les normes manuellement |

> 💡 **Si aucune norme n'est disponible :** utiliser le Prompt 3 pour construire le mode personnalisé "ACME Developer" *avant* le Prompt 1. Les normes définies au Prompt 3 servent ensuite de base à tous les Prompts 1 et 2 de la session.

> 💡 **Reprise de session :** si la génération d'un programme est interrompue, recharger le fichier `*-programme-genere-*.md` produit jusqu'ici dans le contexte Bob. Indiquer dans le prompt "le source en cours est dans [NOM_FICHIER] — reprendre à partir du Prompt [N°]".

> 💡 **Lien vers l'UC suivant :** les fichiers `*-programme-genere-*.md` produits dans cet UC sont les inputs de UC 13 (tests). Voir la Carte des livrables dans `plan-poc-bob-acme.md`.

---

## Prérequis

- UC 15 (maîtrise de Bob) et UC 12 Phase 0 (MCP) complétés
- Normes de développement ACME disponibles : au minimum les conventions de nommage, le header standard et les types de données préférés — à récupérer auprès de l'équipe avant la session
- Optionnel : `*-spec-fonc-*.md` et `*-spec-tech-*.md` (UC 6) si le programme s'intègre dans une application documentée
- Optionnel : `*-ddl-complet-*.md` (UC 14) si le programme accède des tables DDL

> 💡 Si les normes ACME ne sont pas disponibles sous forme documentée, demander à Bob d'analyser 2 à 3 programmes RPG existants (via UC 4) et d'en extraire les patterns communs — le Prompt 3 documente ce processus.

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | **IBM i Developer** (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : **Ask** pour la qualification/génération, **Agent** pour la compilation et la sauvegarde. |
| **Scope** | Library List → bibliothèque applicative ACME |
| **MCP actifs** | IBM i MCP (compilation CRTBNDRPG, lecture des sources de référence) |
| **MCP différés** | IBM i Database MCP (introspection QSYS2 pour les programmes accédant des tables), Confluence MCP (publication, si token disponible) |

### Pourquoi le mode IBM i Developer pour la génération de code ?

Le mode **IBM i Developer** apporte la connaissance RPG/ILE spécialisée pour toute la session — conventions de nommage IBM i, structures DCL-F/DCL-DS, templates CRUD, patterns ILE. La discipline de travail repose sur **la validation humaine avant toute écriture** : Bob génère le source dans le chat, l'équipe valide le header, les conventions et la logique, puis autorise explicitement l'écriture sur l'IBM i.

> 💡 **Sans Premium Package IBM i :** remplacer IBM i Developer par le mode **Ask** pour la qualification et la génération, et le mode **Agent** pour la compilation et la sauvegarde. La discipline de validation reste identique.

| Phase | Comportement attendu | Ce que Bob fait |
|-------|----------------------|----------------|
| Qualification du programme (Prompt 0) | Génère dans le chat — pas d'écriture | Analyse le besoin, détermine la catégorie (SIMPLE/STANDARD/COMPLEXE), recommande la séquence |
| Génération header + DCL (Prompt 1) | Génère dans le chat — pas d'écriture | Génère CTL-OPT, header documentaire, DCL-F, DCL-DS dans le chat |
| Génération logique métier (Prompt 2) | Génère dans le chat — pas d'écriture | Génère les procédures ou sous-routines, templates CRUD dans le chat |
| Test de compilation (Prompt 2-bis) | **Écriture et exécution autorisées** — après validation de l'équipe | Lance `CRTBNDRPG` via IBM i MCP, rapporte et corrige les erreurs |
| Création du mode personnalisé (Prompt 3) | Génère dans le chat — pas d'écriture | Génère le fichier `.bob/custom_modes.yaml` du mode "ACME Developer" dans le chat |
| Plan de génération (Prompt 4) | Génère dans le chat — pas d'écriture | Génère le backlog et le plan du mode dans le chat |
| Sauvegarde des livrables | **Écriture autorisée** — après validation complète | Écrit les fichiers `.md` dans le workspace |

> 💡 **Règle d'or pour UC 9 :** L'écriture et la compilation sur l'IBM i ne sont autorisées qu'après validation explicite de l'équipe. Pendant toute la phase de génération (Prompts 0 à 4), Bob produit uniquement dans le chat — le mode IBM i Developer le permet, mais l'équipe ne donne pas l'instruction d'écrire.

> 💡 **Règle d'or pour UC 9 :** L'écriture et la compilation sur l'IBM i ne sont autorisées qu'après validation explicite de l'équipe. Pendant toute la phase de génération (Prompts 0 à 4), Bob produit uniquement dans le chat — le mode IBM i Developer le permet, mais l'équipe ne donne pas l'instruction d'écrire.

### Spécificité ARCAD — MCP non disponible

ACME utilise ARCAD pour la gestion du code source IBM i. Le MCP ARCAD n'est **pas actif** dans ce POC (incompatibilité de version).

**Impact sur UC 9 : faible.** Les programmes générés sont de nouveaux membres sources (`QRPGSRC`) qui n'existent pas encore dans ARCAD. Ces nouveaux membres devront être enregistrés manuellement dans ARCAD après validation.

| Ce que l'absence du MCP ARCAD change | Ce qui fonctionne quand même |
|--------------------------------------|------------------------------|
| Impossible de créer automatiquement le nouveau membre dans ARCAD lors de la génération | IBM i MCP peut créer le membre directement dans les bibliothèques source — ARCAD le verra lors de la synchro manuelle |
| Impossible de vérifier si un programme de même nom est déjà géré dans ARCAD | Vérification manuelle dans l'interface ARCAD avant de nommer le programme cible |
| Les programmes générés ne sont pas automatiquement intégrés dans les packages de déploiement ARCAD | Réintégration manuelle — ajouter le placeholder `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` dans l'en-tête de chaque source généré |

> 💡 **Contournement :** avant de générer un programme, vérifier dans ARCAD qu'aucun membre de même nom n'est en cours de modification. Documenter le nom du programme généré dans le fichier `*-plan-generation-*.md` pour faciliter la réintégration ARCAD post-validation.

---

## Prompts clés

> 💡 **Atelier Bob Industrialisation**
> Les prompts récurrents (Prompt 0, génération header, génération logique) peuvent être disponibles sous forme de **commandes slash personnalisées** dans le mode "ACME Developer" une fois celui-ci créé.
> Voir la fiche `atelier-bob-industrialisation-prompts.md` pour la liste complète des commandes disponibles.

### Prompt 0 — Qualification du programme à générer

> **Ce prompt est le point d'entrée obligatoire de UC 9 pour chaque programme.**
> Il traduit le besoin fonctionnel en catégorie technique et recommande la séquence de prompts adaptée.
> Il se lance **avant** le Prompt 1 — son résultat conditionne toute la séquence suivante.

```
Je dois générer un programme RPG ILE Free pour [NOM_LIB].

Besoin fonctionnel : [décrire ce que doit faire le programme — en termes métier, pas techniques]
[Si disponible : La spécification fonctionnelle {appArcad}-{fonction}-{composant}-spec-fonc-{date}.md est disponible.]
[Si disponible : La spécification technique {appArcad}-{fonction}-{composant}-spec-tech-{date}.md est disponible.]

Produis en français, en markdown, une fiche de qualification pour la génération :

## Qualification UC 9 — [NOM_PROGRAMME_CIBLE]

### 1. Analyse du besoin
| Critère | Valeur détectée |
|---------|----------------|
| Type de programme (utilitaire / service / batch / interactif) | ? |
| Nombre de paramètres d'entrée | ? |
| Fichiers ou tables accédés | ? (liste) |
| Appels externes (CALLP vers d'autres programmes) | OUI / NON |
| Présence d'une interface utilisateur (display file) | OUI / NON |

### 2. Catégorie recommandée
- [ ] SIMPLE — utilitaire ou service sans état, 1-3 paramètres, aucun fichier
- [ ] STANDARD — batch avec 1-3 fichiers, logique CRUD, validation de paramètres
- [ ] COMPLEXE — interactif, plusieurs fichiers liés, logique métier significative

Note : pour COMPLEXE avec interface 5250, UC 10 est plus adapté que UC 9 pour la partie display file.

### 3. Interfaces à générer
- DCL-F requises : [liste des fichiers / tables]
- DCL-DS requises : [structures de données principales]
- Paramètres (DCL-PI) : [liste type + longueur]

### 4. Séquence recommandée
Selon la catégorie :
- SIMPLE   → Prompt 1 (header + DCL) → Prompt 2 (logique) → Prompt 2-bis (compilation)
- STANDARD → Prompt 1 (header + DCL-F + DCL-DS) → Prompt 2 (logique CRUD) → Prompt 2-bis
- COMPLEXE → délimiter le périmètre ; utiliser UC 10 pour le display file si présent
```

**Analyse ligne à ligne :**

- `Besoin fonctionnel en termes métier` → décrire ce que doit faire le programme en langage métier, pas en termes techniques. "Calculer le total d'une commande et écrire le résultat dans le fichier CMDTOT" est mieux que "lire CMDDTE, boucler sur CMDLIG, écrire CMDTOT". Le second force Bob à reproduire une implémentation supposée ; le premier laisse Bob proposer la meilleure implémentation selon les normes ACME.

- `Type de programme` → la distinction utilitaire / service / batch / interactif conditionne la structure générée. Un service (appelé via CALLP) a une DCL-PI avec paramètres et un RETURN. Un batch a une DCL-F et une boucle de traitement. Un utilitaire peut ne pas avoir de DCL-F ni de DCL-PI.

- `Interfaces à générer` → lister explicitement les fichiers et tables accédés au Prompt 0 évite la principale source d'hallucination de UC 9 : Bob qui invente des noms de fichiers. Ces interfaces sont validées contre QSYS2 avant de démarrer le Prompt 1.

> 💡 **Sauvegarder la qualification** (mode Agent) dès la fin du Prompt 0 si la session est longue.

> ⚠️ **Piège évité :** sans le Prompt 0, l'équipe démarre directement la génération sans avoir défini les interfaces. Bob génère des DCL-F avec des noms de fichiers inventés ou des DCL-DS ne correspondant pas aux structures réelles de ACME.

---

### Prompt 1 — Génération du header et de la structure DCL

> **Ce prompt génère le squelette du programme — CTL-OPT, header documentaire, DCL-F, DCL-DS.**
> Il doit être précédé du Prompt 0 et, si disponible, du mode personnalisé "ACME Developer" chargé dans le contexte.

```
Sur la base de la qualification UC 9 que nous venons de faire pour [NOM_PROGRAMME_CIBLE],
génère en français le squelette RPG ILE Free conforme aux normes ACME :

## Source RPG — [NOM_PROGRAMME_CIBLE] — Section DCL

### CTL-OPT
[Générer les options de compilation ACME : DFTACTGRP, ACTGRP, OPTION, DATFMT, TIMFMT, etc.]

### Header documentaire standard ACME
[Générer le bloc de commentaires de début de source conforme aux normes du client :
 auteur, date, version, description fonctionnelle, historique des modifications]

### DCL-F — Fichiers accédés
[Pour chaque fichier de la qualification : DCL-F avec USAGE, KEYED / SEQONLY selon l'accès]

### DCL-DS — Structures de données
[Pour chaque structure identifiée : DCL-DS avec les sous-champs conformes aux conventions ACME]

### DCL-S — Variables standalone
[Variables de travail, compteurs, indicateurs — nommés selon les conventions ACME]

### DCL-PI — Interface du programme (si applicable)
[Pour un programme service ou utilitaire : paramètres d'entrée/sortie avec types et longueurs]

Appliquer les conventions ACME suivantes :
[Si mode personnalisé disponible : voir le mode "ACME Developer" chargé dans le contexte]
[Si pas de mode : appliquer les normes fournies — liste les normes à respecter]
- Convention de nommage des variables : [WK pour variables de travail / etc.]
- Convention de nommage des structures de données : [DS pour ds / etc.]
- Types de données préférés : [PACKED pour numériques / CHAR pour alphanumériques / etc.]
```

**Analyse ligne à ligne :**

- `Générer le squelette d'abord, la logique ensuite` → séparer la génération des DCL de la logique métier permet une validation étape par étape. Un header incorrect ou une DCL-F avec le mauvais accès est détecté avant d'aller plus loin.

- `CTL-OPT` → les options de compilation ACME doivent être explicites. DFTACTGRP(*NO) est la valeur moderne pour les programmes ILE. Si ACME a une convention de groupe d'activation spécifique, la préciser dans le prompt — Bob ne peut pas la deviner.

- `Convention de nommage` → c'est ici que le mode personnalisé "ACME Developer" prend toute sa valeur. Sans le mode, Bob utilise ses conventions par défaut (variables en anglais, pas de préfixe, header minimal). Avec le mode, Bob applique automatiquement WK, DS, les abbréviations métier ACME.

> 💡 **Sauvegarder le squelette** (mode Agent) avant de démarrer le Prompt 2 :
> ```
> "Sauvegarde ce squelette dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-programme-genere-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-PGMUTIL-programme-genere-20250622-1000.md"
> ```

---

### Prompt 2 — Génération de la logique métier

```
Sur la base du squelette UC 9 de [NOM_PROGRAMME_CIBLE] et de la qualification,
génère la logique métier du programme :

## Source RPG — [NOM_PROGRAMME_CIBLE] — Logique métier

### Procédure principale (ou mainline)
[Générer la logique de haut niveau : boucle de traitement, appels aux procédures]

### Procédures ou sous-routines
Pour chaque procédure identifiée au Prompt 0, générer :

DCL-PROC [nomProcedure] ;
  DCL-PI [nomProcedure] [TYPE_RETOUR] ;
    [paramètres]
  END-PI ;
  [corps de la procédure]
  RETURN [valeur] ;
END-PROC ;

Règles de génération :
- Templates CRUD selon les fichiers DCL-F générés au Prompt 1 :
  - READ : utiliser CHAIN pour accès par clé, READ pour accès séquentiel
  - WRITE : valider les paramètres obligatoires avant l'écriture
  - UPDATE : lire pour mise à jour avec CHAIN(E), READ(E), READE(E) ou équivalent ; tester %ERROR immédiatement — %ERROR n'est actif que si l'extender (E) est présent. Le verrou RPG est libéré automatiquement par UPDATE hors commitment control. Sous commitment control, les verrous transactionnels restent actifs jusqu'au COMMIT ou ROLLBACK. Si la validation entre la lecture et l'UPDATE est longue, déverrouiller explicitement avec UNLOCK avant de continuer.
  - DELETE : confirmer que l'enregistrement existe via CHAIN(E) avant DELETE
- Gestion d'erreur : utiliser l'extender (E) sur chaque opération fichier pouvant échouer (CHAIN, READ, WRITE, UPDATE, DELETE) et tester %ERROR immédiatement après. Sans l'extender (E), ne pas utiliser %ERROR pour détecter l'erreur de cette opération. Utiliser un indicateur d'erreur, un bloc MONITOR/ON-ERROR, une routine INFSR/*PSSR ou le gestionnaire d'exception par défaut.
- Validation des paramètres : vérifier les champs obligatoires en entrée de la procédure principale
- Ne pas utiliser d'opcodes obsolètes (MOVE, MOVEL, SETON, GOTO)

Expliquer brièvement chaque choix de conception (pourquoi cette structure, pourquoi ce type de données)
pour faciliter la revue développeur.
```

**Analyse ligne à ligne :**

- `Expliquer les choix de conception` → demander à Bob d'expliquer ses choix produit deux bénéfices : les développeurs comprennent le code généré, et Bob expose ses hypothèses (si une hypothèse est incorrecte par rapport aux normes ACME, elle est visible et corrigeable avant compilation).

- `Templates CRUD` → spécifier les templates d'accès évite que Bob génère un UPDATE sans maîtriser quel enregistrement est courant. `UPDATE` modifie l'enregistrement courant préalablement lu pour mise à jour par `CHAIN`, `READ`, `READE`, `READP` ou `READPE` sur un fichier déclaré en update. Ne pas exécuter `UPDATE` sans connaître explicitement l'enregistrement courant — c'est la source la plus fréquente d'écrasement de données sur IBM i.

- `Pas d'opcodes obsolètes` → sans cette instruction explicite, Bob peut utiliser MOVE ou SETON sur des programmes RPG — surtout si le style du programme de référence fourni au contexte utilise ces opcodes. La règle doit être explicite dans chaque prompt de génération.

> 💡 **Sauvegarder la logique générée** (mode Agent) — compléter le fichier existant :
> ```
> "Complète le fichier {appArcad}-{fonction}-{composant}-programme-genere-{YYYYMMDD-HHmm}.md
>  avec la logique métier générée."
> ```

> ⚠️ **Piège évité :** générer la logique sans avoir chargé le `*-ddl-complet-*.md` (UC 14). Bob invente les noms des colonnes des tables accédées — le source compile parfois (colonnes génériques), mais la logique accède les mauvaises données.

---

### Prompt 2-bis — Écriture du membre source + Test de compilation CRTBNDRPG

> **Ce prompt s'exécute en mode Agent.** Bob vérifie que le membre n'existe pas, écrit le source, puis compile.

```
Le programme [NOM_PROGRAMME_CIBLE] est prêt à être sauvegardé.

Étape 1 — Vérifier que le membre n'existe pas encore :
SELECT COUNT(*) AS MEMBER_COUNT
FROM QSYS2.SYSMEMBERSTAT
WHERE SYSTEM_TABLE_SCHEMA = '[NOM_LIB]'
  AND SYSTEM_TABLE_NAME   = '[SRCFILE_RPG]'
  AND SYSTEM_TABLE_MEMBER = '[NOM_PROGRAMME_CIBLE]';
Si MEMBER_COUNT > 0 : arrêter l'opération et signaler — ne pas écraser sans validation explicite.

Étape 2 — Écrire le source RPG validé dans le membre :
Écrire uniquement le source RPG (sans balises Markdown ni commentaires explicatifs extérieurs au source)
dans [NOM_LIB]/[SRCFILE_RPG]([NOM_PROGRAMME_CIBLE]).

Étape 3 — Compiler avec CRTBNDRPG et rapporter le résultat :
1. Si la compilation réussit sans erreur ni avertissement : confirmer
2. Si des erreurs de compilation sont présentes :
   - Lister chaque erreur (code, numéro de ligne, description)
   - Proposer la correction dans le source pour chaque erreur
   - Préciser si la correction est mécanique (syntaxe) ou nécessite une décision du développeur
3. Si des avertissements sont présents : les lister avec leur niveau de sévérité
4. Rappeler que la compilation réussie ne valide pas la logique fonctionnelle —
   un cas nominal doit être exécuté sur l'IBM i de test pour déclarer le programme validé
```

> 💡 **Ce prompt s'exécute en mode Agent** — droits `*CHANGE` sur la bibliothèque cible requis. La vérification via `QSYS2.SYSMEMBERSTAT` (Étape 1) est obligatoire : ne jamais écraser un membre existant sans confirmation explicite.

> ⚠️ **Ce prompt ne remplace pas le test fonctionnel.** La compilation valide la syntaxe, pas le comportement. Un cas nominal doit être exécuté avec des données représentatives avant de déclarer le programme validé.

---

### Prompt 3 — Création du mode personnalisé "ACME Developer"

> **Ce prompt est à exécuter en priorité si aucune norme n'est disponible avant le Prompt 1.**
> Si le mode a déjà été créé lors d'une session précédente, ce prompt devient une "Mise à jour du mode".

```
[Si les normes sont disponibles dans le contexte :]
Sur la base des normes de développement ACME fournies, génère le mode personnalisé
"ACME Developer" pour Bob en français, au format YAML (fichier `.bob/custom_modes.yaml`).

[Si les normes ne sont pas disponibles :]
Sur la base des programmes RPG existants dans [NOM_LIB]/[SRCFILE_RPG] que tu peux lire via IBM i MCP,
analyse les patterns communs (header, conventions de nommage, types de données, opcodes préférés)
et génère le mode personnalisé "ACME Developer" pour Bob en français, au format YAML (`.bob/custom_modes.yaml`).

Le mode doit inclure :

## Mode "ACME Developer" — Spécification

### roleDefinition
[Description du rôle : expert RPG ILE Free appliquant systématiquement les normes ACME]

### Conventions de nommage
| Élément | Convention ACME | Exemple |
|---------|-------------------|---------|
| Variables de travail | ? | wkNomClient |
| Structures de données | ? | dsCommande |
| Procédures | camelCase verbe + nom | calculerMontantTTC |
| Paramètres | ? | pNomParam |

### Header documentaire standard
[Bloc de commentaires en début de source — exactement tel qu'utilisé chez ACME]

### Types de données préférés
[Tableau : numérique, alphanumérique, date, heure, booléen]

### CTL-OPT standard
[Options de compilation ACME par défaut]

### Opcodes préférés
[Liste des opcodes modernes utilisés — et les opcodes obsolètes à ne jamais générer]

### Export du mode
Générer le fichier YAML complet au format `.bob/custom_modes.yaml` prêt à être placé dans le dossier `.bob/` du workspace Bob.
Le mode sera disponible pour toute l'équipe dès que le fichier est présent dans le workspace partagé.
```

**Analyse ligne à ligne :**

- `Extraire les patterns depuis des programmes existants` → si les normes ACME ne sont pas documentées, Bob peut les inférer en analysant les sources existants via UC 4. 2 à 3 programmes constituent un corpus initial pour extraire les conventions stables — les règles inférées doivent être validées par le référent technique avant intégration dans le mode. Cette approche produit un mode basé sur les pratiques réelles de l'équipe, pas sur des suppositions.

- `Format YAML pour `.bob/custom_modes.yaml`` → les modes personnalisés Bob sont enregistrés en YAML dans `.bob/custom_modes.yaml` au niveau projet. Préciser le format évite que Bob génère du JSON non interprété ou un document descriptif non importable.

- `Procédure de distribution` → le mode est un actif d'équipe, pas un outil individuel. Une fois validé sur au moins 2 programmes représentatifs, le distribuer à tous les membres de l'équipe Bob via le fichier `.bob/custom_modes.yaml` partagé.

> 💡 **Sauvegarder le mode** (mode Agent) dans `.bob/custom_modes.yaml` dans le workspace Bob — ou dans un fichier versionné séparément `acme-developer-mode-v1.yaml` pour historiser les versions. Ce fichier n'est pas dans la convention de nommage `*-programme-genere-*` mais fait partie des livrables de UC 9.

> ⚠️ **Piège évité :** distribuer le mode à l'équipe avant de le valider sur au moins 2 programmes représentatifs. Un mode avec des conventions incorrectes génère du mauvais code pour toute l'équipe — plus difficile à corriger qu'un code généré sans mode.

---

### Prompt 4 — Plan de génération et plan du mode personnalisé

```
Sur la base des fichiers de compréhension ({appArcad}-{fonction}-*-comprehension-*.md),
des spécifications techniques ({appArcad}-{fonction}-*-spec-tech-*.md) disponibles,
et du mode "ACME Developer" créé dans cette session,
génère en français, en markdown, le plan de génération pour l'application [NOM_APPLICATION].

## Plan de génération UC 9 — [NOM_APPLICATION]

### Angle 1 — Backlog de programmes à générer
| Programme | Besoin fonctionnel | Type | Catégorie (S/St/C) | Dépendances | Effort estimé | Priorité |
Catégorie : S = SIMPLE / St = STANDARD / C = COMPLEXE (critères UC 9)
Effort estimé : durée Bob avec le mode personnalisé

### Angle 2 — Plan de mise en œuvre du mode "ACME Developer"
| Étape | Action | Responsable | Délai |
|-------|--------|-------------|-------|
| Validation du mode | Tester sur 2 programmes représentatifs avant distribution | Dev référent | Avant distribution |
| Export du mode | Sauvegarder `.bob/custom_modes.yaml` dans le workspace et le partager | Tech lead | J+1 |
| Distribution à l'équipe | Import dans Bob pour chaque membre | Chaque membre | J+2 |
| Processus de mise à jour | Définir qui peut modifier les normes et comment versionner le mode | Tech lead | Avant production |
| Versionning du mode | Nommer le mode "ACME Developer v1.0" — incrémenter à chaque évolution des normes | Tech lead | En continu |

Ne pas inventer de programmes ou de structures non visibles dans les sources disponibles.
```

**Analyse ligne à ligne :**

- `Double angle` → ce prompt produit deux livrables complémentaires : le backlog opérationnel (qui sert de feuille de route pour UC 9 sur le périmètre complet) et le plan de gouvernance du mode personnalisé (qui assure que l'investissement du mode bénéficie à toute l'équipe sur la durée).

- `Versionning du mode` → les normes de développement ACME peuvent évoluer. Un mode sans version devient rapidement obsolète et difficile à tracer. La convention "ACME Developer v1.0" et son incrément à chaque évolution est une pratique à mettre en place dès la première version.

> 💡 **Sauvegarder ce plan** (mode Agent) :
> ```
> "Sauvegarde ce plan dans un fichier nommé
>  {appArcad}-{fonction}-{domaine}-plan-generation-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-APPVTE-plan-generation-20250622-0800.md"
> ```

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Ouverture des membres sources RPG de référence, création et écriture du nouveau membre, compilation et navigation dans les erreurs inline (Prompt 2-bis) |
| **IBM i Languages** | Coloration syntaxique RPG ILE Free — indispensable pour valider le source généré dans l'éditeur avant compilation |
| **Markdown All in One** | Prévisualisation des sources générés et des plans sauvegardés |

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture des sources de référence (`QRPGSRC`) pour le Prompt 3 (extraction des patterns) ; écriture du nouveau membre source ; compilation via `execute_compile_action` ou `execute_cl_command` (Prompt 2-bis) |
| **IBM i Database MCP** | Introspection QSYS2 — valider que les fichiers et tables référencés dans les DCL-F existent réellement avant de compiler |
| **Confluence MCP** *(si disponible)* | Publication des sources générés et du plan de génération dans l'espace POC |

> 💡 **Requêtes QSYS2 utiles pour UC 9 (si IBM i Database MCP actif) :**
> ```sql
> -- Valider qu'une table existe et connaître ses colonnes avant de générer les DCL-F
> SELECT COLUMN_NAME, DATA_TYPE, LENGTH, NUMERIC_SCALE, IS_NULLABLE
> FROM QSYS2.SYSCOLUMNS
> WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_TABLE]'
> ORDER BY ORDINAL_POSITION;
>
> -- Lister les programmes existants d'une bibliothèque (pour identifier les programmes à générer)
> -- OBJECT_STATISTICS est une fonction table — syntaxe obligatoire : TABLE(...)
> SELECT OBJNAME, OBJTEXT, LAST_USED_TIMESTAMP
> FROM TABLE(
>   QSYS2.OBJECT_STATISTICS(
>     OBJECT_SCHEMA    => '[NOM_LIB]',
>     OBJECT_TYPE_LIST => '*PGM'
>   )
> )
> ORDER BY OBJNAME;
> ```
> Ces requêtes permettent de valider les interfaces avant de générer les DCL-F — première ligne de défense contre les hallucinations de noms de colonnes.

> ⚠️ **ARCAD MCP : NON DISPONIBLE dans ce POC.** Voir la section "Spécificité ARCAD" ci-dessus.

---

## Cas avancé — Ajout d'un champ end-to-end (feature cross-couches)

Ce pattern correspond à l'ajout d'un nouveau champ qui traverse toutes les couches d'une application IBM i existante : base de données (PF DDS), fichier logique (LF DDS), display file (DSPF), et programme RPG. C'est un cas d'usage fréquent en maintenance évolutive — différent de la génération d'un programme nouveau.

> ⚠️ **Ce cas est une extension de UC 9 + UC 14 combinés.** Il ne s'agit pas de créer un programme nouveau (UC 9 standard) ni de convertir un DDS existant (UC 14 standard) — il s'agit de **propager une modification** cohérente à travers plusieurs objets liés. Démarrer par un scope explicite : lister les 4 objets impactés avant toute génération.

**Séquence recommandée :**

| Étape | Objet | Opération | Risque principal |
|-------|-------|-----------|-----------------|
| 1 | Fichier physique (PF DDS) | Ajouter le champ avec son type, longueur, `COLHDG` | Attention à ne pas modifier les clés (`K`) ni les champs existants |
| 2 | Fichier logique (LF DDS) | Ajouter l'alias `RENAME` du nouveau champ | La ligne `K [CLE]` doit rester intacte |
| 3 | Display file (DSPF) | Ajouter le champ screen avec position, label, `CHECK(RZ)`, **pas de `COLHDG`** | Vérifier la position — chevauchement avec un champ existant = erreur de compilation |
| 4 | Programme RPG OPM/ILE | Propager le champ dans les specs I (input), C (calcul) et O (output) | Voir piège O-specs OPM ci-dessous |
| 5 | Compilation + validation | Dans l'ordre : PF → LF → DSPF → RPG | Un échec à une étape bloque les suivantes — s'arrêter et corriger |

**Prompts à utiliser :**

```
Je dois ajouter le champ [NOM_CHAMP] (type [TYPE], longueur [LG]) à travers les couches
suivantes de l'application [NOM_LIB] :
1. Fichier physique (PF) : [NOM_PF] dans [NOM_LIB]/QDDSSRCF
2. Fichier logique (LF) : [NOM_LF] dans [NOM_LIB]/QDDSSRCF
3. Display file (DSPF) : [NOM_DSPF] dans [NOM_LIB]/QDDSSRCD
4. Programme RPG : [NOM_RPG] dans [NOM_LIB]/QRPGSRC

Commence par lister pour chaque objet les lignes à modifier ou ajouter (diff uniquement),
sans générer de code complet. Attends ma validation avant de démarrer les modifications.
Pour chaque objet, procède séparément — montrer le diff, attendre l'approbation, puis sauvegarder
le membre uniquement (sans compiler). Compilation uniquement à l'étape 5, dans l'ordre donné.
```

> ⚠️ **Piège O-specs OPM RPG (alignement colonne critique) :** dans un programme RPG OPM (format colonné), les O-specs (spécifications de sortie) sont **sensibles à l'alignement de colonne**. La position de fin de champ est une valeur décimale positionnée exactement dans les colonnes 40-43. Si Bob génère la ligne en estimant l'espacement visuellement plutôt qu'en copiant le modèle exact d'une ligne voisine (ex. la ligne du champ `FMILES`), il peut positionner `234` à la place de `233` — ce qui provoque une erreur de compilation silencieuse ou un accès au mauvais champ. **Instruction à inclure dans le prompt :** *"Pour les O-specs, copie l'alignement exact de la ligne voisine et change uniquement le nom du champ et la valeur de fin — ne recalcule pas les colonnes à partir de zéro."*

> 💡 **Stratégie 3 diffs pour les O-specs :** sur un programme OPM RPG dense, scinder la modification en 3 diffs successifs (1. F-specs + I-specs, 2. C-specs, 3. O-specs) plutôt qu'un seul diff global. Chaque diff est plus petit, plus facile à valider, et une erreur d'alignement est immédiatement localisée.

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Générer sans charger les normes ACME | Bob utilise ses conventions par défaut : variables en anglais, pas de header, CTL-OPT minimal — le source généré ne respecte pas les standards de l'équipe | Toujours charger le mode "ACME Developer" ou fournir les normes explicitement avant le Prompt 1 ; si aucune norme n'est disponible, commencer par le Prompt 3 |
| Confondre UC 9 et UC 10 | UC 9 génère des programmes RPG ; UC 10 génère des interfaces (DDS display file, Web). Pour un programme interactif complet : utiliser UC 9 pour le programme RPG et UC 10 pour le display file — pas UC 9 seul | Le Prompt 0 catégorise le besoin — si "interactif 5250" est détecté, orienter vers UC 10 pour la partie interface |
| Hallucination des noms de fichiers et de champs | Bob génère des DCL-F avec des noms de fichiers inventés ou des noms de colonnes qui n'existent pas dans les tables — le source compile parfois, accède les mauvaises données | Demander à Bob de lister les fichiers et colonnes qu'il va utiliser *avant* de générer les DCL-F (section 3 du Prompt 0), et valider contre QSYS2 via IBM i Database MCP |
| Ne pas tester la compilation avant de sauvegarder | Des erreurs de syntaxe ou de type découvertes après sauvegarde nécessitent une reprise du diff — perte de temps | Le Prompt 2-bis est obligatoire avant toute sauvegarde du source final |
| Distribuer le mode personnalisé sans le valider | Un mode avec des conventions incorrectes génère du mauvais code pour toute l'équipe — plus difficile à corriger | Valider le mode "ACME Developer" sur au moins 2 programmes représentatifs avant de le distribuer (Prompt 3) |
| Travailler en mode Agent pendant la génération | Bob peut sauvegarder un source intermédiaire non validé dans QRPGSRC, ou écraser un source existant | Rester en mode **Ask** pendant Prompts 0 à 4 — mode Agent uniquement pour Prompt 2-bis (compilation) et sauvegarde finale |
| Modifier plusieurs couches en un seul diff sur un programme OPM colonné | Un diff global mélange I-specs, C-specs et O-specs — si une erreur d'alignement est dans les O-specs, elle est masquée par les erreurs des specs précédentes | Utiliser 3 diffs successifs (F/I, C, O) sur les programmes OPM colonné — voir section "Cas avancé cross-couches" |

---

## Check-list de validation UC 9

Avant de passer à UC 11 (voir `UC11-objets-sql.md`) ou de déclarer un programme généré comme livrable, valider chaque point :

- [ ] **Pour chaque programme généré : le Prompt 0 a été exécuté** — la catégorie (SIMPLE / STANDARD / COMPLEXE) est documentée et les interfaces (fichiers, paramètres) ont été validées avant la génération
- [ ] Le mode personnalisé "ACME Developer" est créé, enregistré dans `.bob/custom_modes.yaml` et accessible à toute l'équipe Bob — toute session UC 9 ultérieure charge ce mode avant le Prompt 1
- [ ] Le mode "ACME Developer" a été **testé sur au moins 2 programmes représentatifs** avant distribution à l'équipe — un mode avec des conventions incorrectes génère du mauvais code pour tous
- [ ] Les conventions de nommage ACME (variables, structures, procédures, header) sont respectées dans chaque source généré — validées par un développeur de l'équipe ACME
- [ ] Les noms de fichiers et de colonnes dans les DCL-F ont été vérifiés contre QSYS2 via IBM i Database MCP — aucun nom inventé dans le source final
- [ ] Chaque programme généré a été **compilé sans erreur** sur l'IBM i de test (Prompt 2-bis) — le membre source a été créé dans `[SRCFILE_RPG]` après vérification via `QSYS2.SYSMEMBERSTAT` qu'il n'existait pas (Étape 1 du Prompt 2-bis)
- [ ] Chaque programme généré a été **testé fonctionnellement** avec un jeu de données représentatif — la logique CRUD produit les résultats attendus
- [ ] Le plan de génération (Prompt 4) est produit et sauvegardé dans `{appArcad}-{fonction}-{domaine}-plan-generation-{YYYYMMDD-HHmm}.md` — tous les programmes du périmètre sont listés avec leur catégorie et leur priorité
- [ ] La mention `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` est présente dans l'en-tête de chaque source généré

---

## Points à compléter avant passage en production

> Ces points ne bloquent pas le POC — ils concernent des cas avancés à traiter avant d'industrialiser la génération de code sur l'ensemble du parc en production.

### Maintenance du mode personnalisé

**Contexte :** les normes de développement ACME évoluent. Le mode "ACME Developer" doit refléter les conventions actuelles — un mode obsolète génère du code non conforme qui passe en code review et consomme du temps développeur.

**À faire avant production :** définir un processus de mise à jour du mode : qui peut soumettre une modification des normes ? Quel est le processus de validation (revue par le tech lead) ? Comment versionner le mode (v1.0, v1.1...) ? Comment distribuer la nouvelle version à toute l'équipe ?

### Templates additionnels à intégrer au mode

**Contexte :** le POC couvre les programmes SIMPLE et STANDARD. En production, des patterns plus avancés seront nécessaires :
- **Journalisation** : écriture dans un fichier journal ACME (`JRNPGM`, `QSYS2.JOBLOG_INFO`) — template à ajouter au mode
- **Gestion d'exceptions** : pattern `MONITOR / ON-ERROR / ENDMON` avec code d'erreur et message de diagnostic
- **Sécurité des profils utilisateurs** : vérification des droits avant exécution des procédures critiques (`QSYS2.CHECK_AUTHORITY_TO_OBJECT`)

**À faire avant production :** générer un programme de test pour chaque template additionnel, le valider avec l'équipe, et l'intégrer dans le mode "ACME Developer".

---

## Référence complémentaire — Lab IBM

> **Lab FLIGHT400 — Exercise 3 (Add Field End-to-End)**
> https://github.com/bmarolleau/flight400-demo

L'Exercise 3 du lab illustre le cas "ajout d'un champ end-to-end" décrit dans la section ci-dessus : ajout du champ `FLHRS` (Flight Hours) à travers toutes les couches de l'application FLIGHT400 — PF (`FLIGHTS`), LF (`FLIGHTSZ`), display file (`FRS021DF`), programme RPG OPM (`FRS021`). Il montre concrètement la stratégie en 3 diffs pour les O-specs OPM (étapes 3d, 3e, 3f, 3g) et la séquence de compilation en ordre strict (`CHGPF` → `CRTLF` → `CRTDSPF` → `CRTRPGPGM`).

**Ce lab est la référence de terrain pour le pattern cross-couches.** Les prompts de l'Exercise 3 sont directement adaptables en remplaçant les noms `FRS021`, `FLIGHTS`, `FLIGHTSZ`, `FLGHT4nn` par les noms de l'application cible.

---

*Fiche UC 9 — Document évolutif à mettre à jour au fil du POC.*
