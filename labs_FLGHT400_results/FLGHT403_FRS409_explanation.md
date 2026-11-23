# FLGHT403_FRS409 — Explication du programme

**Source :** `FLGHT403/QRPGSRC(FRS409.RPG)`  
**Display file :** `FLGHT403/QDDSSRCD(FRS409DF.DSPF)`  
**Langage :** OPM RPG/400 (fixe, indicateur-driven)  
**Date d'analyse :** 2026-10-29

---

## Rôle fonctionnel

`FLGHT403_FRS409` est une **fenêtre de confirmation de modification de billet** (Ticket Confirmation Window).

Il est appelé après qu'une modification a été enregistrée dans le fichier des commandes (Orders). Il informe l'opérateur que le billet a bien été modifié et lui propose deux options :

- **ENTER** → retourner dans la maintenance de la commande pour effectuer d'autres modifications de vol  
- **F3** → quitter la maintenance de la commande (Ticket Order Maintenance)

---

## Interface utilisateur — `FLGHT403_FRS409DF`

La display file `FRS409DF.DSPF` définit une **fenêtre popup** (WINDOW) de 15 lignes × 29 colonnes, positionnée en ligne 5 colonne 2 de l'écran. Elle contient :

| Élément | Contenu |
|--------|---------|
| Titre | `TICKET CONFIRMATION WINDOW` |
| Champ affiché | `ORDER` — numéro de billet (9 chiffres, output-only) |
| Texte bleu | Message de confirmation + instructions |
| Touches actives | F3 = Exit, F12 = Cancel (défini dans le DDS) |

---

## Logique du programme — `FLGHT403_FRS409`

### Paramètres reçus à l'appel (`*ENTRY PLIST`)

| Paramètre | Longueur | Direction | Description |
|-----------|----------|-----------|-------------|
| `ORDER#`  | 9        | Entrée    | Numéro de billet à afficher dans la fenêtre |
| `ENDYN`   | 1        | Sortie    | Indicateur retour : `'Y'` = sortir, `'N'` = continuer |

### Flux d'exécution

```
Appel entrant
    │
    ▼
INZSR — Initialisation
    │  ORDER# → ORDER (copie pour affichage)
    │  ENDYN  ← 'N'  (par défaut : ne pas quitter)
    │  @EXIT  ← '0'  (boucle principale active)
    │
    ▼
DOUEQ @TRUE  ──── Boucle principale ────────────────────
    │                                                    │
    ├── WRITE FRSWIN    (affiche la fenêtre popup)       │
    ├── EXFMT FKEYS     (affiche + attend une touche)    │
    │                                                    │
    ├── KEY = F03 → @F03SR  ENDYN='Y', @EXIT='1' ───────┘
    ├── KEY = F12 → @F12SR  ENDYN='Y', @EXIT='1' ───────┘
    └── KEY = ENTER → @ENTKY  @EXIT='1' ────────────────┘
                                                         │
    ▼                                                    │
MOVE *ON → *INLR  (fin de programme) ◄──────────────────┘
```

### Subroutines

| Subroutine | Déclencheur | Effet |
|-----------|-------------|-------|
| `INZSR`   | Début de programme | Copie `ORDER#` → `ORDER`, met `ENDYN='N'`, `@EXIT='0'` |
| `@ENTKY`  | Touche ENTER | Met `@EXIT='1'` — retour à la maintenance |
| `@F03SR`  | F3 | Met `ENDYN='Y'` + `@EXIT='1'` — quitter |
| `@F12SR`  | F12 | Met `ENDYN='Y'` + `@EXIT='1'` — quitter (idem F3) |

### Valeur retournée `ENDYN`

| Valeur | Signification pour l'appelant |
|--------|-------------------------------|
| `'N'`  | L'opérateur veut continuer les modifications |
| `'Y'`  | L'opérateur veut quitter la maintenance |

---

## Résumé

`FLGHT403_FRS409` est un programme OPM RPG court, **purement interactif**, sans accès fichier direct. Son seul rôle est d'afficher une fenêtre popup de confirmation après la sauvegarde d'une modification de billet, puis de retourner à l'appelant une valeur `ENDYN` indiquant si l'opérateur souhaite continuer ou quitter.

---

*Généré par Bob — analyse de `FLGHT403/QRPGSRC(FRS409.RPG)` et `FLGHT403/QDDSSRCD(FRS409DF.DSPF)`*
