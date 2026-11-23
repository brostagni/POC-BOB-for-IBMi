# FLGHT403_FRS409 — Journal de modernisation RPG (session complète)

> Généré le 29/10/2026 à 13:09 — Session de modernisation OPM RPG → ILE RPGLE free-format

---

## Membres concernés

| Rôle | Chemin |
|---|---|
| Source OPM original | `FLGHT403/QRPGSRC/FRS409.RPG` |
| Intermédiaire ILE fixe (temp) | `FLGHT403/QRPGLESRC/FRS409_T.RPGLE` *(supprimé)* |
| Objet compilé intermédiaire | `FLGHT403/FRS409_T (*PGM)` *(supprimé)* |
| Sortie finale free-format | `FLGHT403/QRPGLESRC/FRS409.RPGLE` |

---

## Étape 1 — Conversion OPM RPG → ILE fixe (`FRS409_T`)

Le workflow a d'abord converti le membre OPM `FLGHT403/QRPGSRC/FRS409.RPG` vers un membre ILE format fixe temporaire `FLGHT403/QRPGLESRC/FRS409_T.RPGLE` via la commande `CVTRPGSRC`.

### Résultat de compilation (ILE fixe)

| Paramètre | Valeur |
|---|---|
| Commande | `CRTBNDRPG` |
| Programme | `FLGHT403/FRS409_T` |
| Tentatives | 1 |
| Erreurs | 0 |
| Statut | ✅ Succès |

---

## Étape 2 — Détection I-specs (fichiers décrits par programme en entrée)

Résultat : **Aucune I-spec programme-décrite détectée.**
Le fichier `FRS409DF` est un fichier écran décrit externement. Étape ignorée.

---

## Étape 3 — Détection O-specs (fichiers imprimante décrits par programme)

Résultat : **Aucune O-spec programme-décrite détectée.**
Étape ignorée.

---

## Étape 4 — Conversion ILE fixe → ILE free-format (`FRS409.RPGLE`)

### Transformations appliquées

#### CTL-OPT (H-spec)
Aucune H-spec dans l'original. Ajout du CTL-OPT standard projet :
```rpgle
CTL-OPT DFTACTGRP(*NO) ACTGRP(*CALLER) OPTION(*NODEBUGIO:*SRCSTMT);
```

#### F-spec → DCL-F
```
Avant : FFRS409DFCF  E                    WORKSTN      KINFDS WSDS
Après : DCL-F FRS409DF WORKSTN INFDS(WSDS);
```

#### I-specs (constantes et DS) → DCL-C / DCL-DS
```
Avant :
  I    X'31'   C  @F01
  I    X'33'   C  @F03
  I    X'3C'   C  @F12
  I    X'F1'   C  @ENTER
  I    X'F3'   C  @HELP
  I    X'F6'   C  @PRINT
  I    '0'     C  @FALSE
  I    '1'     C  @TRUE
  IWSDS  DS
  I            369 369 KEY

Après :
  DCL-C @F01   X'31';
  DCL-C @F03   X'33';
  DCL-C @F12   X'3C';
  DCL-C @ENTER X'F1';
  DCL-C @HELP  X'F3';
  DCL-C @PRINT X'F6';
  DCL-C @FALSE '0';
  DCL-C @TRUE  '1';
  DCL-DS WSDS;
    KEY CHAR(1) POS(369);
  END-DS;
```

#### *ENTRY PLIST → DCL-PI
```
Avant :
  C  *ENTRY  PLIST
  C          PARM  ORDER#  90
  C          PARM  ENDYN    1

Après :
  DCL-PI *N;
    ORDER#  PACKED(9:0);
    ENDYN   CHAR(1);
  END-PI;
```

#### Déclaration implicite → DCL-S
```
Avant : @EXIT défini implicitement via MOVE @FALSE @EXIT 1
Après : DCL-S @EXIT CHAR(1);
```

#### Boucle DOUEQ → DOU
```
Avant : C  @EXIT  DOUEQ @TRUE
Après : DOU @EXIT = @TRUE;
```

#### CAS/ENDCS → IF/ELSEIF/ELSE/ENDIF
```
Avant :
  C  KEY  CASEQ @F03   @F03SR
  C  KEY  CASEQ @F12   @F12SR
  C        CAS         @ENTKY
  C        ENDCS

Après :
  IF KEY = @F03;
    EXSR @F03SR;
  ELSEIF KEY = @F12;
    EXSR @F12SR;
  ELSE;
    EXSR @ENTKY;
  ENDIF;
```

#### MOVE → affectation directe
```
Avant : C  MOVE *ON     *INLR
Après : *INLR = *ON;

Avant : C  MOVE ORDER#  ORDER
Après : ORDER = ORDER#;

Avant : C  MOVE 'N'     ENDYN
Après : ENDYN = 'N';

etc.
```

#### TAG supprimé
```
Avant : C  NEXT  TAG   (inutilisée — aucun GOTO ne la ciblait)
Après : (supprimé)
```

### Résultat de compilation (free-format final)

| Paramètre | Valeur |
|---|---|
| Commande | `CRTBNDRPG` |
| Programme | `FLGHT403/FRS409` |
| Source | `FLGHT403/QRPGLESRC/FRS409.RPGLE` |
| Tentatives | 1 |
| Erreurs | 0 |
| Statut | ✅ Succès |

---

## Étape 5 — Nettoyage

- Membre intermédiaire `FLGHT403/QRPGLESRC/FRS409_T.RPGLE` : **supprimé**
- Objet compilé intermédiaire `FLGHT403/FRS409_T (*PGM)` : **supprimé**

---

## Code source final : FLGHT403/QRPGLESRC/FRS409.RPGLE

```rpgle
**FREE
// FRS409 - Flight Reservation System - Display Panel
// Converted from fixed-format RPG to fully free-format RPGLE
CTL-OPT DFTACTGRP(*NO) ACTGRP(*CALLER) OPTION(*NODEBUGIO:*SRCSTMT);

// ---------------------------------------------------------
// File declarations
// ---------------------------------------------------------
DCL-F FRS409DF WORKSTN INFDS(WSDS);

// ---------------------------------------------------------
// Function key constants
// ---------------------------------------------------------
DCL-C @F01    X'31';
DCL-C @F03    X'33';
DCL-C @F12    X'3C';
DCL-C @ENTER  X'F1';
DCL-C @HELP   X'F3';
DCL-C @PRINT  X'F6';

// ---------------------------------------------------------
// Boolean constants
// ---------------------------------------------------------
DCL-C @FALSE  '0';
DCL-C @TRUE   '1';

// ---------------------------------------------------------
// INFDS data structure
// ---------------------------------------------------------
DCL-DS WSDS;
  KEY  CHAR(1) POS(369);
END-DS;

// ---------------------------------------------------------
// Program parameters
// ---------------------------------------------------------
DCL-PI *N;
  ORDER#  PACKED(9:0);
  ENDYN   CHAR(1);
END-PI;

// ---------------------------------------------------------
// Work variables
// ---------------------------------------------------------
DCL-S ORDER  PACKED(9:0);
DCL-S @EXIT  CHAR(1);

// ---------------------------------------------------------
// Main logic
// ---------------------------------------------------------
EXSR INZSR;

DOU @EXIT = @TRUE;

  WRITE FRSWIN;
  EXFMT FKEYS;

  IF KEY = @F03;
    EXSR @F03SR;
  ELSEIF KEY = @F12;
    EXSR @F12SR;
  ELSE;
    EXSR @ENTKY;
  ENDIF;

ENDDO;

*INLR = *ON;

BEGSR INZSR;
  ORDER = ORDER#;
  ENDYN = 'N';
  @EXIT = @FALSE;
ENDSR;

BEGSR @ENTKY;
  @EXIT = @TRUE;
ENDSR;

BEGSR @F03SR;
  ENDYN = 'Y';
  @EXIT = @TRUE;
ENDSR;

BEGSR @F12SR;
  ENDYN = 'Y';
  @EXIT = @TRUE;
ENDSR;
```

---

## Points d'attention pour la recette

1. **Tests fonctionnels** : vérifier F3, F12, Entrée, retour de `ENDYN` (`'Y'`/`'N'`), affichage de `ORDER`.
2. **Groupe d'activation** : confirmer que les programmes appelants sont compatibles avec `ACTGRP(*CALLER)`.
3. **Source original** : `FLGHT403/QRPGSRC/FRS409.RPG` conservé jusqu'à validation complète.
4. **Modernisation optionnelle future** : conversion des sous-routines en procédures, constantes nommées pour les octets AID.
