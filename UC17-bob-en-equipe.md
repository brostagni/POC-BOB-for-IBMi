# UC 17 — Bob en équipe : workspace d'entreprise

> **Catégorie :** Utilisation de Bob
>
> **Priorité dans le POC :** Transversal — à mettre en place dès la Phase 0, avant tout autre UC
>
> **Durée de mise en place initiale :** 2 à 4 heures — création du workspace entreprise, conversion des normes ACME en rules Bob, configuration des modes et MCP partagés
>
> **Durée d'onboarding d'un nouveau développeur :** 30 à 45 minutes — `git clone` du repo entreprise + configuration du poste (`~/.bob/`)
>
> **Mode Bob recommandé :** Agent (pour la création et l'organisation des fichiers), Ask (pour les questions sur la configuration)

---

## Objectif

Industrialiser l'utilisation de Bob au sein d'une équipe de développement IBM i, en garantissant que **tous les développeurs travaillent avec les mêmes conventions, les mêmes normes, les mêmes outils** — quelle que soit leur machine, quel que soit le projet.

**Ce UC répond à trois problèmes concrets :**

1. **L'incohérence** — sans cadrage, chaque développeur configure Bob différemment : normes de code différentes, modes différents, comportements imprévisibles.
2. **L'oubli de contexte** — Bob est stateless. Sans `AGENTS.md` et sans rules, il redécouvre le projet à chaque session.
3. **Le temps perdu** — onboarder un nouveau développeur sur Bob prend des heures si la configuration n'est pas centralisée.

**La solution Bob :** une hiérarchie de workspaces en 4 niveaux, tous versionnés sauf le niveau poste, qui se combinent et se complètent.

---

## Les 4 niveaux de configuration Bob

Bob applique ses configurations en combinant plusieurs sources, **du plus spécifique au moins spécifique** — le niveau projet l'emporte toujours sur le niveau global.

```
Niveau 1 — Poste développeur (~/.bob/)
  Portée : un seul poste, un seul développeur
  Versionné : NON — jamais
  Contenu : préférences personnelles, tokens d'authentification réels

Niveau 2 — Workspace entreprise (repo Git partagé)
  Portée : toute l'entreprise, tous les projets
  Versionné : OUI — git clone par chaque développeur
  Contenu : normes de code, modes personnalisés, skills partagés, MCP sans tokens

Niveau 3 — Workspace projet (repo applicatif)
  Portée : un seul projet applicatif
  Versionné : OUI — dans le repo du projet
  Contenu : contexte projet, rules spécifiques, skills projet, .bobignore

Niveau 4 — Projet initialisé avec /init
  Portée : un seul projet, toutes les sessions Bob
  Versionné : OUI — AGENTS.md dans le repo
  Contenu : contexte généré par /init + enrichi manuellement
```

**Règle de priorité documentée :**
- Rules : workspace > global ([doc](https://bob.ibm.com/docs/ide/configuration/rules#rule-priority))
- Skills : projet > global ([doc](https://bob.ibm.com/docs/ide/features/skills#skill-locations))
- Custom modes : projet > global > défaut ([doc](https://bob.ibm.com/docs/ide/configuration/custom-modes#edit-configuration-files-manually))
- MCP : projet > global ([doc](https://bob.ibm.com/docs/ide/configuration/mcp/mcp-in-bob#configuration-levels))

---

## Arborescence cible

### Workspace entreprise — `acme-bob-commons/`

```
acme-bob-commons/                      ← repo Git dédié aux standards Bob
├── .bob/
│   ├── rules/
│   │   ├── normes-programmation.md    ← NORMES/ACME-Normes Programmation.md → rules Bob
│   │   ├── normes-nomenclature.md     ← NORMES/ACME-Normes nomenclature.md → rules Bob
│   │   └── conventions-arcad.md       ← NORMES/ACME-Normes Arcad.md → rules Bob
│   ├── rules-agent/
│   │   └── workflow-ibmi.md           ← instructions spécifiques au mode Agent IBM i
│   ├── skills/
│   │   ├── uc04-comprehension/        ← skill UC4 partagé
│   │   │   └── SKILL.md
│   │   ├── uc07-optimisation-rpg/     ← skill UC7 partagé
│   │   │   └── SKILL.md
│   │   └── uc13-rpgunit/              ← skill UC13 partagé
│   │       └── SKILL.md
│   ├── custom_modes.yaml              ← modes IBM i Developer, IBM i Database, etc.
│   ├── mcp.json                       ← MCP partagés SANS tokens (env vars uniquement)
│   └── commands/
│       ├── revue-code.md              → /revue-code
│       ├── init-projet.md             → /init-projet
│       └── doc-programme.md           → /doc-programme
├── .bobignore.template                ← modèle à copier dans chaque projet
├── .gitignore                         ← exclure les fichiers avec credentials
└── README.md                          ← guide d'installation pour les développeurs
```

### Workspace projet — `acme/{appArcad}/`

```
acme/TRC0018/                          ← repo Git du projet applicatif
├── SOURCES/                           ← sources IBM i
├── OKF/                               ← livrables Bob
├── .bob/
│   ├── rules/
│   │   └── contexte-projet.md         ← spécificités TRC0018 (bibliothèques, contraintes)
│   ├── skills/
│   │   └── trc0018-metier/            ← skill spécifique à ce projet
│   │       └── SKILL.md
│   ├── mcp.json                       ← MCP projet SANS tokens (env vars)
│   ├── rules-agent/AGENTS.md          ← généré par /init
│   ├── rules-ask/AGENTS.md            ← généré par /init
│   └── rules-plan/AGENTS.md           ← généré par /init
├── AGENTS.md                          ← généré par /init + enrichi manuellement
├── .bobignore                         ← issu du template entreprise + adapté
└── .gitignore                         ← inclure *.env, secrets/, tokens
```

### Poste développeur — `~/.bob/` (jamais versionné)

```
~/.bob/
├── rules/
│   └── preferences-perso.md           ← langue de réponse, niveau de verbosité
├── settings/
│   └── custom_modes.yaml              ← éventuels modes personnels
├── mcp.json                           ← tokens RÉELS (JIRA, Confluence, IBM i)
├── skills/
│   └── (skills personnels)
└── commands/
    └── (commandes personnelles)
```

---

## Étape 1 — Créer le workspace entreprise

### 1.1 Créer le repo Git

```bash
mkdir acme-bob-commons
cd acme-bob-commons
git init
mkdir -p .bob/rules .bob/rules-agent .bob/skills .bob/commands
touch .gitignore README.md .bobignore.template
```

### 1.2 Convertir les normes ACME en rules Bob

Les normes ACME existantes (`NORMES/ACME-Normes Programmation.md`, `NORMES/ACME-Normes nomenclature.md`, `NORMES/ACME-Normes Arcad.md`) doivent être transformées en **instructions directes pour Bob** — pas une documentation à lire, mais des directives à appliquer.

**Format recommandé pour une rule Bob :**

```markdown
# Normes de programmation RPG — ACME

## Conventions de nommage des variables
- Les variables de travail commencent par W_ (ex: W_MONTANT, W_CPTEUR)
- Les paramètres de procédure commencent par P_ (ex: P_CODCLI, P_DATCMD)
- Les constantes sont en MAJUSCULES avec underscores (ex: MAX_LIGNES, TVA_TAUX)

## Conventions de nommage des procédures
- Verbe + Objet en camelCase (ex: calculerTaxe, validerCommande)
- Procédures de lecture : LireXxx (ex: LireClient, LireCommande)
- Procédures d'écriture : EcrireXxx ou MettreAJourXxx

## Structure des programmes
- Toujours déclarer les variables en début de programme dans une section dédiée
- Les sous-routines sont interdites — utiliser des procédures ILE
- Chaque procédure fait au maximum 50 lignes
```

> 📚 Doc : [Custom rules — Rule scopes](https://bob.ibm.com/docs/ide/configuration/rules#rule-scopes) · [Team standardization](https://bob.ibm.com/docs/ide/configuration/rules#team-standardization)

**Demander à Bob de faire la conversion :**
```
En mode Agent, ouvrir @NORMES/ACME-Normes Programmation.md et me demander :
"Convertis ces normes en rules Bob : extrait les directives actionnables sous forme
d'instructions courtes et directes, au format Markdown, dans le fichier
.bob/rules/normes-programmation.md"
```

### 1.3 Créer les custom modes partagés

Fichier `.bob/custom_modes.yaml` du repo entreprise :

```yaml
customModes:
  - slug: ibmi-developer
    name: 🖥️ IBM i Developer
    description: Développement et modernisation IBM i — RPG, CL, DDS, SQL embarqué
    roleDefinition: >-
      Tu es un expert en développement IBM i avec une maîtrise approfondie de
      RPG ILE Free Format, CL, DDS, SQL embarqué, et des normes de développement
      ACME. Tu connais les conventions de nommage ACME, les normes ARCAD,
      et les pratiques de modernisation IBM i.
    whenToUse: >-
      Utilise ce mode pour tout développement, modernisation ou analyse de code
      IBM i (RPG, CL, DDS, COBOL). Active-le pour les UC 1 à 11 et UC 13-14.
    customInstructions: >-
      Applique systématiquement les normes de programmation ACME.
      Utilise le FREE RPG ILE pour tout nouveau code.
      Respecte les conventions de nommage ACME définies dans les rules.
      Pour les accès aux données, privilégie le SQL embarqué (UC3).
    groups:
      - read
      - edit
      - command
      - mcp
      - skill
```

> 📚 Doc : [Custom modes — Edit configuration files manually](https://bob.ibm.com/docs/ide/configuration/custom-modes#edit-configuration-files-manually) · [YAML format](https://bob.ibm.com/docs/ide/configuration/custom-modes#yaml-format)

### 1.4 Configurer les MCP partagés (SANS tokens)

Fichier `.bob/mcp.json` du repo entreprise — **variables d'environnement uniquement, jamais de tokens en clair** :

```json
{
  "mcpServers": {
    "jira": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@ibm/mcp-jira"],
      "env": {
        "JIRA_URL": "${JIRA_URL}",
        "JIRA_TOKEN": "${JIRA_TOKEN}"
      }
    },
    "ibmi": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@ibm/mcp-ibmi"],
      "env": {
        "IBMI_HOST": "${IBMI_HOST}",
        "IBMI_USER": "${IBMI_USER}",
        "IBMI_PASSWORD": "${IBMI_PASSWORD}"
      }
    }
  }
}
```

Les valeurs réelles (`${JIRA_TOKEN}`, `${IBMI_PASSWORD}`) sont définies sur le **poste développeur** dans un fichier `.env` ou via les variables d'environnement du système — jamais dans ce fichier versionné.

> ⚠️ **Règle absolue** ([doc sécurité MCP](https://bob.ibm.com/docs/ide/configuration/mcp/understanding-mcp#common-questions)) : Ne jamais committer de tokens, mots de passe ou API keys dans un fichier `mcp.json`. Ajouter `mcp.json` au `.gitignore` si des credentials y apparaissent.

> 📚 Doc : [MCP — Configuration levels](https://bob.ibm.com/docs/ide/configuration/mcp/mcp-in-bob#configuration-levels) · [Edit configuration files](https://bob.ibm.com/docs/ide/configuration/mcp/mcp-in-bob#edit-configuration-files)

### 1.5 Créer le template .bobignore

Fichier `.bobignore.template` du repo entreprise — à copier et renommer `.bobignore` dans chaque projet :

```
# Credentials et secrets — NE JAMAIS laisser Bob accéder à ces fichiers
.env
.env.*
secrets/
*.key
*.pem
config/credentials.json
config/connexion.json

# Fichiers de connexion IBM i
**/QSYS.LIB/**/*.SRVPGM
connexion-ibmi.json

# Tokens ARCAD
arcad-token.txt
.arcad-credentials

# Build artifacts — pas utile pour Bob
build/
dist/
*.o
*.pgm.bak

# Gros fichiers binaires
*.bmp
*.jpg
*.png
```

> 📚 Doc : [Using .bobignore](https://bob.ibm.com/docs/ide/configuration/bobignore) · [Security guidelines — .bobignore](https://bob.ibm.com/docs/ide/security/bob-security-guidance#restrict-file-access-with-bobignore)

### 1.6 Créer le .gitignore du repo entreprise

```
# Jamais de credentials dans ce repo
.env
.env.*
*.key
secrets/
mcp-with-tokens.json

# macOS
.DS_Store

# Bob local
.bob/mcp-local.json
```

### 1.7 Versionner et partager

```bash
git add .bob/
git add .bobignore.template .gitignore README.md
git commit -m "feat: socle Bob commun ACME — rules, modes, skills, MCP template"
git push origin main
```

> 📚 Doc : [Project-level standards — version control](https://bob.ibm.com/docs/ide/configuration/rules#project-level-standards)

---

## Étape 2 — Créer le workspace projet

### 2.1 Structure minimale

Dans le repo du projet applicatif, créer le dossier `.bob/` avec les éléments spécifiques au projet :

```bash
cd acme/TRC0018
mkdir -p .bob/rules .bob/skills .bob/commands
```

### 2.2 Rules spécifiques au projet

Fichier `.bob/rules/contexte-projet.md` :

```markdown
# Contexte projet — TRC0018 (Traitement des commandes)

## Bibliothèques IBM i
- Bibliothèque de développement : ACME_DEV
- Bibliothèque de production : ACME_PRD
- Bibliothèque de test : ACME_TST

## Programmes principaux
- TRC0018 : programme principal de traitement des commandes
- TRC0018R : sous-programme de calcul des remises
- TRC0018W : module d'écriture dans CDEFAUT

## Contraintes spécifiques
- Ne jamais modifier directement les fichiers ARCAD — toujours passer par le workspace Bob
- Les modifications doivent être testées en ACME_TST avant toute promotion
- Référence fonctionnelle : OKF/acme/GEE/TRI/TRC0018/spec-fonc.md

## Conventions spécifiques à ce projet
- Préfixe des variables locales à TRC0018 : T18_
- Les erreurs sont loguées dans le journal JRNL0018
```

### 2.3 Copier et adapter le .bobignore

```bash
cp ../acme-bob-commons/.bobignore.template .bobignore
# Adapter si nécessaire pour ce projet
```

### 2.4 Configurer le .gitignore projet

```
# Secrets et credentials — JAMAIS dans Git
.env
.env.*
secrets/
*.key
.bob/mcp-local.json

# Bob local uniquement
~/.bob/

# macOS
.DS_Store
```

---

## Étape 3 — Initialiser le projet avec `/init`

### Pourquoi /init est indispensable

Bob est **stateless** : chaque nouvelle conversation repart de zéro, sans mémoire des échanges précédents. Sans `AGENTS.md`, Bob doit redécouvrir le projet à chaque session — ce qui est lent, coûteux en Bob Coins, et produit des résultats incohérents.

`/init` résout ce problème en générant un fichier `AGENTS.md` que Bob relit automatiquement à chaque conversation, lui donnant un contexte persistant sur le projet.

> 📚 Doc : [Start a project with /init and AGENTS.md](https://bob.ibm.com/docs/ide/getting-started/tutorials/start-a-project)

### 3.1 Lancer /init

1. Ouvrir le workspace projet dans Bob
2. Passer en mode **Agent**
3. Taper dans le chat :

```
/init
```

Bob scanne le projet et génère :

```
AGENTS.md                         ← contexte racine (structure, stack, conventions)
.bob/rules-agent/AGENTS.md        ← contexte spécifique au mode Agent
.bob/rules-ask/AGENTS.md          ← contexte spécifique au mode Ask
.bob/rules-plan/AGENTS.md         ← contexte spécifique au mode Plan
```

> 📚 Doc : [Run /init](https://bob.ibm.com/docs/ide/getting-started/tutorials/start-a-project#run-init) · [Mode-specific AGENTS.md files](https://bob.ibm.com/docs/ide/getting-started/tutorials/start-a-project#mode-specific-agentsmd-files)

### 3.2 Enrichir manuellement l'AGENTS.md

`/init` génère un contexte structurel (arborescence, stack technique). Il ne peut pas inférer les règles métier, les conventions ARCAD, ni les contraintes d'exploitation. Ces éléments doivent être ajoutés manuellement.

**Sections à ajouter manuellement dans `AGENTS.md` :**

```markdown
## Règles métier clés
- Le calcul des remises TRC0018R s'applique uniquement aux commandes de type CODE > 3
- Les commandes en attente (STATUS = 'A') ne doivent jamais être modifiées par un batch
- Le fichier CDEFAUT est partagé entre TRC0018 et TRC0021 — toute écriture doit être journalisée

## Dépendances applicatives
- TRC0018 appelle TRC0018R via CALLB — ne jamais modifier la signature de procédure sans adapter l'appelant
- CDEFAUT est aussi utilisé par la facturation (FAC0001) — validation croisée obligatoire

## Workflow de développement
- Développement dans ACME_DEV
- Test unitaire avec RPGUnit avant promotion
- Promotion via ARCAD uniquement — ne pas copier les objets manuellement

## Références
- Spécifications fonctionnelles : OKF/acme/GEE/TRI/TRC0018/spec-fonc.md
- Règles métier extraites : OKF/acme/GEE/TRI/TRC0018/regles.md
- Architecture : OKF/acme/GEE/TRI/TRC0018/architecture.md
```

### 3.3 Quand relancer /init

Relancer `/init` après :
- Ajout d'un nouveau module ou d'une nouvelle bibliothèque
- Changement de structure de répertoires
- Arrivée d'un nouveau développeur sur le projet
- Changement de stack technique (ex: ajout SQL embarqué après UC3)

> 📚 Doc : [Maintain the AGENTS.md files](https://bob.ibm.com/docs/ide/getting-started/tutorials/start-a-project#maintain-the-agentsmd-files)

### 3.4 Versionner l'AGENTS.md

```bash
git add AGENTS.md .bob/rules-agent/AGENTS.md .bob/rules-ask/AGENTS.md .bob/rules-plan/AGENTS.md
git commit -m "feat: contexte Bob initialisé avec /init + enrichissement manuel"
```

---

## Étape 4 — Configurer le poste développeur

### Ce qui appartient au poste (`~/.bob/`) — jamais versionné

Le poste développeur contient tout ce qui est **personnel et confidentiel** : les vrais tokens d'authentification, les préférences de communication, les modes personnels.

> ⚠️ Ces fichiers ne doivent **jamais** être ajoutés à Git, même dans un repo privé.

### 4.1 Tokens MCP réels

Fichier `~/.bob/mcp.json` (global, macOS/Linux) :

```json
{
  "mcpServers": {
    "jira": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@ibm/mcp-jira"],
      "env": {
        "JIRA_URL": "https://acme.atlassian.net",
        "JIRA_TOKEN": "votre-token-jira-personnel"
      }
    },
    "ibmi": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@ibm/mcp-ibmi"],
      "env": {
        "IBMI_HOST": "10.x.x.x",
        "IBMI_USER": "VOTRE_PROFIL",
        "IBMI_PASSWORD": "votre-mot-de-passe"
      }
    }
  }
}
```

> Windows : `%USERPROFILE%\.bob\mcp.json`

> 📚 Doc : [Edit Global MCP](https://bob.ibm.com/docs/ide/configuration/mcp/mcp-in-bob#edit-configuration-files) · [MCP security warning](https://bob.ibm.com/docs/ide/configuration/mcp/understanding-mcp#common-questions)

### 4.2 Préférences personnelles

Fichier `~/.bob/rules/preferences-perso.md` :

```markdown
# Préférences personnelles Bob

## Langue et communication
- Répondre toujours en français
- Réponses concises par défaut — développer uniquement si demandé
- Toujours donner le raisonnement avant la solution pour les sujets complexes

## Style de code
- Pour les exemples RPG : utiliser le FREE FORMAT ILE uniquement
- Nommer les exemples de variables avec le préfixe W_ conforme aux normes ACME
```

> 📚 Doc : [Global rules](https://bob.ibm.com/docs/ide/configuration/rules#rule-scopes)

### 4.3 Activer le workspace entreprise sur le poste

Pour que Bob charge les rules et skills du workspace entreprise partagé **sur tous les projets**, deux approches sont possibles :

**Option A — Copie locale (plus simple)**
```bash
# Cloner le repo entreprise
git clone https://github.com/acme/acme-bob-commons ~/.bob-commons

# Créer des liens symboliques vers les répertoires partagés
ln -s ~/.bob-commons/.bob/rules ~/.bob/rules-entreprise
ln -s ~/.bob-commons/.bob/skills ~/.bob/skills-entreprise
```

**Option B — Git pull régulier**
```bash
# Script à ajouter dans le profil shell (.zshrc ou .bashrc)
alias bob-update="cd ~/.bob-commons && git pull && echo 'Bob commons mis à jour'"
```

> **Note :** Bob charge les rules depuis `~/.bob/rules/` (global) et `.bob/rules/` (workspace). Pour les skills, il charge depuis `~/.bob/skills/` et `.bob/skills/`. La meilleure pratique est de cloner le repo entreprise et de mettre à jour régulièrement.

---

## Onboarding d'un nouveau développeur

### Checklist complète

```
□ 1. Installer Bob V2 depuis bob.ibm.com
□ 2. Installer le package Premium (IBM i Developer + IBM i Database)
□ 3. Cloner le repo entreprise :
      git clone https://github.com/acme/acme-bob-commons ~/.bob-commons
□ 4. Configurer les tokens MCP personnels dans ~/.bob/mcp.json
□ 5. Créer ~/.bob/rules/preferences-perso.md (langue, style)
□ 6. Cloner le repo du projet applicatif
□ 7. Ouvrir le projet dans Bob
□ 8. Vérifier que les modes personnalisés sont bien chargés :
      Paramètres Bob → onglet Modes → vérifier "IBM i Developer"
□ 9. Vérifier que les skills sont chargés :
      Paramètres Bob → onglet Skills → vérifier les skills partagés
□ 10. Vérifier que les MCP sont connectés :
       Paramètres Bob → onglet MCP → vérifier JIRA et IBM i
□ 11. Lire AGENTS.md à la racine du projet
□ 12. Faire une première session Ask pour valider le contexte :
       "Décris-moi la structure de ce projet et les normes de développement qui s'appliquent"
```

---

## Maintenance du workspace entreprise

### Cycle de vie des normes Bob

Les rules Bob doivent évoluer avec les normes ACME. Processus recommandé :

| Déclencheur | Action |
|---|---|
| Mise à jour des normes ACME | Mettre à jour `.bob/rules/normes-*.md` dans le repo entreprise |
| Ajout d'un nouveau skill partagé | Ajouter dans `.bob/skills/` + commit + PR |
| Ajout d'un nouveau MCP d'équipe | Ajouter dans `.bob/mcp.json` (sans token) + documenter dans README |
| Nouveau mode personnalisé validé | Ajouter dans `.bob/custom_modes.yaml` + commit + PR |
| Changement de convention majeur | Relancer `/init` sur tous les projets actifs |

### Gouvernance recommandée

- Le repo `acme-bob-commons` suit le même processus de PR que le code applicatif
- Toute modification des rules doit être testée sur au moins un projet avant merge
- Un développeur référent Bob est désigné pour chaque équipe — il valide les PR sur ce repo

---

## Pièges à éviter

| Piège | Conséquence | Solution |
|---|---|---|
| Mettre des tokens dans `.bob/mcp.json` versionné | Exposition des credentials dans Git | Utiliser des variables d'environnement — tokens uniquement dans `~/.bob/mcp.json` |
| Oublier le `.bobignore` | Bob accède aux fichiers de connexion et credentials | Copier `.bobignore.template` dans chaque projet dès la création |
| Ne pas lancer `/init` après un changement majeur | Bob travaille sur un contexte obsolète | Relancer `/init` après tout changement structurel ou onboarding |
| Rules trop longues et exhaustives | Bob ignore ou tronque les rules dépassant la fenêtre de contexte | Garder les rules concises — 1 fichier = 1 sujet, max 100 lignes par fichier |
| Tout mettre dans `~/.bob/rules/` | Les règles ne se propagent pas aux autres développeurs | Normes d'équipe → repo entreprise ; préférences perso → `~/.bob/` |
| Versionner `~/.bob/` | Exposition des credentials, conflits entre développeurs | `~/.bob/` est strictement personnel — jamais dans Git |
| Oublier de relancer `/init` à l'onboarding | Le nouveau développeur travaille sans contexte projet | Étape 12 de la checklist d'onboarding |

---

## Add-ons Bob à activer

| Add-on | Utilité pour ce UC |
|---|---|
| **IBM i Developer** (Premium) | Mode principal pour le développement IBM i |
| **IBM i Database** (Premium) | Mode base de données pour la migration DDS → DDL |

---

## MCP à utiliser

| MCP | Portée | Où configurer |
|---|---|---|
| **IBM i MCP** | Accès aux sources IBM i depuis Bob | `~/.bob/mcp.json` (token) + `.bob/mcp.json` projet (host) |
| **JIRA MCP** | Lecture des tickets, création d'issues | `~/.bob/mcp.json` (token) + `.bob/mcp.json` entreprise (URL) |
| **Confluence MCP** | Publication de la documentation produite | `~/.bob/mcp.json` (token) + `.bob/mcp.json` entreprise (URL) |

---

## Voir aussi

- **[`UC18-strategie-skills.md`](UC18-strategie-skills.md)** — suite directe de ce UC : transformer les fiches UCxx.md en skills Bob réutilisables placés dans le workspace entreprise créé ici.

---

## Références documentation officielle Bob

| Sujet | Lien |
|---|---|
| Custom rules — scopes et priorités | [bob.ibm.com/docs/ide/configuration/rules](https://bob.ibm.com/docs/ide/configuration/rules) |
| Custom rules — team standardization | [bob.ibm.com/docs/ide/configuration/rules#team-standardization](https://bob.ibm.com/docs/ide/configuration/rules#team-standardization) |
| Skills — locations et priorités | [bob.ibm.com/docs/ide/features/skills#skill-locations](https://bob.ibm.com/docs/ide/features/skills#skill-locations) |
| Skills — tips and best practices | [bob.ibm.com/docs/ide/features/skills#tips-and-best-practices](https://bob.ibm.com/docs/ide/features/skills#tips-and-best-practices) |
| Custom modes — edit config files | [bob.ibm.com/docs/ide/configuration/custom-modes#edit-configuration-files-manually](https://bob.ibm.com/docs/ide/configuration/custom-modes#edit-configuration-files-manually) |
| Custom modes — YAML format | [bob.ibm.com/docs/ide/configuration/custom-modes#yaml-format](https://bob.ibm.com/docs/ide/configuration/custom-modes#yaml-format) |
| MCP — configuration levels | [bob.ibm.com/docs/ide/configuration/mcp/mcp-in-bob#configuration-levels](https://bob.ibm.com/docs/ide/configuration/mcp/mcp-in-bob#configuration-levels) |
| MCP — security warning (tokens) | [bob.ibm.com/docs/ide/configuration/mcp/understanding-mcp#common-questions](https://bob.ibm.com/docs/ide/configuration/mcp/understanding-mcp#common-questions) |
| Slash commands — creating custom | [bob.ibm.com/docs/ide/features/slash-commands#creating-custom-commands](https://bob.ibm.com/docs/ide/features/slash-commands#creating-custom-commands) |
| /init — tutorial complet | [bob.ibm.com/docs/ide/getting-started/tutorials/start-a-project](https://bob.ibm.com/docs/ide/getting-started/tutorials/start-a-project) |
| AGENTS.md — maintain | [bob.ibm.com/docs/ide/getting-started/tutorials/start-a-project#maintain-the-agentsmd-files](https://bob.ibm.com/docs/ide/getting-started/tutorials/start-a-project#maintain-the-agentsmd-files) |
| .bobignore | [bob.ibm.com/docs/ide/configuration/bobignore](https://bob.ibm.com/docs/ide/configuration/bobignore) |
| Security guidelines | [bob.ibm.com/docs/ide/security/bob-security-guidance](https://bob.ibm.com/docs/ide/security/bob-security-guidance) |
| Standardize Bob's behavior (tutorial) | [bob.ibm.com/docs/ide/getting-started/tutorials/standardize-bobs-behavior](https://bob.ibm.com/docs/ide/getting-started/tutorials/standardize-bobs-behavior) |
| Enterprise — plan overview | [bob.ibm.com/docs/ide/enterprise/enterprise-index](https://bob.ibm.com/docs/ide/enterprise/enterprise-index) |
