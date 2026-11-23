# UC 00 — Bob for IBM i Premium Package: Overview and Implementation

> **This guide is the entry point for all other UC in the POC.**
> The IBM i Premium Package takes precedence over all historical prompts. Before starting any UC,
> verify that the Premium Package is installed and its capabilities are understood.
>
> **Reference source:** [IBM Bob Premium Package for i — Overview](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/bob-for-i-index). This guide is aligned with the online documentation as read in early November 2026 (no dated changelog is published).

---

## ⭐ IBM i Premium Package Capabilities

The Premium Package extends Bob with IBM i-specific capabilities, all accessible directly from the Bob IDE:

| Capability | Type | How to trigger |
|---|---|---|
| 2 modes: IBM i Developer, IBM i Database | Modes | Bob mode selector |
| RPG Modernization | Guided workflow | *Start Workflow* button at the top of the chat |
| SQL Index Strategy Advisor | Guided workflow | *Start Workflow* button at the top of the chat |
| RPGUnit Test Plan Creation | Guided workflow | *Start Workflow* button at the top of the chat |
| RPGUnit Test Suite Implementation | Guided workflow | *Start Workflow* button at the top of the chat |
| Business Rules Extraction | Guided workflow | *Start Workflow* button at the top of the chat |
| Understand (Dashboard + 6 tools + 3 skills) | Application mapping | *Open Understand Dashboard* command or the Dashboard icon in the Bob chat title bar |
| `/init` | Slash command | Bob chat input area |
| `/erd` | Slash command | Bob chat input area |
| `/review_SQL` | Slash command | Bob chat input area |
| `/review_rpg` | Slash command | Bob chat input area |
| 71 IBM i skills | Contextual skills | Automatically activated by Bob based on context |
| IBM i tools (read/write QSYS-IFS, CL, SQL, PASE, build, test, docs) | Native tools (native Premium Package connection) | Called by Bob in both IBM i modes |
| Documentation search (RAG) | Tool | Called by Bob (or requested explicitly) |

---

## Prerequisites

Source: [Prerequisites](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/prerequisites).

- **Subscriptions:** active IBM Bob **and** IBM Bob Premium Package for i subscriptions
- **Workstation:** Windows x64 or macOS ARM64 (Apple Silicon). Windows ARM64, macOS Intel and Linux may work but are not supported
- **Versions:** IBM Bob IDE **2.0.0 or higher**; Premium Package for i extension **1.0.0 or higher**
- **Installation:** Bob Marketplace → install the **IBM i Premium** package (see *installing premium packages*)
- **Open source extensions installed automatically** (minimum versions): Code for IBM i 3.0.10, Db2 for IBM i 2.0.1, RPGLE 0.33.7, CLLE 1.2.6, IBM i Testing 1.3.4, IBMi Languages 0.6.25
- **IBM i:** version **7.4 or higher**; up-to-date Database PTF Group (for 7.4: SF99704 level 13 minimum); SSH daemon started (`STRTCPSVR *SSHD`, product 5733-SC1, `QSSHD` profile enabled). Code for IBM i connects **over SSH only**
- **Understand:** to be installed on the IBM i from the Dashboard (see §2); the `vscode-ibmi-bob.understand.installLibrary` setting must first point to a dedicated library
- **RPGUnit:** installed if using the RPGUnit workflows (`RPGUNIT` and `QDEVTOOLS` libraries in the library list; installation offered by the IBM i Testing extension)
- **Bob Coins:** plan for ● ● ○ for workflows, ● ○ ○ for slash commands

---

## 0. Modes, Workspaces and AGENTS.md

### The two IBM i modes

Sources: [Modes](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/modes), [Workspaces](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workspaces).

| Mode | Usage | Note |
|---|---|---|
| **IBM i Developer** | Explain, generate, compile, test and document code (RPG OPM/ILE, CL, DDS, SQL, COBOL), locally or on IBM i (QSYS/IFS) | Replaces Agent mode for any IBM i use case |
| **IBM i Database** | Write, modernize, tune and review Db2 for i SQL; analyze indexes | Same for SQL questions |

Both modes automatically inject the Code for IBM i connection context (host, user profile, OS version, CCSID, current library, library list, `AGENTS.md`). Database mode adds the active SQL job. They can be customized like any Bob mode. You can switch to Plan or Agent when a task goes beyond IBM i.

### The three workspaces

| Workspace | Content | Prerequisite |
|---|---|---|
| **Local** | Folder on the workstation (default) | — |
| **Library List** ⭐ | Libraries, objects and members of the Code for IBM i User Library List | IBM i connection |
| **IFS Working Directory** | Files and directories of the configured IFS directory (default `/home/<user>`) | IBM i connection |

For any IBM i exercise, the **Library List** workspace is the default space. Tasks are attached to a workspace and can be filtered by workspace in the history. `@` mentions target a library, an open member, or an IFS file.

### AGENTS.md

Bob automatically loads `AGENTS.md` in both IBM i modes:

| Workspace | Location |
|---|---|
| QSYS | `{current library}/QBOBSRC/AGENTS.MD` |
| IFS | `{IFS working directory}/.bob/AGENTS.md` |

`/init` generates or updates it (see §3.1).

---

## 1. The 5 Guided Workflows

Source: [Workflows](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows).

Each workflow chains **agentic** steps (subagents or subtasks where Bob reasons and uses its tools) and **non-agentic** steps (deterministic operations that reduce token consumption and make results more predictable).

**How to start them:**
- **Start Workflow** button at the top of the chat → choose the workspace → **Start** on the desired workflow (the FLGHT400 lab refers to a *Workflows* icon); or
- Bob can **suggest** a workflow in the chat when the request fits → **Start workflow**.

> Most workflows are only visible when connected to an IBM i.

| Workflow | Supported workspaces |
|---|---|
| RPG Modernization | Local, Library List |
| Business Rules Extraction | Local, IFS Working Directory, Library List |
| RPGUnit Test Plan Creation | Local, Library List |
| RPGUnit Test Suite Implementation | Local, Library List |
| SQL Index Strategy Advisor | Library List only |

### 1.1 RPG Modernization

**What it does:** converts an RPG source program (OPM RPG II/III/400 or fixed-format ILE RPG IV) to free-format following best practices. It automatically detects the RPG variant and picks the conversion path. It also detects cycle-driven programs (Primary/Secondary files) and then loads the `rpg-cycle-explained` skill. Conversion runs in **two phases**: (1) externalize I-specs and O-specs, (2) convert the remaining H, F, D and C specifications. An optional pre-conversion compilation validates the source, and a summary report closes the workflow.

**How to run:**
1. Open the source to modernize in the editor
2. **Start Workflow** → **RPG Modernization**
3. Enter: source library, source file, member, output library, output file, output member
4. Let Bob execute the workflow step by step

**Official documentation:** [Workflows — RPG Modernization](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

**Related UCs:** UC13 (RPG Columnar→FREE), UC11 (Restructuring). UC12 (COBOL→RPG) does not use this workflow, only as a reference for the target standards.

**FLGHT400 example:** Modernization of `FRS409` (OPM RPG → FREE ILE) — result in [`labs_FLGHT400_results/FRS409-Modernisation-report.md`](../labs_FLGHT400_results/FRS409-Modernisation-report.md). The workflow analyzed the fixed-format source, produced the matching FREE code with `CTL-OPT DFTACTGRP(*NO) ACTGRP(*NEW)`, and documented the transformations applied.

---

### 1.2 SQL Index Strategy Advisor

**What it does:** analyzes Db2 for i SQL performance data — Index Advisor recommendations, MTI (Maintained Temporary Indexes) usage, execution statistics — and establishes an indexing strategy. It can **capture new data** or use existing **Plan Cache snapshots** or a **database monitor**. It proposes indexes on physical tables **and** logical files, presents them for **selective approval**, creates them on the IBM i after approval, then produces a final report (data source, findings, indexes created, next steps).

**How to run:**
1. **IBM i Database** mode, **Library List** workspace (the only supported workspace)
2. **Start Workflow** → **SQL Index Strategy Advisor**
3. Enter: Data Source (`Capture New Performance Data`), Capture Method, Output Library, Output Object Name, Top N Queries
4. Bob analyzes the Plan Cache, identifies costly queries and proposes priority indexes
5. Validate: *"Apply the highest-priority index only"*

**Official documentation:** [Workflows — SQL Index Strategy Advisor](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

**Related UCs:** UC09 (Embedded SQL), UC10 (Optimization), UC16 (SQL Objects)

**FLGHT400 example:** Lab exercise 4 — analysis of the `ORDERS × FLIGHTS × AGENTS` query. Bob identified `ORDERS` as a large table and the `DEPARTURE_DATE` filter as highly selective, and proposed:
```sql
CREATE INDEX FLGHT400.ORDERS_IDX_DEPDT_FLT
    ON FLGHT400.ORDERS (DEPARTURE_DATE, FLIGHT_NUMBER);
```
Full result: [`labs_FLGHT400_results/FLGHT403_sql-review-bottleneck-analysis.md`](../labs_FLGHT400_results/FLGHT403_sql-review-bottleneck-analysis.md)

---

### 1.3 RPGUnit Test Plan Creation

**What it does:** analyzes how the code is used, runs existing test suites to learn the current state, then generates **four types of Markdown documents**: reusable *templates*, one *module* document per module (each exported API: purpose, parameters, return values, side effects, flow chart of branches), a *test suites* document per module (cases, setup, inputs, assertions, implementation status) and a shared *test utilities* document (e.g. mock table setup).

**How to run:**
1. **IBM i Developer** mode
2. **Start Workflow** → **RPGUnit Test Plan Creation**
3. Select the target library
4. Enter the output IFS folder
5. Choose **create a new test plan** → Bob finds the source members automatically
6. Select the exported procedures to test → **validate the environment**

**Official documentation:** [Workflows — RPGUnit Test Plan Creation](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

**Related UCs:** UC14 (RPGUnit Unit Tests)

**FLGHT400 example:** Lab exercise 6 — test plan creation for `CUSTCHK.SQLRPGLE` (`checkCustomerExists` procedure). Bob scanned `FLGHT400/QRPGLESRC`, found the member, identified the exported procedure and generated the structured test plan. Result: [`labs_FLGHT400_results/FLGHT403_tests_RPGUnit.md`](../labs_FLGHT400_results/FLGHT403_tests_RPGUnit.md)

---

### 1.4 RPGUnit Test Suite Implementation

**What it does:** reads the test plan documents (produced by workflow 1.3, a **prerequisite**), writes or updates the test suites, creates or updates the RPGUnit configuration file `testing.json`, runs the suites and **iterates until the newly generated tests pass**. Pre-existing failures are flagged and Bob asks whether to fix them. It ends with a summary (suite files created, number of test cases implemented).

**How to run:**
1. **IBM i Developer** mode (after completing the Test Plan Creation workflow)
2. **Start Workflow** → **RPGUnit Test Suite Implementation**
3. **Proceed** (the previous plan is ready)
4. Confirm the library and IFS folder
5. **validate the environment** → Bob generates the test source members
6. On failure: *"Explain the failure and help fix it."*

**Official documentation:** [Workflows — RPGUnit Test Suite Implementation](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

**Related UCs:** UC14 (RPGUnit Unit Tests)

**FLGHT400 example:** Continuation of exercise 6 — test suite implementation for `CUSTCHK`. Result: [`labs_FLGHT400_results/FLGHT403_tests_RPGUnit.md`](../labs_FLGHT400_results/FLGHT403_tests_RPGUnit.md)

---

### 1.5 Business Rules Extraction

**What it does:** analyzes **RPG** sources (OPM RPG II, RPG III, RPG/400, RPGLE, SQLRPGLE) and extracts business rules, decision logic, validations and process knowledge into a business-friendly Markdown report. The workflow identifies external dependencies (files, programs, service programs, copy members, SQL, system APIs), skips sources with no meaningful RPG constructs (with a warning), splits the source into sections (procedures, subroutines, mainline) processed by **parallel subagents**, enriches with the fields of the files read, then synthesizes an executive report in **8 sections** with up to **3 Mermaid diagrams** (process flow, decision logic, state transitions). The report is saved locally or in the IFS.

> ⚠️ **Scope per the official documentation:** RPG only. For COBOL and CL, use the UC06 prompts (business logic) and the `cobol-*` / `cl-*` skills.

**How to run:**
1. **IBM i Developer** mode
2. **Start Workflow** → **Business Rules Extraction**
3. Enter: library, source file, member, IFS output path
4. Bob analyzes the program and generates a structured report: executive summary, detailed rules, validations, exceptions, decision logic, process flow

**Official documentation:** [Workflows — Business Rules Extraction](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

**Related UCs:** UC06 (Business Logic), UC05 (Code Understanding), UC07 (Documentation)

**FLGHT400 example:** Lab exercise 1g — extraction of business rules from `FRS401.RPGLE` (agent selection window). The workflow produced a complete report in English ([`labs_FLGHT400_results/FRS401-business-rules.md`](../labs_FLGHT400_results/FRS401-business-rules.md)) and French ([`labs_FLGHT400_results/FRS401-business-rules-FR.md`](../labs_FLGHT400_results/FRS401-business-rules-FR.md)), covering 10 detailed business rules, 5 exceptions, 5 calculation rules, 6 main decisions, 6 data integrity rules, and a Mermaid process flow diagram.

---

## 2. Understand: Dashboard, Tools and Skills

Source: [Understand](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/understand).

**What it does:** a code intelligence component included in the Premium Package. It **scans the application libraries** you select and builds a model of the application: programs, service programs, modules, files, commands, data areas, data queues, and their relationships (calls, file access, SRVPGM bindings, SQL references, triggers, foreign keys, journaling, constraints). Once scanned, Bob answers structural questions **directly from the scan data**, without reading every source member.

> ⚠️ **Important limitation:** the scan reads the **compiled objects**, not the source. If source members are out of sync with the objects, the scan reflects the objects. **Dynamic calls** and **dynamic SQL** are not captured (affected objects are flagged for manual review). Source physical files are not listed as objects.

### Setup

1. Set `vscode-ibmi-bob.understand.installLibrary`: a **dedicated**, non-existing library with restricted access (10 characters max, no period, not a system library)
2. Open the Dashboard (*Open Understand Dashboard* command, or the Dashboard icon in the Bob chat title bar). To discover the tool: *Get Started with Understand* command (built-in walkthrough)
3. **Overview** tab → **Install Understand** (an **Update Understand** action appears when an extension update ships a newer version)
4. **Applications** tab → **New application**: name, description, ordered list of libraries (system libraries QSYS, QSYS2, SYSIBM, SYSIBMADM, SYSPROC and SYSTOOLS are excluded) → **Scan**

Scans run in their **own SQL jobs** with fixed settings (your naming, commit and isolation settings have no effect) and the Dashboard stays usable during a scan. After changing a library, an **incremental per-library scan** is enough (Libraries tab). A scan may recommend libraries that are referenced but missing from the application.

### Dashboard tabs

| Tab | Purpose |
|---|---|
| Overview | Install, update and monitor the component |
| Applications | Create, scan and manage applications |
| Libraries | Add/remove libraries; incremental scan |
| Objects | Search/filter scanned objects; see inbound and outbound dependencies, last used date |
| Insights | Object and dependency counts, most referenced objects, dynamic SQL risks, triggers, foreign keys, journaled objects; **Explore the Application with Bob** button |
| Messages | Diagnostics for each scan step |
| Settings | Access to the install library setting and list of accessible Understand libraries |
| Access Control | Audit and manage the profiles that can access the Understand library (see below) |

**Contextual entry points:** right-click a Code for IBM i connection profile → *Open Understand Dashboard* (the application whose library list matches the active profile is tagged *Active Profile*); right-click a member or object in the Object Browser → *Open in Understand*.

### Using Bob with Understand

When Understand is installed and an application is scanned, Bob has **6 Understand tools** and **3 Understand skills** in **both IBM i modes**. It uses them first for any question about structure, relationships or dependencies.

| Tool | Role |
|---|---|
| `understand_list_apps` | Lists registered applications |
| `understand_app_summary` | One-call architecture overview: inventory by type and language, dependencies, most coupled objects, risk and modernization signals (OPM, dynamic SQL, adopted authority, entry points, files without a journal), database rules (foreign keys, cascades, triggers) |
| `understand_find_objects` | Filtered search: name, type, library, language, OPM/ILE, dynamic SQL, no inbound dependencies, not used for N days |
| `understand_describe_object` | Complete profile of an object (metadata, dependencies, bindings, exports/imports, columns, triggers, foreign keys, journaling) |
| `understand_trace_dependencies` | Inbound or outbound dependencies up to a given depth, filterable by kind (`FILE_OPEN`, `PGM_CALL`, `BOUND_SRVPGM`…) |
| `understand_find_symbol` | Where a procedure or exported symbol is defined and who imports it (IBM i 7.6+ scans) |

**Typical questions:**
- *Impact analysis:* "Which programs will be affected if I modify FLIGHTS?" — "Is it safe to delete file CUSTMST? What would break?"
- *Dead objects:* "Which objects in FLGHT400 haven't been used in the last 5 years?"
- *Architecture view:* "What are the high-level functional areas of FLGHT400?" — "Which programs are OPM? Which use dynamic SQL?"
- *Service program:* "What does service program DBSRV export, and who uses those procedures?"

The task's **Understand** (auto-approve) permission controls whether Bob can use these tools without asking.

### Understand security

Scan data is stored in the install library: **a user with access to it may see data from objects they have no direct authority to**. Restrict access to authorized users only. The **Access Control** tab (**Audit Access**) lists profiles, flags authority anomalies and lets you grant, change (`*USE`, `*CHANGE`, `*ALL`) or revoke access. These actions **modify authorities on the IBM i**: review before confirming.

**Official documentation:** [Understand](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/understand)

**Scope:** Understand is part of UC00 and serves the existing UCs (no dedicated UC): UC05 (Code Understanding), UC07 (Documentation), UC11 (Restructuring).

**FLGHT400 example:** Lab exercise 1c — scan of `FLGHT400` (~46 programs, 0 dynamic SQL). Architectural result: [`labs_FLGHT400_results/FLGHT403_functional_areas.md`](../labs_FLGHT400_results/FLGHT403_functional_areas.md), [`labs_FLGHT400_results/FLGHT403_architecture.md`](../labs_FLGHT400_results/FLGHT403_architecture.md).

---

## 3. The 4 Premium Slash Commands

Source: [Slash commands](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/slash-commands).

Slash commands run directly in the Bob chat input area (type `/` to list them). They require an active IBM i connection.

| Command | Argument | Modes |
|---|---|---|
| `/init` | None (free-form instructions allowed) | IBM i Developer, IBM i Database |
| `/erd` | `[schema.table or library/file]` | IBM i Developer, IBM i Database |
| `/review_SQL` | `[SQL statement or source path]` | IBM i Database |
| `/review_rpg` | `[source member or stream file path]` | IBM i Developer |

### 3.1 `/init`

**What it does:** analyzes the active IBM i workspace (current library in QSYS or working directory in IFS) and generates an `AGENTS.md` documenting the non-obvious architecture, source structures, database conventions and project standards: ILE patterns (service programs, binding directories, activation groups), IWS definitions, database objects, naming conventions. For large libraries, Bob summarizes with counts and patterns. If `AGENTS.md` already exists, it is updated while **preserving your manual additions**. The rules are loaded immediately and injected into every conversation of both IBM i modes, with no reconnect.

**How to use:**
1. In the **User Library List** view, right-click the library → *Set as Current Library*
2. In the Bob chat (IBM i Developer mode, *Library List* scope): `/init`
3. Bob analyzes the library and writes `{LIBRARY}/QBOBSRC/AGENTS.MD` (QSYS workspace) or `{IFS directory}/.bob/AGENTS.md` (IFS workspace)

> ⚠️ Without a current library set, `/init` stops without creating anything (QGPL and QTEMP are refused: Bob suggests switching to an application library). In an IFS workspace, a working directory must be configured (not the root `/`).
>
> 💡 **Customize:** `/init and make sure to enforce free-format RPG and our standard error-handling subprocedures`, or edit `AGENTS.md` by hand.
>
> 📝 **To verify:** the FLGHT400 lab also produced a copy `/home/ITZUSER/AGENTS.md`; the current documentation only mentions the two locations above. UC02, UC03 and UC05 still cite this IFS copy (`/home/{USER}/AGENTS.md`).

**Official documentation:** [Slash commands — /init](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/slash-commands)

**Related UCs:** UC02 (Mastering IBM Bob), UC03 (Using Bob Across a Development Team)

**FLGHT400 example:** Lab exercise 1b — `/init` on `FLGHT403` produced an `AGENTS.md` documenting 122 source members, the ILE architecture, cross-library dependencies on `FLGHT400`, logical file naming patterns and modernized code styles. Result: [`labs_FLGHT400_results/FLGHT403_init_report.md`](../labs_FLGHT400_results/FLGHT403_init_report.md), [`labs_FLGHT400_results/FLGHT403_AGENTS.md`](../labs_FLGHT400_results/FLGHT403_AGENTS.md).

---

### 3.2 `/erd`

**What it does:** generates a Mermaid Entity-Relationship diagram from the connected system's Db2 for i tables. Queries `QSYS2.SYSTABLES`, `QSYS2.SYSCOLUMNS`, `QSYS2.SYSCST` and `QSYS2.SYSREFCST` to produce a diagram with native Db2 data types, PK/FK/UQ markers and cardinalities. Without an argument, Bob asks which schema to analyze.

**How to use:**
1. **IBM i Database** mode (or Developer)
2. In the chat: `/erd {LIBRARY}` (e.g. `/erd FLGHT400`, `/erd MYAPP.ORDERS`, `/erd MYPROJ/CUSTOMER`)
3. Bob generates the Mermaid diagram directly in the conversation

**Related UCs:** UC07 (Documentation), UC16 (SQL Objects), UC08 (DDS→DDL)

**FLGHT400 example:** Lab exercise 1e — `/erd FLGHT400` produced the full ERD of the Flight Reservation System application. Result: [`labs_FLGHT400_results/FLGHT403_erd.md`](../labs_FLGHT400_results/FLGHT403_erd.md), [`labs_FLGHT400_results/FLGHT403_erd_report.md`](../labs_FLGHT400_results/FLGHT403_erd_report.md).

---

### 3.3 `/review_SQL`

**What it does:** performs a complete review of an SQL query or of a source member containing SQL, against a checklist: correctness and simplification, readability, qualification and naming, best practices (built-in functions, CTEs), security (least privilege, SQL injection), SQE performance, statement-type guidance (DDL, DML, embedded and dynamic SQL), diagnostics (Visual Explain, plan cache, index advice), reusable logic (UDFs, procedures) and transaction isolation. Produces: (1) Summary, (2) Confirmed findings, (3) Conditional recommendations, (4) Revised SQL, (5) Explanation of changes.

**How to use:**
1. **IBM i Database** mode
2. In the chat: `/review_SQL` then paste the query, or `/review_SQL mylib/mysrc(mycode)` or `/review_SQL /home/user/src/mypgm.rpglesrc`

**Related UCs:** UC09 (Embedded SQL), UC10 (Optimization), UC16 (SQL Objects)

**FLGHT400 example:** Lab exercise 4b — review of the analytical query on `ORDERS × FLIGHTS × AGENTS`. Bob detected the intentional issue (`ORDER BY o.DEPAR00001` on a column absent from the `GROUP BY`) and proposed `MIN(o.DEPAR00001)`. Result: [`labs_FLGHT400_results/FLGHT403_sql-review-bottleneck-analysis.md`](../labs_FLGHT400_results/FLGHT403_sql-review-bottleneck-analysis.md).

---

### 3.4 `/review_rpg`

**What it does:** performs a code review of an RPG source member or stream file (fixed, free or mixed). Bob **rejects** non-RPG sources (CL, COBOL, DDS, pure SQL) before loading the review rules. It detects the format, loads the appropriate skills, then evaluates syntax, standards, error handling, security (adopted authority, injection), performance (RLA vs embedded SQL, blocking, cursors) and modernization. Produces: (1) Detected format, (2) Summary, (3) Critical issues, (4) Major, (5) Minor, (6) Modernization roadmap.

**How to use:**
1. Open the RPG source member in the editor
2. **IBM i Developer** mode
3. In the chat: `/review_rpg` or `/review_rpg MYLIB/QRPGLESRC/MYPGM.RPGLE`

**Related UCs:** UC12 (COBOL→FREE, on the generated RPG only), UC13 (RPG→FREE), UC10 (Optimization), UC11 (Restructuring)

---

## 4. The 71 IBM i Skills — Quick Reference

Source: [Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills).

Skills are activated automatically by Bob based on context. They do not need to be invoked manually. Each one is available in IBM i Developer mode, IBM i Database mode, or both:

| Domain | Count | Representative Skills |
|---|---|---|
| RPG | 22 | `rpg-fixed-to-free`, `rpg-cycle-explained`, `rpg-rla-to-sql`, `rpg-opm-to-ile`, `rpg-code-review` |
| CL | 4 | `cl-programming`, `cl-best-practices` |
| DDS | 5 | `dds-physical-files`, `dds-logical-files`, `dds-display-files` |
| COBOL for i | 12 | `cobol-ile-understanding`, `cobol-embedded-sql`, `cobol-opm-understanding`, `cobol-source-debugging`, `cobol-code-refactoring`, `cobol-code-review`, `cobol-module-service-program` |
| Db2 for i & SQL | 19 | `db2-dds-to-ddl`, `db2-sql-performance-analysis`, `db2-sql-stored-procedure-performance`, `db2-index-strategy`, `db2-security-best-practices` |
| IBM i IWS | 5 | `iws-service-ops`, `iws-server-config` |
| Understand | 3 | `understand-app-overview`, `understand-impact-analysis`, `understand-program-explainer` |
| Unit testing | 1 | `rpgunit-testing` |

**Points to note:**
- Understand skills are only active when Understand is installed.
- Skills **limited to IBM i Database mode**: `db2-dds-understanding`, `db2-ddl-generation`, `db2-sqlpl-performance`, `db2-sql-find-performance-data`, `db2-index-strategy`, `db2-journaling-commitment`, `db2-temporal-tables`.
- Skills available in **both modes**: `rpg-rla-to-sql`, `cl-*` (except `clle-api`), `dds-primer-basics`, `dds-physical-files`, `dds-logical-files`, several `db2-*` skills and the 3 `understand-*` skills. The other RPG, COBOL and IWS skills, plus `dds-display-files`, `dds-printer-files`, `clle-api` and `rpgunit-testing`, belong to IBM i Developer.

**Full documentation:** [IBM i Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills)

---

## 5. IBM i Tools, Documentation Search (RAG) and Guardrails

### 5.1 IBM i tools

Source: [Tools](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/tools). These tools are called by Bob; they are grouped by auto-approve permission.

| Family | Tools | Worth knowing for the POC |
|---|---|---|
| Read | `read_member`, `search_qsys`, `read_stream_file`, `search_ifs` | Regex search in members and stream files |
| Write | `write_member`, `write_stream_file`, `generate_dds` | `generate_dds`: DDS for PF, LF, DSPF, PRTF with layout validation (UC08, UC17) |
| Execute | `execute_cl_command`, `execute_sql_statement`, `execute_pase_command`, `manage_library_list`, `convert_rpg_source` | `convert_rpg_source` = `CVTRPGSRC` (RPG III → RPG IV); `manage_library_list` reads/updates the library list and current library |
| Build | `get_compile_actions`, `execute_compile_action` | Code for IBM i actions (`.vscode/actions.json` locally) |
| Test | `generate_rpg_unit_test_stub`, `run_rpg_unit_test_suite` | `*LINE` or `*PROC` code coverage (UC14) |
| SQL | `get_schema_info`, `get_file_info`, `get_table_columns`, `get_cl_command_library` | System/SQL name resolution, long names, aliases, delimited identifiers |
| Documentation | `search_ibm_i_docs_with_rag`, `fetch_cl_command_doc`, `search_sql_examples`, `fetch_sql_example` | Official documentation and SQL Services examples |
| Understand | 6 `understand_*` tools | See §2 |

### 5.2 Documentation search (RAG)

Source: [Retrieval-augmented generation](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/retrieval-augmented-generation). Both IBM i modes have `search_ibm_i_docs_with_rag`, which queries an indexed corpus of IBM i documentation, Redbooks and reference manuals (ILE, Db2 for i, IFS/QSYS, security, performance, networking, troubleshooting). Bob uses it for conceptual questions or when it is stuck in a loop (e.g. repeated compilation errors). You can request it explicitly: *"Search the IBM i docs for…"* or *"Check the documentation before answering"*.

### 5.3 Guardrails and settings

Sources: [Settings](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/settings), [Security guidelines](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/security-guidelines).

| Setting | Default | Effect |
|---|---|---|
| `vscode-ibmi-bob.guardrails.enforceQualifiedCLCommands` | `true` | CL commands must be qualified (`QSYS/…`); otherwise an error with qualified alternatives |
| `vscode-ibmi-bob.guardrails.deniedSQLStatementTypes` | 17 types including `UPDATE`, `DROP`, `TRUNCATE`, `GRANT`, `CALL`, `MERGE` | Explicit approval **always** required, even if execute auto-approve is granted |
| `vscode-ibmi-bob.general.maxRowsToFetch` | `100` | Row cap returned by `execute_sql_statement` |
| `vscode-ibmi-bob.understand.installLibrary` | none (required) | Understand install library |

**Security:** Bob acts with the **user profile of the Code for IBM i connection**: anything that profile can do, Bob can do → apply least privilege. Bob's activity is traceable through the `CLIENT_APPLNAME` special register of its SQL jobs. Restrict access to the `RPGUNIT` library (developers read, administrators update) and to Code for IBM i's temporary library and directory.

### 5.4 Troubleshooting

Source: [Troubleshooting](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/troubleshooting). The **IBM Bob Premium Package for i** Output channel logs every tool call (INFO at start, ERROR on failure, WARN after 30 seconds, final duration). To isolate messages in the *IBM Bob* channel, search for `[source:ibmi]`. Other useful channels: *Code for IBM i* (CL, SQL, PASE executed) and *IBM i Testing*.

---

## 6. Usage Map by UC

The following table shows which Premium capabilities are used by each POC UC (current numbering of the `uc-en/` guides):

| UC | Title | Workflow | Slash Cmd | Understand | Key Skills |
|---|---|---|---|---|---|
| UC01 | IBM i Connection and third-party MCPs | — | — | — | — |
| UC02 | Mastering IBM Bob | — | `/init` | — | — |
| UC03 | Using Bob Across a Development Team | — | `/init` → AGENTS.md | — | — |
| UC04 | Skills Strategy | — | — | — | (link to skill `poc-ibmi-premium-uc-writer`) |
| UC05 | Code Understanding | Business Rules Extraction | `/init` | ✅ | `understand-app-overview`, `understand-program-explainer` |
| UC06 | Business Logic | **Business Rules Extraction** (RPG) | — | ✅ | `rpg-code-review`, `cobol-procedure-division` |
| UC07 | Documentation | — | `/erd`, `/init` | ✅ | `understand-impact-analysis`, `db2-system-catalog` |
| UC08 | DDS→DDL | — | `/erd` | — | `db2-dds-to-ddl`, `dds-physical-files` |
| UC09 | Embedded SQL | SQL Index Strategy (partial) | `/review_SQL` | — | `rpg-rla-to-sql`, `rpg-embedded-sql`, `db2-sql-stored-procedure-performance` |
| UC10 | Optimization | SQL Index Strategy (partial) | `/review_rpg`, `/review_SQL` | — | `rpg-code-review`, `rpg-rla-to-sql`, `db2-sql-stored-procedure-performance` |
| UC11 | Restructuring | RPG Modernization (partial) | `/review_rpg` | ✅ | `rpg-fixed-to-free`, `rpg-refactor-legacy`, `cobol-code-refactoring` |
| UC12 | COBOL→FREE | — (guided approach with the COBOL skills; RPG Modernization does not apply to COBOL sources) | `/review_rpg` (generated RPG members only) | — | `rpg-opm-to-ile`, `cobol-*` (incl. `cobol-code-review`, `cobol-code-refactoring`) |
| UC13 | RPG Columnar→FREE | RPG Modernization | `/review_rpg` | — | `rpg-fixed-to-free`, `rpg-cycle-explained` |
| UC14 | RPGUnit Tests | **RPGUnit Test Plan + Impl.** | — | — | `rpgunit-testing`, `rpg-debugging-techniques` |
| UC15 | Code Generation | — | — | — | `rpg-procedures-functions`, `rpg-free-format-fundamentals` |
| UC16 | SQL Objects | SQL Index Strategy (partial) | `/review_SQL` | — | `db2-stored-procedures`, `db2-index-strategy`, `db2-sql-stored-procedure-performance` |
| UC17 | App Generation | — | — | — | `iws-service-ops`, `dds-display-files` |
| UC18 | REST API Exposure (IWS) | — | `/init`, `/review_rpg` | — | `iws-admin`, `iws-server-ops`, `iws-server-config`, `iws-service-ops`, `iws-backup` |
| UC19 | Operations | — | `/init` | — | `db2-security-best-practices`, `db2-system-catalog`, `db2-services-catalog`, `db2-sql-performance-analysis` |

---

## 7. Lab FLGHT400 — Implementation Guide

The Flight400 lab is the official hands-on support for the IBM i Premium Package. It runs on a TechZone instance and covers the exercises below, each illustrating one or more Premium capabilities.

**Deployment guide:** [`labs_FLGHT400/FICHE-REDEPLOIEMENT.en.md`](../labs_FLGHT400/FICHE-REDEPLOIEMENT.en.md)
**Lab GitHub:** https://github.com/bmarolleau/flight400-demo

| Exercise | Duration | Premium Capabilities | Available Results |
|---|---|---|---|
| 1a — Object Browser exploration | 5 min | — (manual exploration) | — |
| 1b — `/init` on the library | 5 min | `/init` slash command | [`FLGHT403_init_report.md`](../labs_FLGHT400_results/FLGHT403_init_report.md) |
| 1c — Understand scan | 10 min | Understand Dashboard | [`FLGHT403_functional_areas.md`](../labs_FLGHT400_results/FLGHT403_functional_areas.md) |
| 1d — Architecture overview | 10 min | Understand + skills | [`FLGHT403_architecture.md`](../labs_FLGHT400_results/FLGHT403_architecture.md) |
| 1e — `/erd` | 5 min | `/erd` slash command | [`FLGHT403_erd.md`](../labs_FLGHT400_results/FLGHT403_erd.md) |
| 1g — Business Rules | 10 min | **Business Rules Extraction** | [`FRS401-business-rules-FR.md`](../labs_FLGHT400_results/FRS401-business-rules-FR.md) |
| 2 — Modernize FRS409 | 15 min | **RPG Modernization** | [`FRS409-Modernisation-report.md`](../labs_FLGHT400_results/FRS409-Modernisation-report.md) |
| 3 — Add Total Flight Hours | 30 min | IBM i Developer (native IBM i tools) | [`FLGHT403_Change_Total_Flight_Hours.md`](../labs_FLGHT400_results/FLGHT403_Change_Total_Flight_Hours.md) |
| 4 — SQL optimization | 15 min | `/review_SQL` + **SQL Index Strategy Advisor** | [`FLGHT403_sql-review-bottleneck-analysis.md`](../labs_FLGHT400_results/FLGHT403_sql-review-bottleneck-analysis.md) |
| 5 — System questions | 10 min | IBM i Database (native IBM i tools) | [`FLGHT403_cpu_jobs_programs_audit.md`](../labs_FLGHT400_results/FLGHT403_cpu_jobs_programs_audit.md) |
| 6 — RPGUnit tests | 20 min | **RPGUnit Test Plan + Implementation** | [`FLGHT403_tests_RPGUnit.md`](../labs_FLGHT400_results/FLGHT403_tests_RPGUnit.md) |
| 7 — React app (optional) | 30 min | App generation | [`FLGHT403_App_React.md`](../labs_FLGHT400_results/FLGHT403_App_React.md) |
| 8 — 5250 to HTML (optional) | 5 min | App generation | [`FLGHT403_App_5250_web.md`](../labs_FLGHT400_results/FLGHT403_App_5250_web.md) |

---

## Deliverables of this UC

| File produced | Description | Consumed by |
|---|---|---|
| `{app}-init-all-{date}.md` | `/init` report — documented AGENTS.md | All following UCs (context) |
| `{app}-erd-all-{date}.md` | Full Mermaid ERD | UC07, UC08, UC16 |
| `{app}-understand-overview-{date}.md` | Understand architecture overview | UC05, UC07, UC11 |

---

## Sources

IBM Bob Premium Package for i documentation: [Overview](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/bob-for-i-index) · [Prerequisites](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/prerequisites) · [Modes](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/modes) · [Workspaces](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workspaces) · [Understand](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/understand) · [Tools](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/tools) · [Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills) · [Slash commands](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/slash-commands) · [Workflows](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows) · [RAG](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/retrieval-augmented-generation) · [Settings](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/settings) · [Security guidelines](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/security-guidelines) · [Troubleshooting](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/troubleshooting)
