# Plan — Anonymisation des fichiers MD du répertoire POC-Bob-SystemU

## Vue d'ensemble

**Objectif :** Produire des versions anonymisées des fichiers Markdown du POC dans `POC-Bob-SystemU/Anonyme/`.
Les documents originaux ne sont pas modifiés.

**Périmètre :** Fichiers UCxx à la racine de `POC-Bob-SystemU/` (les sous-répertoires sont exclus).
Les fichiers internes de travail (bob-report, bob-task, plan-uc, prompt-*, task*) sont déplacés dans `/Trash`, pas anonymisés.

**Approche :** Repartir des fichiers **originaux** pour chaque anonymisation.
Pour chaque fichier, appliquer les substitutions ci-dessous et enregistrer dans `POC-Bob-SystemU/Anonyme/`.
Les noms de fichiers contenant `systemu` sont renommés (ex. `plan-poc-bob-systemu.md` → `plan-poc-bob-acme.md`).

**GitHub :** Le répertoire `Anonyme/` est publié sur https://github.com/brostagni/POC-BOB-for-IBMi.
Un `README.md` bilingue (français + anglais) doit être créé à la racine du dépôt.

---

## Table de substitutions

| Terme original | Remplacement | Notes |
|---|---|---|
| `System-U` | `ACME` | Toutes casses et variantes |
| `U-Tech` | `ACME` | Abréviation du client |
| `Système U` | `ACME` | Variante accentuée |
| `SystemU` (sans tiret) | `ACME` | Variante sans tiret |
| `systemu` (minuscules) | `acme` | Dans noms de fichiers/code/chemins |
| `systemu.fr` | `acme.fr` | Domaine email |
| `TECHZONE-TS022467569` | `[TECHZONE_ID]` | Identifiant de réservation TechZone |
| `ARCAD` | *(conservé)* | Outil public connu, pas de substitution |

---

## Fichiers concernés

### Fichiers déjà dans `Anonyme/` — statut vérifié par comparaison de contenu

| Fichier source | Fichier dans `Anonyme/` | Statut |
|---|---|---|
| `UC01-cobol-free-rpg.md` | `UC01-cobol-free-rpg.md` | ⚠️ source tronqué dans Anonyme (1075 vs ~1050 lignes) → à re-anonymiser |
| `UC02-rpg-colonne-free.md` | `UC02-rpg-colonne-free.md` | ⚠️ source tronqué dans Anonyme (1054 vs ~1040 lignes) → à re-anonymiser |
| `UC03-sql-embarque.md` | `UC03-sql-embarque.md` | ✅ sync, propre |
| `UC04-comprehension-code.md` | `UC04-comprehension-code.md` | ✅ sync, propre |
| `UC05-logique-metier.md` | `UC05-logique-metier.md` | ✅ sync, propre |
| `UC06-documentation-complete.md` | `UC06-documentation-complete.md` | ⚠️ terme `System-U` résiduel ligne 622 → à re-anonymiser |
| `UC07-optimisation-code.md` | `UC07-optimisation-code.md` | ✅ sync, propre |
| `UC08-restructuration-code.md` | `UC08-restructuration-code.md` | ✅ sync, propre |
| `UC12-mcp-enabler.md` | `UC12-mcp-enabler.md` | ✅ sync, propre |
| `UC14-dds-ddl.md` | `UC14-dds-ddl.md` | ✅ sync, propre |
| `UC15-maitrise-bob.md` | `UC15-maitrise-bob.md` | ✅ sync, propre |
| `plan-poc-bob-systemu.md` | `plan-poc-bob-acme.md` | ⚠️ source mis à jour (section 6 ajoutée) → à re-anonymiser |

### Fichiers à anonymiser (cette session)

| Fichier source | Fichier destination dans `Anonyme/` | Notes |
|---|---|---|
| `UC01-cobol-free-rpg.md` | `UC01-cobol-free-rpg.md` | Source enrichi depuis dernière anonymisation |
| `UC02-rpg-colonne-free.md` | `UC02-rpg-colonne-free.md` | Source enrichi depuis dernière anonymisation |
| `UC06-documentation-complete.md` | `UC06-documentation-complete.md` | Terme client résiduel détecté ligne 622 |
| `UC09-generation-code.md` | `UC09-generation-code.md` | Nouveau UC Phase 4 |
| `UC10-generation-app.md` | `UC10-generation-app.md` | Nouveau UC Phase 4 |
| `UC11-objets-sql.md` | `UC11-objets-sql.md` | Nouveau UC Phase 4 |
| `UC13-tests-unitaires.md` | `UC13-tests-unitaires.md` | Nouveau UC Phase 5 |

### Fichiers à déplacer dans `/Trash` (pas anonymisés, pas publiés)

Ces fichiers sont des outils internes de travail qui ne font pas partie des livrables du POC :
- `Bob-Report-2026-07-28.md`
- `bob-task-*.json` (tous les fichiers de tâches JSON)
- `plan-uc09-generation-code.md`
- `plan-uc10-generation-app.md`
- `plan-uc11-objets-sql.md`
- `plan-uc12-mcp-enabler.md`
- `plan-uc13-tests-unitaires.md`
- `prompt-revision-nomenclature-contexte.md`
- `prompt-uc-phase4.md`
- `prompt-uc7-uc8.md`
- `task2-uc11-plan.md`
- `task3-uc10-plan.md`

---

## Sous-tâches

### Sous-tâche 1 — Déplacer les fichiers internes dans `/Trash`

**Intent :** Ranger dans `/Trash` les fichiers de travail interne qui ne font pas partie
des livrables publiables : rapports, tâches bob, plans de rédaction, prompts de session, task-plans.
Ces fichiers **ne sont pas supprimés** — ils restent dans la Corbeille.

**Expected Outcomes :**
- Les fichiers listés dans "Fichiers à déplacer dans `/Trash`" ont été déplacés (pas supprimés).
- La racine de `POC-Bob-SystemU/` ne contient plus que : les UCxx, `plan-poc-bob-systemu.md`, `anonymisation-plan.md`.

**Todo List :**
1. Déplacer `Bob-Report-2026-07-28.md` vers `/Trash/`.
2. Déplacer tous les `bob-task-*.json` vers `/Trash/`.
3. Déplacer `plan-uc09-generation-code.md`, `plan-uc10-generation-app.md`, `plan-uc11-objets-sql.md`, `plan-uc12-mcp-enabler.md`, `plan-uc13-tests-unitaires.md` vers `/Trash/`.
4. Déplacer `prompt-revision-nomenclature-contexte.md`, `prompt-uc-phase4.md`, `prompt-uc7-uc8.md` vers `/Trash/`.
5. Déplacer `task2-uc11-plan.md`, `task3-uc10-plan.md` vers `/Trash/`.

**Relevant Context :** `/Trash` = répertoire `~/.Trash` sur macOS. Utiliser `mv` pour déplacer sans supprimer.
L'atelier d'industrialisation des prompts n'est plus valide dans Bob — supprimer `Anonyme/atelier-bob-industrialisation-prompts.md` (suppression définitive).

**Todo List (suite) :**
6. Supprimer `Anonyme/atelier-bob-industrialisation-prompts.md`.

**Status :** `[ ] pending`

---

### Sous-tâche 2 — Anonymiser UC01, UC02, UC06 (mis à jour / terme résiduel) + UC09, UC10, UC11, UC13 (nouveaux)

**Intent :** Re-anonymiser les UC dont la source a été enrichie depuis la dernière anonymisation (UC01, UC02) ou qui contiennent encore un terme client (UC06), et produire les quatre nouveaux UC de Phase 4/5. UC01 et UC02 ont été significativement étoffés (sections "Points à compléter avant production", constructs spéciaux COBOL/RPG).

**Expected Outcomes :**
- 7 fichiers dans `Anonyme/` sans aucune occurrence de terme client.
- `plan-poc-bob-systemu.md` → `plan-poc-bob-acme.md` dans les liens internes (UC01 ligne 149, UC02 ligne 163, UC09 ligne 120).
- `systemu-APPVTE-` → `acme-APPVTE-` dans tous les exemples de noms de fichiers.
- `System-U Developer` → `ACME Developer` (mode personnalisé UC09).
- `systemu-developer-mode-v1.yaml` → `acme-developer-mode-v1.yaml` (UC09 ligne 441).

**Todo List :**
1. Lire `UC01-cobol-free-rpg.md`, appliquer les substitutions, écrire dans `Anonyme/UC01-cobol-free-rpg.md`.
2. Lire `UC02-rpg-colonne-free.md`, appliquer les substitutions, écrire dans `Anonyme/UC02-rpg-colonne-free.md`.
3. Lire `UC06-documentation-complete.md`, appliquer les substitutions, écrire dans `Anonyme/UC06-documentation-complete.md`.
4. Lire `UC09-generation-code.md`, appliquer les substitutions, écrire dans `Anonyme/UC09-generation-code.md`.
5. Lire `UC10-generation-app.md`, appliquer les substitutions, écrire dans `Anonyme/UC10-generation-app.md`.
6. Lire `UC11-objets-sql.md`, appliquer les substitutions, écrire dans `Anonyme/UC11-objets-sql.md`.
7. Lire `UC13-tests-unitaires.md`, appliquer les substitutions, écrire dans `Anonyme/UC13-tests-unitaires.md`.
8. Vérifier l'absence de toute occurrence des termes interdits dans les 7 fichiers produits.

**Relevant Context :**
- UC01 ligne 56 : `System-U` dans recommandation → `ACME` ; `systemu-APPVTE-` dans exemples de nommage.
- UC02 ligne 68 : `System-U` ; `plan-poc-bob-systemu.md` ligne 163 → `plan-poc-bob-acme.md`.
- UC06 ligne 622 : `à valider par l'équipe System-U` → `à valider par l'équipe ACME`.
- UC09 : nombreuses occurrences de `System-U Developer`, `systemu-developer-mode-v1.yaml`, `systemu-APPVTE-`.
- UC10 : `System-U` pour choix de framework Web ; `systemu-APPVTE-` dans exemples.
- UC11 : conventions SQL `System-U` ; `systemu-APPVTE-`.
- UC13 : `System-U` dans contexte RPGUnit/ARCAD ; `systemu-APPVTE-`.

**Status :** `[ ] pending`

---

### Sous-tâche 3 — Re-anonymiser le plan global mis à jour

**Intent :** Remplacer la version existante de `Anonyme/plan-poc-bob-acme.md` par une version fraîche
depuis `plan-poc-bob-systemu.md`, qui a été enrichi depuis la dernière anonymisation
(section 6 avec UC 9-11-13, ligne 263 avec `systemu-developer-mode-v1.json`).

**Expected Outcomes :**
- `Anonyme/plan-poc-bob-acme.md` mis à jour, sans termes clients, incluant les nouvelles sections.

**Todo List :**
1. Lire `plan-poc-bob-systemu.md` dans son intégralité.
2. Appliquer toutes les substitutions (notamment `systemu-developer-mode-v1.json` → `acme-developer-mode-v1.json`).
3. Écrire dans `Anonyme/plan-poc-bob-acme.md` (écrase la version précédente).
4. Vérifier l'absence de toute occurrence des termes interdits.

**Relevant Context :**
- `plan-poc-bob-systemu.md` ligne 263 : `systemu-developer-mode-v1.json` → `acme-developer-mode-v1.json`.
- Lignes 169 et 193 : occurrences de `System-U` dans le contexte ARCAD.
- Ligne 118 : `System-U Developer` (mode personnalisé) → `ACME Developer`.

**Status :** `[ ] pending`

---

### Sous-tâche 4 — Mettre à jour `anonymisation-plan.md` dans `Anonyme/`

**Intent :** Placer la version mise à jour de ce fichier plan dans `Anonyme/`.

**Expected Outcomes :**
- `Anonyme/anonymisation-plan.md` identique à `POC-Bob-SystemU/anonymisation-plan.md`.

**Todo List :**
1. Copier `anonymisation-plan.md` vers `Anonyme/anonymisation-plan.md`.

**Status :** `[ ] pending`

---

### Sous-tâche 5 — Créer le README bilingue pour le dépôt GitHub

**Intent :** Créer un fichier `README.md` dans `Anonyme/` qui décrit le projet de façon claire et
publique, sans aucune référence au client. Bilingue français + anglais. Ce README sera le point
d'entrée du dépôt GitHub `https://github.com/brostagni/POC-BOB-for-IBMi`.

**Expected Outcomes :**
- `Anonyme/README.md` décrit : contexte IBM i, objectif du POC Bob, les 16 use cases par phase, comment utiliser les fiches.
- Aucune mention du nom du client, de son secteur, de l'ID TechZone.
- Deux sections : version française complète, puis version anglaise complète.

**Todo List :**
1. Lire `Anonyme/plan-poc-bob-acme.md` pour extraire la structure du POC (phases, UC, objectifs).
2. Rédiger la section française : présentation, phases, liste des livrables, guide d'utilisation.
3. Rédiger la section anglaise avec le même contenu.
4. Écrire dans `Anonyme/README.md`.
5. Vérifier l'absence de tout terme client.

**Relevant Context :**
- Source d'information principale : `Anonyme/plan-poc-bob-acme.md` (version déjà propre).
- Structure : 6 phases, 16 UC (UC01–UC15 présents, UC16 hors scope MCP ARCAD).
- Le README doit être auto-suffisant pour un lecteur externe qui découvre le dépôt.

**Status :** `[ ] pending`

---

### Sous-tâche 6 — Vérification globale et push GitHub

**Intent :** Vérifier que le répertoire `Anonyme/` est complet et propre, puis pousser vers
`https://github.com/brostagni/POC-BOB-for-IBMi`.

**Expected Outcomes :**
- Grep global sur `Anonyme/` : zéro occurrence de `System-U`, `U-Tech`, `Système U`, `SystemU`, `systemu`, `TECHZONE-TS022467569`.
- Le dépôt GitHub est à jour et le README s'affiche correctement.

**Todo List :**
1. Lancer un grep global sur `Anonyme/` pour chaque terme interdit.
2. Corriger toute occurrence résiduelle trouvée.
3. Configurer le remote git et pousser sur `https://github.com/brostagni/POC-BOB-for-IBMi`.
4. Confirmer que le dépôt est accessible et que le README s'affiche correctement.

**Relevant Context :** Le dépôt vient d'être créé — vérifier s'il faut initialiser git ou s'il y a déjà un remote configuré.

**Status :** `[ ] pending`

---

## Validation finale

Après completion de toutes les sous-tâches, vérifier que :
- `POC-Bob-SystemU/Anonyme/` contient : UC01–UC15 (15 fichiers), `plan-poc-bob-acme.md`,
  `anonymisation-plan.md`, `README.md` = **18 fichiers**.
- Aucun fichier dans `Anonyme/` ne contient : `System-U`, `U-Tech`, `Système U`, `SystemU`,
  `systemu`, `TECHZONE-TS022467569`.
- Le fichier `plan-poc-bob-acme.md` est bien la version mise à jour.
- Les fichiers originaux dans `POC-Bob-SystemU/` sont intacts.
- Le mot commun "client" dans les textes courants n'a pas été altéré.
- Le dépôt GitHub `https://github.com/brostagni/POC-BOB-for-IBMi` est à jour avec le README visible.
