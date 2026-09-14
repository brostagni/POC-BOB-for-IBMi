# UC 18 — Stratégie Skills : industrialisation des use cases Bob

> **Catégorie :** Utilisation de Bob
>
> **Priorité dans le POC :** Phase 0 — après UC17 (workspace entreprise en place), avant les UC métier
>
> **Durée de mise en place initiale :** 1 à 2 heures par skill créé — rédaction du SKILL.md, test sur un composant pilote, versioning
>
> **Durée d'utilisation :** 0 — le skill s'active automatiquement, zéro effort développeur
>
> **Mode Bob recommandé :** Agent (pour créer et écrire les SKILL.md), n'importe quel mode (pour utiliser un skill)

---

## Objectif

Transformer les fiches UCxx.md en **skills Bob réutilisables** — des workflows automatisés que Bob active de lui-même quand le contexte correspond, sans que le développeur ait à copier-coller des prompts ou à connaître la séquence par cœur.

**Ce UC résout trois problèmes :**

1. **La friction d'utilisation** — sans skill, utiliser UC07 demande d'ouvrir la fiche, de lire la séquence, de copier chaque prompt un par un. Avec un skill, le développeur dit *"Optimise TRC0018"* et Bob orchestre tout.
2. **L'incohérence entre développeurs** — chaque développeur exécute les prompts légèrement différemment. Le skill garantit la même séquence, les mêmes règles, les mêmes livrables pour tous.
3. **La maintenance dispersée** — sans stratégie, les règles métier se dupliquent entre la fiche UCxx et le skill. Ce UC définit une architecture "source unique de vérité" qui élimine la double maintenance.

---

## Principe fondamental : source unique de vérité

Les fichiers `UCxx-*.md` restent la **source de vérité**. Ils contiennent les règles métier, les prompts, les check-lists, les conventions, les pièges. Le skill ne duplique pas ce contenu — il **pointe** vers le UCxx.md et ajoute le workflow d'orchestration par-dessus.

```
UC07-optimisation-code.md        ← source de vérité (ne jamais modifier pour le skill)
         ↑
         pointé par @UC07-optimisation-code.md
         ↓
.bob/skills/uc07-optimisation-rpg/SKILL.md   ← pilote : workflow + règles absolues + pointeur
```

**Conséquence directe :** si un UCxx.md évolue (nouveau prompt, nouveau piège détecté), le skill en bénéficie automatiquement lors de la prochaine activation — zéro double maintenance.

---

## Comment fonctionne un skill Bob

Quand Bob reçoit une demande, il analyse la `description` de chaque skill disponible pour décider lequel activer. Si la description correspond à la demande, Bob charge le `SKILL.md` en contexte et suit ses instructions.

**Points clés documentés :**
- Un skill se charge **une seule fois par conversation** — pas de doublons
- Bob active le skill **automatiquement** basé sur la description — pas besoin d'invocation explicite
- Les fichiers supports dans le dossier du skill sont accessibles à Bob une fois le skill activé
- Par défaut Bob demande confirmation avant d'activer un skill — désactivable dans Paramètres → Auto-Approve → Skills

> 📚 Doc : [How skills work](https://bob.ibm.com/docs/ide/features/skills#how-skills-work) · [Approving skills](https://bob.ibm.com/docs/ide/features/skills#approving-skills)

---

## Localisation des skills — 2 niveaux

| Niveau | Emplacement | Portée | Versionné |
|---|---|---|---|
| **Global** | `~/.bob/skills/` | Tous les projets du poste | Non — personnel |
| **Projet** | `.bob/skills/` dans le repo | Ce projet uniquement | Oui — dans Git |

**Règle de priorité :** si un skill avec le même nom existe aux deux niveaux, le skill projet prend le dessus sur le skill global.

Dans le contexte ACME, les skills sont placés dans le **repo entreprise** `acme-bob-commons/.bob/skills/` (voir UC17) pour être partagés avec toute l'équipe via Git.

> 📚 Doc : [Skill locations](https://bob.ibm.com/docs/ide/features/skills#skill-locations)

---

## Structure d'un skill

### Arborescence

```
.bob/skills/
├── uc07-optimisation-rpg/
│   ├── SKILL.md              ← obligatoire — workflow + frontmatter
│   └── (fichiers supports optionnels)
├── uc04-comprehension-rpg/
│   └── SKILL.md
└── uc14-dds-vers-ddl/
    └── SKILL.md
```

Le **nom du dossier** porte l'identité du skill. Le fichier s'appelle toujours `SKILL.md` — convention fixe Bob.

> 📚 Doc : [Creating a skill — Basic setup](https://bob.ibm.com/docs/ide/features/skills#basic-setup)

### Format du SKILL.md

```markdown
---
name: UC07 — Optimisation RPG IBM i
description: >
  Optimise un programme RPG IBM i sans modifier sa logique fonctionnelle.
  Applique la méthodologie UC7 ACME : renommage des variables cryptiques,
  remplacement des opcodes obsolètes (MOVE, SETON, GOTO…), extraction de
  subroutines en procédures Free RPG.
  À activer quand l'utilisateur demande d'optimiser, moderniser, refactorer
  ou améliorer la lisibilité d'un programme RPG.
---

## Source de vérité

Ce skill applique la méthodologie définie dans @UC07-optimisation-code.md.
Lire ce fichier en entier avant de démarrer — il contient les prompts P0 à P5,
les règles de mode, les conventions de nommage et la check-list de validation.

## Règles absolues

1. Toujours commencer par le Prompt 0 — jamais directement les renommages.
2. Rester en mode Ask pendant les Prompts 0 à 4 (analyse, diffs).
   Basculer en Agent uniquement pour P2-bis et P3-bis (compilation) et la sauvegarde.
3. Vérifier les prérequis avant de démarrer :
   - *-comprehension-*.md (UC4) disponible → OBLIGATOIRE
   - *-regles-*.md (UC5) disponible → OBLIGATOIRE
   - IBM i MCP actif
4. Nommer les livrables : {appArcad}-{fonction}-{composant}-{type}-{YYYYMMDD-HHmm}.md
5. Section "Questions ouvertes" obligatoire en fin de chaque livrable.
6. Mention ARCAD dans chaque diff : ⚠️ Réintégration ARCAD — à effectuer manuellement

## Démarrage

Demander à l'utilisateur :
- Quel programme RPG optimiser ? (nom + bibliothèque)
- Les fichiers *-comprehension-*.md et *-regles-*.md sont-ils dans le workspace ?

Si les prérequis sont absents, recommander une session UC4 (30 min) avant de continuer.
```

**Champs obligatoires du frontmatter :**
- `name` : nom affiché dans l'interface Bob
- `description` : texte que Bob analyse pour décider d'activer le skill — **c'est le déclencheur**

> ⚠️ Un skill sans `description` est ignoré par Bob.

> 📚 Doc : [SKILL.md format](https://bob.ibm.com/docs/ide/features/skills#skillmd-format) · [Writing effective skills — Clear descriptions](https://bob.ibm.com/docs/ide/features/skills#clear-descriptions)

### Fichiers supports optionnels

Un skill peut embarquer des fichiers complémentaires dans son dossier — checklist, templates, guide de sévérité, scripts. Bob y accède automatiquement une fois le skill activé.

```
.bob/skills/uc07-optimisation-rpg/
├── SKILL.md
├── checklist-validation.md     ← check-list de validation UC7
└── conventions-nommage.md      ← référence rapide des conventions ACME
```

> 📚 Doc : [Adding supporting files](https://bob.ibm.com/docs/ide/features/skills#adding-supporting-files)

---

## Choisir le bon mode pour un skill

Le mode Bob dans lequel le skill s'active conditionne les outils disponibles. Pour les UC IBM i, deux modes sont pertinents :

| Mode | Quand l'utiliser | Avantage |
|---|---|---|
| **`acme-poc`** (mode personnalisé) | Situation actuelle du POC — Premium Pack non confirmé | Contexte ACME embarqué (conventions, ARCAD, MCP) |
| **`IBM i Developer`** (Premium Pack) | Dès que le Premium Pack IBM i est actif | Connaissance RPG/ILE spécialisée — meilleure qualité des renommages, détection opcodes, risques MOVE/MOVEL |

**Ces deux modes sont complémentaires :** le mode IBM i Developer apporte la connaissance RPG native, le skill ajoute les règles ACME (conventions de nommage, contraintes ARCAD, section "Questions ouvertes") que le mode natif ne connaît pas.

```
Décision selon la situation :

Premium Pack IBM i non actif  →  Mode acme-poc  +  skill UCxx
Premium Pack IBM i actif      →  Mode IBM i Developer  +  skill UCxx
```

> 📚 Doc : [Custom modes](https://bob.ibm.com/docs/ide/configuration/custom-modes)

---

## Cycle de vie d'un skill

```
1. Formalisation du use case dans UCxx.md (source de vérité)
        ↓
2. Rédaction du SKILL.md (workflow + règles absolues + pointeur vers UCxx.md)
        ↓
3. Test sur un composant pilote (ex. TRC0018) — vérifier que Bob active bien le skill
        ↓
4. Versioning dans le repo entreprise acme-bob-commons
        ↓
5. Distribution via git pull (tous les développeurs récupèrent le skill)
        ↓
6. Amélioration continue — UCxx.md évolue, skill en bénéficie automatiquement
```

---

## Roadmap de création des skills ACME

| Priorité | UC | Slug du skill | Statut |
|----------|----|---------------|--------|
| 1 | UC07 — Optimisation code | `uc07-optimisation-rpg` | ✅ Créé |
| 2 | UC04 — Compréhension programme | `uc04-comprehension-rpg` | À créer |
| 3 | UC05 — Règles métier | `uc05-regles-metier` | À créer |
| 4 | UC14 — DDS vers DDL | `uc14-dds-vers-ddl` | À créer |
| 5 | UC06 — Documentation complète | `uc06-documentation` | À créer |
| 6 | UC08 — Restructuration code | `uc08-restructuration-rpg` | À créer |
| 7 | UC03 — SQL embarqué | `uc03-sql-embarque` | À créer |
| 8 | UC13 — Tests unitaires | `uc13-rpgunit` | À créer |

**Critère de priorité :** maturité du UCxx.md + fréquence d'utilisation prévue dans le POC.

---

## Rôle de chaque composant

| Composant | Rôle | Modifié par |
|---|---|---|
| `UCxx-*.md` | Source de vérité — règles, prompts, check-lists, pièges | Équipe méthode |
| `SKILL.md` | Pilote — workflow, règles absolues, pointeur vers UCxx.md | Équipe méthode |
| Mode `acme-poc` | Contexte ACME — conventions, ARCAD, langue | Équipe Bob |
| Mode `IBM i Developer` | Contexte RPG/ILE spécialisé (Premium Pack) | IBM (fourni) |

---

## Ce que le skill apporte par rapport aux prompts directs

| Approche prompts (avant) | Approche skill (après) |
|---|---|
| Trouver la fiche UCxx.md | Dire naturellement ce qu'on veut faire |
| Lire la séquence des prompts | Bob détecte et active le skill automatiquement |
| Copier-coller chaque prompt | Bob suit la séquence P0 → P1 → P2 → ... |
| Gérer les changements de mode manuellement | Bob gère les basculements Ask ↔ Agent |
| Respecter la convention de nommage | Bob applique la convention ACME |
| Vérifier manuellement les prérequis | Bob vérifie les prérequis et alerte si manquants |

---

## Pièges à éviter

| Piège | Conséquence | Solution |
|---|---|---|
| Description trop vague (`"skill UC7"`) | Bob n'active jamais le skill | Écrire une description qui décrit le **déclencheur** — quand, pour quoi, sur quoi |
| Dupliquer les règles UCxx dans le SKILL.md | Double maintenance — les deux fichiers divergent | Pointer vers `@UCxx.md` — ne jamais copier-coller les règles |
| SKILL.md trop long (> 200 lignes) | Bob tronque ou ignore les instructions en fin de fichier | Garder SKILL.md court — déplacer le détail dans des fichiers supports |
| Oublier la section "Démarrage" | Bob commence sans collecter les prérequis | Toujours inclure une section qui demande à l'utilisateur le contexte minimum |
| Skill dans `~/.bob/skills/` uniquement | Skill non partagé — chaque développeur doit le recréer | Skills d'équipe dans le repo entreprise `acme-bob-commons` |

---

## Vérifier les skills chargés

Pour confirmer qu'un skill est bien détecté par Bob :

1. Ouvrir les **Paramètres Bob**
2. Aller dans l'onglet **Skills**
3. Vérifier que les skills du projet et du global sont listés

> 📚 Doc : [Skills settings tab](https://bob.ibm.com/docs/ide/features/skills#skills-settings-tab)

---

## Références documentation officielle Bob

| Sujet | Lien |
|---|---|
| Skills — introduction et utilité | [bob.ibm.com/docs/ide/features/skills](https://bob.ibm.com/docs/ide/features/skills) |
| Skills — how skills work | [bob.ibm.com/docs/ide/features/skills#how-skills-work](https://bob.ibm.com/docs/ide/features/skills#how-skills-work) |
| Skills — creating a skill (basic setup) | [bob.ibm.com/docs/ide/features/skills#basic-setup](https://bob.ibm.com/docs/ide/features/skills#basic-setup) |
| Skills — SKILL.md format | [bob.ibm.com/docs/ide/features/skills#skillmd-format](https://bob.ibm.com/docs/ide/features/skills#skillmd-format) |
| Skills — adding supporting files | [bob.ibm.com/docs/ide/features/skills#adding-supporting-files](https://bob.ibm.com/docs/ide/features/skills#adding-supporting-files) |
| Skills — skill locations (projet vs global) | [bob.ibm.com/docs/ide/features/skills#skill-locations](https://bob.ibm.com/docs/ide/features/skills#skill-locations) |
| Skills — approving skills (auto-approve) | [bob.ibm.com/docs/ide/features/skills#approving-skills](https://bob.ibm.com/docs/ide/features/skills#approving-skills) |
| Skills — writing effective skills | [bob.ibm.com/docs/ide/features/skills#writing-effective-skills](https://bob.ibm.com/docs/ide/features/skills#writing-effective-skills) |
| Skills — tips and best practices | [bob.ibm.com/docs/ide/features/skills#tips-and-best-practices](https://bob.ibm.com/docs/ide/features/skills#tips-and-best-practices) |
| Skills — settings tab | [bob.ibm.com/docs/ide/features/skills#skills-settings-tab](https://bob.ibm.com/docs/ide/features/skills#skills-settings-tab) |
