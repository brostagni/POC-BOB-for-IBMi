# FLGHT403 — App React : Conversation complète

---

## Contexte

Construction d'une SPA React 18 + Vite 4 + Carbon g100 qui modernise l'écran 5250 « Create Order » du système Flight Reservation System (FLGHT403), déployée sur IBM i PASE.

---

## Échanges

---

**User**  
Given this screenshot of the 5250 flight order screen from the Application Flight403 in @FLGHT403, Build a single-page React 18 + Vite 4 app on IBM i (PASE) using @carbon/react ^1.x with the g100 dark theme that modernises the IBM i 5250 screen shown in the attached screenshot. Create the app in the IFS at $HOME/flight403-frontend-apps/screen-name/. Use the g100 dark theme. All fields should have a list of values to select from. The dev server must run in the background using nohup ... & and write output to /tmp/vite-dev.log. Pin the Vite dev server to port 3004 if available.

**Bob**  
Chargement du skill `ibmi-react-vite`. Vérification de l'environnement IBM i PASE.

- Node v22.23.4 détecté sur `/QOpenSys/pkgs/lib/nodejs22/bin/node` (nécessite `LIBPATH`)
- Port 3004 libre
- Projet scaffoldé manuellement (npm create vite non utilisable — TTY)
- `package.json` écrit avec toutes les dépendances
- `npm install --ignore-scripts` exécuté (4 min)
- `patch-esbuild.cjs` écrit et exécuté → esbuild patché pour os400
- `vite.config.js`, `src/styles/theme.scss`, `src/main.jsx`, `src/App.jsx`, `src/CreateOrder.jsx` écrits
- `build.sh`, `dev.sh`, `start-dev.sh` écrits

**Problème rencontré :** `node: No such file or directory` dans tous les scripts shell.  
**Cause :** `LIBPATH` non hérité par les processus enfants. Tous les shebangs `#!/usr/bin/env node` (vite, esbuild-wasm) échouent.  
**Fix :** Création de `.bin/node` wrapper qui exporte `LIBPATH` avant d'exec le vrai binaire. Tous les scripts préfixent `$PROJECT/.bin` dans PATH.

- Build de production : succès — 925 modules, 22s
- Dev server démarré : PID 4729, port 3004

---

**User**  
Managing the server — `# Restart /QOpenSys/pkgs/bin/bash /home/ITZUSER/.../start-dev.sh` comment je fais ?  
*(screenshot : `zsh: no such file or directory: /QOpenSys/pkgs/bin/bash`)*

**Bob**  
Tu lances la commande depuis ton terminal macOS local. `/QOpenSys/pkgs/bin/bash` n'existe que sur l'IBM i.  
→ Option A : demander à Bob directement.  
→ Option B : SSH depuis le terminal Mac.

---

**User**  
Lance le dev server create-order

**Bob**  
Ancien PID 4729 tué. Nouveau démarrage via `start-dev.sh`. Erreur intermédiaire `MODULE_NOT_FOUND` sur `ESBUILD_BINARY_PATH` (résolution hors répertoire projet) mais sans impact car le `cd` précède le lancement de Vite.  
Log confirmé : `VITE v4.5.14 ready in 2442 ms` — PID 4758, port 3004.  
URL : http://192.168.141.xxx:3004/

---

**User**  
nan *(screenshot : Firefox "The connection has timed out" sur 192.168.141.xxx:3004)*

**Bob**  
`192.168.141.xxx` est une IP VETH interne à TechZone — inaccessible depuis l'extérieur.  
Il faut un tunnel SSH :
```bash
ssh -L 3004:192.168.141.xxx:3004 -p 22 ITZUSER@161.156.191.xxx -N
```

---

**User**  
je ne sais pas, je ne le fait pas *(erreur `Permission denied (publickey)` sur port 22)*

**Bob**  
Port 22 bloqué. Recherche du `.env` et du script `tunnel.sh` sur l'IBM i.  
Trouvé : `/home/ITZUSER/builds/Bob_L4/ibmi-lab/scripts/tunnel.sh`  
`.env` contient : `IBMI_SSH_KEY`, `IBMI_SSH_PORT=22`, `IBMI_HOST=161.156.191.xxx`

Commande correcte depuis le Mac :
```bash
cd "<WORKSPACE>/ibmi-lab"
scripts/tunnel.sh open 3004
```

Note importante : la cible du tunnel doit être `localhost:3004` (pas `192.168.141.xxx:3004`).

---

**User**  
`port 3004: already open (pid 44776)` puis `bind: Address already in use`

**Bob**  
Le tunnel était déjà ouvert depuis le premier `scripts/tunnel.sh open 3004`. Le deuxième `ssh` a échoué car le port était pris — c'est normal. Aller directement sur http://localhost:3004/

---

**User**  
c'est bon *(screenshot : app Carbon g100 fonctionnelle, réservation #31842 créée)*

**Bob**  
App opérationnelle. Deux points relevés :
1. Label "F10" tronqué en "0" sur le bouton (largeur)
2. Notification reste ouverte après soumission

---

**User**  
je voudrais que tu me donne des instructions sur les changements à faire dans la doc pour monter la connection vers le server web.

**Bob**  
Documentation produite :
- Utiliser `scripts/tunnel.sh open 3004` depuis `ibmi-lab/`
- Cible : `localhost:3004` (pas l'IP interne)
- Port 22 accessible uniquement via clé PEM
- `bind: Address already in use` = tunnel déjà ouvert = OK

---

**User**  
1) Rapport MD de session. 2) Conversation mémorisée en MD. 3) Préfixe FLGHT403_. 4) Nom explicite de session. 5) Nommage : `FLGHT403_<nom-session>.md` et `FLGHT403_<nom-session>_conversation.md`. 6) Aucune date/time dans les noms. 7) session = App_React

**Bob**  
Création de `FLGHT403_App_React.md` et `FLGHT403_App_React_conversation.md` dans `/home/ITZUSER/`.

---

## Décisions techniques retenues

| Décision | Raison |
|---|---|
| Wrapper `.bin/node` avec `LIBPATH` | Seule solution fiable pour que tous les shebangs `#!/usr/bin/env node` fonctionnent sur os400 |
| `patch-esbuild.cjs` | esbuild n'a pas de binaire natif ppc64 BE — WASM fallback obligatoire |
| Scaffolding manuel (sans `npm create vite`) | `npm create vite` freeze en PASE sans TTY |
| Tunnel `localhost:3004` et non `192.168.141.xxx:3004` | IP VETH interne, non routable depuis Mac |
| `scripts/tunnel.sh open 3004` | Réutilise la clé PEM et la config `.env` de ibmi-lab |
