# POC Bob × IBM i — Practical Modernisation Guides

> **Tool:** [IBM Bob](https://bob.ibm.com) — IBM i Premium package  
> **Environment:** IBM i — VS Code with IBM i extensions  
> **Language:** English — [Version française](README.md)

---

## Overview

This repository contains **practical use-case guides from an IBM Bob POC on IBM i**: 20 use cases (UC00 to UC19), published in French and English. They cover shared team workspace setup, skills and rules strategy, code understanding and documentation, RPG/COBOL and database-layer modernisation, code generation, unit testing, and IBM i operations.

Each guide is an operational runbook: prerequisites, Bob prompts, expected deliverables, pitfalls to avoid, and a validation checklist. The guides are designed to be played in a Bob session, using the IBM i Developer and IBM i Database modes of the Premium package.

**What this repository is not:** a general tutorial on Bob or IBM i. The guides assume Bob is installed and the IBM i connection is configured (see UC01).

The POC customer is referred to as "ACME"; the content is anonymised.

---

## POC Structure — UC00 to UC19

**Prior reading:** [UC00 — Bob for IBM i Premium Package](uc-en/UC00-premium-package-ibmi-en.md) presents the Premium package and the FLGHT400 lab. The following UCs refer to it.

### Phase 0 — Bob Onboarding

| UC | Title | FR | EN |
|----|-------|----|----|
| UC01 | Connecting Bob to IBM i and Optional Third-Party MCPs | [FR](uc-fr/UC01-connexion-ibmi-fr.md) | [EN](uc-en/UC01-connexion-ibmi-en.md) |
| UC02 | Mastering IBM Bob | [FR](uc-fr/UC02-maitrise-bob-fr.md) | [EN](uc-en/UC02-maitrise-bob-en.md) |
| UC03 | Using Bob Across a Development Team: Shared Development Workspace | [FR](uc-fr/UC03-bob-en-equipe-fr.md) | [EN](uc-en/UC03-bob-en-equipe-en.md) |
| UC04 | Skills and Rules Strategy: industrializing Bob use cases | [FR](uc-fr/UC04-strategie-skills-fr.md) | [EN](uc-en/UC04-strategie-skills-en.md) |

### Phase 1 — Understand Before Modernising

| UC | Title | FR | EN |
|----|-------|----|----|
| UC05 | Code and Application Understanding | [FR](uc-fr/UC05-comprehension-code-fr.md) | [EN](uc-en/UC05-comprehension-code-en.md) |
| UC06 | Code Explanation and Business Logic Extraction | [FR](uc-fr/UC06-logique-metier-fr.md) | [EN](uc-en/UC06-logique-metier-en.md) |
| UC07 | Full Documentation | [FR](uc-fr/UC07-documentation-complete-fr.md) | [EN](uc-en/UC07-documentation-complete-en.md) |

### Phase 2 — Database Layer Modernisation

| UC | Title | FR | EN |
|----|-------|----|----|
| UC08 | DDS → DDL Conversion | [FR](uc-fr/UC08-dds-ddl-fr.md) | [EN](uc-en/UC08-dds-ddl-en.md) |
| UC09 | Native database access → Embedded SQL | [FR](uc-fr/UC09-sql-embarque-fr.md) | [EN](uc-en/UC09-sql-embarque-en.md) |

### Phase 3 — Code Modernisation

| UC | Title | FR | EN |
|----|-------|----|----|
| UC10 | Code Improvement and Optimization | [FR](uc-fr/UC10-optimisation-code-fr.md) | [EN](uc-en/UC10-optimisation-code-en.md) |
| UC11 | Code Restructuring, Transformation, Conversion | [FR](uc-fr/UC11-restructuration-code-fr.md) | [EN](uc-en/UC11-restructuration-code-en.md) |

### Phase 4 — Advanced Modernisation (two tracks)

**Track A — Languages**

| UC | Title | FR | EN |
|----|-------|----|----|
| UC12 | COBOL → FREE RPG ILE conversion | [FR](uc-fr/UC12-cobol-free-rpg-fr.md) | [EN](uc-en/UC12-cobol-free-rpg-en.md) |
| UC13 | Fixed-format RPG / RPG III → FREE RPG ILE conversion | [FR](uc-fr/UC13-rpg-colonne-free-fr.md) | [EN](uc-en/UC13-rpg-colonne-free-en.md) |

**Track B — Generation**

| UC | Title | FR | EN |
|----|-------|----|----|
| UC15 | Code generation | [FR](uc-fr/UC15-generation-code-fr.md) | [EN](uc-en/UC15-generation-code-en.md) |
| UC16 | SQL object generation | [FR](uc-fr/UC16-objets-sql-fr.md) | [EN](uc-en/UC16-objets-sql-en.md) |
| UC17 | Application generation | [FR](uc-fr/UC17-generation-applications-fr.md) | [EN](uc-en/UC17-generation-applications-en.md) |
| UC18 | Exposing RPG Programs and Procedures as REST APIs (IWS) | [FR](uc-fr/UC18-exposition-api-iws-fr.md) | [EN](uc-en/UC18-exposition-api-iws-en.md) |

### Phase 5 — Testing and Operations

| UC | Title | FR | EN |
|----|-------|----|----|
| UC14 | Unit Test Development with RPGUnit | [FR](uc-fr/UC14-tests-unitaires-fr.md) | [EN](uc-en/UC14-tests-unitaires-en.md) |
| UC19 | IBM i operations and administration with Bob (Ops) | [FR](uc-fr/UC19-exploitation-ops-fr.md) | [EN](uc-en/UC19-exploitation-ops-en.md) |

UC14 is a cross-cutting mechanism: it can be replayed after each modernisation workstream.

---

## How to Use These Guides

1. **Read UC00** to understand the Premium package (skills, workflows, commands, modes).
2. **Go through Phase 0 (UC01 to UC04)**: IBM i connection, Bob mastery, team workspace, skills and rules.
3. **Follow the phase order**: each phase produces deliverables that feed the next one.
4. **Use IBM i Developer mode** (or IBM i Database for database UCs) in Bob.
5. **Load source files in the editor** before starting a prompt (Open in Editor via Code for IBM i).
6. **Save every deliverable** using the naming convention below.

### Deliverable naming convention

```
{appId}-{function}-{component}-{type}-{YYYYMMDD-HHmm}.md
```

---

## IBM i Premium Package

The IBM i Premium package provides **71 skills**, **5 guided workflows**, **4 slash commands** (`/init`, `/erd`, `/review_SQL`, `/review_rpg`) and **2 modes** (IBM i Developer, IBM i Database).

- Detailed content: [`Package_Premium_IBMi/premium_ibmi_content_en.md`](Package_Premium_IBMi/premium_ibmi_content_en.md) ([FR](Package_Premium_IBMi/premium_ibmi_content.md))
- Overview and implementation: [UC00](uc-en/UC00-premium-package-ibmi-en.md)

---

## FLGHT400 Labs

The guides rely on the **FLGHT400** lab (IBM i demo application).

- [`labs_FLGHT400/FICHE-REDEPLOIEMENT.en.md`](labs_FLGHT400/FICHE-REDEPLOIEMENT.en.md) ([FR](labs_FLGHT400/FICHE-REDEPLOIEMENT.md)): redeployment sheet for the environment and the lab. The `ibmi-lab/` and `scripts/` folders it mentions are not provided in this repository.
- [`labs_FLGHT400_results/`](labs_FLGHT400_results/): examples of results produced by Bob during the lab (reports, architecture, business rules, tests, conversations).
- **To go further:** Benoît Marolleau's full repository, [IBM-i-Application-Modernization-with-Bob](https://github.com/bmarolleau/IBM-i-Application-Modernization-with-Bob).

---

## Guide-writing skill

The skill [`.bob/skills/poc-ibmi-premium-uc-writer`](.bob/skills/poc-ibmi-premium-uc-writer/SKILL.md) guides the writing, review and enrichment of a UC guide that follows the template and is aligned with the Premium package.

---

## Technical Prerequisites

| # | Prerequisite | Detail |
|---|-------------|--------|
| 1 | **IBM Bob** | Install from [bob.ibm.com](https://bob.ibm.com) + Premium package (IBM i Developer and IBM i Database modes) |
| 2 | **Code for IBM i extension** | SSH/ODBC connection to IBM i |
| 3 | **IBM i Languages extension** | RPG, CL, DDS, COBOL syntax highlighting |
| 4 | **IBM i access** | User profile, source libraries, object permissions |
| 5 | **Third-party MCP servers** (optional) | API tokens (JIRA, Confluence…) — see UC01 |

---

## References & Proof of Functionality

> Provisional section: public sources only (YouTube videos, Redbooks, IBM Docs). It will be completed in a second pass.

### Demo videos by UC

| UC | Videos | Level of evidence |
|----|--------|-------------------|
| **UC05** — Code understanding | [Detailed Design Document from a Decades-Old RPG III Program](https://www.youtube.com/watch?v=FiBbop1IIdE) · [Live Impact Analysis, Architecture Docs and RPG Modernization](https://www.youtube.com/watch?v=Ry-8ODREvJc) · [Onboarding — Explain the Architecture of This Application](https://youtu.be/fO43F8LlOSE) | ✅ Very strong evidence |
| **UC06** — Business logic extraction | [IBM Bob Extracts Business Rules from Legacy RPG](https://www.youtube.com/watch?v=u6ZTuWQZ404) · [Full Modernization Strategy from a 1980s RPG Application](https://www.youtube.com/watch?v=uty0W6WxwgU) | ✅ Exactly aligned |
| **UC07** — Full documentation | [IBM Bob — Generate Documentation for RPG](https://www.youtube.com/watch?v=ULEJQwgzieg) · [IBM Bob Analyzes a 1980s RPG Application and Builds a Full Modernization Strategy](https://www.youtube.com/watch?v=uty0W6WxwgU) · [Generate an ERD from a Live DB2 for i Schema](https://www.youtube.com/watch?v=PJ1PrREsDV0) | ✅ Very strong evidence |
| **UC08** — DDS → DDL | [Full Modernization Strategy from a 1980s RPG Application](https://www.youtube.com/watch?v=uty0W6WxwgU) · [AI-Powered Database Engineering for IBM i](https://www.youtube.com/watch?v=yalsyhGALAE) | ⚠️ Insufficient — full PF→DDL conversion not demonstrated |
| **UC09** — Native access → Embedded SQL | [IBM BOB vs VS Code — Convert Native FILE IO to SQL RPGLE](https://www.youtube.com/watch?v=N_2KHeaFgvQ) · [IBM BOB — The King of IBM i Code Modernization?](https://www.youtube.com/watch?v=b9531omCNKs) | ✅ Direct evidence |
| **UC10** — Code optimisation | [Fixed to Free, Variable Rename and Modular Extraction](https://www.youtube.com/watch?v=d158kV8hSvE) · [IBM Bob — Refactor RPG Variable Names](https://www.youtube.com/watch?v=4wtDCmVpi5k) · [IBM Bob — Extract Procedures from RPG](https://www.youtube.com/watch?v=c2GMCoxtrYQ) · [Shift-Left Code Review](https://youtu.be/xzxnH305SEE) | ✅ Strong evidence |
| **UC11** — Restructuring | [IBM Bob — Extract Procedures from RPG](https://www.youtube.com/watch?v=c2GMCoxtrYQ) · [Fixed to Free, Variable Rename, Modular Extraction, and COBOL to RPG](https://www.youtube.com/watch?v=d158kV8hSvE) · [IBM Bob Analyzes a 1980s RPG Application](https://www.youtube.com/watch?v=uty0W6WxwgU) | ✅ Very strong evidence |
| **UC12** — COBOL → Free RPG | [Fixed to Free, Variable Rename, Modular Extraction, and COBOL to RPG](https://www.youtube.com/watch?v=d158kV8hSvE) | ✅ Direct evidence |
| **UC13** — Fixed-format RPG → Free RPG | [IBM Bob Converts OPM RPG III to Free-Form ILE](https://www.youtube.com/watch?v=FnaLzc6Zmhk) · [IBM Bob Converts Fixed-Form RPG to Free-Form RPG](https://www.youtube.com/watch?v=ojjinfQ2HLc) · [Modernize OPM RPG to Free-Format ILE in One Session](https://www.youtube.com/watch?v=R-BTDZWVUgs) | ✅ Very strong evidence |
| **UC14** — RPGUnit unit tests | [RPGUnit Part 1 — Test Plan Creation](https://www.youtube.com/watch?v=NGddesYKQUc) · [RPGUnit Part 2 — Implement and Execute 24 Tests](https://www.youtube.com/watch?v=AnSQl8R-9ZU) · [Full RPGUnit Lifecycle End to End](https://www.youtube.com/watch?v=cvi3ZT4J0H0) · [Implement a New Feature](https://youtu.be/9KEw_glsLPw) | ✅ Strongest evidence of the corpus |
| **UC15** — RPG code generation | [IBM Bob — Create a New RPG Application](https://www.youtube.com/watch?v=GacBr38qbjU) · [IBM Bob Builds a New RPG Feature from Scratch](https://www.youtube.com/watch?v=2TiPCmA8GOM) · [Architect a New Feature](https://youtu.be/lBa6-rVq6eE) · [Implement a New Feature](https://youtu.be/9KEw_glsLPw) · [World-First: Literate Coding](https://youtu.be/sBWzf1QiXgQ) | ✅ Direct and well complemented evidence |
| **UC16** — SQL object generation | [Create a New RPG Application](https://www.youtube.com/watch?v=GacBr38qbjU) · [AI-Powered Database Engineering for IBM i](https://www.youtube.com/watch?v=yalsyhGALAE) · [Generate an ERD from Live DB2 for i](https://www.youtube.com/watch?v=PJ1PrREsDV0) | ⚠️ Partial — stored procedures/triggers/UDFs not demonstrated |
| **UC17** — Application generation | [New RPG Feature: Display File, Backend and Business Logic](https://www.youtube.com/watch?v=2TiPCmA8GOM) · [Complete RPG/DB2/Flask/Web Application](https://www.youtube.com/watch?v=GacBr38qbjU) · [Java UI Modernization Using Quarkus](https://youtu.be/gqZG_afyQyg) | ⚠️ Strong — SUBFILE not explicitly demonstrated |

UC01 to UC04, UC18 and UC19 have no reference video listed at this stage.

### Reference documents by UC (Redbooks and IBM Docs)

| UC | Document | Source | Relevance |
|----|----------|--------|-----------|
| UC05 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | RPGLEdoc, ILEDocs: tools to extract documentation embedded in RPG code |
| UC07 | [Tools and Solutions for Modernizing IBM i (REDP-5095)](https://www.redbooks.ibm.com/redpapers/pdfs/redp5095.pdf) | Redbooks | Modern IBM i architecture, ISV tools for diagrams and visualisation |
| UC07 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | Coverage of DB2, RPG, Web services, HTTP — reference for specifications and architecture |
| UC08 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks §9 | Main reference: PF → `CREATE TABLE`, `CREATE INDEX`, `CREATE VIEW`/ALIAS |
| UC08 | [Tools and Solutions for Modernizing IBM i (REDP-5095)](https://www.redbooks.ibm.com/redpapers/pdfs/redp5095.pdf) | Redbooks | "Modernize DB" tool: DDS→SQL data validation workflow |
| UC08 | [Data Description Specifications (DDS) — IBM i 7.1](https://www.ibm.com/docs/en/i/7.1.0?topic=processing-data-description-specifications-dds) | IBM Docs | DDS reference: PF, LF examples, field layouts |
| UC08 | [Creating the DDS file — IBM i 7.3](https://www.ibm.com/docs/en/i/7.3.0?topic=dds-creating-file) | IBM Docs | `CRTPF`, `CRTLF` commands, replaced by `CREATE TABLE`/`CREATE INDEX` |
| UC09 | [DB2 for i: Coding SQL in ILE RPG (IBM i 7.6)](https://www.ibm.com/docs/en/i/7.6.0?topic=programming-coding-sql-statements-in-ile-rpg-applications) | IBM Docs | Host variables, SQLCA, EXEC SQL in ILE RPG |
| UC09 | [Example: SQL statements in RPG/400 programs (IBM i 7.1)](https://www.ibm.com/docs/en/i/7.1.0?topic=statements-example-sql-in-rpg400-programs) | IBM Docs | DECLARE CURSOR → OPEN → FETCH example replacing SETLL+READE |
| UC09 | [SQL Procedures, Triggers, Functions on DB2 for i (SG24-8326)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248326.pdf) | Redbooks | DECLARE/OPEN/FETCH/CLOSE cycle, SQL equivalent of the SETLL→READE→EOF pattern |
| UC10 | [Refactor ILE RPG code — RDi 9.8.0 Docs](https://www.ibm.com/docs/en/rdfi/9.8.0?topic=editing-refactor-ile-rpg-code) | IBM Docs | IBM definition of ILE RPG refactoring |
| UC10 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | §7.4.2: replace `INnn` with named variables |
| UC11 | [Who Knew You Could Do That with RPG IV? (SG24-5402)](https://www.redbooks.ibm.com/redbooks/pdfs/sg245402.pdf) | Redbooks | §4.1: modules, service programs, binding directories, `CRTRPGMOD`/`CRTSRVPGM` |
| UC11 | [RPG Programming in ILE (IBM i 7.5)](https://www.ibm.com/docs/en/i/7.5.0?topic=introduction-rpg-programming-in-ile) | IBM Docs | ILE programming model, activation groups, binding |
| UC12 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | Fixed→free conversion "can be substantially automated" |
| UC12 | [Migration Strategy — ILE COBOL (IBM i 7.3)](https://www.ibm.com/docs/en/i/7.3.0?topic=cobol-migration-strategy) | IBM Docs | OPM→ILE COBOL migration: CRTBNDCBL, activation groups |
| UC13 | [Introducing IBM i Modernization Engine — Merlin (SG24-8583)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248583.pdf) | Redbooks | Known limits of fixed→free conversion (I/O specs, P/S/T files) |
| UC13 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | History of `/Free`, syntax evolution |
| UC14 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks §4.3.3 | RPGUnit: assertions, test detection, setup/teardown, CI integration |
| UC14 | [IBM i 7.6 Features (SG24-8588)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248588.pdf) | Redbooks | IBM direction for AI-generated test cases |
| UC15 | [Inserting Specification Templates — IBM i 7.1 Docs](https://www.ibm.com/docs/en/i/7.1.0?topic=source-inserting-specification-templates) | IBM Docs | `INSRTPL(YES)` for scaffolding RPG IV programs |
| UC15 | [Who Knew You Could Do That with RPG IV? (SG24-5402)](https://www.redbooks.ibm.com/redbooks/pdfs/sg245402.pdf) | Redbooks | ILE RPG patterns: `LIKEDS`, main procedures, free-form syntax |
| UC16 | [SQL Procedures, Triggers, Functions on DB2 for i (SG24-8326)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248326.pdf) | Redbooks | `CREATE PROCEDURE`, triggers, UDFs on Db2 for i |
| UC16 | [External Procedures, Triggers, UDFs on DB2 for i (SG24-6503)](https://www.redbooks.ibm.com/redbooks/pdfs/sg246503.pdf) | Redbooks | External procedures (RPG, COBOL, C, Java), external triggers, UDFs |
| UC17 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | UI modernisation strategies: browser-based, Mobile RPG, custom client |
| UC17 | [Tools and Solutions for Modernizing IBM i (REDP-5095)](https://www.redbooks.ibm.com/redpapers/pdfs/redp5095.pdf) | Redbooks | Mobile RPG: DDS-compatible export, subfile loading code generation |

---

## Other languages

- [Version française](README.md)
