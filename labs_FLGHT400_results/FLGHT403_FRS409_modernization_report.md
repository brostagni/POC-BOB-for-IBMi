# FLGHT403_FRS409 — Rapport de modernisation RPG

## 1. Vue d'ensemble

| Élément | Détail |
|---|---|
| Membre source original | `FLGHT403/QRPGSRC/FRS409.RPG` |
| Membre intermédiaire (ILE fixe, temporaire) | `FLGHT403/QRPGLESRC/FRS409_T.RPGLE` (supprimé) |
| Objet intermédiaire compilé | `FLGHT403/FRS409_T (*PGM)` (supprimé) |
| Membre de sortie final | `FLGHT403/QRPGLESRC/FRS409.RPGLE` |
| Type RPG source | OPM RPG (format fixe, sans H-spec) |
| Commande de compilation finale | `CRTBNDRPG` |
| Tentatives de compilation | 1 (succès immédiat) |
| Statut final | ✅ Terminé avec succès |

`FLGHT403_FRS409` est un petit programme interactif WORKSTN. Il reçoit un numéro de commande et un indicateur Y/N en paramètres, affiche une fenêtre avec des touches de fonction, et boucle jusqu'à ce que l'utilisateur quitte via F3, F12 ou Entrée. Il a été converti d'OPM RPG format fixe vers RPGLE entièrement libre.

---

## 2. Externalisation des I-specs

**Non applicable.** Le programme utilise un fichier écran décrit externement (`FRS409DF`). Aucune I-spec décrite par programme n'était présente.

---

## 3. Externalisation des O-specs

**Non applicable.** Le seul fichier est un fichier `WORKSTN`. Aucune O-spec décrite par programme n'était présente.

---

## 4. Conversion format fixe → format libre

**Résultat :** Succès à la tentative 1, sans erreur de compilation.

### 4.1 H-Spec → `CTL-OPT`

L'original n'avait pas de H-spec. Une spécification de contrôle ILE standard a été ajoutée :

```rpgle
CTL-OPT DFTACTGRP(*NO) ACTGRP(*CALLER) OPTION(*NODEBUGIO:*SRCSTMT);
```

### 4.2 F-Spec → `DCL-F`

| Original | Converti |
|---|---|
| `FFRS409DF CF E WORKSTN INFDS(WSDS)` | `DCL-F FRS409DF WORKSTN INFDS(WSDS);` |

- Le comportement combined-file (`CF`) est le défaut pour `WORKSTN` en libre, aucun mot-clé supplémentaire nécessaire.
- Le mot-clé `INFDS(WSDS)` a été conservé.

### 4.3 D-Specs → `DCL-C` / `DCL-DS` / `DCL-S`

| Élément | Déclaration convertie |
|---|---|
| Constantes touches de fonction (`@F01`, `@F03`, `@F12`, `@ENTER`, `@HELP`, `@PRINT`) | `DCL-C name X'..';` (valeurs hex 31, 33, 3C, F1, F3, F6) |
| Constantes booléennes (`@FALSE`, `@TRUE`) | `DCL-C @FALSE '0';` et `DCL-C @TRUE '1';` |
| Structure `WSDS` avec `KEY` à la position 369 | `DCL-DS WSDS; KEY CHAR(1) POS(369); END-DS;` |
| `@EXIT` (implicite à l'origine) | `DCL-S @EXIT CHAR(1);` |

### 4.4 C-Specs → Logique libre

- **Paramètres d'entrée :** `*ENTRY PLIST` avec `PARM ORDER# 9 0` et `PARM ENDYN 1` devient un bloc `DCL-PI *N; ... END-PI;` avec `ORDER#` en `PACKED(9:0)` et `ENDYN` en `CHAR(1)`.
- **Boucle :** `DOUEQ @TRUE` devient `DOU @EXIT = @TRUE;` avec comparaison explicite.
- **Gestion des touches :** Le bloc `CAS`/`ENDCS` devient une structure `IF / ELSEIF / ELSE / ENDIF` :
  - `KEY = @F03` → `EXSR @F03SR`
  - `KEY = @F12` → `EXSR @F12SR`
  - Défaut → `EXSR @ENTKY`
- **Affectations :** Toutes les opérations `MOVE` deviennent des affectations directes :
  - `*INLR = *ON;`
  - `ORDER = ORDER#;`
  - `ENDYN = 'N';` et `ENDYN = 'Y';`
  - `@EXIT = @FALSE;` et `@EXIT = @TRUE;`
- **Opérations inchangées :** `EXSR`, `WRITE FRSWIN`, `EXFMT FKEYS` et `ENDDO` conservés avec terminaison libre.
- **Supprimé :** `TAG NEXT`, étiquette inutilisée (aucun `GOTO` ne la ciblait).

### 4.5 Sous-routines

Les quatre sous-routines (`INZSR`, `@ENTKY`, `@F03SR`, `@F12SR`) ont été conservées comme blocs `BEGSR`/`ENDSR`, valides en RPG entièrement libre.

---

## 5. Détails techniques

### Cas limites traités

1. **Déclaration implicite de `@EXIT`.** Dans la source fixe, `@EXIT` était défini implicitement par la colonne résultat de `MOVE @FALSE @EXIT 1`. Le libre exige une déclaration explicite : `DCL-S @EXIT CHAR(1);`.
2. **Suppression de `TAG NEXT`.** L'étiquette se trouvait entre `ENDCS` et `ENDDO` et n'était la cible d'aucun `GOTO` → suppression sans impact fonctionnel.
3. **Champ `ORDER`.** Provient du fichier écran décrit externement `FRS409DF` — pas de déclaration séparée nécessaire.

### Indicateurs

Aucun indicateur numéroté (`*INxx`) n'est utilisé. La détection des touches repose sur l'octet AID dans `WSDS.KEY` (position 369), conservé exactement. Seul `*INLR` est utilisé.

---

## 6. Bilan et recommandations

### Tableau de bord final

| Phase | Statut | Tentatives |
|---|---|---|
| Externalisation I-specs | N/A | — |
| Externalisation O-specs | N/A | — |
| Conversion format fixe → libre | ✅ Succès | 1 |

### Recommandations

1. **Tester fonctionnellement.** Exécuter le programme en mode interactif et vérifier que F3, F12 et Entrée atteignent bien la sous-routine correcte, que `ENDYN` retourne `'Y'`/`'N'` correctement, et que `ORDER` s'affiche correctement.
2. **Vérifier l'environnement appelant.** Confirmer que les programmes appelants fonctionnent avec `ACTGRP(*CALLER)`.
3. **Source originale à conserver.** Garder `FLGHT403/QRPGSRC/FRS409.RPG` jusqu'à la fin des tests fonctionnels. Le membre intermédiaire `FRS409_T` a été supprimé par le workflow.
4. **Modernisation complémentaire (optionnelle).** Les sous-routines pourraient être converties en procédures, ou des constantes nommées utilisées pour les octets AID. Non requis pour la conversion.
