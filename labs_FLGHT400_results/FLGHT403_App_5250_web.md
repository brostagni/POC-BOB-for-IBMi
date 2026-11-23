# Rapport de session — FLGHT403 App 5250 Web

## Objectif

Créer un prototype web HTML/CSS/JavaScript (sans framework) émulant fidèlement
les écrans 5250 (terminal vert sur noir) de l'application FLGHT403 Flights Reservation System,
et le déployer sur le serveur Apache QICCQXML existant.

---

## Sources DDS analysées

| Membre | Fichier source | Écran émulé |
|--------|----------------|-------------|
| `FRS000DF.DSPF` | `FLGHT403/QDDSSRCD` | LOGON — saisie agent + mot de passe |
| `FRS001DF.DSPF` | `FLGHT403/QDDSSRCD` | Create Order — réservation vol |
| `FRS002DF.DSPF` | `FLGHT403/QDDSSRCD` | Modify Order — modification réservation |
| `FRS003DF.DSPF` | `FLGHT403/QDDSSRCD` | Display Order — consultation en lecture seule |

---

## Fichiers déployés

Répertoire cible : `/www/qiccqxml/htdocs/flght400/`

| Fichier | Description |
|---------|-------------|
| `index.html` | Structure HTML, barre de navigation entre les 4 écrans |
| `style.css`  | Terminal vert sur noir, grille 24×80, scanlines CRT, attributs DDS |
| `frs.js`     | Moteur de rendu 5250, définition des 4 écrans, navigation clavier |

**URL d'accès** :
```
http://161.156.191.xxx:57700/flght400/
```

---

## Fidélité 5250 reproduite

- Grille **24 lignes × 80 colonnes** en `Courier New 14px`
- **Couleurs DDS** : `COLOR(BLU)` → bleu, `DSPATR(HI)` → vert brillant, `DSPATR(RI)` → fond inverse
- **Bordures verticales** reverse-image (colonnes 2, 35, 79) identiques au DDS
- **TIME / DATE / SYSNAME** en ligne 1 (mis à jour toutes les 5 s)
- Champs `B` (both) → `<input>` avec majuscules forcées, focus, bordure verte
- Champs `O` (output) → `<span>` non éditable, vert brillant
- Ligne 24 → messages d'erreur en rouge (style `CPFxxxx / FRSxxxx (C G D F)`)
- **OIA** (Operator Information Area) avec nom d'écran et position curseur
- **Barre de touches Fn** cliquable reproduisant exactement les touches DDS de chaque écran
- Raccourcis clavier : F1–F12 natifs, Tab / Shift+Tab, Enter, Escape (= F3)
- Scanlines CRT en overlay CSS

---

## Comportement interactif

| Action | Résultat |
|--------|----------|
| F10 sur FRS000 (Agent Name vide) | Message d'erreur ligne 24, champ en rouge |
| F10 sur FRS000 (champs remplis)  | Navigation vers FRS001 |
| F10 sur FRS001 (Customer vide)   | Message d'erreur ligne 24 |
| F10 sur FRS001 (valide)          | Confirmation de réservation |
| F3 / F12 / Escape               | Retour à FRS000 (LOGON) |
| Tab / Shift+Tab                 | Navigation entre champs de saisie |
| F2                              | Rechargement de l'écran courant |

---

## Contraintes respectées

- **Aucun objet FLGHT403 modifié** (lecture seule des sources DDS)
- **Aucun nouveau serveur démarré** — utilisation exclusive du serveur Apache QICCQXML existant (port 57700)
- **Aucun framework JavaScript** — HTML/CSS/JS vanilla uniquement
- Permissions IFS : `644` fichiers, `755` répertoire, héritage `Allow From all` du `<Directory /www/qiccqxml/htdocs>`

---

## Serveur Apache QICCQXML

- **État** : actif (confirmé via `WRKACTJOB SBS(QHTTPSVR)`)
- **Port** : 57700
- **DocumentRoot** : `/www/qiccqxml/htdocs`
- **Jobs actifs** : `QZHBMAIN`, `QZSRLOG` ×2, `QZSRHTTP` ×2

---

## Prochaines étapes suggérées

1. Ajouter les écrans FRS004–FRS024 (listes subfiles, menus, rapports)
2. Connecter les champs à une API REST IWS pour des données réelles
3. Ajouter un mode "demo guided" avec scénario pas-à-pas
