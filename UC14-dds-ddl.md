# UC 14 — Conversion DDS → DDL

> **Catégorie :** Conversion base de données
>
> **Priorité dans le POC :** 6 — premier UC de la Phase 2, prérequis de UC 3
>
> **Durée POC (avec Bob) :** 3 à 5 heures — conversion d'un périmètre applicatif représentatif (5 à 10 PF + leurs LF), itérations et validation des DDL générés
>
> **Durée PROD (avec Bob) :** 30 à 60 min / fichier DDS — analyse Bob + génération DDL + revue développeur + test de création de table
>
> **Durée PROD (sans Bob) :** 1 à 3 jours / fichier complexe — lecture DDS, rédaction DDL manuelle, gestion des types, création des index correspondants, tests
>
> **Gain Bob estimé :** ~8× — 10 fichiers physiques avec leurs logiques convertis en une journée au lieu d'une semaine ; gain encore plus fort sur les fichiers volumineux (50+ champs) où la transcription manuelle est longue et source d'erreurs
>
> **Mode Bob recommandé :** IBM i Developer (mode Ask pour l'analyse et la génération, Agent pour la sauvegarde)

---

## Objectif

Convertir les **fichiers physiques (PF) et fichiers logiques (LF) DDS** d'IBM i en **instructions DDL SQL** (`CREATE TABLE`, `CREATE INDEX`, `CREATE VIEW`) conformes à Db2 for i.

**Ce UC est le point d'entrée de la Phase 2 — Modernisation de la couche base de données.** Il précède UC 3 (accès natifs → SQL embarqué) car les requêtes SQL de UC 3 doivent cibler des tables DDL correctement typées. Convertir les accès avant d'avoir converti les structures revient à écrire des requêtes SQL contre des cibles mal définies.

**Ce UC se différencie de UC 6 (Documentation) sur un point fondamental :** UC 6 documente ce qui existe pour comprendre et conserver. UC 14 produit du code exécutable destiné à **remplacer** les définitions DDS — il porte un risque d'exécution réel. Chaque DDL généré doit être reviewé et testé avant d'être utilisé en production.

**Livrable attendu :** Des scripts DDL SQL (`CREATE TABLE` + `CREATE INDEX` + optionnellement `CREATE VIEW`) pour chaque fichier physique et logique converti, avec commentaires de traçabilité, prêts à être soumis à un test de création sur l'IBM i de test.

**Convention de nommage des fichiers générés :**
```
{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md

Types pour cet UC :
  ddl-table   → script DDL de la table (CREATE TABLE)
  ddl-index   → script DDL des index (CREATE INDEX)
  ddl-complet → table + index + vue dans un seul fichier
  analyse-dds → analyse préalable du fichier DDS, avant toute conversion
```

Exemples :
```
acme-APPVTE-FCOMMANDES-analyse-dds-20250617-0900.md   ← analyse du DDS avant conversion
acme-APPVTE-FCOMMANDES-ddl-complet-20250617-1030.md   ← CREATE TABLE + CREATE INDEX + commentaires
acme-APPVTE-APPVTE-ddl-complet-20250617-1400.md       ← périmètre applicatif complet
```

> 💡 Cette convention est valable en dehors du contexte POC — réutilisable en production tel quel.

---

## Démarrer par un fichier DDS connu

> **Recommandation forte avant d'aborder les fichiers critiques de ACME.**

Commencer par un **fichier physique que l'équipe connaît bien** — un PF simple, peu de champs, dont la structure est connue d'au moins un développeur qui peut valider la sortie de Bob.

Pourquoi ? Parce que la première conversion sert à **calibrer le niveau de précision** :
- Est-ce que Bob produit les bons types SQL pour les champs DDS (`6S2` → `DECIMAL(6,2)`, `10A` → `VARCHAR(10)` ou `CHAR(10)` ?) ?
- Est-ce que la clé primaire est correctement déduite ?
- Est-ce que les fichiers logiques sont convertis en `CREATE INDEX` ou en `CREATE VIEW` selon leur nature ?
- Est-ce que Bob signale les champs dont le type est ambigu ou les contraintes non exprimables en DDL standard ?

Si la conversion sur un fichier connu est correcte, l'équipe peut aborder les fichiers plus complexes avec confiance.

**Progression recommandée :**

> 💡 Pour chaque fichier de cette progression, **commencer par le Prompt 0** — il donne la catégorie (SIMPLE / STANDARD / COMPLEXE) et la séquence exacte à suivre. Ne pas aller directement au Prompt 1.

| Étape | Fichier à choisir | Catégorie attendue | Objectif |
|-------|------------------|--------------------|---------|
| 1 | PF simple, < 20 champs, clé évidente | SIMPLE | Calibrer la correspondance de types et valider le CREATE TABLE ; vérifier que le Prompt 0 classe correctement |
| 2 | PF métier central avec plusieurs LF associés | STANDARD | Valider la conversion des LF en CREATE INDEX et CREATE VIEW |
| 3 | PF avec champs spéciaux (dates, zones décimales, champs COMP) | STANDARD à COMPLEXE | Identifier les cas nécessitant une décision manuelle sur le type SQL cible |
| 4 | PF partagé par de nombreux programmes (identifié dans la matrice UC 6) | COMPLEXE + risque | Valider contre la matrice de références croisées avant toute modification |

---

## Impact de la taille du fichier DDS sur la stratégie de conversion

La taille du fichier DDS ne se mesure pas en lignes de programme, mais en **nombre de champs** et en **nombre de fichiers logiques associés**.

Les vrais facteurs qui compliquent la conversion sont :
- Les **champs COMP-3** (packed decimal) ou **COMP** (binary) : types DDS anciens qui ont plusieurs équivalents SQL possibles
- Les **champs de date au format numérique** (`8S0` stocké comme `YYYYMMDD`) : non typés comme dates en DDS, à convertir en `DATE` ou laisser en `DECIMAL` selon l'usage
- Les **fichiers logiques avec SELECT/OMIT** : peuvent se traduire en `CREATE INDEX` avec filtre ou en `CREATE VIEW` selon que l'on a besoin d'un accès en lecture seule ou d'un index de tri
- Les **jointures dans les fichiers logiques multi-format** : ne se traduisent pas directement en DDL — elles deviennent des vues (`CREATE VIEW`)
- Les **REFLD** (champs référencés depuis un autre fichier) : Bob doit résoudre la référence avant de produire le DDL

### Fichiers DDS < 20 champs, 1 à 3 LF — Conversion directe

Bob gère sans difficulté. **Séquence : Prompt 0 → Prompt 1 → Prompt 2 → Prompt 3-bis (test création table) → Prompt 3 (LF) → Prompt 3-bis.**

> 💡 Le Prompt 0 confirme la catégorie "Simple" et recommande directement cette séquence — pas de décision manuelle requise.

### Fichiers DDS 20 à 50 champs, plusieurs LF — Analyse préalable recommandée

Le Prompt 0 identifie les champs ambigus et pose les questions de décision de types. **Séquence : Prompt 0 → décisions de types → Prompt 1 → Prompt 2 avec décisions intégrées → Prompt 3-bis → Prompt 3 (LF) → Prompt 3-bis.**

> 💡 Ne pas demander à Bob de trancher seul sur les types ambigus — le Prompt 0 les liste explicitement pour les soumettre à un DBA ou développeur senior avant de lancer le Prompt 1.

### Fichiers DDS > 50 champs ou LF avec SELECT/OMIT — Analyse en deux passes

Le Prompt 0 détecte le volume et la présence de LF complexes. Il recommande automatiquement la stratégie deux passes. **Séquence : Prompt 0 → Prompt 1 (analyse complète) → décisions sur les ambiguïtés → Prompt 2 avec décisions → Prompt 3-bis → Prompt 3 (LF) → Prompt 3-bis.**

> ⚠️ Sur un fichier de 50+ champs avec LF SELECT/OMIT, ne jamais sauter le Prompt 0 — le risque de types incorrects et de LF mal convertis est trop élevé pour une approche directe.

### Fichiers DDS avec REFLD (champs référencés)

Le Prompt 0 détecte les REFLD présents et signale le blocage. Résoudre avant de continuer :

```
→ Utiliser IBM i Database MCP : SELECT depuis QSYS2.SYSCOLUMNS pour lire
  la définition réelle du champ référencé
→ Ou ouvrir le fichier référencé dans l'éditeur et le fournir en contexte à Bob
→ Ne jamais laisser un REFLD non résolu dans le DDL généré
```

> ⚠️ Un REFLD non résolu produit un CREATE TABLE avec des types incorrects — les données pourront être insérées sans erreur mais avec une troncature silencieuse ou une perte de précision.

---

## Démarrer une session Bob

> **À lire avant chaque session UC 14 — nouvelle conversation ou reprise.**

### 1. Nouvelle conversation Bob

Chaque session de travail sur un fichier DDS doit démarrer dans une **nouvelle conversation Bob** (bouton `+` en haut du panneau Chat). Ne pas réutiliser une conversation d'un UC précédent — le contexte accumulé pollue les réponses et consomme des Bob Coins inutilement.

**Mode à sélectionner :** `IBM i Developer`

### 2. Ouvrir les fichiers sources dans l'éditeur (Open in Editor)

Avant de lancer le Prompt 0, ouvrir dans l'éditeur Bob les fichiers sources que Bob devra analyser. L'ouverture dans l'éditeur les rend accessibles au MCP IBM i sans avoir à les copier-coller dans le chat.

**Procédure :** dans le panneau **IBM i — Object Browser** (extension Code for IBM i), naviguer jusqu'à la bibliothèque source, faire un clic droit sur le membre DDS → **Open in Editor**.

Fichiers à ouvrir pour chaque session UC 14 :
- Le fichier DDS source cible (`[NOM_LIB]/QDDSSRC([NOM_PF_OU_LF])`)
- Les fichiers DDS des fichiers référencés (`REFLD`) si présents
- Le fichier de compréhension du périmètre (`*-comprehension-*.md`) si disponible depuis UC 4

### 3. Fichiers de contexte à charger

Ces fichiers produits par les UC précédents doivent être disponibles dans le workspace Bob **avant** de démarrer. Utiliser **Add File to Chat** (icône trombone dans le chat) ou les ouvrir dans l'éditeur pour que Bob puisse les lire via MCP.

| Fichier | Produit par | Obligatoire / Recommandé |
|---------|-------------|--------------------------|
| `{appArcad}-{fonction}-{composant}-comprehension-{date}.md` | UC 4 | **Recommandé** — liste des PF/LF utilisés par chaque programme |
| `{appArcad}-{fonction}-{fonction}-spec-tech-{date}.md` | UC 6 | **Recommandé** — inventaire complet des objets et PF partagés |
| `{appArcad}-{fonction}-{fonction}-matrice-{date}.md` | UC 6 | **Recommandé** — criticité des PF partagés entre programmes |

> 💡 Si ces fichiers ne sont pas disponibles, UC 14 fonctionne quand même — mais le Prompt 0 ne pourra pas identifier les PF partagés à fort risque ni les REFLD entre fichiers hors scope. Anticiper ces manques avant de démarrer.

> ⚠️ **Risque de réduction de contexte — sauvegarde intermédiaire recommandée :** une session UC 14 avec plusieurs fichiers DDS STANDARD ou COMPLEXE peut atteindre la limite de contexte. Si Bob semble oublier une décision prise au Prompt 0 (types ambigus, REFLD résolus, catégorie retenue), c'est un signal de compression de contexte. Sauvegarder le livrable en cours en mode Agent après chaque prompt majeur — pas seulement en fin de session. À chaque reprise de passe, commencer le prompt par : "Le fichier [NOM_FICHIER] contient les décisions prises — continuer à partir de [ÉTAPE]."

> 💡 **Reprise de session :** si la session UC 14 est interrompue et reprise le lendemain, ouvrir dans l'éditeur les fichiers `*-analyse-dds-*.md` déjà produits — Bob retrouve ainsi le contexte des décisions de types déjà prises sans relire les DDS depuis zéro.

---

## Prérequis

- UC 4 complété : les fichiers `*-comprehension-*.md` sont présents — ils fournissent la liste des PF/LF utilisés par chaque programme
- UC 6 complété (recommandé) : le fichier `*-spec-tech-*.md` liste l'inventaire complet des objets et le fichier `*-matrice-*.md` identifie les PF partagés et leur niveau de criticité
- IBM i MCP actif (lecture des membres sources DDS dans QDDSSRC ou QDDSSRCD)
- IBM i Database MCP actif (`QSYS2.SYSCOLUMNS`, `QSYS2.SYSTABLES`, `QSYS2.SYSKEYS`)
- Accès à l'IBM i de test pour valider les scripts DDL générés (`CREATE TABLE` en mode test)
- Un DBA ou développeur senior disponible pour trancher les décisions de types ambigus

---

## Mode Bob et MCP à utiliser

| Élément | Valeur |
|---------|--------|
| **Mode Bob** | **IBM i Developer** (Premium Package IBM i) — mode unique pour toute la session. Sans Premium Package : Ask pour l'analyse DDS et la génération DDL, Agent pour le test de création et la sauvegarde. |
| **Scope** | Library List → bibliothèque source ACME (QDDSSRC ou QDDSSRCD) |
| **MCP actifs** | IBM i MCP (lecture DDS) + IBM i Database MCP (QSYS2 pour validation et métadonnées) |
| **MCP différés** | Confluence MCP (publication des scripts DDL, si token disponible) |

### Pourquoi le mode IBM i Developer pour la conversion DDS → DDL ?

Le mode **IBM i Developer** pré-charge le contexte IBM i (DDS, SQL Db2 for i, IBM i MCP) dans chaque conversation. Bob génère le DDL dans le chat — l'équipe le relit, le corrige, itère sans risque avant d'autoriser l'écriture ou l'exécution. Un script DDL incorrect exécuté sur l'IBM i peut créer une table avec de mauvais types — difficile à corriger une fois des données chargées.

| Phase | Comportement attendu | Ce que Bob fait |
|-------|----------------------|----------------|
| Qualification du fichier DDS (Prompt 0) | Génère dans le chat — pas d'écriture | Compte les champs, détecte les facteurs de complexité, liste les REFLD, recommande la stratégie |
| Analyse DDS (Prompt 1) | Génère dans le chat — pas d'écriture | Lit les membres DDS via IBM i MCP, produit l'analyse complète dans le chat |
| Génération DDL (Prompts 2 et 3) | Génère dans le chat — pas d'écriture | Génère `CREATE TABLE`, `CREATE INDEX`, `CREATE VIEW` dans le chat |
| Décisions sur types ambigus | Génère dans le chat — pas d'écriture | Itération dans le chat jusqu'à validation de l'équipe |
| Test de création DDL (Prompt 3-bis) | Écriture et exécution autorisées — après validation | Exécute le `CREATE TABLE` / `CREATE INDEX` sur l'IBM i de test, rapporte les erreurs SQL |
| Validation post-création (Prompt 5) | Génère dans le chat — pas d'écriture | Interroge `QSYS2` pour vérifier la structure créée — comparaison avec le DDS source |
| Sauvegarde du script DDL final | Écriture autorisée — après validation | Écrit le fichier `.md` dans le workspace — uniquement une fois le DDL validé |

> 💡 **Règle d'or pour UC 14 :** Bob génère dans le chat. L'écriture (`write_member`) et l'exécution DDL ne sont autorisées qu'après relecture et validation explicite par un développeur.

> 💡 **Sans Premium Package IBM i :** utiliser le mode Ask pour l'analyse DDS et la génération DDL (Prompts 0 à 3), puis basculer en mode Agent uniquement pour le test de création DDL (Prompt 3-bis) et la sauvegarde du script validé.

### Intégration ARCAD

Le MCP ARCAD n'était pas disponible dans le contexte de ce POC de référence (version ARCAD non compatible avec le MCP). Si le MCP ARCAD est disponible dans votre environnement, les étapes manuelles de réintégration décrites ci-dessous peuvent être automatisées. N'hésitez pas à demander à Bob de modifier cette fiche UC en intégrant la disponibilité du MCP ARCAD.

**Impact sur UC 14 : faible.** ARCAD gère les versions et les déploiements — la lecture des sources DDS se fait via IBM i MCP directement dans les bibliothèques, indépendamment d'ARCAD.

| Sans MCP ARCAD (contexte de ce POC) | Avec MCP ARCAD disponible |
|--------------------------------------|---------------------------|
| Exporter manuellement l'historique des versions DDS depuis ARCAD | Le MCP ARCAD peut exposer l'historique directement dans le contexte Bob |
| Vérifier manuellement si un PF est "en promotion" dans ARCAD | Le MCP ARCAD peut vérifier le statut de promotion directement depuis Bob |
| Réintégrer manuellement les scripts DDL dans ARCAD après validation | Le MCP ARCAD peut versionner et intégrer les scripts automatiquement |

> 💡 **Dans les deux cas :** avant de démarrer UC 14, vérifier dans ARCAD la liste des PF/LF managés et leur statut (promotions en cours) pour que la génération DDL tienne compte des objets actuellement verrouillés.

---

## Prompts clés

### Prompt 0 — Qualification et choix de stratégie

> **Ce prompt est le point d'entrée obligatoire de UC 14 pour chaque fichier DDS.**
> Il remplace la décision manuelle sur la stratégie (Conversion directe / Analyse préalable / Deux passes).
> Il se lance **avant** le Prompt 1 — son résultat conditionne toute la séquence suivante.

```
Le fichier DDS [NOM_PF_OU_LF] se trouve dans [NOM_LIB]/QDDSSRC (ou QDDSSRCD).

Analyse ce fichier DDS et produis en français, en markdown,
une fiche de qualification pour la conversion DDL :

## Qualification UC 14 — [NOM_PF_OU_LF]

### 1. Inventaire rapide
| Indicateur                        | Valeur |
|-----------------------------------|--------|
| Type d'objet                      | PF / LF |
| Nombre de champs                  | ?      |
| Nombre de fichiers logiques (LF) associés | ? (si PF) |
| Présence de REFLD                 | OUI / NON |
| Présence de champs COMP-3 ou COMP | OUI / NON |
| Présence de champs date numériques (ex. 8S0) | OUI / NON |
| LF avec SELECT/OMIT               | OUI / NON / ? |
| LF multi-format (jointure de PF)  | OUI / NON / ? |

### 2. Facteurs de complexité détectés
Réponds par OUI / NON / À CONFIRMER pour chaque facteur :
- [ ] REFLD non résolvables depuis ce seul fichier (fichier référencé hors scope)
- [ ] Champs dont le type SQL est ambigu (date numérique, COMP-3, binaire, champ très long)
- [ ] LF avec SELECT/OMIT nécessitant une CREATE VIEW plutôt qu'un CREATE INDEX
- [ ] LF multi-format (jointure) → CREATE VIEW avec JOIN complexe
- [ ] Contraintes DDS sans équivalent DDL (ALWNULL implicite, COLHDG, COMP)
- [ ] PF référencé par plusieurs programmes (criticité élevée — matrice UC 6)

### 3. REFLD à résoudre avant conversion
Si des REFLD sont présents, lister chaque champ référencé :
| Nom champ | Fichier référencé | Champ source | Type SQL à utiliser |
| [CHAMP]   | [NOM_FICHIER_REF] | [NOM_CHAMP_SOURCE] | ? (à résoudre) |

Si le fichier référencé n'est pas dans le scope actuel :
→ Signaler le blocage — résoudre via QSYS2.SYSCOLUMNS avant de continuer.

### 4. Champs ambigus — décisions requises
Pour chaque champ dont le type SQL est ambigu :
| Nom champ | Type DDS | Valeur de la colonne | Usage probable | Type SQL recommandé | Décision requise |
| [CHAMP]   | 8S0      | YYYYMMDD ?           | Date de création ? | DATE ou DECIMAL(8,0) | À confirmer |

### 5. Catégorie et stratégie recommandée
Sur la base de l'inventaire, conclure :

**Catégorie :**
- [ ] SIMPLE — ≤ 20 champs, 1 à 3 LF sans SELECT/OMIT, pas de REFLD, pas de type ambigu
- [ ] STANDARD — 20 à 50 champs, ou LF simples, ou quelques types ambigus
- [ ] COMPLEXE — > 50 champs, ou LF avec SELECT/OMIT / jointure, ou REFLD non résolus

**Recommandation :**
- SIMPLE   → Séquence directe : Prompt 1 → Prompt 2 → Prompt 3-bis → Prompt 3 (LF) → Prompt 3-bis
- STANDARD → Résoudre les ambiguïtés d'abord, puis : Prompt 1 → Prompt 2 avec décisions → Prompt 3-bis → Prompt 3 → Prompt 3-bis
- COMPLEXE → Analyse en deux passes : Prompt 1 complet → décisions de types → Prompt 2 avec décisions → Prompt 3-bis → Prompt 3 LF par LF → Prompt 3-bis

Signale clairement les REFLD bloquants et les décisions de types qui doivent être prises
par un DBA ou développeur senior avant de continuer.
```

**Analyse ligne à ligne :**

- `Inventaire rapide en tableau` → les indicateurs clés lisibles en un coup d'œil. Le Prompt 0 répond à la question "par où commencer ?" sans lire le DDS en détail — Bob fait le comptage, l'équipe lit le résultat.

- `OUI / NON / À CONFIRMER` → trois états, comme dans UC 3. Le statut "À CONFIRMER" est délibéré pour les cas que Bob ne peut pas trancher seul depuis le seul source DDS (ex. : un `8S0` est-il une date ou un compteur ? Bob peut suspecter mais pas certifier sans le contexte d'usage).

- `REFLD à résoudre avant conversion` → tableau dédié aux blocages. Un REFLD non résolu est un **bloquant** — le Prompt 0 l'identifie explicitement avec le fichier source et le type à résoudre, avant que le Prompt 1 ne génère un tableau de champs avec des types incorrects.

- `Champs ambigus — décisions requises` → liste actionnable pour le DBA. La colonne "Décision requise" est le livrable de cette section — l'équipe remplit cette colonne et injecte les décisions dans le Prompt 2.

- `Catégorie SIMPLE / STANDARD / COMPLEXE` → trois niveaux adaptés à la réalité des DDS IBM i (différents de UC 3 où le critère est le nombre d'opcodes). SIMPLE = conversion directe sans réflexion ; STANDARD = quelques décisions manuelles ; COMPLEXE = deux passes obligatoires.

- `Recommandation avec séquence exacte` → comme UC 3, la sortie du Prompt 0 est un plan d'action précis avec les numéros de prompts à enchaîner.

> 💡 **Sauvegarder la fiche de qualification** (mode Agent) dès la fin du Prompt 0 :
> ```
> "Sauvegarde cette fiche de qualification dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-analyse-dds-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-FCOMMANDES-analyse-dds-20250617-0900.md"
> ```
> Le Prompt 1 (analyse détaillée) complète ensuite ce même fichier.

> ⚠️ **Piège évité :** sans le Prompt 0, l'équipe découvre en milieu de Prompt 1 qu'un champ est un REFLD non résolvable ou qu'un LF est une jointure complexe — la session doit être interrompue pour résoudre le blocage. Le Prompt 0 déplace cette découverte au tout début.

---

### Prompt 1 — Analyse préalable d'un fichier DDS

```
Le fichier DDS [NOM_PF_OU_LF] se trouve dans [NOM_LIB]/QDDSSRC (ou QDDSSRCD).

Analyse ce fichier DDS et réponds en français en markdown :

1. Type d'objet DDS : fichier physique (PF) ou fichier logique (LF)
2. Rôle fonctionnel supposé : ce que représente chaque enregistrement (1 à 3 phrases)
3. Liste de tous les champs :
   | Nom champ | Type DDS | Long. | Déc. | Texte / Libellé DDS | Type SQL proposé | Remarque |
4. Clé primaire : champs composant la clé, dans l'ordre
5. Si fichier logique :
   - Fichier(s) physique(s) de base
   - Clé du LF et critères de tri
   - Clauses SELECT/OMIT si présentes
   - Nature de conversion recommandée : CREATE INDEX ou CREATE VIEW ? Pourquoi ?
6. Champs dont la conversion SQL est ambiguë ou nécessite une décision :
   (ex. champ date numérique, COMP-3, REFLD non résolu, champ très long)
7. Contraintes DDS sans équivalent DDL direct :
   (ex. COMP, ALWNULL implicite, COLHDG, ALIAS)

Ne pas générer le DDL dans ce prompt.
Signale clairement ce que tu ne peux pas déterminer depuis le seul source DDS.
```

**Analyse ligne à ligne :**

- `[NOM_LIB]/QDDSSRC (ou QDDSSRCD)` → ancrage objet explicite avec la bibliothèque membre. Les sources DDS peuvent se trouver dans différents membres selon les conventions du client (`QDDSSRC`, `QDDSSRCD`, `QSRVSRC`…) — les nommer évite que Bob cherche dans le mauvais membre.

- `Type d'objet DDS : fichier physique (PF) ou fichier logique (LF)` → point 1 délibérément simple. La distinction PF/LF est fondamentale pour la stratégie de conversion — un LF ne donne jamais un `CREATE TABLE` mais un `CREATE INDEX` ou une `CREATE VIEW`. Forcer Bob à l'annoncer explicitement en point 1 évite des confusions dans les prompts suivants.

- `| Nom champ | Type DDS | Long. | Déc. | Texte / Libellé DDS | Type SQL proposé | Remarque |` → tableau structuré avec le type SQL **proposé** — pas définitif. La colonne "Remarque" est le lieu où Bob signale les cas ambigus. Ce tableau devient la feuille de travail pour les décisions manuelles.

- `Nature de conversion recommandée : CREATE INDEX ou CREATE VIEW ?` → question explicite qui force Bob à justifier son choix. Sur IBM i, la règle est : LF avec tri → `CREATE INDEX` ; LF avec SELECT/OMIT ou colonnes réduites → `CREATE VIEW`. Bob connaît cette règle mais il faut lui demander de l'expliciter pour que l'équipe puisse la valider.

- `Ne pas générer le DDL dans ce prompt` → séparation délibérée entre l'analyse et la génération. Analyser d'abord permet à l'équipe de prendre les décisions sur les types ambigus **avant** que le DDL soit généré. Sans cette séparation, Bob produit un DDL avec des choix par défaut qui peuvent être incorrects — et il faut corriger le DDL après coup plutôt que de corriger les décisions avant.

- `Signale clairement ce que tu ne peux pas déterminer depuis le seul source DDS` → garde-fou anti-hallucination. Les DDS d'IBM i ne décrivent pas toujours le contexte d'usage d'un champ — un champ `8S0` peut stocker une date, un montant, ou un compteur selon le contexte. Bob ne peut pas trancher sans contexte supplémentaire — il doit le signaler.

> ⚠️ **Piège évité :** sans la distinction explicite PF/LF, Bob peut proposer un `CREATE TABLE` pour un fichier logique — une erreur qui produit une table redondante et ne remplace pas l'index d'origine.

> ⚠️ **Piège évité :** sans la demande explicite du tableau de champs, Bob résume la structure en prose — difficile à utiliser comme feuille de travail pour les décisions de types.

---

### Prompt 2 — Génération du CREATE TABLE (fichier physique)

```
Sur la base de l'analyse du fichier physique [NOM_PF] que nous venons de faire,
génère le script DDL SQL CREATE TABLE pour Db2 for i, en français dans les commentaires,
en prenant en compte les décisions suivantes :
[Lister ici les décisions de types prises lors de l'analyse du Prompt 1]

Structure attendue du script :

-- ============================================================
-- TABLE : [NOM_LIB].[NOM_TABLE]
-- Fichier DDS d'origine : [NOM_LIB]/QDDSSRC([NOM_PF])
-- Converti le : [DATE]
-- Statut : À valider et tester sur IBM i de test avant tout usage production
-- ⚠️ Réintégration ARCAD — à effectuer manuellement après validation
-- ============================================================

CREATE OR REPLACE TABLE [NOM_LIB].[NOM_TABLE] (
  -- [NOM_CHAMP]  [TYPE_SQL]  [NULL/NOT NULL]  -- [LIBELLÉ DDS ORIGINAL]
  ...
  CONSTRAINT [NOM_TABLE]_PK PRIMARY KEY ([CHAMPS_CLE])
)
;

Règles de conversion à respecter :
- Utiliser les types SQL Db2 for i natifs : CHAR, VARCHAR, DECIMAL, INTEGER, SMALLINT, DATE, TIME, TIMESTAMP
- Champs alphanumériques courts (≤ 30) : CHAR(n)
- Champs alphanumériques longs (> 30) ou texte libre : VARCHAR(n)
- Zones décimales packed (DDS type S) : DECIMAL(long, déc)
- Ne pas inventer de contraintes non visibles dans le DDS source
- Conserver les libellés DDS originaux en commentaire de chaque champ
- Si un champ a un ALIAS DDS, l'indiquer en commentaire
- Signaler en commentaire TODO les champs dont la conversion n'est pas certaine
```

**Analyse ligne à ligne :**

- `Sur la base de l'analyse […] que nous venons de faire` → s'appuie sur le contexte de la conversation. Bob utilise le tableau produit au Prompt 1 plutôt que de relire le DDS — plus rapide et cohérent avec les décisions déjà prises.

- `en prenant en compte les décisions suivantes` → espace explicite pour injecter les décisions de l'équipe sur les types ambigus. Sans ce point, Bob applique ses propres défauts — qui peuvent différer des choix de l'équipe.

- `CREATE OR REPLACE TABLE` → syntaxe Db2 for i préférable à `CREATE TABLE` seul. En environnement de test, `CREATE OR REPLACE` permet de relancer le script sans `DROP TABLE` préalable — moins de risque d'erreur lors des itérations.

- `-- Fichier DDS d'origine` → en-tête de traçabilité dans le script. Permet de retrouver le DDS source 6 mois après la conversion, quand le contexte est perdu.

- `Statut : À valider et tester sur IBM i de test` → mention obligatoire dans l'en-tête. Tout script DDL généré par Bob doit être identifié comme non validé jusqu'à son test sur l'IBM i de test.

- `-- [LIBELLÉ DDS ORIGINAL]` → conserver les libellés DDS en commentaire de chaque champ. Sur IBM i, les `COLHDG` et `TEXT` du DDS sont la documentation du champ — les perdre dans la conversion efface de la connaissance métier.

- `Signaler en commentaire TODO les champs dont la conversion n'est pas certaine` → garde-fou anti-hallucination dans le code généré lui-même. Un commentaire `-- TODO: vérifier si ce champ est une date (format YYYYMMDD)` dans le script est plus utile qu'un choix silencieux incorrect.

- `Ne pas inventer de contraintes non visibles dans le DDS source` → les contraintes référentielles (FOREIGN KEY) n'existent pas en DDS. Bob ne doit pas en inférer — elles seront ajoutées ultérieurement lors d'une étape dédiée si l'équipe le décide.

> 💡 **Sauvegarder ce livrable** (mode Agent) :
> ```
> "Sauvegarde ce script CREATE TABLE dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-ddl-table-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-FCOMMANDES-ddl-table-20250617-1030.md"
> ```

> ⚠️ **Piège évité :** sans la règle `VARCHAR(n)` pour les champs longs, Bob peut générer des `CHAR(200)` pour des champs de texte libre — ce qui gaspille de l'espace et dégrade les performances d'indexation.

---

### Prompt 3 — Génération des CREATE INDEX et CREATE VIEW (fichiers logiques)

```
Sur la base de l'analyse des fichiers logiques de [NOM_PF] que nous venons de faire,
génère les scripts DDL pour chaque LF identifié, en respectant les règles suivantes :

Pour chaque LF à convertir en CREATE INDEX :
-- ============================================================
-- INDEX : [NOM_INDEX] sur [NOM_LIB].[NOM_TABLE]
-- Fichier logique DDS d'origine : [NOM_LIB]/QDDSSRC([NOM_LF])
-- Clé : [CHAMPS_CLE_LF]
-- Type : [UNIQUE / non unique]
-- ============================================================

CREATE [UNIQUE] INDEX [NOM_LIB].[NOM_INDEX]
  ON [NOM_LIB].[NOM_TABLE] ([CHAMPS_CLE_LF ASC/DESC])
;

Pour chaque LF à convertir en CREATE VIEW (SELECT/OMIT ou colonnes réduites) :
-- ============================================================
-- VUE : [NOM_VUE] sur [NOM_LIB].[NOM_TABLE]
-- Fichier logique DDS d'origine : [NOM_LIB]/QDDSSRC([NOM_LF])
-- SELECT/OMIT d'origine : [CONDITION_DDS]
-- ============================================================

CREATE OR REPLACE VIEW [NOM_LIB].[NOM_VUE] AS
  SELECT [COLONNES]
  FROM [NOM_LIB].[NOM_TABLE]
  WHERE [CONDITION_SQL_EQUIVALENTE]
;

Règles :
- LF avec tri uniquement (pas de SELECT/OMIT, pas de réduction de colonnes) → CREATE INDEX
- LF avec SELECT/OMIT → CREATE VIEW avec WHERE clause équivalente
- LF multi-format (jointure de PF) → CREATE VIEW avec JOIN — signaler si la jointure est complexe et nécessite une révision manuelle
- Ne pas inventer de colonnes ou de conditions non visibles dans le DDS source
- Signaler en commentaire les cas où la conversion n'est pas directe
```

**Analyse ligne à ligne :**

- `Pour chaque LF à convertir en CREATE INDEX` / `CREATE VIEW` → la séparation des deux cas dans le prompt lui-même renforce la décision prise à l'analyse. Bob ne peut pas mélanger les deux stratégies par erreur si les règles sont explicites dans la structure du prompt.

- `[UNIQUE / non unique]` → indication explicite. Un LF avec clé unique en DDS doit produire un `CREATE UNIQUE INDEX` — un index non unique produit des accès en doublon si la contrainte est perdue.

- `WHERE [CONDITION_SQL_EQUIVALENTE]` → le plus délicat de la conversion. Les clauses `SELECT/OMIT` DDS ont une syntaxe différente des `WHERE` SQL — Bob doit les traduire. Si la traduction n'est pas directe (conditions composées, RANGE…), il doit le signaler.

- `LF multi-format (jointure de PF) → CREATE VIEW avec JOIN` → les LF de jointure (join logical files) sont une fonctionnalité DDS sans équivalent direct en index. Ils doivent devenir des vues — ce qui implique que les programmes qui les utilisent devront accéder à la vue et non à un index.

- `Ne pas inventer de colonnes ou de conditions` → garde-fou anti-hallucination pour les LF. Bob ne doit pas ajouter des colonnes absentes du LF d'origine ni déduire des conditions SELECT/OMIT qui n'y sont pas.

> 💡 **Sauvegarder ce livrable** (mode Agent) :
> ```
> "Sauvegarde ces scripts CREATE INDEX et CREATE VIEW dans un fichier nommé
>  {appArcad}-{fonction}-{composant}-ddl-index-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-FCOMMANDES-ddl-index-20250617-1100.md"
> ```

> ⚠️ **Piège évité :** un LF avec `SELECT/OMIT` converti en `CREATE INDEX` (au lieu de `CREATE VIEW`) ne filtre rien — tous les enregistrements sont accessibles, y compris ceux qui devaient être exclus. Une erreur silencieuse avec un impact fonctionnel direct.

---

### Prompt 3-bis — Test de création DDL par Bob (après chaque CREATE TABLE / CREATE INDEX)

> **Ce que Bob peut faire :** exécuter le script DDL sur l'IBM i de test via IBM i MCP en mode **Agent** et rapporter les erreurs SQL.
> **Ce que Bob ne peut pas faire :** valider que la table créée est fonctionnellement correcte ni qu'elle contient les bonnes données — c'est la vérification humaine du Prompt 5.
> **Quand l'utiliser :** après chaque Prompt 2 (CREATE TABLE) et après chaque Prompt 3 (CREATE INDEX / CREATE VIEW), avant de passer au fichier suivant.

```
Le script DDL suivant a été généré pour [NOM_PF_OU_LF] dans [NOM_LIB] :

[Coller ici le CREATE TABLE / CREATE INDEX / CREATE VIEW généré au Prompt 2 ou 3]

Exécute ce script sur l'IBM i de test [NOM_LIB_TEST] :
RUNSQLSTM SRCSTMF('[CHEMIN_SCRIPT]') COMMIT(*NONE) NAMING(*SQL)

Ou si le script est court, exécute directement via IBM i Database MCP :
EXEC SQL [CREATE TABLE / CREATE INDEX / CREATE VIEW ...]

Analyse le résultat et produis en français :

1. Statut : CRÉATION RÉUSSIE / ERREURS SQL
2. Si erreurs : liste des erreurs avec code SQL et description
   | Code SQL | Description | Objet concerné | Cause probable |
3. Si création réussie : confirmer le nom complet de l'objet créé
   ([NOM_LIB_TEST].[NOM_TABLE_OU_INDEX_OU_VUE])
4. Rappeler que la structure créée doit être validée via le Prompt 5 (QSYS2)
   avant de passer au fichier suivant

Ne pas modifier les données existantes — création de structure uniquement.
```

**Analyse ligne à ligne :**

- `RUNSQLSTM` ou `EXEC SQL direct` → deux options selon la taille du script. `RUNSQLSTM` exécute un script depuis un fichier stream (IFS) — utile pour les scripts longs avec plusieurs instructions. L'exécution directe via IBM i Database MCP est plus rapide pour un seul `CREATE TABLE`.

- `COMMIT(*NONE)` → les instructions DDL sur IBM i ne nécessitent pas de commit en mode journal — `COMMIT(*NONE)` évite les erreurs liées à l'absence de transaction ouverte.

- `| Code SQL | Description | Objet concerné | Cause probable |` → tableau structuré des erreurs. Bob connaît les codes SQL courants : `-601` (objet déjà existant), `-204` (table de base introuvable pour un index), `-104` (erreur de syntaxe SQL). La colonne "Cause probable" permet un diagnostic immédiat.

- `confirmer le nom complet de l'objet créé` → vérification que l'objet a bien été créé dans la bonne bibliothèque. Sur IBM i, `CREATE TABLE` sans schéma explicite utilise la bibliothèque courante — qui peut ne pas être la bibliothèque cible.

- `Ne pas modifier les données existantes — création de structure uniquement` → rappel de périmètre. Ce prompt ne lit ni n'écrit de données — il crée des structures vides.

> 💡 **Ce prompt s'exécute en mode Agent** — Bob doit pouvoir exécuter du SQL DDL via IBM i Database MCP. Vérifier que l'utilisateur IBM i associé au MCP a les droits `*CHANGE` sur la bibliothèque cible de test et `*USE` sur `RUNSQLSTM`.

> 💡 **En cas d'erreur SQL `-601` (objet déjà existant) :** le script utilise `CREATE OR REPLACE` — cette erreur ne devrait pas apparaître. Si elle apparaît, vérifier que la syntaxe `CREATE OR REPLACE TABLE` est bien présente (et non `CREATE TABLE`).

> ⚠️ **Ce prompt ne remplace pas la validation fonctionnelle du Prompt 5.** Une table créée sans erreur SQL peut avoir des types incorrects ou des champs manquants. Le Prompt 5 (`QSYS2.SYSCOLUMNS`) est la vérification de structure — obligatoire après chaque création.

---

### Prompt 4 — Script de migration complet (PF + LF d'un périmètre)

```
Sur la base de l'analyse et des scripts DDL générés pour le périmètre applicatif
[NOM_APPLICATION] / [NOM_LIB], génère un script de migration SQL complet en français,
en markdown, regroupant tous les CREATE TABLE, CREATE INDEX et CREATE VIEW
dans le bon ordre d'exécution.

Structure du script de migration :

-- ============================================================
-- SCRIPT DE MIGRATION DDS → DDL
-- Application : [NOM_APPLICATION]
-- Bibliothèque : [NOM_LIB]
-- Périmètre : [liste des PF/LF couverts]
-- Généré par : Bob × ACME POC
-- Date : [DATE]
-- Statut : ⚠️ À valider et tester sur IBM i de test avant tout usage
-- ============================================================

-- SECTION 1 : TABLES (fichiers physiques)
-- Ordre : tables indépendantes d'abord, tables avec dépendances ensuite
[CREATE TABLE ...]

-- SECTION 2 : INDEX (fichiers logiques avec tri)
-- Les index doivent être créés après les tables qu'ils indexent
[CREATE INDEX ...]

-- SECTION 3 : VUES (fichiers logiques avec SELECT/OMIT ou jointure)
-- Les vues doivent être créées après les tables qu'elles référencent
[CREATE VIEW ...]

Règles d'ordre :
- Une table ne peut pas être créée après une table qui la référence
- Les index doivent suivre leurs tables
- Les vues de jointure doivent suivre toutes leurs tables sources

Inclure en fin de script un tableau récapitulatif :
| Objet DDS original | Type DDS | Objet SQL créé | Type SQL | Statut |
```

**Analyse ligne à ligne :**

- `dans le bon ordre d'exécution` → sur IBM i, les `CREATE INDEX` et `CREATE VIEW` doivent référencer des tables qui existent déjà. Bob gère l'ordre si on lui demande explicitement — sans cette instruction, il peut produire les scripts dans l'ordre du DDS d'origine, qui ne respecte pas les dépendances SQL.

- `SECTION 1 : TABLES / SECTION 2 : INDEX / SECTION 3 : VUES` → structure imposée dans le prompt. Un script de migration mal ordonné produit des erreurs SQL à l'exécution, pas des erreurs de compilation — difficiles à diagnostiquer sur un gros périmètre.

- `Inclure un tableau récapitulatif` → la table de mapping DDS → SQL est le livrable de traçabilité de UC 14. Permet de vérifier que chaque PF/LF a été traité et que rien n'a été oublié.

> 💡 **Sauvegarder ce livrable** (mode Agent) :
> ```
> "Sauvegarde ce script de migration dans un fichier nommé
>  {appArcad}-{fonction}-{fonction}-ddl-complet-{YYYYMMDD-HHmm}.md
>  Exemple : acme-APPVTE-APPVTE-ddl-complet-20250617-1400.md"
> ```

> 💡 Ce script est directement l'input de UC 3 (SQL embarqué) — les noms de tables et d'index générés ici sont les cibles des requêtes SQL qui seront produites en UC 3. S'assurer que ce fichier est présent et validé avant de démarrer UC 3.

---

### Prompt 5 — Validation croisée avec QSYS2 (après exécution du DDL)

```
Les tables suivantes ont été créées sur l'IBM i de test à partir du script DDL
que nous avons généré : [NOM_TABLE1], [NOM_TABLE2], ...

Utilise IBM i Database MCP pour valider la création :
1. Pour chaque table : vérifie que QSYS2.SYSCOLUMNS retourne bien tous les champs attendus
   avec les types SQL corrects
2. Compare le nombre de champs entre le DDS d'origine et la table créée :
   SELECT COUNT(*) FROM QSYS2.SYSCOLUMNS WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_TABLE]'
3. Vérifie que les index ont été créés :
   SELECT INDEX_NAME, COLUMN_NAMES, IS_UNIQUE
   FROM QSYS2.SYSINDEXES
   WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_TABLE]'
4. Signale toute différence entre le DDS source et la structure SQL créée

Ne pas modifier les objets créés — se limiter à la validation.
```

**Analyse ligne à ligne :**

- `Les tables suivantes ont été créées sur l'IBM i de test` → ce prompt s'utilise **après** l'exécution manuelle du script DDL sur l'IBM i de test, pas avant. Séquençage explicite.

- `QSYS2.SYSCOLUMNS` / `QSYS2.SYSINDEXES` → les vues QSYS2 sont la source de vérité après création. Elles reflètent la structure réelle des objets créés — pas le script DDL généré.

- `Compare le nombre de champs` → vérification arithmétique simple. Un écart de champs entre DDS et table SQL indique un champ perdu dans la conversion — souvent un champ avec un type non géré.

- `Ne pas modifier les objets créés` → garde-fou en mode Ask. Bob ne peut pas modifier des objets SQL directement en Ask (le MCP Database est en lecture seule pour les requêtes), mais l'instruction est là pour clarifier l'intention.

> 💡 Ce prompt peut être exécuté en mode **Ask** avec IBM i Database MCP actif — Bob interroge `QSYS2` et compare les résultats au DDS d'origine analysé en Prompt 1. C'est la validation la plus rapide disponible sans sortir de Bob.

> ⚠️ **Piège évité :** ne pas se contenter de vérifier que le `CREATE TABLE` s'est exécuté sans erreur — une table peut être créée avec succès mais avec un champ en moins si un type non reconnu a été silencieusement ignoré.

---

### Note — Correspondance de types DDS → SQL : référence rapide

Tableau de référence à inclure dans les prompts ou à garder sous la main lors des sessions UC 14 :

| Type DDS | Signification | Type SQL Db2 for i recommandé | Cas particuliers |
|----------|--------------|-------------------------------|-----------------|
| `nA` | Alphanumérique (n caractères) | `CHAR(n)` si n ≤ 30 ; `VARCHAR(n)` si n > 30 | Champs de texte libre : toujours `VARCHAR` |
| `nS d` | Packed decimal (n chiffres, d décimales) | `DECIMAL(n, d)` | `0S0` → `INTEGER` si compteur |
| `nP d` | Packed (même que S en DDS moderne) | `DECIMAL(n, d)` | — |
| `nB` | Binary (entier binaire) | `SMALLINT` (n ≤ 4) ou `INTEGER` (n ≤ 9) | Vérifier l'usage avant de choisir |
| `nF d` | Floating point | `FLOAT` ou `DOUBLE` | Rare sur IBM i legacy |
| `nL` | Date (format ISO/JUL/MDY…) | `DATE` | Vérifier le format DATFMT du fichier |
| `nT` | Time | `TIME` | — |
| `nZ` | Timestamp | `TIMESTAMP` | — |
| `nA` (8 chars) utilisé comme date | Pseudo-date numérique YYYYMMDD | `CHAR(8)` avec commentaire `-- date au format YYYYMMDD` | Ne pas forcer `DATE` sans valider le contenu |
| `REFLD(champ, fichier)` | Référence à un autre champ | Résoudre la référence → copier le type du champ référencé | Toujours résoudre avant génération DDL |

> 💡 Ce tableau peut être collé directement dans un prompt Bob pour lui donner la grille de conversion à utiliser — évite que Bob invente ses propres équivalences.

---

## Add-ons Bob à activer

| Extension | Rôle dans cet UC |
|-----------|-----------------|
| **Code for IBM i** | Ouverture des membres DDS dans QDDSSRC/QDDSSRCD, navigation Object Browser vers les PF/LF |
| **IBM i Languages** | Coloration syntaxique DDS — indispensable pour lire les sources DDS dans l'éditeur et valider les champs |
| **Markdown All in One** | Prévisualisation des scripts DDL sauvegardés en markdown, vérification des tableaux de correspondance |
| **SQL Notebook** *(si disponible)* | Exécution interactive des requêtes QSYS2 de validation (Prompt 5) |

---

## MCP à utiliser

| MCP | Usage dans cet UC |
|-----|------------------|
| **IBM i MCP** | Lecture des membres sources DDS (`QDDSSRC`, `QDDSSRCD`) via `read_member` ou `search_qsys` |
| **IBM i Database MCP** | Interrogation `QSYS2.SYSCOLUMNS`, `QSYS2.SYSTABLES`, `QSYS2.SYSINDEXES`, `QSYS2.SYSKEYS` pour la validation post-création (Prompt 5) et la résolution des REFLD |
| **Confluence MCP** *(si disponible)* | Publication des scripts DDL et du tableau de correspondance dans l'espace POC |

> 💡 **Requêtes QSYS2 utiles pour UC 14 :**
> ```sql
> -- Structure d'un fichier physique (équivalent du DDS en SQL)
> SELECT COLUMN_NAME, DATA_TYPE, LENGTH, NUMERIC_SCALE,
>        IS_NULLABLE, COLUMN_TEXT, COLUMN_HEADING
> FROM QSYS2.SYSCOLUMNS
> WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_PF]'
> ORDER BY ORDINAL_POSITION;
>
> -- Fichiers logiques associés à un PF
> SELECT TABLE_NAME, TABLE_SCHEMA, TABLE_TYPE, BASE_TABLE_NAME
> FROM QSYS2.SYSTABLES
> WHERE BASE_TABLE_NAME = '[NOM_PF]' AND TABLE_SCHEMA = '[NOM_LIB]'
>   AND TABLE_TYPE IN ('L', 'V');
>
> -- Clés d'un fichier (PF ou LF)
> SELECT CONSTRAINT_NAME, COLUMN_NAME, ORDINAL_POSITION
> FROM QSYS2.SYSKEYS
> WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_PF_OU_LF]'
> ORDER BY ORDINAL_POSITION;
>
> -- Index sur une table (après création DDL)
> SELECT INDEX_NAME, COLUMN_NAMES, IS_UNIQUE, INDEX_TYPE
> FROM QSYS2.SYSINDEXES
> WHERE TABLE_SCHEMA = '[NOM_LIB]' AND TABLE_NAME = '[NOM_TABLE]';
> ```
> Ces requêtes permettent de résoudre les REFLD et de valider les objets créés sans sortir de Bob.

---

## Pièges à éviter

| Piège | Ce qui se passe | Comment l'éviter |
|-------|----------------|-----------------|
| Sauter le Prompt 0 et aller directement au Prompt 1 sur un fichier COMPLEXE | Les REFLD bloquants et les LF avec SELECT/OMIT complexes sont découverts en milieu d'analyse — la session est interrompue et doit recommencer | Toujours lancer le Prompt 0 en premier — il prend 2 minutes et évite de perdre 1 heure |
| Confondre PF et LF dans la stratégie de conversion | Un LF est converti en `CREATE TABLE` au lieu de `CREATE INDEX` ou `CREATE VIEW` — une table redondante est créée, l'index d'origine est perdu | Le Prompt 0 identifie le type dès l'inventaire rapide — vérifier la case "Type d'objet" avant tout |
| Ne pas résoudre les REFLD avant la génération | Le DDL produit des types erronés pour les champs référencés — données tronquées silencieusement à l'insertion | Le Prompt 0 liste les REFLD bloquants dans la section 3 — résoudre via `QSYS2.SYSCOLUMNS` avant de lancer le Prompt 1 |
| Ignorer les champs pseudo-date (`8S0` stocké YYYYMMDD) | Le champ est converti en `DECIMAL(8,0)` alors que l'usage est une date — les programmes SQL en UC 3 devront gérer la conversion manuellement | Le Prompt 0 liste ces champs dans la section 4 (champs ambigus) — décider du type cible avant le Prompt 2 |
| Convertir un LF avec SELECT/OMIT en CREATE INDEX | L'index n'a pas de capacité de filtrage — tous les enregistrements sont accessibles, le SELECT/OMIT est perdu | Pour tout LF avec SELECT/OMIT, utiliser `CREATE VIEW` avec clause `WHERE` équivalente |
| Générer le DDL sans l'en-tête de traçabilité | Impossible de retrouver le DDS d'origine 3 mois plus tard ; statut de validation inconnu | Toujours inclure l'en-tête structuré avec fichier DDS d'origine, date et statut "À valider" |
| Exécuter le CREATE TABLE en production sans test préalable | Erreur de type ou champ manquant détecté seulement lors d'un chargement de données — coûteux à corriger | Utiliser le Prompt 3-bis pour tester sur l'IBM i de test — ne pas aller en production sans ce test |
| Laisser les contraintes référentielles (FOREIGN KEY) implicites | Les intégrités entre tables sont perdues — des données orphelines peuvent apparaître après migration | UC 14 se limite au schéma DDS → DDL ; la définition des FOREIGN KEY est une étape séparée, après validation du modèle de données en UC 6 Prompt 3 |
| Démarrer UC 14 sans les fichiers `*-spec-tech-*.md` de UC 6 | L'inventaire des PF/LF est incomplet — des fichiers sont oubliés dans le périmètre de conversion | Vérifier que `QSYS2.SYSTABLES` (via IBM i Database MCP) retourne le même nombre de PF/LF que le document de spec technique avant de démarrer |
| Travailler en mode Agent pendant la génération DDL | Bob peut sauvegarder des scripts intermédiaires incomplets ou invalider des versions validées | Rester en mode **Ask** pendant toute la phase de génération (Prompts 0 à 3) — mode Agent uniquement pour Prompt 3-bis (test DDL) et sauvegarde finale |

---

## Check-list de validation UC 14

Avant de passer à UC 3 (voir `UC03-sql-embarque.md`), valider chaque point :

- [ ] L'inventaire des PF/LF à convertir est complet — vérifier contre `QSYS2.SYSTABLES` et les fichiers `*-spec-tech-*.md` de UC 6 : aucun PF oublié
- [ ] **Pour chaque fichier DDS converti : le Prompt 0 a été exécuté** — la fiche de qualification `{appArcad}-{fonction}-{composant}-analyse-dds-{YYYYMMDD-HHmm}.md` existe et mentionne la catégorie (SIMPLE / STANDARD / COMPLEXE) et la stratégie retenue
- [ ] Tous les REFLD bloquants identifiés dans les fiches de qualification (Prompt 0, section 3) ont été résolus avant la génération DDL
- [ ] Chaque PF a été analysé avec le Prompt 1 et le tableau de correspondance de champs est produit
- [ ] Les champs ambigus (pseudo-dates, REFLD, COMP-3) ont été soumis à décision par un développeur senior — les choix sont documentés dans le fichier `*-analyse-dds-*.md`
- [ ] Chaque PF a un script `CREATE TABLE` généré et relu par un développeur
- [ ] Chaque LF a été converti en `CREATE INDEX` ou `CREATE VIEW` selon sa nature — le choix est justifié dans le fichier analyse
- [ ] Le script de migration complet (Prompt 4) est produit et sauvegardé dans `{appArcad}-{fonction}-{fonction}-ddl-complet-{YYYYMMDD-HHmm}.md` — **ce fichier est l'input obligatoire de UC 3** (section "Démarrer une session Bob" de `UC03-sql-embarque.md`)
- [ ] Le script de migration a été exécuté sur l'IBM i de test — sans erreur SQL
- [ ] La validation post-création (Prompt 5) a été réalisée via `QSYS2` — aucun écart entre DDS d'origine et tables créées
- [ ] Le tableau récapitulatif DDS → SQL (Prompt 4) est complet : chaque PF/LF source a un objet SQL cible documenté
- [ ] Les scripts DDL sont sauvegardés avec la convention de nommage `{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md` dans le workspace ET publiés sur Confluence (si MCP disponible)
- [ ] La mention `⚠️ Réintégration ARCAD — à effectuer manuellement après validation` est présente dans l'en-tête de chaque script DDL

---

## Points à compléter avant passage en production

> Ces points ne bloquent pas le POC — ils concernent des cas avancés peu probables sur les fichiers pilotes. Ils deviennent critiques dès que la conversion s'étend à l'ensemble du parc applicatif en production.

### Triggers natifs IBM i (`ADDPFTRG`)

**Contexte :** certains fichiers physiques ont des triggers définis via la commande CL `ADDPFTRG` — des programmes RPG déclenchés automatiquement lors d'un `INSERT`, `UPDATE` ou `DELETE` sur le PF. Ces triggers sont attachés à l'objet PF DDS, **pas à la table SQL** créée par UC 14. Résultat : la table SQL créée se comporte différemment du PF DDS d'origine sans aucune erreur visible — les triggers ne se déclenchent plus.

**Pourquoi absent de la fiche POC :** les PF pilotes sont choisis parmi des fichiers simples sans triggers. Les triggers `ADDPFTRG` sont rares sur les fichiers candidats à un POC de démonstration.

**À faire avant production :** avant de convertir un PF en production, vérifier l'existence de triggers avec :
```sql
SELECT TRIGGER_NAME, EVENT_MANIPULATION, ACTION_TIMING,
       TRIGGER_BODY
FROM QSYS2.SYSTRIGGERS
WHERE EVENT_OBJECT_SCHEMA = '[NOM_LIB]'
  AND EVENT_OBJECT_TABLE  = '[NOM_PF]';
```
Si des triggers existent : les recréer sur la table SQL avec `CREATE TRIGGER` DDL, ou les remplacer par des procédures stockées selon l'architecture cible. Ajouter cette vérification dans le Prompt 0 (section "Facteurs de complexité") et dans le Prompt 5 (validation post-création).

> ⚠️ **Signal d'alerte sur le terrain :** si après la création de la table SQL des comportements métier disparaissent silencieusement (mise à jour d'une table dérivée qui ne se fait plus, journal applicatif non alimenté) — vérifier en premier lieu si le PF d'origine avait des triggers `ADDPFTRG`.

---

*Fiche UC 14 — Document évolutif à mettre à jour au fil du POC.*
