# Plan POC — Bob × ACME (ACME) sur IBM i

> **Client :** ACME (ACME)  
> **Environnement :** TechZone — `[TECHZONE_ID]` (region: us-east)  
> **Outil :** IBM Bob V2 avec package Premium  
> **Accès :** IBM i de test fourni dans le cadre du POC

---

## 1. Prérequis

Avant de démarrer les use cases, chaque membre de l'équipe doit avoir complété les étapes suivantes :

| # | Prérequis | Détail |
|---|-----------|--------|
| P1 | **IBM ID** | Vérifier que vous disposez d'un IBM ID associé à l'adresse e-mail du POC. Si ce n'est pas le cas, créer un IBM ID sur [ibm.com/account](https://www.ibm.com/account). |
| P2 | **Installer Bob V2** | Télécharger et installer la dernière version (V2) de Bob depuis [bob.ibm.com](https://bob.ibm.com). |
| P3 | **Package Premium** | Dans Bob, ouvrir le **Marketplace** et installer le package **Premium** (nécessaire pour les modes IBM i Developer et IBM i Database). |
| P4 | **Profil Entreprise TechZone** | Configurer le profil entreprise dans Bob en liant le compte TechZone du POC : `[TECHZONE_ID]` (region: us-east). Ce profil donne accès aux Bob Coins alloués au POC. |
| P5 | **Extension Code for IBM i** | Bob étant un fork de VS Code, les extensions VS Code sont installables directement dans Bob. Installer l'extension **Code for IBM i** (éditeur IBM) via le marketplace de Bob. Configurer la connexion SSH/ODBC vers l'IBM i de test. |
| P6 | **Extension IBM i Languages** | Installer l'extension **IBM i Languages** dans Bob (coloration syntaxique RPG, CL, DDS, COBOL). |
| P7 | **Accès IBM i de test** | Vérifier l'accès à l'IBM i de test fourni dans le POC (profil utilisateur, bibliothèques, droits sur les objets sources). |
| P8 | **Lab Flight400** | Accéder au lab de référence Bob pour IBM i : [IBM i Application Modernization with Bob](https://github.ibm.com/ClientEngineering/bob/blob/main/LABs/IBM-i-Application-Modernization-with-Bob/README.md). Ce lab sera utilisé lors de l'UC 15 (onboarding). |

---

## 2. Liste des use cases

| N° | Catégorie | Intitulé | Sous-cas |
|----|-----------|----------|----------|
| 1 | Conversion FREE RPG | Du COBOL vers le FREE RPG | COBOL → FREE RPG |
| 2 | Conversion FREE RPG | Du RPG Colonné vers le FREE RPG | RPG IV colonné → FREE ; RPG III → FREE |
| 3 | Modernisation | Accès natif base de données → SQL embarqué | Conversion programme / partie |
| 4 | Documentation | Compréhension de code et d'applications | Explication / documentation |
| 5 | Documentation | Explication de code et d'applications | Extraction logique métier ; Requêtes SQL |
| 6 | Documentation | Documentation de code et d'applications | Modèles d'architecture ; Spécifications fonctionnelles et techniques ; Diagrammes des relations ; Références croisées |
| 7 | Modernisation | Amélioration, optimisation de code | Refactoring procédures ; Renommage variables ; Optimisation performance / maintenabilité |
| 8 | Modernisation | Restructuration, transformation, conversion | Restructuration et transformation d'applications |
| 9 | Développement | Génération du code | Génération rapide programme/procédures ; Normes de développement ; Conventions de nommage |
| 10 | Développement | Génération d'applications | Nouvelle app 5250 ; Web UI moderne (JS) ; 5250 → Web (React/Angular) |
| 11 | Développement | Génération d'objets SQL | Tables, vues, index, procédures stockées, fonctions, triggers |
| 12 | Développement | Utilisation des serveurs MCP | JIRA, Confluence, IBM i MCP, IBM i Database MCP |
| 13 | Tests | Création de procédures de test | Tests unitaires (RPGUnit) ; Automatisation |
| 14 | Conversion base de données | Conversion DDS → DDL | Fichiers physiques/logiques DDS → Tables/Index SQL |
| 15 | Utilisation de BOB | Maîtrise de Bob | Modes ASK, Code, Plan, Agent ; Modes personnalisés ; Télécharger des modes |
| 16 | DevOps | Pipelines CI/CD | ARCAD ; Automatisation des déploiements IBM i |

---

## 3. Ordre des use cases et justification

### Séquence globale

```
UC 15 + UC 12 ⚡ (enabler transversal MCP — activer dès la Phase 0)
  → UC 4 → 5 → 6
    → UC 14 + UC 3 (en parallèle possible)
      → UC 7 → UC 8
        → [UC 1 & UC 2]  ‖  [UC 9, UC 11, UC 10]  (2 tracks parallèles)
          → UC 13
            → UC 16
```

---

### Phase 0 — Onboarding Bob

| Ordre | UC | Justification |
|-------|----|---------------|
| 1 | **UC 15 — Maîtrise de Bob** | L'équipe doit être autonome sur Bob avant tout autre use case : navigation entre les modes (ASK, Code, Plan, Agent), création et import de modes personnalisés, gestion des Bob Coins. Le lab Flight400 sert de support pratique. Sans cette phase, les autres UC seront sous-exploités. |
| 2 ⚡ | **UC 12 — Serveurs MCP** | Enabler transversal à activer dès la Phase 0, en parallèle de UC 15. L'accès aux MCP JIRA et Confluence accélère tous les UC suivants : lecture des tickets pour contextualiser le code, publication automatique des docs produites en Phase 1, création d'issues depuis les anomalies détectées. Nécessite la préparation des droits API JIRA/Confluence avec l'équipe IT du client — ne pas attendre la dernière minute. |

---

### Phase 1 — Comprendre avant de moderniser

| Ordre | UC | Justification |
|-------|----|---------------|
| 3 | **UC 4 — Compréhension de code** | On ne modernise pas ce qu'on ne comprend pas. Ce UC produit une première lecture fonctionnelle du code legacy — rôle, entrées, sorties, dépendances. |
| 4 | **UC 5 — Extraction logique métier** | Approfondissement de UC 4 : extraction des règles métier implicites et des requêtes de données. Ces règles serviront de référence pour valider les conversions ultérieures. |
| 5 | **UC 6 — Documentation complète** | Capitalisation de tout ce qui a été compris : specs fonctionnelles et techniques, diagrammes d'architecture, matrices de références croisées. Ces livrables sont les inputs des phases de modernisation. |

---

### Phase 2 — Modernisation de la couche base de données

| Ordre | UC | Justification |
|-------|----|---------------|
| 6 | **UC 14 — DDS → DDL** | Moderniser la *structure* des données : fichiers physiques et logiques DDS vers tables et index SQL DDL. Placé avant UC 3 car il faut connaître les tables cibles SQL avant de convertir les accès. |
| 7 | **UC 3 — SQL embarqué** | Moderniser l'*accès* aux données : remplacement des accès natifs (CHAIN, READ, SETLL, WRITE…) par du SQL embarqué. UC 14 et UC 3 sont complémentaires et peuvent être menés en parallèle sur le même périmètre applicatif si l'équipe est suffisante. |

---

### Phase 3 — Modernisation du code

| Ordre | UC | Justification |
|-------|----|---------------|
| 8 | **UC 7 — Optimisation de code** | UC le moins risqué pour démarrer la modernisation du code : renommage de variables, extraction de procédures, optimisation de performance. Pas de changement de logique fonctionnelle. |
| 9 | **UC 8 — Restructuration** | Transformation structurelle plus profonde. Nécessite d'avoir la documentation (Phase 1) et une équipe à l'aise avec Bob (Phase 0). C'est le UC le plus risqué de cette phase — toujours partir des specs produites en Phase 1. |

---

### Phase 4 — Tracks parallèles (Conversion RPG ‖ Développement)

Ces deux tracks peuvent être menées en parallèle par des sous-équipes différentes.

**Track A — Conversion RPG**

| Ordre | UC | Justification |
|-------|----|---------------|
| 10a | **UC 1 — COBOL → FREE RPG** | Conversion la plus complexe : nécessite une bonne maîtrise de Bob et des deux langages. Placé après la modernisation pour que l'équipe ait de l'expérience avec Bob avant d'aborder ce UC à haut risque métier. |
| 10b | **UC 2 — RPG Colonné → FREE RPG** | Conversion plus accessible que UC 1. RPG IV colonné → FREE est le cas le plus courant chez les clients IBM i. RPG III → FREE est plus délicat (gestion du cycle RPG). |

**Track B — Développement**

| Ordre | UC | Justification |
|-------|----|---------------|
| 11a | **UC 9 — Génération de code** | Génération de programmes et procédures selon les normes ACME. UC accessible, fort ROI démonstrable rapidement. Idéal pour créer un mode personnalisé "ACME Developer" intégrant les normes de développement du client. |
| 11b | **UC 11 — Objets SQL** | Génération de tables, vues, index, procédures stockées, fonctions, triggers. Complémentaire de UC 14. |
| 11c | **UC 10 — Génération d'applications** | UC le plus consommateur en Bob Coins et en effort de cadrage. Abordé en dernier de la track développement car il nécessite une maîtrise solide de Bob et une réflexion UX pour la conversion 5250 → Web. |

---

### Phase 5 — Tests

| Ordre | UC | Justification |
|-------|----|---------------|
| 12 | **UC 13 — Tests unitaires** | Déclenché dès qu'il existe du code modernisé à valider (après UC 7-8 au plus tôt). Les tests sont la preuve de non-régression de toutes les conversions et modernisations. |

---

### Phase 6 — DevOps (capstone)

| Ordre | UC | Justification |
|-------|----|---------------|
| 13 | **UC 16 — CI/CD avec ARCAD** | Capstone du POC : intégrer tout le cycle (build, test, déploiement) dans un pipeline automatisé. Ne démarrer qu'une fois le code stable et les tests passants. Vérifier préalablement la disponibilité de la licence ARCAD sur l'IBM i de test. **Note : le MCP ARCAD n'est pas actif dans ce POC (voir section 5 ci-dessous) — la portée de UC 16 est adaptée en conséquence.** |

---

## 4. Vue synthétique — Difficulté, risque et consommation Bob Coins

> **Risque métier** = risque en cas d'absence de préparation avant d'aborder le use case.

| Ordre | UC | Difficulté Bob | Risque métier (si absence de préparation) | Bob Coins |
|-------|----|---------------|-------------------------------------------|-----------|
| 1 | UC 15 — Maîtrise de Bob | Faible | Nul | ● ○ ○ |
| 2 ⚡ | UC 12 — MCP (enabler transversal) | Moyen (config) | Élevé — sans MCP, les autres UC perdent en efficacité et la doc ne se publie pas automatiquement | ● ○ ○ |
| 3 | UC 4 — Compréhension | Faible | Moyen — moderniser sans comprendre le code = risque de casser la logique métier | ● ○ ○ |
| 4 | UC 5 — Logique métier | Faible | Moyen — règles métier non identifiées = erreurs silencieuses dans les conversions | ● ○ ○ |
| 5 | UC 6 — Documentation | Moyen | Faible | ● ● ○ |
| 6 | UC 14 — DDS → DDL | Moyen | Élevé — conversion sans analyse = perte de contraintes, types incorrects, programmes appelants cassés | ● ● ○ |
| 7 | UC 3 — SQL embarqué | Moyen | Élevé — conversion des accès sans connaître les tables cibles DDL = requêtes incorrectes | ● ● ○ |
| 8 | UC 7 — Optimisation | Faible | Faible | ● ○ ○ |
| 9 | UC 8 — Restructuration | Élevé | Élevé — restructurer sans documentation = risque de régression fonctionnelle majeure | ● ● ● |
| 10a | UC 1 — COBOL → FREE RPG | Élevé | Élevé — validation par expert COBOL + RPG indispensable | ● ● ○ |
| 10b | UC 2 — RPG Colonné → FREE | Moyen | Moyen — cycle RPG et indicateurs de niveaux de contrôle mal convertis | ● ● ○ |
| 11a | UC 9 — Génération code | Faible | Faible | ● ○ ○ |
| 11b | UC 11 — Objets SQL | Faible | Faible | ● ○ ○ |
| 11c | UC 10 — Génération app | Élevé | Moyen — sans réflexion UX préalable, la conversion 5250 → Web sera un simple copier-coller sans valeur | ● ● ● |
| 12 | UC 13 — Tests | Moyen | Élevé — sans tests, aucune garantie de non-régression sur les modernisations | ● ● ○ |
| 13 | UC 16 — DevOps / ARCAD | Élevé | **Élevé** — MCP ARCAD non actif (incompatibilité version) : Bob ne peut pas piloter les pipelines directement ; portée de UC 16 réduite à la documentation et aux scripts | ● ● ○ |

---

## 5. Contrainte transversale — ARCAD sans MCP

> ⚠️ **Contrainte identifiée dès la Phase 0 — à communiquer à l'équipe avant de démarrer.**

ACME utilise **ARCAD** pour la gestion du code source IBM i (versioning, packaging, déploiement). La version ARCAD en production chez ACME n'est **pas compatible avec le MCP ARCAD** de Bob V2.

### Impact par phase

| Phase | UC concernés | Impact | Adaptation |
|-------|-------------|--------|-----------|
| Phase 1 — Comprendre | UC 4, 5, 6 | **Faible** — ARCAD gère les versions, pas le code en lui-même. IBM i MCP accède aux sources directement dans les bibliothèques, indépendamment d'ARCAD | Ajouter le placeholder `⚠️ Historique ARCAD — à compléter manuellement` dans les livrables de spec technique (UC 6 Prompt 3) |
| Phase 2 — Base de données | UC 14, 3 | **Faible** — même logique que Phase 1 | Aucune adaptation nécessaire |
| Phase 3 — Code | UC 7, 8 | **Faible** — les modifications de code se font dans le workspace Bob, pas via ARCAD | Les sources modifiés devront être réintégrés dans ARCAD manuellement après la session Bob |
| Phase 4 — Conversion/Dev | UC 1, 2, 9, 10, 11 | **Faible à moyen** — même logique Phase 3 | Définir un workflow de réintégration ARCAD en dehors de Bob |
| Phase 5 — Tests | UC 13 | **Faible** | Les sources de test (`QTESTSRC`) et rapports sont créés hors ARCAD — réintégrer manuellement dans ARCAD si versioning souhaité. Inclure le **manifest de traçabilité** dans chaque `*-rapport-test-*.md` (OBJCREATED, CHANGE_TIMESTAMP, SOURCE_TIMESTAMP, SOURCE_FILE/LIBRARY/MEMBER) pour lier le rapport à l'objet exact testé |
| Phase 6 — DevOps | **UC 16** | **Élevé** — UC 16 est centré sur ARCAD. Sans MCP, Bob ne peut pas lire les environnements, déclencher des builds, ni suivre les déploiements | Portée de UC 16 réduite : Bob documente le pipeline et génère des scripts, mais l'exécution reste manuelle dans l'interface ARCAD |

### Contournements disponibles

| Besoin | Contournement sans MCP ARCAD |
|--------|------------------------------|
| Lire l'historique des modifications d'un objet | Exporter depuis ARCAD (CSV/texte) → charger dans le contexte Bob |
| Connaître la liste des objets managés par ARCAD | Exporter la liste depuis ARCAD → charger dans le contexte Bob |
| Déclencher un déploiement | Non contournable via Bob — exécution manuelle dans l'interface ARCAD |
| Créer une tâche ARCAD depuis Bob | Non contournable — utiliser JIRA MCP comme alternative pour le suivi des tâches |

### Recommandation

Anticiper dès la Phase 0 avec l'équipe IT de ACME : vérifier si une mise à jour de la version ARCAD est envisageable avant UC 16, ou si une instance ARCAD compatible peut être déployée sur l'IBM i de test du POC. Si ce n'est pas possible, **recadrer UC 16 comme un UC de documentation du pipeline** plutôt que d'automatisation.

---

## 6. Carte des livrables — par UC, dans l'ordre de travail

> Cette section est la référence centrale pour savoir **quels fichiers sont produits par chaque UC** et **quels fichiers sont nécessaires pour démarrer l'UC suivant**.
> Avant de démarrer un UC, vérifier que tous les fichiers marqués **Obligatoire** sont présents dans le workspace.
> Convention de nommage : `{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md`

---

### Phase 0 — Onboarding

| UC | Fiche | Fichiers produits | Nécessaires pour |
|----|-------|-------------------|-----------------|
| **UC 15** — Maîtrise de Bob | — | *(aucun livrable fichier)* | Toutes les phases suivantes |
| **UC 12** — MCP Enabler | — | *(configuration MCP — pas de fichier .md)* | Toutes les phases suivantes |

---

### Phase 1 — Comprendre

| UC | Fiche | Fichiers produits | Nécessaires pour |
|----|-------|-------------------|-----------------|
| **UC 4** — Compréhension | `UC04-comprehension-code.md` | `*-comprehension-{date}.md` | UC 5, UC 6, UC 14, UC 3, UC 7, UC 8 |
| **UC 5** — Logique métier | `UC05-logique-metier.md` | `*-regles-{date}.md` | UC 6, UC 7, UC 8 |
| **UC 6** — Documentation | `UC06-documentation-complete.md` | `*-spec-fonc-{date}.md`<br>`*-spec-tech-{date}.md`<br>`*-matrice-{date}.md`<br>`*-architecture-{date}.md` | UC 14, UC 3, UC 7, UC 8 |

---

### Phase 2 — Modernisation base de données

| UC | Fiche | Fichiers produits | Nécessaires pour |
|----|-------|-------------------|-----------------|
| **UC 14** — DDS → DDL | `UC14-dds-ddl.md` | `*-analyse-dds-{date}.md`<br>`*-ddl-table-{date}.md`<br>`*-ddl-index-{date}.md`<br>**`*-ddl-complet-{date}.md`** ← clé | UC 3 (**Obligatoire**) |
| **UC 3** — SQL embarqué | `UC03-sql-embarque.md` | `*-analyse-acces-{date}.md`<br>`*-sql-embarque-{date}.md`<br>`*-plan-conversion-{date}.md` | UC 7, UC 8 (Si UC 3 appliqué) |

> ⚠️ **Point de synchronisation Phase 2 → Phase 3 :** le fichier `*-ddl-complet-{date}.md` (UC 14) doit exister et avoir été testé sur l'IBM i avant de démarrer UC 3. UC 3 peut ensuite être mené en parallèle avec UC 7 sur des programmes différents.

---

### Phase 3 — Modernisation du code

| UC | Fiche | Fichiers produits | Nécessaires pour |
|----|-------|-------------------|-----------------|
| **UC 7** — Optimisation | `UC07-optimisation-code.md` | `*-qualification-optim-{date}.md`<br>`*-diff-optim-{date}.md`<br>`*-plan-optim-{date}.md` | UC 8 (si UC 7 appliqué avant UC 8 sur le même programme) |
| **UC 8** — Restructuration | `UC08-restructuration-code.md` | `*-qualification-restr-{date}.md`<br>`*-plan-archi-{date}.md`<br>`*-spec-module-{date}.md`<br>`*-diff-restr-{date}.md`<br>`*-plan-restr-{date}.md` | UC 13 (tests de non-régression sur le code restructuré) |

> 💡 **Ordre sur un même programme :** UC 7 d'abord (optimisation), UC 8 ensuite (restructuration). Exception : si le Prompt 0 de UC 3 ou UC 7 conclut "restructuration préalable recommandée", démarrer UC 8 sur ce programme avant de continuer.

---

### Phase 4 — Tracks parallèles (Conversion RPG ‖ Développement)

**Track A — Conversion RPG**

| UC | Fiche | Fichiers produits | Nécessaires pour |
|----|-------|-------------------|-----------------|
| **UC 1** — COBOL → FREE RPG ✅ | `UC01-cobol-free-rpg.md` | `*-analyse-cobol-*`<br>`*-cobol-converti-*`<br>`*-plan-cobol-*` | UC 13 (tests) |
| **UC 2** — RPG Colonné → FREE ✅ | `UC02-rpg-colonne-free.md` | `*-qualification-conv-*`<br>`*-rpg-converti-*`<br>`*-plan-conv-rpg-*` | UC 13 (tests) |

**Track B — Développement**

| UC | Fiche | Fichiers produits | Nécessaires pour |
|----|-------|-------------------|-----------------|
| **UC 9** — Génération code | `UC09-generation-code.md` | `*-programme-genere-{date}.md`<br>`*-plan-generation-{date}.md` | UC 13 (tests) |
| **UC 11** — Objets SQL | `UC11-objets-sql.md` | `*-objet-sql-*`<br>`*-procedure-sql-*`<br>`*-trigger-sql-*`<br>`*-plan-sql-*` | UC 13 (tests) |
| **UC 10** — Génération apps | `UC10-generation-app.md` | `*-analyse-ecrans-*`<br>`*-dspf-genere-*`<br>`*-web-genere-*`<br>`*-plan-ecrans-*` | UC 13 (tests) |

> 💡 Le mode personnalisé "ACME Developer" (fichier JSON/YAML) est également produit par UC 9. Il n'obéit pas à la convention `*-programme-genere-*` — il est sauvegardé séparément (ex. `acme-developer-mode-v1.json`) et distribué à l'équipe via l'import de modes Bob.

---

### Phase 5 — Tests

| UC | Fiche | Fichiers produits | Nécessaires pour |
|----|-------|-------------------|-----------------|
| **UC 13** — Tests unitaires | `UC13-tests-unitaires.md` | `*-analyse-test-{date}.md`<br>`*-programme-test-{date}.md`<br>`*-plan-test-{date}.md`<br>`*-rapport-test-{date}.md` | UC 16 (preuve de non-régression avant déploiement) |

> 💡 **Point de synchronisation Phase 5 → Phase 6 :** les fichiers `*-rapport-test-*.md` doivent exister pour tous les programmes modernisés avant de démarrer UC 16. Toute régression détectée (cas FAILED) doit être corrigée dans l'UC de modernisation source avant de continuer.

---

### Tableau récapitulatif — tous types de fichiers

| Type de fichier | Produit par | Consommé par |
|-----------------|-------------|-------------|
| `*-comprehension-*` | UC 4 | UC 5, UC 6, UC 14, UC 3, UC 7, UC 8 |
| `*-regles-*` | UC 5 | UC 6, UC 7, UC 8 |
| `*-spec-fonc-*` | UC 6 | UC 8 (référence fonctionnelle) |
| `*-spec-tech-*` | UC 6 | UC 14, UC 3, UC 7, UC 8 |
| `*-matrice-*` | UC 6 | UC 14, UC 3, UC 7, UC 8 |
| `*-analyse-dds-*` | UC 14 | UC 14 (reprise de session) |
| `*-ddl-table-*` | UC 14 | UC 14 (assemblage ddl-complet) |
| `*-ddl-index-*` | UC 14 | UC 14 (assemblage ddl-complet) |
| **`*-ddl-complet-*`** | **UC 14** | **UC 3 (Obligatoire)** |
| `*-analyse-acces-*` | UC 3 | UC 3 (reprise de session) |
| `*-sql-embarque-*` | UC 3 | UC 7, UC 8 (si UC 3 appliqué) |
| `*-plan-conversion-*` | UC 3 | Pilotage périmètre UC 3 |
| `*-qualification-optim-*` | UC 7 | UC 7 (reprise), UC 8 (contexte) |
| `*-diff-optim-*` | UC 7 | UC 8 (source optimisé en entrée) |
| `*-plan-optim-*` | UC 7 | Pilotage périmètre UC 7 |
| `*-qualification-restr-*` | UC 8 | UC 8 (reprise de session) |
| `*-plan-archi-*` | UC 8 | UC 8 (reprise après Prompt 1) |
| `*-spec-module-*` | UC 8 | UC 8 (Prompt 3 — programme principal) |
| `*-diff-restr-*` | UC 8 | UC 13 (tests de non-régression) |
| `*-plan-restr-*` | UC 8 | Pilotage périmètre UC 8 |
| `*-analyse-cobol-*`      | UC 1  | UC 1 (reprise de session) |
| `*-cobol-converti-*`     | UC 1  | UC 13 (tests non-régression) |
| `*-plan-cobol-*`         | UC 1  | Pilotage périmètre UC 1 |
| `*-qualification-conv-*` | UC 2  | UC 2 (reprise de session) |
| `*-rpg-converti-*`       | UC 2  | UC 13 (tests non-régression) |
| `*-plan-conv-rpg-*`      | UC 2  | Pilotage périmètre UC 2 |
| `*-programme-genere-*`   | UC 9  | UC 13 (tests) |
| `*-plan-generation-*`    | UC 9  | Pilotage périmètre UC 9 |
| `*-objet-sql-*`          | UC 11 | UC 13 (tests) |
| `*-procedure-sql-*`      | UC 11 | UC 13 (tests) |
| `*-trigger-sql-*`        | UC 11 | UC 13 (tests) |
| `*-plan-sql-*`           | UC 11 | Pilotage périmètre UC 11 |
| `*-analyse-ecrans-*`     | UC 10 | UC 10 (reprise) |
| `*-dspf-genere-*`        | UC 10 | UC 13 (tests) |
| `*-web-genere-*`         | UC 10 | UC 13 (tests) |
| `*-plan-ecrans-*`        | UC 10 | Pilotage périmètre UC 10 |
| `*-analyse-test-*`       | UC 13 | UC 13 (reprise de session P0) |
| `*-programme-test-*`     | UC 13 | UC 13 (exécution P4), UC 16 (intégration pipeline) |
| `*-plan-test-*`          | UC 13 | Pilotage périmètre UC 13 |
| `*-rapport-test-*`       | UC 13 | UC 16 (preuve de qualité avant déploiement) |

---

## 7. État d'avancement des fiches

| UC | Fiche | Statut | Notes |
|----|-------|--------|-------|
| UC 1 — COBOL → FREE RPG | `UC01-cobol-free-rpg.md` | ✅ Figée — prête à distribuer | Correction `READ(E)` conditionnel, règle `(E)` clarifiée, `**FREE` directive, `%ERROR` sans paramètre, `MOVE/DIVIDE/STOP RUN` encadrés, architecture hybride COBOL+RPG, golden master obligatoire |
| UC 2 — RPG Colonné → FREE | `UC02-rpg-colonne-free.md` | ✅ Figée — prête à distribuer | Détection RPG III vs RPG IV, `MOVE/MOVEL` protocole complet, indicateurs INDARA, cycle L1–L9 → UC 8, séquence Prompt 2→3→4→4-bis pour STANDARD/COMPLEXE, `SETON/SETOF` non compilables en fully free-form, `CHAIN/READ` formes conditionnelles, `DCL-PI` au Prompt 2 |
| UC 3 — SQL embarqué | `UC03-sql-embarque.md` | ✅ Fiche de référence — testée POC | — |
| UC 4 — Compréhension de code | `UC04-comprehension-code.md` | ⬜ Fiche à créer | — |
| UC 5 — Logique métier | `UC05-logique-metier.md` | ⬜ Fiche à créer | — |
| UC 6 — Documentation complète | `UC06-documentation-complete.md` | ⬜ Fiche à créer | — |
| UC 7 — Optimisation de code | `UC07-optimisation-code.md` | ✅ Fiche de référence — testée POC | — |
| UC 8 — Restructuration | `UC08-restructuration-code.md` | ✅ Fiche de référence — testée POC | — |
| UC 9 — Génération de code | `UC09-generation-code.md` | ⬜ Fiche à créer | — |
| UC 10 — Génération d'applications | `UC10-generation-app.md` | ⬜ Fiche à créer | — |
| UC 11 — Objets SQL | `UC11-objets-sql.md` | ⬜ Fiche à créer | — |
| UC 12 — Serveurs MCP | `UC12-mcp.md` | ⬜ Fiche à créer | — |
| UC 13 — Tests unitaires | `UC13-tests-unitaires.md` | ✅ Figée — validée après 5 cycles de révision | Neutralisation RPGUnit : copybook/assertions/hooks/commande de compilation découverts depuis l'installation réelle (P0). Ordre P4 : `VALUES QSYS2.JOB_NAME` + `VALUES CURRENT_TIMESTAMP` *avant* `RUCALLTST`. Spool filtré par `CREATE_TIMESTAMP` (pas QPRINT hardcodé). `JOB_STATUS` / `COMPLETION_STATUS` dissociés. Manifest ARCAD : OBJCREATED, CHANGE_TIMESTAMP, SOURCE_TIMESTAMP. 8 prompts dans `U_BOB_IBM/.bob/prompts/` |
| UC 14 — DDS → DDL | `UC14-dds-ddl.md` | ✅ Fiche de référence — testée POC | — |
| UC 15 — Maîtrise de Bob | `UC15-maitrise-bob.md` | ⬜ Fiche à créer | — |
| UC 16 — CI/CD ARCAD | `UC16-cicd-arcad.md` | ⬜ Fiche à créer | Portée réduite — MCP ARCAD non actif (voir section 5) |

---

*Document évolutif — à mettre à jour au fil de l'avancement du POC.*
