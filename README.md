# POC Bob × IBM i — Fiches pratiques de modernisation

> **Outil :** [IBM Bob](https://bob.ibm.com) — package Premium IBM i  
> **Environnement :** IBM i — VS Code avec extensions IBM i  
> **Langue :** Français — [English version](README.en.md)

---

## Présentation

Ce dépôt contient les **fiches pratiques d'un POC IBM Bob sur IBM i** : 20 use cases (UC00 à UC19), publiés en français et en anglais. Ils couvrent la mise en place du workspace d'équipe, la stratégie skills et rules, la compréhension et la documentation de code, la modernisation RPG/COBOL et de la couche base de données, la génération de code, les tests unitaires et l'exploitation IBM i.

Chaque fiche est un guide opérationnel : prérequis, prompts à utiliser dans Bob, livrables attendus, pièges à éviter et check-list de validation. Les fiches sont conçues pour être jouées en session Bob, avec les modes IBM i Developer et IBM i Database du package Premium.

**Ce que ce dépôt n'est pas :** un tutoriel général sur Bob ou sur IBM i. Les fiches supposent que Bob est installé et que la connexion à l'IBM i est configurée (voir UC01).

Le client du POC est désigné par « ACME » ; le contenu est anonymisé.

---

## Structure du POC — UC00 à UC19

**Lecture préalable :** [UC00 — Bob for IBM i Premium Package](uc-fr/UC00-premium-package-ibmi-fr.md) présente le package Premium et le lab FLGHT400. Les UC suivants s'y réfèrent.

### Phase 0 — Onboarding Bob

| UC | Titre | FR | EN |
|----|-------|----|----|
| UC01 | Connecter Bob à l'IBM i et MCP tiers optionnels | [FR](uc-fr/UC01-connexion-ibmi-fr.md) | [EN](uc-en/UC01-connexion-ibmi-en.md) |
| UC02 | Maîtrise de Bob | [FR](uc-fr/UC02-maitrise-bob-fr.md) | [EN](uc-en/UC02-maitrise-bob-en.md) |
| UC03 | Bob en équipe : workspace d'entreprise | [FR](uc-fr/UC03-bob-en-equipe-fr.md) | [EN](uc-en/UC03-bob-en-equipe-en.md) |
| UC04 | Stratégie Skills et Rules : industrialisation des use cases Bob | [FR](uc-fr/UC04-strategie-skills-fr.md) | [EN](uc-en/UC04-strategie-skills-en.md) |

### Phase 1 — Comprendre avant de moderniser

| UC | Titre | FR | EN |
|----|-------|----|----|
| UC05 | Compréhension de code et d'applications | [FR](uc-fr/UC05-comprehension-code-fr.md) | [EN](uc-en/UC05-comprehension-code-en.md) |
| UC06 | Explication de code et extraction de la logique métier | [FR](uc-fr/UC06-logique-metier-fr.md) | [EN](uc-en/UC06-logique-metier-en.md) |
| UC07 | Documentation complète | [FR](uc-fr/UC07-documentation-complete-fr.md) | [EN](uc-en/UC07-documentation-complete-en.md) |

### Phase 2 — Modernisation de la couche base de données

| UC | Titre | FR | EN |
|----|-------|----|----|
| UC08 | Conversion DDS → DDL | [FR](uc-fr/UC08-dds-ddl-fr.md) | [EN](uc-en/UC08-dds-ddl-en.md) |
| UC09 | Accès natif base de données → SQL embarqué | [FR](uc-fr/UC09-sql-embarque-fr.md) | [EN](uc-en/UC09-sql-embarque-en.md) |

### Phase 3 — Modernisation du code

| UC | Titre | FR | EN |
|----|-------|----|----|
| UC10 | Amélioration, optimisation de code | [FR](uc-fr/UC10-optimisation-code-fr.md) | [EN](uc-en/UC10-optimisation-code-en.md) |
| UC11 | Restructuration, transformation, conversion | [FR](uc-fr/UC11-restructuration-code-fr.md) | [EN](uc-en/UC11-restructuration-code-en.md) |

### Phase 4 — Modernisation avancée (deux pistes)

**Piste A — Langages**

| UC | Titre | FR | EN |
|----|-------|----|----|
| UC12 | Conversion COBOL → FREE RPG ILE | [FR](uc-fr/UC12-cobol-free-rpg-fr.md) | [EN](uc-en/UC12-cobol-free-rpg-en.md) |
| UC13 | Conversion RPG Colonné / RPG III → FREE RPG ILE | [FR](uc-fr/UC13-rpg-colonne-free-fr.md) | [EN](uc-en/UC13-rpg-colonne-free-en.md) |

**Piste B — Génération**

| UC | Titre | FR | EN |
|----|-------|----|----|
| UC15 | Génération du code | [FR](uc-fr/UC15-generation-code-fr.md) | [EN](uc-en/UC15-generation-code-en.md) |
| UC16 | Génération d'objets SQL | [FR](uc-fr/UC16-objets-sql-fr.md) | [EN](uc-en/UC16-objets-sql-en.md) |
| UC17 | Génération d'applications | [FR](uc-fr/UC17-generation-applications-fr.md) | [EN](uc-en/UC17-generation-applications-en.md) |
| UC18 | Exposition de services RPG en API REST (IWS) | [FR](uc-fr/UC18-exposition-api-iws-fr.md) | [EN](uc-en/UC18-exposition-api-iws-en.md) |

### Phase 5 — Tests et exploitation

| UC | Titre | FR | EN |
|----|-------|----|----|
| UC14 | Création de procédures de test (RPGUnit) | [FR](uc-fr/UC14-tests-unitaires-fr.md) | [EN](uc-en/UC14-tests-unitaires-en.md) |
| UC19 | Exploitation et administration IBM i avec Bob (Ops) | [FR](uc-fr/UC19-exploitation-ops-fr.md) | [EN](uc-en/UC19-exploitation-ops-en.md) |

UC14 est un mécanisme transverse : il peut être rejoué après chaque chantier de modernisation.

---

## Comment utiliser ces fiches

1. **Lire UC00** pour comprendre le package Premium (skills, workflows, commandes, modes).
2. **Dérouler la Phase 0 (UC01 à UC04)** : connexion IBM i, maîtrise de Bob, workspace d'équipe, skills et rules.
3. **Respecter l'ordre des phases** : chaque phase produit des livrables qui alimentent la suivante.
4. **Utiliser le mode IBM i Developer** (ou IBM i Database pour les UC base de données) dans Bob.
5. **Charger les sources dans l'éditeur** avant de lancer un prompt (Open in Editor via Code for IBM i).
6. **Sauvegarder chaque livrable** avec la convention de nommage ci-dessous.

### Convention de nommage des livrables

```
{appId}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md
```

---

## Package Premium IBM i

Le package Premium IBM i fournit **71 skills**, **5 workflows guidés**, **4 commandes slash** (`/init`, `/erd`, `/review_SQL`, `/review_rpg`) et **2 modes** (IBM i Developer, IBM i Database).

- Contenu détaillé : [`Package_Premium_IBMi/premium_ibmi_content.md`](Package_Premium_IBMi/premium_ibmi_content.md) ([EN](Package_Premium_IBMi/premium_ibmi_content_en.md))
- Présentation et mise en œuvre : [UC00](uc-fr/UC00-premium-package-ibmi-fr.md)

---

## Labs FLGHT400

Les fiches s'appuient sur le lab **FLGHT400** (application de démonstration IBM i).

- [`labs_FLGHT400/FICHE-REDEPLOIEMENT.md`](labs_FLGHT400/FICHE-REDEPLOIEMENT.md) ([EN](labs_FLGHT400/FICHE-REDEPLOIEMENT.en.md)) : fiche de redéploiement de l'environnement et du lab. Les dossiers `ibmi-lab/` et `scripts/` qu'elle cite ne sont pas fournis dans ce dépôt.
- [`labs_FLGHT400_results/`](labs_FLGHT400_results/) : exemples de résultats produits par Bob lors du lab (rapports, architecture, règles métier, tests, conversations).
- **Pour aller plus loin :** le dépôt complet de Benoît Marolleau, [IBM-i-Application-Modernization-with-Bob](https://github.com/bmarolleau/IBM-i-Application-Modernization-with-Bob).

---

## Skill de rédaction des fiches

Le skill [`.bob/skills/poc-ibmi-premium-uc-writer`](.bob/skills/poc-ibmi-premium-uc-writer/SKILL.md) guide la rédaction, la révision et l'enrichissement d'une fiche UC conforme au template, alignée sur le package Premium.

---

## Prérequis techniques

| # | Prérequis | Détail |
|---|-----------|--------|
| 1 | **IBM Bob** | Installer depuis [bob.ibm.com](https://bob.ibm.com) + package Premium (modes IBM i Developer et IBM i Database) |
| 2 | **Extension Code for IBM i** | Connexion SSH/ODBC vers l'IBM i |
| 3 | **Extension IBM i Languages** | Coloration syntaxique RPG, CL, DDS, COBOL |
| 4 | **Accès IBM i** | Profil utilisateur, bibliothèques sources, droits sur les objets |
| 5 | **Serveurs MCP tiers** (optionnel) | Jetons API (JIRA, Confluence…) — voir UC01 |

---

## Références & preuves de fonctionnalité

> Section provisoire : sources publiques uniquement (vidéos YouTube, Redbooks, IBM Docs). Elle sera complétée lors d'une seconde passe.

### Vidéos de démonstration par UC

| UC | Vidéos | Niveau de preuve |
|----|--------|------------------|
| **UC05** — Compréhension de code | [Detailed Design Document from a Decades-Old RPG III Program](https://www.youtube.com/watch?v=FiBbop1IIdE) · [Live Impact Analysis, Architecture Docs and RPG Modernization](https://www.youtube.com/watch?v=Ry-8ODREvJc) · [Onboarding — Explain the Architecture of This Application](https://youtu.be/fO43F8LlOSE) | ✅ Preuve très forte |
| **UC06** — Extraction logique métier | [IBM Bob Extracts Business Rules from Legacy RPG](https://www.youtube.com/watch?v=u6ZTuWQZ404) · [Full Modernization Strategy from a 1980s RPG Application](https://www.youtube.com/watch?v=uty0W6WxwgU) | ✅ Exactement alignée |
| **UC07** — Documentation complète | [IBM Bob — Generate Documentation for RPG](https://www.youtube.com/watch?v=ULEJQwgzieg) · [IBM Bob Analyzes a 1980s RPG Application and Builds a Full Modernization Strategy](https://www.youtube.com/watch?v=uty0W6WxwgU) · [Generate an ERD from a Live DB2 for i Schema](https://www.youtube.com/watch?v=PJ1PrREsDV0) | ✅ Preuve très forte |
| **UC08** — DDS → DDL | [Full Modernization Strategy from a 1980s RPG Application](https://www.youtube.com/watch?v=uty0W6WxwgU) · [AI-Powered Database Engineering for IBM i](https://www.youtube.com/watch?v=yalsyhGALAE) | ⚠️ Insuffisante — conversion complète PF→DDL non démontrée |
| **UC09** — Accès natif → SQL embarqué | [IBM BOB vs VS Code — Convert Native FILE IO to SQL RPGLE](https://www.youtube.com/watch?v=N_2KHeaFgvQ) · [IBM BOB — The King of IBM i Code Modernization?](https://www.youtube.com/watch?v=b9531omCNKs) | ✅ Preuve directe |
| **UC10** — Optimisation de code | [Fixed to Free, Variable Rename and Modular Extraction](https://www.youtube.com/watch?v=d158kV8hSvE) · [IBM Bob — Refactor RPG Variable Names](https://www.youtube.com/watch?v=4wtDCmVpi5k) · [IBM Bob — Extract Procedures from RPG](https://www.youtube.com/watch?v=c2GMCoxtrYQ) · [Shift-Left Code Review](https://youtu.be/xzxnH305SEE) | ✅ Preuve forte |
| **UC11** — Restructuration | [IBM Bob — Extract Procedures from RPG](https://www.youtube.com/watch?v=c2GMCoxtrYQ) · [Fixed to Free, Variable Rename, Modular Extraction, and COBOL to RPG](https://www.youtube.com/watch?v=d158kV8hSvE) · [IBM Bob Analyzes a 1980s RPG Application](https://www.youtube.com/watch?v=uty0W6WxwgU) | ✅ Preuve très forte |
| **UC12** — COBOL → Free RPG | [Fixed to Free, Variable Rename, Modular Extraction, and COBOL to RPG](https://www.youtube.com/watch?v=d158kV8hSvE) | ✅ Preuve directe |
| **UC13** — RPG colonné → Free RPG | [IBM Bob Converts OPM RPG III to Free-Form ILE](https://www.youtube.com/watch?v=FnaLzc6Zmhk) · [IBM Bob Converts Fixed-Form RPG to Free-Form RPG](https://www.youtube.com/watch?v=ojjinfQ2HLc) · [Modernize OPM RPG to Free-Format ILE in One Session](https://www.youtube.com/watch?v=R-BTDZWVUgs) | ✅ Preuve très forte |
| **UC14** — Tests unitaires RPGUnit | [RPGUnit Part 1 — Test Plan Creation](https://www.youtube.com/watch?v=NGddesYKQUc) · [RPGUnit Part 2 — Implement and Execute 24 Tests](https://www.youtube.com/watch?v=AnSQl8R-9ZU) · [Full RPGUnit Lifecycle End to End](https://www.youtube.com/watch?v=cvi3ZT4J0H0) · [Implement a New Feature](https://youtu.be/9KEw_glsLPw) | ✅ Preuve la plus forte du corpus |
| **UC15** — Génération de code RPG | [IBM Bob — Create a New RPG Application](https://www.youtube.com/watch?v=GacBr38qbjU) · [IBM Bob Builds a New RPG Feature from Scratch](https://www.youtube.com/watch?v=2TiPCmA8GOM) · [Architect a New Feature](https://youtu.be/lBa6-rVq6eE) · [Implement a New Feature](https://youtu.be/9KEw_glsLPw) · [World-First: Literate Coding](https://youtu.be/sBWzf1QiXgQ) | ✅ Preuve directe et bien complétée |
| **UC16** — Génération d'objets SQL | [Create a New RPG Application](https://www.youtube.com/watch?v=GacBr38qbjU) · [AI-Powered Database Engineering for IBM i](https://www.youtube.com/watch?v=yalsyhGALAE) · [Generate an ERD from Live DB2 for i](https://www.youtube.com/watch?v=PJ1PrREsDV0) | ⚠️ Partielle — procédures stockées/triggers/UDF non démontrés |
| **UC17** — Génération d'applications | [New RPG Feature: Display File, Backend and Business Logic](https://www.youtube.com/watch?v=2TiPCmA8GOM) · [Complete RPG/DB2/Flask/Web Application](https://www.youtube.com/watch?v=GacBr38qbjU) · [Java UI Modernization Using Quarkus](https://youtu.be/gqZG_afyQyg) | ⚠️ Forte — SUBFILE non démontré explicitement |

UC01 à UC04, UC18 et UC19 n'ont pas de vidéo de référence recensée à ce stade.

### Documents de référence par UC (Redbooks et IBM Docs)

| UC | Document | Source | Pertinence |
|----|----------|--------|------------|
| UC05 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | RPGLEdoc, ILEDocs : outils d'extraction de documentation embarquée dans le code RPG |
| UC07 | [Tools and Solutions for Modernizing IBM i (REDP-5095)](https://www.redbooks.ibm.com/redpapers/pdfs/redp5095.pdf) | Redbooks | Architecture IBM i moderne, outils ISV pour diagrammes et visualisation |
| UC07 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | Couverture DB2, RPG, Web services, HTTP — référence pour spécifications et architecture |
| UC08 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks §9 | Référence principale : PF → `CREATE TABLE`, `CREATE INDEX`, `CREATE VIEW`/ALIAS |
| UC08 | [Tools and Solutions for Modernizing IBM i (REDP-5095)](https://www.redbooks.ibm.com/redpapers/pdfs/redp5095.pdf) | Redbooks | Outil « Modernize DB » : workflow de validation des données DDS→SQL |
| UC08 | [Data Description Specifications (DDS) — IBM i 7.1](https://www.ibm.com/docs/en/i/7.1.0?topic=processing-data-description-specifications-dds) | IBM Docs | Référence DDS : exemples PF, LF, field layouts |
| UC08 | [Creating the DDS file — IBM i 7.3](https://www.ibm.com/docs/en/i/7.3.0?topic=dds-creating-file) | IBM Docs | Commandes `CRTPF`, `CRTLF`, remplacées par `CREATE TABLE`/`CREATE INDEX` |
| UC09 | [DB2 for i: Coding SQL in ILE RPG (IBM i 7.6)](https://www.ibm.com/docs/en/i/7.6.0?topic=programming-coding-sql-statements-in-ile-rpg-applications) | IBM Docs | Host variables, SQLCA, EXEC SQL en RPG ILE |
| UC09 | [Example: SQL statements in RPG/400 programs (IBM i 7.1)](https://www.ibm.com/docs/en/i/7.1.0?topic=statements-example-sql-in-rpg400-programs) | IBM Docs | Exemple DECLARE CURSOR → OPEN → FETCH remplaçant SETLL+READE |
| UC09 | [SQL Procedures, Triggers, Functions on DB2 for i (SG24-8326)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248326.pdf) | Redbooks | Cycle DECLARE/OPEN/FETCH/CLOSE, équivalent SQL du pattern SETLL→READE→EOF |
| UC10 | [Refactor ILE RPG code — RDi 9.8.0 Docs](https://www.ibm.com/docs/en/rdfi/9.8.0?topic=editing-refactor-ile-rpg-code) | IBM Docs | Définition IBM du refactoring RPG ILE |
| UC10 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | §7.4.2 : remplacer `INnn` par des variables nommées |
| UC11 | [Who Knew You Could Do That with RPG IV? (SG24-5402)](https://www.redbooks.ibm.com/redbooks/pdfs/sg245402.pdf) | Redbooks | §4.1 : modules, service programs, binding directories, `CRTRPGMOD`/`CRTSRVPGM` |
| UC11 | [RPG Programming in ILE (IBM i 7.5)](https://www.ibm.com/docs/en/i/7.5.0?topic=introduction-rpg-programming-in-ile) | IBM Docs | Modèle de programmation ILE, activation groups, binding |
| UC12 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | La conversion fixed→free « peut être substantiellement automatisée » |
| UC12 | [Migration Strategy — ILE COBOL (IBM i 7.3)](https://www.ibm.com/docs/en/i/7.3.0?topic=cobol-migration-strategy) | IBM Docs | Migration OPM→ILE COBOL : CRTBNDCBL, activation groups |
| UC13 | [Introducing IBM i Modernization Engine — Merlin (SG24-8583)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248583.pdf) | Redbooks | Limites connues de la conversion fixed→free (I/O specs, fichiers P/S/T) |
| UC13 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | Historique `/Free`, évolution de la syntaxe |
| UC14 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks §4.3.3 | RPGUnit : assertions, détection des tests, setup/teardown, intégration CI |
| UC14 | [IBM i 7.6 Features (SG24-8588)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248588.pdf) | Redbooks | Orientation IBM pour la génération de cas de test par IA |
| UC15 | [Inserting Specification Templates — IBM i 7.1 Docs](https://www.ibm.com/docs/en/i/7.1.0?topic=source-inserting-specification-templates) | IBM Docs | `INSRTPL(YES)` pour le scaffolding de programmes RPG IV |
| UC15 | [Who Knew You Could Do That with RPG IV? (SG24-5402)](https://www.redbooks.ibm.com/redbooks/pdfs/sg245402.pdf) | Redbooks | Patterns ILE RPG : `LIKEDS`, main procedures, syntaxe free-form |
| UC16 | [SQL Procedures, Triggers, Functions on DB2 for i (SG24-8326)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248326.pdf) | Redbooks | `CREATE PROCEDURE`, triggers, UDF sur Db2 for i |
| UC16 | [External Procedures, Triggers, UDFs on DB2 for i (SG24-6503)](https://www.redbooks.ibm.com/redbooks/pdfs/sg246503.pdf) | Redbooks | Procédures externes (RPG, COBOL, C, Java), triggers externes, UDF |
| UC17 | [Modernizing IBM i Applications (SG24-8185)](https://www.redbooks.ibm.com/redbooks/pdfs/sg248185.pdf) | Redbooks | Stratégies de modernisation UI : navigateur, Mobile RPG, client sur mesure |
| UC17 | [Tools and Solutions for Modernizing IBM i (REDP-5095)](https://www.redbooks.ibm.com/redpapers/pdfs/redp5095.pdf) | Redbooks | Mobile RPG : export compatible DDS, génération de code de chargement de subfile |

---

## Autres langues

- [English README](README.en.md)
