# UC 12 — Utilisation des serveurs MCP ⚡ Enabler transversal

> **Catégorie :** Développement
> **Priorité dans le POC :** 2 — activer dès la Phase 0, en parallèle de UC 15
> **Durée estimée :** 2 à 4 heures (configuration + validation des MCP IBM i)
> **Mode Bob utilisé :** IBM i Developer (Agent), IBM i Database

---

## Objectif

Le MCP (Model Context Protocol) est le mécanisme par lequel Bob se connecte à des **systèmes externes** pour lire et écrire des données en temps réel : IBM i, Db2 for i, JIRA, Confluence, etc. Sans MCP configuré, Bob travaille uniquement sur le texte qu'on lui colle dans le chat. Avec MCP, Bob devient un **agent actif** capable d'interroger l'IBM i, d'exécuter du SQL, de lire des tickets JIRA et de publier de la documentation sur Confluence.

Ce use case est un **enabler transversal** : il ne produit pas de livrable visible par lui-même, mais il multiplie l'efficacité de tous les UC suivants.

> **Séquencement de l'installation :**
>
> | MCP | Quand l'installer | Bloquant si absent ? |
> |-----|------------------|----------------------|
> | IBM i MCP | **Phase 0 — immédiatement** | Oui — prérequis à tout travail IBM i avec Bob |
> | IBM i Database MCP | **Phase 0 — immédiatement** | Oui — prérequis aux UC SQL, ERD, Index Advisor |
> | JIRA MCP | **Dès que le token IT est disponible** — préparer la demande en Phase 0 | Non — enrichit les UC mais ne les bloque pas |
> | Confluence MCP | **Dès que le token IT est disponible** — préparer la demande en Phase 0 | Non — enrichit la doc mais ne la bloque pas |

---

## 1. Comprendre le MCP — Pourquoi, quoi, comment

### Qu'est-ce que le Model Context Protocol ?

MCP est un protocole standard (initié par Anthropic, adopté par IBM et l'industrie) qui permet à un LLM de communiquer avec des **serveurs de contexte externes** de façon structurée. Chaque serveur MCP expose un ensemble d'**outils** (tools) que Bob peut appeler durant une conversation.

```
Bob (LLM)  ←──── MCP Protocol ────→  Serveur MCP  ←───→  Système cible
                                      (IBM i MCP)          (IBM i LPAR)
                                      (JIRA MCP)           (JIRA Cloud)
                                      (Confluence MCP)     (Confluence)
```

### Pourquoi configurer MCP dès la Phase 0 ?

| Sans MCP | Avec MCP |
|----------|----------|
| Bob répond uniquement sur le code copié-collé dans le chat | Bob lit les membres sources directement sur l'IBM i |
| L'utilisateur doit copier/coller chaque résultat manuellement | Bob écrit la doc directement sur Confluence |
| Pas de corrélation entre tickets JIRA et code | Bob lit un ticket JIRA et génère le code correspondant |
| Analyse d'impact manuelle | Bob interroge `QSYS2` pour une analyse d'impact en temps réel |
| Compilation manuelle hors Bob | Bob déclenche et surveille les compilations IBM i |

---

## 2. Les serveurs MCP — Installation en deux temps

### Phase 0 — À installer immédiatement

### 2.1 IBM i MCP — Connexion à l'IBM i

**Rôle :** Permet à Bob d'accéder aux membres sources, objets, bibliothèques et à l'IFS de l'IBM i de test. C'est le MCP le plus fondamental pour tout travail IBM i.

**Ce que Bob peut faire avec :**
- Lire des membres sources (`QRPGSRC`, `QCLSRC`, `QDDSSRCD`, etc.)
- Naviguer dans les bibliothèques et l'Object Browser
- Écrire/modifier des membres sources (mode Agent)
- Exécuter des commandes CL (`CRTBNDRPG`, `DSPDBR`, `ADDLIBLE`, etc.)
- Lire et écrire dans l'IFS
- Déployer des fichiers depuis le workspace local

**Prérequis :**
- Connexion SSH active vers l'IBM i de test
- Profil utilisateur IBM i avec droits suffisants (`*USE`, `*CHANGE` sur les bibliothèques sources)
- Port SSH ouvert (par défaut : 22)

---

### 2.2 IBM i Database MCP — Db2 for i

**Rôle :** Permet à Bob d'exécuter des requêtes SQL sur Db2 for i, d'interroger les vues système (`QSYS2`), de lancer l'Index Advisor et de générer des ERD.

**Ce que Bob peut faire avec :**
- Exécuter des `SELECT`, `INSERT`, `UPDATE`, `ALTER TABLE` directement depuis le chat
- Interroger `QSYS2.SYSCOLUMNS`, `QSYS2.SYSKEYS`, `QSYS2.OBJECT_STATISTICS`, etc.
- Lancer le `/erd` slash command
- Analyser les performances SQL (Index Advisor, Plan Cache)
- Valider les conversions DDS → DDL en temps réel

**Prérequis :**
- Connexion ODBC/JDBC vers l'IBM i (port 8471 ou via SSH tunnel selon la config)
- Droits `*EXECUTE` sur les procédures système
- IBM i Access ODBC Driver installé sur le poste (ou via le package IBM i Developer)

---

### Dès réception des tokens IT — À préparer dès Phase 0, installer quand disponible

### 2.3 JIRA MCP — Gestion de projet

**Rôle :** Permet à Bob de lire et créer des tickets JIRA directement depuis le chat. Transforme Bob en un assistant capable de comprendre le backlog du projet.

**Ce que Bob peut faire avec :**
- Lire un ticket JIRA (`"Lis le ticket PROJ-123 et génère le code RPG correspondant"`)
- Créer des issues depuis les anomalies détectées dans le code
- Lister les tickets d'un sprint pour prioriser le travail
- Mettre à jour le statut d'un ticket après livraison

**Prérequis :**
- URL de l'instance JIRA du client (Cloud ou Server)
- Token d'API JIRA (Personal Access Token) — à demander à l'équipe IT de ACME
- Droits de lecture sur le projet POC, droits d'écriture si création de tickets nécessaire

> ⚠️ **Action à mener dès le kick-off** : contacter l'équipe IT de ACME pour obtenir les tokens API JIRA et Confluence. Ce sont souvent les délais les plus longs du POC — lancer la demande en Phase 0 même si l'installation est différée.

---

### 2.4 Confluence MCP — Documentation

**Rôle :** Permet à Bob de lire des pages Confluence existantes (normes de dev, architecture en place) et de publier automatiquement la documentation générée.

**Ce que Bob peut faire avec :**
- Lire une page Confluence pour contextualiser un prompt (`"Respecte les normes documentées sur [page]"`)
- Publier la documentation architecture générée en UC 6 directement sur Confluence
- Mettre à jour des pages existantes après modernisation
- Créer des pages depuis les rapports de modernisation générés en UC 3

**Prérequis :**
- URL de l'instance Confluence du client
- Token d'API Confluence (même démarche que JIRA)
- Identifiant de l'espace Confluence dédié au POC

---

## 3. Configuration du fichier `mcp.json`

> La configuration ci-dessous montre la structure complète avec les 4 serveurs. **En Phase 0, ne configurer que les blocs `ibmi` et `ibmi-db`**. Ajouter les blocs `jira` et `confluence` lorsque les tokens sont disponibles.

> ⚠️ **Prérequis poste développeur :** les serveurs MCP Bob tournent via `npx` — **Node.js 18+** doit être installé sur chaque poste. Vérifier avec `node --version`. Si absent, télécharger sur [nodejs.org](https://nodejs.org).

> ⚠️ **Noms des packages npm :** les noms de packages indiqués dans ce fichier (`@ibm/ibmi-mcp-server`, etc.) sont à **vérifier sur [bob.ibm.com](https://bob.ibm.com)** ou dans la documentation du Premium Package au moment de l'installation — ils peuvent évoluer entre versions. Ne pas copier-coller sans vérification.

### Emplacement du fichier

Le fichier de configuration MCP de Bob se place dans le dossier `.bob/` à la racine du workspace :

```
mon-workspace/
├── .bob/
│   └── mcp.json        ← fichier de configuration MCP
├── src/
└── ...
```

### Structure du fichier `mcp.json`

```json
{
  "mcpServers": {
    "ibmi": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@ibm/ibmi-mcp-server"],
      "env": {
        "IBMI_HOST": "votre-ibmi-hostname",
        "IBMI_USER": "VOTRE_PROFIL",
        "IBMI_PASSWORD": "votre-mot-de-passe",
        "IBMI_PORT": "22"
      }
    },
    "ibmi-db": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@ibm/ibmi-db-mcp-server"],
      "env": {
        "IBMI_HOST": "votre-ibmi-hostname",
        "IBMI_USER": "VOTRE_PROFIL",
        "IBMI_PASSWORD": "votre-mot-de-passe"
      }
    },
    "jira": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@atlassian/jira-mcp-server"],
      "env": {
        "JIRA_URL": "https://votre-instance.atlassian.net",
        "JIRA_EMAIL": "votre-email@acme.fr",
        "JIRA_API_TOKEN": "votre-token-api-jira"
      }
    },
    "confluence": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@atlassian/confluence-mcp-server"],
      "env": {
        "CONFLUENCE_URL": "https://votre-instance.atlassian.net",
        "CONFLUENCE_EMAIL": "votre-email@acme.fr",
        "CONFLUENCE_API_TOKEN": "votre-token-api-confluence"
      }
    }
  }
}
```

> ⚠️ **Sécurité absolue** : ne jamais committer `mcp.json` avec des credentials dans Git. Ajouter `.bob/mcp.json` au `.gitignore` du workspace. Utiliser des variables d'environnement système ou un fichier `.env` non commité pour les secrets.

### Fichier `.gitignore` à créer dans le workspace

```
.bob/mcp.json
.env
*.credentials
```

---

## 4. Validation de la configuration MCP

### Vérifier que les serveurs MCP sont actifs

Dans Bob, ouvrir la palette de commandes (`Cmd+Shift+P`) → rechercher `MCP` → **"Bob: Show MCP Servers"**. Chaque serveur doit afficher un statut **Connected** (vert).

### Tests de validation rapides

Une fois configuré, tester chaque MCP avec un prompt simple :

| MCP | Prompt de test | Résultat attendu |
|-----|---------------|-----------------|
| IBM i MCP | `"Liste les membres sources dans FLGHT400/QRPGSRC"` | Liste des membres RPG |
| IBM i Database | `/erd FLGHT400` | Diagramme ERD Mermaid généré |
| JIRA | `"Liste les tickets ouverts du sprint en cours"` | Liste des tickets JIRA |
| Confluence | `"Lis le contenu de la page [URL de la page des normes]"` | Contenu de la page affiché |

### Symptômes d'un MCP mal configuré

| Symptôme | Cause probable | Remède |
|----------|---------------|--------|
| Bob répond "Je n'ai pas accès à l'IBM i" | IBM i MCP non démarré ou credentials incorrects | Vérifier `mcp.json`, relancer Bob |
| Erreur `ECONNREFUSED` dans le terminal | Port SSH/ODBC fermé sur l'IBM i | Vérifier avec l'admin IBM i le port 22 et 8471 |
| Bob lit les sources mais ne peut pas écrire | Droits insuffisants sur le profil IBM i | Demander `*CHANGE` sur les bibliothèques sources |
| JIRA MCP connecté mais pas de tickets | Token API expiré ou mauvais projet | Régénérer le token, vérifier le nom du projet |
| Confluence MCP : erreur 403 | Droits insuffisants sur l'espace Confluence | Vérifier les permissions avec l'admin Confluence |

---

## 5. Utilisation dans les autres UC — Exemples de prompts croisés

### Avec UC 4-5-6 (Documentation)

```
"Lis le programme @FRS409 dans FLGHT400/QRPGSRC et publie 
 sa documentation technique sur la page Confluence [URL]."

"Génère le diagramme d'architecture de l'application FLGHT400 
 et crée une nouvelle page dans l'espace Confluence 'POC-IBM-i'."
```

### Avec UC 3-7-8 (Modernisation)

```
"Lis le ticket JIRA IBMI-42 (demande de modernisation de FRS021), 
 applique les changements demandés sur le membre source, et mets 
 à jour le ticket en statut 'In Review'."

"Après modernisation de FRS409, crée un ticket JIRA 
 'Valider FRS409 modernisé' dans le projet IBMI."
```

### Avec UC 14 (DDS → DDL)

```
"Lis tous les fichiers DDS dans FLGHT400/QDDSSRCD, génère les 
 CREATE TABLE SQL équivalents, exécute-les sur l'IBM i de test, 
 et publie le rapport de conversion sur Confluence."
```

### Avec UC 13 (Tests)

```
"Génère les cas de test RPGUnit pour FRS409, écris-les dans 
 FLGHT400/QTESTSRC, compile-les et retourne le résultat."
```

---

## 6. Pièges à éviter

| Piège | Conséquence | Comment l'éviter |
|-------|-------------|-----------------|
| Committer `mcp.json` avec les credentials | Exposition des mots de passe IBM i et tokens API dans Git | `.gitignore` obligatoire avant tout `git add` |
| Partager le même profil IBM i entre tous les membres de l'équipe | Impossible de tracer qui a fait quoi, risque de conflits | Un profil IBM i par développeur |
| Oublier de demander les tokens JIRA/Confluence en amont | Blocage de l'équipe sur les UC 5-6-12 | Demander les tokens dès le kick-off du POC |
| Utiliser un profil IBM i avec `*ALLOBJ` pour Bob | Risque de modification accidentelle d'objets de prod | Créer un profil dédié avec droits limités aux libs de dev/test |
| MCP IBM i et MCP Database avec des credentials différents | Incohérences de comportement entre les modes | Utiliser le même profil IBM i pour les deux |
| Laisser le MCP JIRA/Confluence connecté à l'espace de production | Bob pourrait créer/modifier des pages en prod | Utiliser un espace/projet dédié POC |

---

## 7. Check-list de validation UC 12

Avant de passer aux UC suivants, valider chaque point :

- [ ] Le fichier `mcp.json` est créé dans `.bob/` du workspace
- [ ] `.bob/mcp.json` est dans le `.gitignore`
- [ ] IBM i MCP : statut **Connected** dans Bob, test de listage de membres réussi
- [ ] IBM i Database MCP : `/erd` fonctionne et retourne un diagramme
- [ ] JIRA MCP : lecture d'un ticket de test réussie (si token disponible)
- [ ] Confluence MCP : lecture d'une page de test réussie (si token disponible)
- [ ] Un profil IBM i dédié au POC (non `*ALLOBJ`) est utilisé
- [ ] L'équipe IT de ACME a été contactée pour les tokens JIRA/Confluence

---

*Fiche UC 12 — Document évolutif à mettre à jour au fil du POC.*
