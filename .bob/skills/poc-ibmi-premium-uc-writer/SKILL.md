---
name: poc-ibmi-premium-uc-writer
description: >
  Utiliser quand on rédige, révise ou enrichit une fiche Use Case (UC) du POC Bob × IBM i.
  S'active sur les phrases : "rédige UC", "révise UC", "fiche use case IBMi", "premium package IBMi UC",
  "crée la fiche UC", "enrichis UC", "met à jour UC", "nouveau UC", "fiche Use Case".
  Guide la production d'une fiche conforme au template canonique, alignée sur le Premium Package IBM i,
  bilingue FR+EN, avec exemples FLGHT400 et convention de nommage des livrables.
---

# POC IBM i Premium UC Writer

Ce skill guide la rédaction et la révision des fiches Use Cases du POC Bob × IBM i.
Il s'applique à tout UC nouveau, révisé ou enrichi dans ce projet.

## Règle primaire

Le fichier `Package_Premium_IBMi/premium_ibmi_content.md` **prime sur tous les prompts historiques**.
Avant de rédiger, toujours vérifier quel(s) élément(s) Premium couvrent le cas d'usage :

- **Workflow** : RPG Modernization / SQL Index Strategy Advisor / RPGUnit Test Plan Creation / RPGUnit Test Suite Implementation / Business Rules Extraction (RPG uniquement)
- **Slash Command** : `/init` / `/erd` / `/review_SQL` / `/review_rpg`
- **Understand Dashboard** : cartographie applicative, analyse d'impact, objets morts
- **Skills IBM i** : 71 skills auto-activées (RPG 22, CL 4, DDS 5, COBOL 12, Db2 19, IWS 5, Understand 3, Tests unitaires 1)

---

## Étape 1 — Charger le contexte

Avant de rédiger quoi que ce soit :

1. Lire `Package_Premium_IBMi/premium_ibmi_content.md` pour identifier les éléments Premium pertinents à cet UC.
2. Lire le fichier UC existant s'il existe (ex. `uc-fr/UC10-optimisation-code-fr.md`).
3. Identifier les objectifs, les sous-cas et les dépendances de l'UC à rédiger ou réviser.
4. Identifier les exemples disponibles dans `labs_FLGHT400_results/` liés au sujet de l'UC.

---

## Étape 2 — Appliquer le template canonique

Chaque fiche UC doit suivre cette structure exacte :

```
# UC [N] — [Titre en français]

## ⭐ Premium Package IBM i utilisé

[Section OBLIGATOIRE en tête de fiche. Liste les éléments Premium actifs pour cet UC :]
- Workflow : [nom du workflow] → [lien bob.ibm.com]
- Slash Command : `/[commande]` — [rôle en une ligne]
- Skills auto-activées : `[skill-1]`, `[skill-2]`, ...
- Understand Dashboard : [si applicable]

> Si aucun élément Premium ne s'applique directement, indiquer : "Les 71 skills IBM i sont activées
> automatiquement selon le contexte. Cet UC mobilise en particulier : [liste]."

---

## Objectif

[2-3 phrases : ce que fait cet UC, pour qui, et quel problème il résout]

## Prérequis

- **Phase :** [Phase X — Nom]
- **UC précédents requis :** [UCxx, UCyy — ou "Aucun"]
- **Livrables requis :** [noms de fichiers avec pattern `*-xxx-{date}.md` — ou "Aucun"]
- **Niveau Bob recommandé :** [Débutant / Intermédiaire / Avancé]
- **Bob Coins estimés :** [● ○ ○ / ● ● ○ / ● ● ●]

## Exemple concret — Lab FLGHT400

[Référencer le(s) fichier(s) de résultat dans labs_FLGHT400_results/ qui illustrent cet UC.
Format : lien vers le fichier + 1-2 phrases décrivant ce qui a été produit.]

---

## Prompts

### Prompt 0 — [Titre : initialisation / qualification / contexte]

**Objectif :** [ce que ce prompt produit]

**Quand l'utiliser :** [conditions d'application]

**Input requis :** [fichiers, membres source, livrables UC précédents]

**Espace :** [Local (`bob-reports/`) / IBM i QSYS / $HOME IFS]
**Mode :** [IBM i Developer / IBM i Database / Agent]
**Skills clés mobilisées :** `[skill-1]`, `[skill-2]`, ...

**Prompt :**

\`\`\`text
[Texte du prompt]
\`\`\`

**Livrable produit :** `{appId}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md`

---

### Prompt N — [Titre]

[Répéter le bloc Prompt pour chaque étape]

---

## Livrables de cet UC

| Fichier produit | Pattern de nommage | Consommé par |
|---|---|---|
| [description] | `*-[type]-{date}.md` | [UC suivants] |

## Dépendances

```mermaid
graph LR
  [UCprec] --> [CetUC]
  [CetUC] --> [UCsuiv]
```

---
```

---

## Étape 3 — Règles de rédaction

### Règle des trois espaces de travail
Quand un UC décrit "comment démarrer une session Bob", inclure systématiquement les trois espaces :
- **Workspace local** (poste) — fichiers `.md`, skills, rules
- **Librairies IBM i / QSYS** ⭐ — sources RPG/CL/DDS, compilation, slash commands, workflows (espace principal)
- **$HOME IFS** — livrables markdown de sortie, apps React/Node

Indiquer clairement que l'espace Librairies IBM i est l'espace par défaut pour tout exercice IBM i.

### Règle des livrables — où Bob écrit-il réellement ?
Dans la section "Livrables de cet UC", toujours préciser **où Bob écrit** selon l'espace actif :
- Si l'espace actif est **Librairies IBM i** : Bob écrit sur l'IBM i (QSYS ou IFS) — **pas en local**
  - Ex. `/init` → `{BIBLIO}/QBOBSRC/AGENTS.MD` (workspace QSYS) ou `{répertoire IFS}/.bob/AGENTS.md` (workspace IFS) — emplacements selon la doc actuelle
  - Pour avoir un fichier local : demander explicitement à Bob de sauvegarder dans le workspace local
- Si l'espace actif est **Workspace local** : Bob écrit en local
- Les fichiers créés sur IBM i sont optionnellement rapatriables via SFTP/Code for IBM i
- **Ne jamais inventer un fichier local automatique pour une commande qui écrit sur IBM i**

### Règle Premium-first
- Toujours décrire COMMENT utiliser le workflow ou la slash command **en premier**, avant les prompts libres.
- Si un workflow Premium remplace un prompt historique : indiquer `[Remplacé par le workflow XXX — voir ⭐ Premium Package ci-dessus]` à la place du prompt supprimé.
- Si un prompt historique apporte une valeur complémentaire non couverte par le Premium : le conserver avec la mention `[Complément — non couvert par le workflow Premium]`.

### Règle de périmètre et d'historique
- Ne jamais mentionner d'ancienne numérotation ni d'historique de révision dans une fiche UC : pas de ligne `Ancien numéro`, pas de « Cet UC reprend l'ancien UCxx », pas de conversion de références (« UC4 → UC 05 »), pas de « ex-UC xx ». La fiche ne doit citer que la numérotation actuelle UC00–UC19 (les nouveaux lecteurs n'ont pas connu l'ancienne).
- Si un UC regroupe plusieurs sujets, ajouter une section `## Périmètre` décrivant uniquement ce que couvre l'UC aujourd'hui (tableau des types, sous-cas, etc.). Ne pas créer de section vide : si elle n'a rien d'utile à dire, ne pas l'ajouter.

### Règle des durées et gains
- Ne pas renseigner dans l'en-tête les lignes `Durée POC (avec Bob)`, `Durée PROD (avec Bob)`, `Durée PROD (sans Bob)` ni `Gain Bob estimé` tant qu'elles n'ont pas été mesurées lors d'un lab sur FLGHT400 : pas de valeur provisoire, pas de mention « À calibrer » à leur place dans l'en-tête.
- Les éléments restant à calibrer ou à produire font l'objet d'un suivi interne et sont mesurés lors des labs sur FLGHT400.

### Règle des Skills par prompt (guide pour le lecteur)
Dans chaque fiche UC, pour chaque prompt (Prompt 0, Prompt 1, Prompts X-bis, etc.) :
- Indiquer systématiquement la ligne **`**Skills clés mobilisées :** \`[skill-1]\`, \`[skill-2]\``** dans le bloc d'en-tête de métadonnées situé **avant** le corps du prompt.
- **Rôle :** En mode IBM i Developer, Bob auto-active ces skills selon le contexte technique. Expliciter cette liste dans les métadonnées permet au lecteur/développeur de comprendre quelles expertises spécialisées sont convoquées à chaque étape et de pouvoir les forcer via `@nom-du-skill` en cas de besoin.
- **Garde-fou :** Ne jamais injecter artificiellement la liste des skills dans le texte brut du prompt (le prompt doit rester propre, concis et centré sur la consigne métier et technique).

### Convention de nommage des livrables
Format obligatoire : `{appId}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md`
- `appId` : identifiant court de l'application, indépendant de tout outil (ex. `flght403`, `trc0018`, `compta`)
- `fonction` : nom fonctionnel court (ex. `comprehension`, `ddl`, `sql-embarque`)
- `composant` : programme ou module concerné (ex. `frs409`, `all`)
- `type` : nature du livrable (ex. `report`, `plan`, `diff`, `spec-fonc`, `regles`)
- Exemples : `flght403-comprehension-frs409-report-20261030-1400.md`

---

## Étape 4 — Mapping UC ↔ éléments Premium (référence rapide)

| UC | Workflow Premium | Slash Command | Understand | Skills clés |
|---|---|---|---|---|
| UC01 Connexion | — | — | — | — |
| UC02 Maîtrise Bob | — | `/init` | — | — |
| UC03 Bob en équipe | — | `/init` → AGENTS.md | — | — |
| UC04 Stratégie Skills | — | — | — | (lien skill `poc-ibmi-premium-uc-writer`) |
| UC05 Compréhension | Business Rules Extraction | `/init` | ✅ | `understand-app-overview`, `understand-program-explainer` |
| UC06 Logique métier | Business Rules Extraction (RPG uniquement) | — | ✅ | `rpg-code-review`, `cobol-procedure-division` |
| UC07 Documentation | — | `/erd`, `/init` | ✅ | `understand-impact-analysis`, `db2-system-catalog` |
| UC08 DDS→DDL | — | `/erd` | — | `db2-dds-to-ddl`, `dds-physical-files` |
| UC09 SQL embarqué | SQL Index Strategy Advisor (partiel) | `/review_SQL` | — | `rpg-rla-to-sql`, `rpg-embedded-sql`, `db2-sql-stored-procedure-performance` |
| UC10 Optimisation | SQL Index Strategy Advisor (partiel) | `/review_rpg`, `/review_SQL` | — | `rpg-code-review`, `rpg-rla-to-sql`, `db2-sql-stored-procedure-performance` |
| UC11 Restructuration | RPG Modernization (partiel) | `/review_rpg` | ✅ | `rpg-fixed-to-free`, `rpg-refactor-legacy`, `cobol-code-refactoring` |
| UC12 COBOL→FREE | RPG Modernization | `/review_rpg` | — | `rpg-opm-to-ile`, `cobol-*` (dont `cobol-code-review`, `cobol-code-refactoring`) |
| UC13 RPG colonné→FREE | RPG Modernization | `/review_rpg` | — | `rpg-fixed-to-free`, `rpg-cycle-explained` |
| UC14 Tests RPGUnit | RPGUnit Test Plan + Test Suite | — | — | `rpgunit-testing`, `rpg-debugging-techniques` |
| UC15 Génération code | — | — | — | `rpg-procedures-functions`, `rpg-free-format-fundamentals` |
| UC16 Objets SQL | SQL Index Strategy Advisor (partiel) | `/review_SQL` | — | `db2-stored-procedures`, `db2-index-strategy`, `db2-sql-stored-procedure-performance` |
| UC17 Génération apps | — | — | — | `iws-service-ops`, `dds-display-files` |
| UC18 Exposition API REST | — | `/init`, `/review_rpg` | — | `iws-admin`, `iws-server-ops`, `iws-server-config`, `iws-service-ops`, `iws-backup` |
| UC19 Exploitation Ops | — | `/init` | — | `db2-security-best-practices`, `db2-system-catalog`, `db2-services-catalog`, `db2-sql-performance-analysis` |

---

## Étape 5 — Bilinguisme FR + EN

**Règle automatique (aucun rappel nécessaire) :**

Chaque UC produit **deux fichiers séparés** dans deux répertoires dédiés :

| Langue | Répertoire | Nom de fichier |
|---|---|---|
| Français | `uc-fr/` | `UC[N]-[slug]-fr.md` |
| Anglais | `uc-en/` | `UC[N]-[slug]-en.md` |

Exemples :
- `uc-fr/UC00-premium-package-ibmi-fr.md`
- `uc-en/UC00-premium-package-ibmi-en.md`

**Workflow de production :**
1. Rédiger la version FR complète dans `uc-fr/UCxx-...-fr.md`
2. Une fois la FR validée, produire la version EN dans `uc-en/UCxx-...-en.md`
3. Les chemins relatifs vers `labs_FLGHT400_results/` sont `../labs_FLGHT400_results/` (un seul niveau au-dessus — `uc-fr/` et `uc-en/` sont directement dans `Poc_Anonyme/`)

**Règles de traduction :**
- Traduire fidèlement, adapter les tournures idiomatiques (ne pas traduire mot à mot)
- Les noms de fichiers, patterns de nommage et noms de skills restent en anglais
- Les liens `bob.ibm.com` sont identiques dans les deux langues
- Les titres de section H2/H3 sont traduits

---

## Étape 6 — Vérification finale avant livraison

Cocher chaque point avant de déclarer l'UC terminé :

- [ ] Section `⭐ Premium Package IBM i utilisé` présente et complète en tête
- [ ] Mention `**Skills clés mobilisées :**` présente sur chaque prompt (en métadonnées avant le corps du prompt)
- [ ] Au moins un exemple `labs_FLGHT400_results/` référencé
- [ ] Convention de nommage des livrables respectée
- [ ] Tous les liens `bob.ibm.com` présents pour les éléments Premium utilisés
- [ ] Dépendances inter-UC à jour (prérequis + consommateurs)
- [ ] Fichier FR créé dans `uc-fr/UCxx-...-fr.md`
- [ ] Fichier EN créé dans `uc-en/UCxx-...-en.md`
- [ ] Cohérence avec la structure globale du POC
