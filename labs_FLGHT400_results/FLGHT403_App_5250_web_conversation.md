# Conversation — FLGHT403 App 5250 Web

---

## Message utilisateur

> Cette application FLGHT403 utilise des écrans 5250. Lis les sources DDS de
> FLGHT403/QDDSSRCD, en commençant par FRS000DF et FRS001DF, puis génère un
> prototype web en HTML, CSS et JavaScript sans framework qui émule fidèlement le
> terminal vert sur noir (champs de saisie, barre de touches de fonction).
> Déploie les fichiers dans l'IFS, dans /www/qiccqxml/htdocs/flght400/ : le
> serveur Apache QICCQXML existant les sert déjà. Ne démarre aucun autre serveur.
> Ne modifie aucun objet de la bibliothèque FLGHT403.

---

## Réponse Bob

### Analyse DDS

Lecture de 4 membres DDS dans `FLGHT403/QDDSSRCD` :

- **FRS000DF** — écran LOGON (24×80, DSPSIZ DS3)
  - Format `LOGONR` : champs `AGNTNM` (10A, B, ligne 6 col 40) et `PASSWD` (10A, B, ligne 8 col 40)
  - Touches : CA02, CA03, CF04, CF10, CA12
  - Ligne 23 : `F2=Refresh  F3=Exit  F4=Agent Prompt  F10=LOGON`

- **FRS001DF** — Create Order
  - Format `FRSADR` : deux colonnes (Flight Info col 2-34 / Ticket Order col 35-79)
  - Champs de saisie : `DOFMON/DOFDAY/DOFYR` (date vol), `CUSNAM` (25A), `FCITY/TCITY` (16A), `COSFIR/COSBUS/COSECO` (1A), `TCKETS` (2D)
  - Champs output (HI) : `AIRLIN`, `FLIGHT`, `DPTIME`, `ARTIME`, `PRICE`, `TAXDUE`, `TOTAL`
  - Touches : CA02, CA03, CF04-CF07, CF09, CF10, CA12

- **FRS002DF** — Modify Order (structure identique à FRS001DF + `ORDNUM` en output)

- **FRS003DF** — Display Order (tous champs en output `O`, pas de saisie, F3/F9/F12 seulement)

### Fichiers générés et déployés

**`/www/qiccqxml/htdocs/flght400/style.css`**
- Variables CSS : `--ch-w: 9.6px`, `--ch-h: 18px`, `--green: #33ff33`, `--blue: #5599ff`
- Grille 24×80 via `position: absolute` sur chaque ligne `.row[data-row="N"]`
- Classes DDS : `.grn-hi` (DSPATR HI), `.blu` (COLOR BLU), `.rev` (DSPATR RI), `.ul` (underline)
- Champs `<input>` : `border-bottom: 1px solid green`, focus avec `background: #001100`
- OIA en bas, barre Fn cliquable, bezel noir avec glow vert
- Scanlines CRT en `::after` repeating-linear-gradient

**`/www/qiccqxml/htdocs/flght400/index.html`**
- Navigation par 4 boutons (FRS000 / FRS001 / FRS002 / FRS003)
- `<div id="screen">` : zone 24×80 avec `tabindex="0"`
- OIA : gauche (statut MW), centre (nom écran), droite (position curseur)
- `<div id="fkey-bar">` peuplé dynamiquement par `frs.js`

**`/www/qiccqxml/htdocs/flght400/frs.js`**
- Architecture : objets `screens['FRS000']…['FRS003']` avec `cells[]` ou `cells()`
- Helpers : `txt(row,col,text,cls)`, `inp(row,col,id,len,password,value)`, `out(row,col,id,len,value,cls)`, `rev()`, `hline()`
- `buildBorderLines(rowStart, rowEnd)` : génère les séparateurs RI colonnes 2, 35, 79
- `renderScreen(name)` : vide l'écran, crée 24 `.row`, place les cellules, construit la barre Fn, focus premier champ
- `handleFkey(key)` : F3/F12 → FRS000, F10 → validation avec messages ligne 24, F2 → refresh
- Horloge 5 s pour `TIME`/`DATE` sur les écrans avec `cells` fonction
- Navigation clavier : F1–F12, Tab/Shift+Tab, Enter (champ suivant ou F10), Escape (F3)

### Vérification serveur

```
WRKACTJOB SBS(QHTTPSVR) => QICCQXML actif
  QZHBMAIN  SIGW
  QZSRLOG   SIGW  x2
  QZSRHTTP  SIGW  x2
```

Fichiers : `chmod 644`, répertoire `chmod 755`.  
`<Directory /www/qiccqxml/htdocs>` : `Order Allow,Deny` + `Allow From all` — le sous-répertoire `/flght400/` hérite sans configuration supplémentaire.

**URL finale** : `http://161.156.191.xxx:57700/flght400/`

---

## Message utilisateur

> 1) Fait un rapport MD la session et copie le résultat sur /home/ITZUSER.
> 2) Reprends toute cette discussion-conversation pour la mémoriser dans un MD dans le meme répertoire.
> 3) Présente toujours les noms avec un préfixe FLGHT403_
> 4) Utilise après le préfixe un nom explicite de session qui reprend ce qui a été fait, pour les 2 fichiers.
> 5) nouveau nommage : FLGHT403_<nom-session>.md et FLGHT403_<nom-session>_conversation.md.
> 6) AUCUNE DATE TIME dans les noms. 7) session = App_5250_web

---

## Réponse Bob

Deux fichiers créés dans `/home/ITZUSER/` :

| Fichier | Contenu |
|---------|---------|
| `FLGHT403_App_5250_web.md` | Rapport de session (objectif, sources, fichiers, fidélité 5250, comportement, contraintes, serveur, prochaines étapes) |
| `FLGHT403_App_5250_web_conversation.md` | Ce fichier — retranscription complète de la conversation |
