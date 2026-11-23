# FLGHT403 — React Carbon App : Create Order

**Date :** 2026-10-29  
**Objectif :** Moderniser l'écran 5250 « Create Order » du système FRS en une Single Page Application React 18 + Vite 4 + Carbon g100 déployée sur IBM i PASE.

---

## Résultat final

- App live sur : **http://localhost:3004/** (via tunnel SSH)
- IFS : `/home/ITZUSER/flight403-frontend-apps/create-order/`
- Dev server : PID sauvegardé dans `/tmp/vite-create-order.pid`, log dans `/tmp/vite-dev.log`

---

## Stack technique

| Composant | Version |
|---|---|
| React | 18.2 |
| Vite | 4.4.5 |
| @carbon/react | ^1.x (Carbon 11) |
| @carbon/icons-react | ^11 |
| Sass | ^1.63 |
| Node.js | v22.23.4 (IBM i PASE) |
| npm | 10.9.9 |

Thème Carbon : **g100** (dark)

---

## Structure du projet

```
/home/ITZUSER/flight403-frontend-apps/create-order/
├── index.html
├── vite.config.js
├── package.json
├── patch-esbuild.cjs       ← patch os400 ppc64 BE — re-exécuter après npm install
├── build.sh                ← build de production
├── dev.sh                  ← dev server au premier plan
├── start-dev.sh            ← lancement non-bloquant (nohup)
├── .bin/node               ← wrapper LIBPATH — indispensable sur IBM i
└── src/
    ├── main.jsx            ← theme.scss importé EN PREMIER
    ├── App.jsx             ← <Theme theme="g100">
    ├── CreateOrder.jsx     ← composant principal
    └── styles/theme.scss   ← g100 + resets dark
```

---

## Champs modernisés (5250 → Carbon)

| Champ 5250 | Composant Carbon | Valeurs |
|---|---|---|
| Airline | `<Select>` | 8 compagnies |
| Flight | `<Select>` | 7 vols |
| Date of Flight | `<Select>` | 8 dates |
| From City | `<Select>` | 14 villes |
| Depart Time | `<Select>` | 12 horaires |
| To City | `<Select>` | 14 villes |
| Arrival Time | `<Select>` | 12 horaires |
| Customer | `<Select>` | 12 clients |
| Class of Service | `<RadioButtonGroup>` | First / Business / Economy |
| Number of Tickets | `<NumberInput>` | 1–9 |

Calcul de prix live : prix unitaire × tickets + taxe 8,75% = total.  
Bouton **F10 – Make Reservation** génère un numéro de commande aléatoire et affiche une `<InlineNotification>` de confirmation.

---

## Problèmes résolus

### 1. Node.js sur IBM i PASE — LIBPATH manquant
`/QOpenSys/pkgs/bin/node` est un shell script qui appelle le binaire réel sans `LIBPATH=/QOpenSys/pkgs/lib`. Résultat : `node: No such file or directory` pour tout processus enfant.

**Fix :** Créer `.bin/node` wrapper :
```sh
#!/bin/sh
LIBPATH=/QOpenSys/pkgs/lib
export LIBPATH
exec /QOpenSys/pkgs/lib/nodejs22/bin/node "$@"
```
Tous les scripts préfixent `$PROJECT/.bin` dans `PATH` pour que tous les shebangs `#!/usr/bin/env node` (vite, esbuild-wasm) résolvent ce wrapper.

### 2. esbuild — plateforme os400 non supportée
esbuild 0.18 ne fournit pas de binaire natif pour os400 ppc64 BE.

**Fix :** `patch-esbuild.cjs` remplace la constante compile-time dans `esbuild/lib/main.js` par le chemin vers `esbuild-wasm/bin/esbuild`.  
À re-exécuter après chaque `npm install` :
```bash
LIBPATH=/QOpenSys/pkgs/lib /QOpenSys/pkgs/lib/nodejs22/bin/node patch-esbuild.cjs
```

### 3. npm create vite non interactif
`npm create vite@4` freeze si le répertoire existe (prompt TTY sans stdin). Solution : créer le projet manuellement (écrire tous les fichiers directement).

### 4. Carbon theme g100 qui ne s'applique pas
`@use '@carbon/react/scss/theme'` **doit** être déclaré **avant** `@use '@carbon/react/scss/components'` dans le fichier SCSS. Ordre inversé = thème white par défaut silencieux.

---

## Accès depuis le navigateur (tunnel SSH)

Le port 3004 n'est pas accessible directement (IP interne VETH `192.168.141.xxx`).

### Démarrer le serveur (via Bob)
Demander : *"Lance le dev server create-order"*

### Ouvrir le tunnel (terminal Mac)
```bash
cd "<WORKSPACE>/ibmi-lab"
scripts/tunnel.sh open 3004
```

### Navigateur
```
http://localhost:3004/
```

### Fermer le tunnel
```bash
scripts/tunnel.sh close 3004
```

### Pièges à éviter

| Piège | Explication |
|---|---|
| `192.168.141.xxx:3004` timeout | IP VETH interne — inaccessible depuis Mac |
| `ssh -L ... 192.168.141.xxx:3004` | Utiliser `localhost:3004` comme cible distante |
| `bind: Address already in use` | Tunnel déjà ouvert — aller directement au navigateur |
| Port 22 refusé en direct | Fonctionne uniquement via `tunnel.sh` avec la clé PEM |

---

## Commandes utiles

```bash
# Vérifier que le serveur tourne
cat /tmp/vite-dev.log

# Relancer le serveur (via Bob ou directement)
/QOpenSys/pkgs/bin/bash /home/ITZUSER/flight403-frontend-apps/create-order/start-dev.sh

# Arrêter le serveur
kill $(cat /tmp/vite-create-order.pid)

# Build de production
/QOpenSys/pkgs/bin/bash /home/ITZUSER/flight403-frontend-apps/create-order/build.sh
```
