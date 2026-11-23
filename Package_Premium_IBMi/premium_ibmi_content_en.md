# Bob for IBM i Premium Package

## Why the IBM i Premium Package?

The IBM i Premium Package brings together capabilities for maintaining, modernizing, analyzing and understanding IBM i applications:

- **5 Premium workflows** guided step by step
- **1 Understand dashboard** for application mapping
- **4 Premium Slash Commands** that run from the Bob chat
- **71 IBM i skills listed below**, activated automatically based on context

> **Consistency check (November 2026 update, based on the official documentation):** the detailed lists below contain 71 skills: 22 RPG, 4 CL, 5 DDS, 12 COBOL for i, 19 Db2 for i & SQL, 5 IBM i IWS, 3 Understand and 1 Unit testing. The initial source message announced 57 skills, then 65 before the addition of 4 COBOL skills, 1 Db2 skill and `rpgunit-testing`.

---

## 1. The 5 IBM i Premium Workflows

These step-by-step workflows guide developers through automating complex maintenance and modernization tasks.

### 1.1 RPG Modernization

**Role:** converts and modernizes existing RPG source code, fixed-format OPM RPG III or ILE RPG IV, to free-format RPG (`**FREE`) by applying modern standards: `CTL-OPT`, integrated procedures, removal of `GOTO`, etc.

**Official documentation:** [RPG Modernization](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

### 1.2 SQL Index Strategy Advisor

**Role:** analyzes SQL query performance on Db2 for i, including Index Advisor recommendations, MTI usage and execution statistics. It advises on and applies an indexing strategy aimed at improving database response times.

**Official documentation:** [SQL Index Strategy Advisor](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

### 1.3 RPGUnit Test Plan Creation

**Role:** scans the application's RPG programs to identify entry points and exported procedures. It then generates a structured unit test plan in Markdown format, with the test scenarios and shared utilities required.

**Official documentation:** [RPGUnit Test Plan Creation](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

### 1.4 RPGUnit Test Suite Implementation

**Role:** implements the planned RPGUnit unit test suites by generating the test code structure, assertions and `setup`/`teardown` routines. It then runs the test suite directly on the system to fix the logic errors it detects.

**Official documentation:** [RPGUnit Test Suite Implementation](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

### 1.5 Business Rules Extraction

**Role:** automatically analyzes the execution flow and calculation specifications of **RPG** sources to isolate and document, in plain language, the application's business rules, validations and decision-making calculation rules. Official scope: RPG only (Local, IFS and Library List workspaces).

**Official documentation:** [Business Rules Extraction](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/workflows)

---

## 2. Understand Dashboard

The **Understand** dashboard is the built-in application scanner that analyzes the overall structure of the IBM i application estate.

### Role

It records and maps objects and their interdependencies:

- program calls;
- use of DDS PF/LF data files and SQL tables;
- service program (`SRVPGM`) bindings;
- triggers;
- constraints;
- operational usage statistics.

### Key features

- **Impact analysis:** determine which programs or files will be affected by changing, renaming or deleting an item.
- **Dead objects:** identify obsolete or unused objects in a library in order to clean up production environments.
- **Architecture visualization:** get a complete dependency graph without manually reading each source member.

### Access

The dashboard opens directly from the IDE with the action:

```text
vscode-ibmi-bob.openUnderstandDashboard
```

**Official documentation:** [Understand Dashboard](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/understand)

---

## 3. The 4 IBM i Premium Slash Commands

These commands, prefixed with `/`, run directly in the Bob chat input area and trigger a predefined action on the connected IBM i.

### 3.1 `/init`

**Role:** analyzes the active IBM i workspace, the current library in QSYS or the working directory in IFS, then automatically generates an `AGENTS.md` file tailored to the application. This file documents the ILE architecture, naming conventions, DDS/SQL objects and project standards. It is then injected into the context of Bob conversations without reconnecting.

### 3.2 `/erd`

**Role:** generates an entity-relationship diagram in Mermaid format from the Db2 for i tables of the connected system. The command queries `QSYS2.SYSTABLES`, `QSYS2.SYSCOLUMNS`, `QSYS2.SYSCST` and `QSYS2.SYSREFCST` to produce a diagram that includes native Db2 data types, PK, FK and UQ column markers, and cardinalities.

### 3.3 `/review_SQL`

**Role:** performs a complete review of an SQL query or of a source member containing SQL. The analysis covers syntax, readability, security, notably SQL injection and least privilege, SQE performance and refactoring opportunities.

**Output structure:**

1. Summary
2. Confirmed findings
3. Conditional recommendations
4. Revised SQL
5. Explanation of changes

### 3.4 `/review_rpg`

**Role:** performs a code review of an RPG source member in fixed, free or mixed format. The analysis covers syntax, coding standards, error handling, security, performance, notably RLA versus embedded SQL, and modernization opportunities.

**Output structure:**

1. Detected format
2. Summary
3. Critical issues
4. Major issues
5. Minor issues
6. Modernization roadmap

**Shared official documentation:** [Bob for IBM i Slash Commands](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/slash-commands)

---

## 4. IBM i Skills

Skills are activated automatically by Bob based on context. They are grouped into seven categories.

### 4.1 RPG

| Skill | Description |
|---|---|
| `rpg-primer-basics` | RPG fundamentals across all variants: RPG II, RPG III, RPG/400, RPGLE and ILE. |
| `rpg-free-format-fundamentals` | Syntax and best practices of modern free-format RPGLE. |
| `rpg-opm-understanding` | OPM architecture, program model differences and migration. |
| `rpg-ile-understanding` | ILE concepts: binding, activation groups, service programs and procedure calls. |
| `rpgle-specs` | Column-by-column reference of fixed-format RPGLE specifications: H, F, D, I, C and O. |
| `rpg-legacy-specs` | Legacy RPG specifications: RPG II, RPG III, RPG36 and RPG38, with column positioning. |
| `rpg-data-structures` | DS patterns, externally described DS, program status DS and file information DS. |
| `rpg-procedures-functions` | Subprocedures, prototypes, parameter passing, exports and service programs. |
| `rpg-file-operations` | RLA access, operation codes, file declarations and error handling. |
| `rpg-indicators` | Indicator types and patterns for replacing them with Boolean variables. |
| `rpg-cycle-explained` | The RPG program cycle, control breaks and situations in which to avoid it in modern code. |
| `rpg-embedded-sql` | Embedded SQL: cursor operations, host variables and compilation differences. |
| `rpg-sql-cursor` | Declaring SQL cursors, scrollable cursors and multi-row processing in SQLRPGLE. |
| `rpg-fixed-to-free` | Conversion of fixed-format RPGLE to free format, including I-specs and O-specs. |
| `rpg-pgmdb-conversion` | Modernization of I-specs and O-specs to free-format RPGLE for program-described files. |
| `rpg-ospec-conversion` | Conversion of print O-specs to externally described printer files. |
| `rpg-opm-to-ile` | Conversion from OPM RPG to ILE RPGLE: `CVTRPGSRC`, activation groups and binding. |
| `rpg-refactor-legacy` | Refactoring: removing `GOTO` and converting routines into subprocedures. |
| `rpg-rla-to-sql` | Conversion from RLA access to embedded SQL to improve performance. |
| `rpg-code-review` | Quality review: technical debt, performance and modernization opportunities. |
| `rpg-debugging-techniques` | RPG debugging: `STRDBG`, breakpoints, watch expressions and service entry points. |
| `rpg-api` | Calling IBM i system APIs from RPG: prototypes, type mapping and error handling. |

**Official documentation:** [RPG Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#rpg)

### 4.2 CL

| Skill | Description |
|---|---|
| `cl-primer-basics` | CL fundamentals: naming conventions, parameter syntax and common commands. |
| `cl-programming` | CL programming: variables, conditional logic, error handling and program structure. |
| `cl-best-practices` | Best practices: `MONMSG`, parameter validation and logging strategies. |
| `clle-api` | Calling system APIs from ILE CL: `CALL PGM/PARM`, `DCL VAR` mapping and `QUSEC` handling. |

**Official documentation:** [CL Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#cl)

### 4.3 DDS

| Skill | Description |
|---|---|
| `dds-primer-basics` | DDS fundamentals for explaining and generating DDS definitions. |
| `dds-physical-files` | PF physical files: field specifications, keys and record formats. |
| `dds-logical-files` | LF logical files: joins, `select/omit` and derived fields. |
| `dds-display-files` | DSPF display files: formats, indicators, subfiles and screen design. |
| `dds-printer-files` | PRTF printer files: spacing, positioning, edit codes and reports. |

**Official documentation:** [DDS Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#dds)

### 4.4 COBOL for i

| Skill | Description |
|---|---|
| `cobol-primer-basics` | COBOL for i fundamentals: structure, the four divisions, common verbs and ILE/OPM standards. |
| `cobol-data-division` | Data Division: level numbers, picture clauses, working-storage and linkage section. |
| `cobol-procedure-division` | Procedure Division: verbs, conditional logic, `PERFORM` loops, file I/O and `CALL` interfaces. |
| `cobol-embedded-sql` | Embedded SQL in COBOL: host variables, indicators, cursors and transaction control. |
| `cobol-ile-understanding` | ILE COBOL: activation groups, binding, service program integration and compiler directives. |
| `cobol-ile-intrinsic-functions` | ILE COBOL intrinsic functions: strings, arithmetic, dates/times and type conversions. |
| `cobol-opm-understanding` | OPM COBOL program model, differences from ILE and migration. |
| `cobol-file-io` | COBOL file I/O: sequential, indexed, relative and file control entries. |
| `cobol-source-debugging` | COBOL debugging: ILE source debugger, breakpoints, step-by-step execution, job log and dump analysis, compile-time and run-time errors. |
| `cobol-code-refactoring` | COBOL restructuring: simplifying logic, removing `GO TO` and dead code, reducing duplication, while preserving business logic. |
| `cobol-code-review` | Structured COBOL code review: defects, logic errors, technical debt and modernization opportunities. |
| `cobol-module-service-program` | ILE COBOL modules and service programs: imports/exports, binder language, binding directories, activation groups, multi-module applications. |

**Official documentation:** [COBOL for i Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#cobol-for-i)

### 4.5 Db2 for i & SQL

| Skill | Description |
|---|---|
| `db2-sql-primer` | Db2 for i SQL fundamentals: CRUD, joins, aggregates, stored procedures and UDFs. |
| `db2-stored-procedures` | SQL PL procedures: `IN`, `OUT`, `INOUT` parameters, control structures and cursors. |
| `db2-functions-udfs` | SQL UDFs, external UDFs, table functions and sourced functions. |
| `db2-sql-debugging` | Interpreting SQL errors, `SQLCODE`/`SQLSTATE`, and diagnostic techniques. |
| `db2-dds-understanding` | Legacy DDS files: PF, LF and keyed access paths. |
| `db2-dds-to-ddl` | Migration from DDS to SQL DDL: tables, views and conversion strategies. |
| `db2-ddl-generation` | DDL syntax: columns, PK, FK, CHECK and UNIQUE constraints, indexes and views. |
| `db2-ccsid-encoding` | Resolving CCSID and character encoding issues. |
| `db2-sql-performance-analysis` | Plan Cache analysis, Visual Explain interpretation and Index Advisor recommendations. |
| `db2-sqlpl-performance` | SQL PL optimization: `SET` optimization and inlining techniques. |
| `db2-sql-stored-procedure-performance` | High-performance SQL PL best practices in stored procedures, functions and triggers. |
| `db2-sql-find-performance-data` | Locating SQL performance data: Plan Cache snapshots and database monitors. |
| `db2-sql-optimization` | Query rewriting, join order, predicate pushdown and index usage. |
| `db2-index-strategy` | Index types, notably Binary Radix and EVI, syntax, selection and maintenance. |
| `db2-security-best-practices` | Authority models, deny-by-default, adopted authority, SQL privileges and audit journal. |
| `db2-journaling-commitment` | Journaling, commitment control, isolation levels and recovery procedures. |
| `db2-temporal-tables` | Temporal tables: historical queries and automatic versioning. |
| `db2-system-catalog` | Db2 for i system catalogs for metadata discovery. |
| `db2-services-catalog` | Exploring the available IBM i SQL Services and their parameters. |

**Official documentation:** [Db2 for i & SQL Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#db2-for-i--sql)

### 4.6 IBM i IWS (Integrated Web Services)

| Skill | Description |
|---|---|
| `iws-admin` | IWS administration: server/service operations, configuration, REST deployment and backup. |
| `iws-server-ops` | List, create, delete, start and stop IWS servers. |
| `iws-server-config` | Read and modify IWS server properties. |
| `iws-service-ops` | List, install, uninstall, start, stop and configure IWS web services. |
| `iws-backup` | Back up and restore IWS servers and services to or from IFS save files. |

**Official documentation:** [IBM i IWS Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#ibm-i-iws)

### 4.7 Understand

> These skills are active only if Understand is installed.

| Skill | Description |
|---|---|
| `understand-app-overview` | Overall application architecture, hotspots, OPM/ILE modernization maturity, dynamic SQL, decommissioning and conversion order. |
| `understand-impact-analysis` | Impact analysis: direct and indirect dependents, risk level and recommended rebuild order. |
| `understand-program-explainer` | Explanation of a program or service program: OPM/ILE composition, bound modules, exports, files used and callers. |

**Official documentation:** [Understand Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills#understand)

### 4.8 Unit testing

| Skill | Description |
|---|---|
| `rpgunit-testing` | RPGUnit suite structure, test case naming, `setup`/`teardown`, assertion API and test configuration (IBM i Developer mode). |

**Official documentation:** [Unit Testing Skills](https://bob.ibm.com/docs/ide/premium-packages/bob-for-i/skills)
