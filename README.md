# POC Bob × IBM i — Fiches pratiques de modernisation

> **Outil :** [IBM Bob V2](https://bob.ibm.com) — package Premium  
> **Environnement :** IBM i (AS/400) — VS Code fork avec extensions IBM i  
> **Langue :** Français — English version below ↓

---

## Présentation

Ce dépôt contient les **fiches pratiques d'un POC IBM Bob sur IBM i** : 17 use cases couvrant la mise en place du workspace d'équipe, la stratégie skills, la modernisation d'applications RPG/COBOL, la documentation, le développement, les tests unitaires et l'activation des serveurs MCP.

Ces fiches sont des guides opérationnels prêts à l'emploi : chaque fiche décrit les prérequis, les prompts à utiliser dans Bob, les livrables attendus, les pièges à éviter et une check-list de validation. Elles sont conçues pour être utilisées directement en session Bob, mode IBM i Developer.

**Ce que ce dépôt n'est pas :** un tutoriel général sur Bob ou sur IBM i. Les fiches supposent que Bob est installé, que la connexion IBM i est configurée, et que l'équipe a complété les UC 17 et UC 15 (workspace d'équipe + onboarding Bob).

---

## Structure du POC — 6 phases, 17 use cases

### Phase 0 — Onboarding Bob

| Fiche | Titre | Description courte |
|-------|-------|-------------------|
| [UC17](UC17-bob-en-equipe.md) | Bob en équipe ⚡ | Workspace entreprise partagé (rules, modes, skills, MCP) — à faire en tout premier |
| [UC18](UC18-strategie-skills.md) | Stratégie Skills ⚡ | Transformer les fiches UCxx en skills Bob réutilisables — juste après UC17 |
| [UC15](UC15-maitrise-bob.md) | Maîtrise de Bob | Modes Ask/Agent/Plan, modes personnalisés, gestion des Bob Coins |
| [UC12](UC12-mcp-enabler.md) | Serveurs MCP ⚡ | Activation JIRA, Confluence, IBM i MCP, IBM i Database MCP — enabler transversal |

### Phase 1 — Comprendre avant de moderniser

| Fiche | Titre | Description courte |
|-------|-------|-------------------|
| [UC04](UC04-comprehension-code.md) | Compréhension de code | Analyser le rôle, les entrées/sorties et les dépendances d'un programme IBM i |
| [UC05](UC05-logique-metier.md) | Extraction de la logique métier | Extraire les règles métier implicites et les patterns d'accès aux données |
| [UC06](UC06-documentation-complete.md) | Documentation complète | Produire specs fonctionnelles, diagrammes d'architecture, matrices de références croisées |

### Phase 2 — Modernisation de la couche base de données

| Fiche | Titre | Description courte |
|-------|-------|-------------------|
| [UC14](UC14-dds-ddl.md) | Conversion DDS → DDL | Fichiers physiques/logiques DDS vers tables et index SQL DDL |
| [UC03](UC03-sql-embarque.md) | SQL embarqué | Remplacement des accès natifs (CHAIN, READ, SETLL…) par du SQL embarqué |

### Phase 3 — Modernisation du code

| Fiche | Titre | Description courte |
|-------|-------|-------------------|
| [UC07](UC07-optimisation-code.md) | Optimisation de code | Renommage variables, extraction procédures, remplacement opcodes obsolètes |
| [UC08](UC08-restructuration-code.md) | Restructuration | Transformation structurelle, conversion du cycle RPG, extraction de modules |

### Phase 4 — Conversion RPG ‖ Développement (tracks parallèles)

**Track A — Conversion RPG**

| Fiche | Titre | Description courte |
|-------|-------|-------------------|
| [UC01](UC01-cobol-free-rpg.md) | COBOL → FREE RPG ILE | Conversion de programmes COBOL IBM i en RPG ILE Free |
| [UC02](UC02-rpg-colonne-free.md) | RPG Colonné → FREE RPG ILE | Conversion RPG IV colonné et RPG III en FREE RPG ILE |

**Track B — Développement**

| Fiche | Titre | Description courte |
|-------|-------|-------------------|
| [UC09](UC09-generation-code.md) | Génération de code | Générer des programmes RPG ILE Free et procédures selon les normes de l'équipe |
| [UC11](UC11-objets-sql.md) | Génération d'objets SQL | Tables, vues, index, procédures stockées, fonctions, triggers |
| [UC10](UC10-generation-app.md) | Génération d'applications | Nouvelle app 5250, display file DDS, conversion 5250 → Web (React/Vue/Angular) |

### Phase 5 — Tests

| Fiche | Titre | Description courte |
|-------|-------|-------------------|
| [UC13](UC13-tests-unitaires.md) | Tests unitaires | Création de tests RPGUnit, automatisation, rapport de non-régression |

---

## Comment utiliser ces fiches

1. **Commencer par UC17** — créer le workspace d'équipe partagé (rules, modes, skills) avant tout le reste.
2. **Enchaîner avec UC18** — transformer les fiches UC en skills Bob pour industrialiser les workflows.
3. **Puis UC15** — l'équipe doit maîtriser Bob avant tout autre use case.
4. **Activer UC12 dès la Phase 0** — les MCP JIRA/Confluence/IBM i accélèrent tous les UC suivants.
5. **Respecter l'ordre des phases** — chaque phase produit des livrables qui sont les inputs de la phase suivante.
6. **Utiliser le mode IBM i Developer** dans Bob — il est pré-configuré pour le contexte IBM i.
7. **Charger les fichiers sources dans l'éditeur** avant de démarrer un prompt (Open in Editor via Code for IBM i).
8. **Sauvegarder chaque livrable** avec la convention `{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md`.

### Note sur ARCAD

**Ce dépôt décrit un POC réalisé sans MCP ARCAD**, car la version ARCAD du client n'était pas compatible au moment de l'étude. Les fiches reposent donc sur un fonctionnement avec réintégration manuelle. Si le MCP ARCAD est disponible dans votre environnement, ces fiches peuvent être adaptées pour exploiter une intégration plus automatisée avec ARCAD.

Une fiche complémentaire anonyme est disponible pour préparer un futur POC avec MCP ARCAD actif : [`apport-mcp-arcad.md`](apport-mcp-arcad.md). Elle détaille, pour les UC 1 à 14, les apports attendus du MCP ARCAD, les bénéfices possibles, les limites et plusieurs options de cadrage à challenger avant lancement.

UC16 (DevOps/ARCAD) n'est pas couvert dans ce dépôt dans sa version complète, mais il représente **le UC avec le plus fort potentiel** lorsque le MCP ARCAD est disponible — Bob peut piloter l'intégralité du pipeline de déploiement IBM i.

---

## Prérequis techniques

| # | Prérequis | Détail |
|---|-----------|--------|
| 1 | **IBM Bob V2** | Installer depuis [bob.ibm.com](https://bob.ibm.com) + package Premium (modes IBM i Developer et IBM i Database) |
| 2 | **Extension Code for IBM i** | Éditeur IBM — connexion SSH/ODBC vers l'IBM i |
| 3 | **Extension IBM i Languages** | Coloration syntaxique RPG, CL, DDS, COBOL |
| 4 | **Accès IBM i** | Profil utilisateur, bibliothèques sources, droits sur les objets |
| 5 | **Tokens MCP** (optionnel) | Tokens API JIRA/Confluence pour UC12 |

---

---

# POC Bob × IBM i — Practical Modernisation Guides

> **Tool:** [IBM Bob V2](https://bob.ibm.com) — Premium package  
> **Environment:** IBM i (AS/400) — VS Code fork with IBM i extensions  
> **Language:** French (fiches) — English README below

---

## Overview

This repository contains **practical use-case guides from an IBM Bob POC on IBM i**: 17 use cases covering enterprise workspace setup, skills strategy, RPG/COBOL application modernisation, documentation, code generation, unit testing, and MCP server activation.

Each guide is an operational runbook: prerequisites, Bob prompts, expected deliverables, pitfalls to avoid, and a validation checklist. They are designed to be used directly in a Bob session, IBM i Developer mode.

**What this repository is not:** a general tutorial on Bob or IBM i. The guides assume Bob is installed, the IBM i connection is configured, and the team has completed UC17 and UC15 (enterprise workspace + Bob onboarding).

---

## POC Structure — 6 Phases, 17 Use Cases

### Phase 0 — Bob Onboarding

| Guide | Title | Summary |
|-------|-------|---------|
| [UC17](UC17-bob-en-equipe.md) | Bob for Teams ⚡ | Shared enterprise workspace (rules, modes, skills, MCP) — set this up first |
| [UC18](UC18-strategie-skills.md) | Skills Strategy ⚡ | Turn UC guides into reusable Bob skills — right after UC17 |
| [UC15](UC15-maitrise-bob.md) | Mastering Bob | Ask/Agent/Plan modes, custom modes, Bob Coins management |
| [UC12](UC12-mcp-enabler.md) | MCP Servers ⚡ | Activate JIRA, Confluence, IBM i MCP, IBM i Database MCP — cross-cutting enabler |

### Phase 1 — Understand Before Modernising

| Guide | Title | Summary |
|-------|-------|---------|
| [UC04](UC04-comprehension-code.md) | Code Comprehension | Analyse role, inputs/outputs and dependencies of an IBM i program |
| [UC05](UC05-logique-metier.md) | Business Logic Extraction | Extract implicit business rules and data access patterns |
| [UC06](UC06-documentation-complete.md) | Full Documentation | Produce functional specs, architecture diagrams, cross-reference matrices |

### Phase 2 — Database Layer Modernisation

| Guide | Title | Summary |
|-------|-------|---------|
| [UC14](UC14-dds-ddl.md) | DDS → DDL Conversion | Physical/logical DDS files to SQL DDL tables and indexes |
| [UC03](UC03-sql-embarque.md) | Embedded SQL | Replace native I/O opcodes (CHAIN, READ, SETLL…) with embedded SQL |

### Phase 3 — Code Modernisation

| Guide | Title | Summary |
|-------|-------|---------|
| [UC07](UC07-optimisation-code.md) | Code Optimisation | Variable renaming, procedure extraction, obsolete opcode replacement |
| [UC08](UC08-restructuration-code.md) | Code Restructuring | Structural transformation, RPG cycle conversion, module extraction |

### Phase 4 — RPG Conversion ‖ Development (parallel tracks)

**Track A — RPG Conversion**

| Guide | Title | Summary |
|-------|-------|---------|
| [UC01](UC01-cobol-free-rpg.md) | COBOL → FREE RPG ILE | Convert IBM i COBOL programs to RPG ILE Free |
| [UC02](UC02-rpg-colonne-free.md) | Fixed-format RPG → FREE RPG ILE | Convert RPG IV fixed-format and RPG III to FREE RPG ILE |

**Track B — Development**

| Guide | Title | Summary |
|-------|-------|---------|
| [UC09](UC09-generation-code.md) | Code Generation | Generate RPG ILE Free programs and procedures following team coding standards |
| [UC11](UC11-objets-sql.md) | SQL Object Generation | Tables, views, indexes, stored procedures, functions, triggers |
| [UC10](UC10-generation-app.md) | Application Generation | New 5250 app, DDS display file, 5250 → Web conversion (React/Vue/Angular) |

### Phase 5 — Testing

| Guide | Title | Summary |
|-------|-------|---------|
| [UC13](UC13-tests-unitaires.md) | Unit Tests | RPGUnit test creation, automation, non-regression report |

---

## How to Use These Guides

1. **Start with UC17** — set up the shared enterprise workspace (rules, modes, skills) before anything else.
2. **Then UC18** — turn the UC guides into Bob skills to industrialise workflows for the whole team.
3. **Then UC15** — the team must be comfortable with Bob before any other use case.
4. **Activate UC12 in Phase 0** — JIRA/Confluence/IBM i MCPs accelerate all subsequent use cases.
5. **Follow the phase order** — each phase produces deliverables that are inputs for the next phase.
6. **Use IBM i Developer mode** in Bob — it is pre-configured for the IBM i context.
7. **Load source files in the editor** before starting a prompt (Open in Editor via Code for IBM i).
8. **Save every deliverable** using the naming convention `{appArcad}-{function}-{component}-{type}-{YYYYMMDD-HHmm}.md`.

### Note on ARCAD

**This repository documents a POC conducted without the ARCAD MCP**, because the client's ARCAD version was not compatible at the time of the study. The guides therefore rely on a workflow with manual reintegration. If the ARCAD MCP is available in your environment, these guides can be adapted to support a more automated integration with ARCAD.

An additional anonymised companion note is available to prepare a future POC with ARCAD MCP enabled: [`apport-mcp-arcad.md`](apport-mcp-arcad.md). It details, for UC 1 to 14, the expected contribution of ARCAD MCP, the potential benefits, the limits and several framing options to challenge before launch.

UC16 (DevOps/ARCAD) is not fully covered in this repository, but it represents **the use case with the highest potential** when the ARCAD MCP is available — Bob can drive the entire IBM i deployment pipeline directly.

---

## Technical Prerequisites

| # | Prerequisite | Detail |
|---|-------------|--------|
| 1 | **IBM Bob V2** | Install from [bob.ibm.com](https://bob.ibm.com) + Premium package (IBM i Developer and IBM i Database modes) |
| 2 | **Code for IBM i extension** | IBM editor — SSH/ODBC connection to IBM i |
| 3 | **IBM i Languages extension** | RPG, CL, DDS, COBOL syntax highlighting |
| 4 | **IBM i access** | User profile, source libraries, object permissions |
| 5 | **MCP tokens** (optional) | JIRA/Confluence API tokens for UC12 |

---

## Repository Contents

| File | Description |
|------|-------------|
| `UC01` – `UC15`, `UC17`, `UC18` | Operational use-case guides |
| `plan-poc-bob-acme.md` | Full POC roadmap: phases, UC ordering, ARCAD integration context, deliverables map |
| `apport-mcp-arcad.md` | Companion analysis for a future POC with ARCAD MCP: impact by use case (UC1–UC14), benefits, limits, and framing options |
| `README.md` | This file |
