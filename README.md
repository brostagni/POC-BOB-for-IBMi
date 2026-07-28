# POC Bob × IBM i — Fiches pratiques de modernisation

> **Outil :** [IBM Bob V2](https://bob.ibm.com) — package Premium  
> **Environnement :** IBM i (AS/400) — VS Code fork avec extensions IBM i  
> **Langue :** Français — English version below ↓

---

## Présentation

Ce dépôt contient les **fiches pratiques d'un POC IBM Bob sur IBM i** : 15 use cases couvrant la modernisation d'applications RPG/COBOL, la documentation, le développement, les tests unitaires et l'activation des serveurs MCP.

Ces fiches sont des guides opérationnels prêts à l'emploi : chaque fiche décrit les prérequis, les prompts à utiliser dans Bob, les livrables attendus, les pièges à éviter et une check-list de validation. Elles sont conçues pour être utilisées directement en session Bob, mode IBM i Developer.

**Ce que ce dépôt n'est pas :** un tutoriel général sur Bob ou sur IBM i. Les fiches supposent que Bob est installé, que la connexion IBM i est configurée, et que l'équipe a complété l'UC 15 (onboarding Bob).

---

## Structure du POC — 6 phases, 15 use cases

### Phase 0 — Onboarding Bob

| Fiche | Titre | Description courte |
|-------|-------|-------------------|
| [UC15](UC15-maitrise-bob.md) | Maîtrise de Bob | Modes Ask/Agent/Plan, modes personnalisés, gestion des Bob Coins — à faire en premier |
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

1. **Commencer par UC15** — l'équipe doit maîtriser Bob avant tout autre use case.
2. **Activer UC12 dès la Phase 0** — les MCP JIRA/Confluence/IBM i accélèrent tous les UC suivants.
3. **Respecter l'ordre des phases** — chaque phase produit des livrables qui sont les inputs de la phase suivante.
4. **Utiliser le mode IBM i Developer** dans Bob — il est pré-configuré pour le contexte IBM i.
5. **Charger les fichiers sources dans l'éditeur** avant de démarrer un prompt (Open in Editor via Code for IBM i).
6. **Sauvegarder chaque livrable** avec la convention `{appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md`.

### Note sur ARCAD

Ces fiches supposent qu'ARCAD est utilisé pour la gestion du code source IBM i. Si le **MCP ARCAD n'est pas disponible** (incompatibilité de version), les sources modifiés par Bob devront être réintégrés dans ARCAD manuellement après chaque session. UC16 (DevOps/ARCAD) est hors périmètre de ce dépôt pour cette raison.

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

This repository contains **practical use-case guides from an IBM Bob POC on IBM i**: 15 use cases covering RPG/COBOL application modernisation, documentation, code generation, unit testing, and MCP server activation.

Each guide is an operational runbook: prerequisites, Bob prompts, expected deliverables, pitfalls to avoid, and a validation checklist. They are designed to be used directly in a Bob session, IBM i Developer mode.

**What this repository is not:** a general tutorial on Bob or IBM i. The guides assume Bob is installed, the IBM i connection is configured, and the team has completed UC15 (Bob onboarding).

---

## POC Structure — 6 Phases, 15 Use Cases

### Phase 0 — Bob Onboarding

| Guide | Title | Summary |
|-------|-------|---------|
| [UC15](UC15-maitrise-bob.md) | Mastering Bob | Ask/Agent/Plan modes, custom modes, Bob Coins management — do this first |
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

1. **Start with UC15** — the team must be comfortable with Bob before any other use case.
2. **Activate UC12 in Phase 0** — JIRA/Confluence/IBM i MCPs accelerate all subsequent use cases.
3. **Follow the phase order** — each phase produces deliverables that are inputs for the next phase.
4. **Use IBM i Developer mode** in Bob — it is pre-configured for the IBM i context.
5. **Load source files in the editor** before starting a prompt (Open in Editor via Code for IBM i).
6. **Save every deliverable** using the naming convention `{appArcad}-{function}-{component}-{type}-{YYYYMMDD-HHmm}.md`.

### Note on ARCAD

These guides assume ARCAD is used for IBM i source management. If the **ARCAD MCP is not available** (version incompatibility), sources modified by Bob must be manually reintegrated into ARCAD after each session. UC16 (DevOps/ARCAD) is out of scope in this repository for this reason.

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
| `UC01` – `UC15` | Operational use-case guides |
| `plan-poc-bob-acme.md` | Full POC roadmap: phases, UC ordering, ARCAD constraint, deliverables map |
| `anonymisation-plan.md` | Anonymisation process documentation |
| `README.md` | This file |
