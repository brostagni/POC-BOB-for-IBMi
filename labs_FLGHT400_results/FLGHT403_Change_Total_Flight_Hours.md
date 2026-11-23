# FLGHT403 — Change Report: Add "Total Flight Hours" Field
**Date:** 2026-10-29  
**Author:** Bob (IBM i Developer AI)  
**Library:** FLGHT403  
**Status:** ✅ All four objects compiled successfully — severity 00

---

## 1. Objective

Add a **Total Flight Hours** field to the Flight Maintenance application, following the exact data flow of the existing Mileage field:

```
FLIGHTS.FLHRS (4P 0)  →  FLIGHTSZ.FHRS (RENAME)  →  FRS021 RPG  →  FRS021DF.SFLHRS (screen)
```

---

## 2. Objects Changed

| # | Object | Library | Source File | Type | Status |
|---|--------|---------|-------------|------|--------|
| 1 | `FLGHT403_FLIGHTS` | FLGHT403 | QDDSSRCF | Physical File (PF) | ✅ CHGPF |
| 2 | `FLGHT403_FLIGHTSZ` | FLGHT403 | QDDSSRCF | Logical File (LF) | ✅ DLTF + CRTLF |
| 3 | `FLGHT403_FRS021DF` | FLGHT403 | QDDSSRCD | Display File (DSPF) | ✅ CRTDSPF |
| 4 | `FLGHT403_FRS021` | FLGHT403 | QRPGSRC | OPM RPG Program | ✅ CRTRPGPGM |

---

## 3. Detailed Changes

### 3.1 — `FLGHT403_FLIGHTS.PF`

**File:** `/FLGHT403/QDDSSRCF/FLIGHTS.PF`  
**Command:** `QSYS/CHGPF FILE(FLGHT403/FLIGHTS) SRCFILE(FLGHT403/QDDSSRCF) SRCMBR(FLIGHTS)`  
**Result:** `CPC7303` — File FLIGHTS changed. 482332 records copied.

**Change:** Added field `FLHRS` as the last data field, immediately after `MILEAGE`, before the key spec.

```dds
                  MILEAGE        9B 0       ALWNULL
                                            COLHDG('MILEAGE')
+                 FLHRS          4P 0       ALWNULL
+                                           COLHDG('FLIGHT HOURS')
                K FLIGH00001
```

**Field properties:**

| Property | Value |
|----------|-------|
| Field name | `FLHRS` |
| Type | Packed decimal (`4P 0`) |
| Storage | 3 bytes |
| Decimals | 0 |
| Nullable | Yes (`ALWNULL`) |
| COLHDG | `'FLIGHT HOURS'` |
| Buffer position (expanded) | 229 |

---

### 3.2 — `FLGHT403_FLIGHTSZ.LF`

**File:** `/FLGHT403/QDDSSRCF/FLIGHTSZ.LF`  
**Command:** `QSYS/DLTF FILE(FLGHT403/FLIGHTSZ)` then `QSYS/CRTLF FILE(FLGHT403/FLIGHTSZ) SRCFILE(FLGHT403/QDDSSRCF) SRCMBR(FLIGHTSZ) AUT(*EXCLUDE)`  
**Result:** `CPC7301` — File FLIGHTSZ created in library FLGHT403.

**Change:** Added `FHRS` as the last field, immediately after `MILES`, before the key spec.

```dds
     A            MILES                     RENAME(MILEAGE)
+    A            FHRS                      RENAME(FLHRS)
     A          K FLGHTN
```

**Field properties:**

| Property | Value |
|----------|-------|
| Field name | `FHRS` |
| Renamed from | `FLHRS` (in FLIGHTS) |
| Type | Packed decimal (inherited, `4P 0`) |
| Buffer position (expanded) | 234 |
| Note | No data-type override needed; packed is readable directly by OPM RPG via `P` in I-spec |

---

### 3.3 — `FLGHT403_FRS021DF.DSPF`

**File:** `/FLGHT403/QDDSSRCD/FRS021DF.DSPF`  
**Command:** `QSYS/CRTDSPF FILE(FLGHT403/FRS021DF) SRCFILE(FLGHT403/QDDSSRCD) SRCMBR(FRS021DF) AUT(*EXCLUDE)`  
**Result:** `CPC7301` — File FRS021DF created in library FLGHT403.

**Change:** Added label constant and field `SFLHRS` on row 16, between Mileage (row 15) and Airline (row 17).

```dds
     A                                 15  2'Mileage. . . . . . . . . . . . .'   (existing)
     A                                      COLOR(BLU)
     A            SMILES         5D 0B 15 35TEXT('Flight Mileage')                (existing)
     A                                      CHECK(RZ)
     A  57                                  DSPATR(RI)
     A  57                                  DSPATR(PC)
+    A                                 16  2'Flight Hours . . . . . . . . .'
+    A                                      COLOR(BLU)
+    A            SFLHRS         4D 0B 16 35TEXT('Flight Hours')
+    A                                      CHECK(RZ)
+    A  61                                  DSPATR(RI)
+    A  61                                  DSPATR(PC)
     A                                 17  2'Airline. . . . . . .'                (existing)
```

**Field properties:**

| Property | Value |
|----------|-------|
| Field name | `SFLHRS` |
| Type | `4D 0` (4 digits, 0 decimals) |
| Usage | `B` (Both — input and output) |
| Row / Column | 16 / 35 |
| CHECK | `RZ` (right-justify, zero-fill) |
| Error indicator | 61 (`DSPATR(RI)` + `DSPATR(PC)`) |
| COLHDG | None (display files do not use COLHDG) |
| Color | Label in `COLOR(BLU)` |
| Buffer position | 74 |

---

### 3.4 — `FLGHT403_FRS021.RPG`

**File:** `/FLGHT403/QRPGSRC/FRS021.RPG`  
**Command:** `QSYS/CRTRPGPGM PGM(FLGHT403/FRS021) SRCFILE(FLGHT403/QRPGSRC) SRCMBR(FRS021) AUT(*EXCLUDE)`  
**Result:** `QRG0003` — Program FRS021 placed in FLGHT403. **Highest severity: 00.**

Four sub-changes applied in three separate diffs:

#### 3.4.1 — F-spec: record length 233 → 236

```rpg
- FFLIGHTSZUF  F     233 09AI     1 DISK                      A
+ FFLIGHTSZUF  F     236 09AI     1 DISK                      A
```

`FHRS` is `4P 0` = 3 bytes. 233 + 3 = 236.

#### 3.4.2 — I-spec: add `FHRS` at positions 234–236, data-type `P`

```rpg
  I                                    B 230 2330FMILES   ← binary (existing)
+ I                                    P 234 2360FHRS     ← packed (new)
```

> Key distinction: `FMILES` uses `B` (binary); `FHRS` uses `P` (packed) in column 43.

#### 3.4.3 — C-specs: validation + ADD/UPDATE moves

**Validation** (added after `SMILES` check, using indicator 61):

```rpg
+ C           SFLHRS    IFEQ *ZEROS                      * RUN INTERACTIVE
+ C                     SETON                       6161 * SET ERR INDICATOR
+ C                     MOVE 'FRS0025' MSGID             * SET ERR MESSAGE
+ C                     EXSR @SNDMS                      * SND ERR MESSAGE
+ C                     GOTO ERREND
+ C                     END
```

**ADD path** (before `EXCPTADDFLT`):

```rpg
  C                     Z-ADDSMILES    FMILES            * GET CMD LINE   (existing)
+ C                     Z-ADDSFLHRS    FHRS              * GET CMD LINE
  C                     EXCPTADDFLT
```

**UPDATE path** (before `EXCPTUPDFLT`):

```rpg
  C                     Z-ADDSMILES    FMILES            * GET CMD LINE   (existing)
+ C                     Z-ADDSFLHRS    FHRS              * GET CMD LINE
  C                     EXCPTUPDFLT
```

#### 3.4.4 — O-specs: add `FHRS` to ADDFLT and UPDFLT

```rpg
  O                         FMILES   233B   ← binary, end pos 233 (existing)
+ O                         FHRS     236P   ← packed, end pos 236 (new)
```

Applied identically to both `ADDFLT` and `UPDFLT` output exception records.

| Property | FMILES (model) | FHRS (new) |
|----------|----------------|------------|
| Field name cols 32–37 | `FMILES` (6 chars) | `FHRS  ` (4 + 2 blanks) |
| End position cols 38–43 | `   233` | `   236` |
| Type col 44 | `B` (binary) | `P` (packed) |

---

## 4. Build Order Executed

| Step | Command | Result |
|------|---------|--------|
| 1 | `QSYS/CHGPF FILE(FLGHT403/FLIGHTS) SRCFILE(FLGHT403/QDDSSRCF) SRCMBR(FLIGHTS)` | ✅ CPC7303 |
| 2 | `QSYS/DLTF FILE(FLGHT403/FLIGHTSZ)` | ✅ CPC2191 |
| 2b | `QSYS/CRTLF FILE(FLGHT403/FLIGHTSZ) SRCFILE(FLGHT403/QDDSSRCF) SRCMBR(FLIGHTSZ) AUT(*EXCLUDE)` | ✅ CPC7301 |
| 3 | `QSYS/CRTDSPF FILE(FLGHT403/FRS021DF) SRCFILE(FLGHT403/QDDSSRCD) SRCMBR(FRS021DF) AUT(*EXCLUDE)` | ✅ CPC7301 |
| 4 | `QSYS/CRTRPGPGM PGM(FLGHT403/FRS021) SRCFILE(FLGHT403/QRPGSRC) SRCMBR(FRS021) AUT(*EXCLUDE)` | ✅ QRG0003 sev 00 |

---

## 5. Validation Results (Étape 3g)

| Check | Result |
|-------|--------|
| `FLGHT403_FLIGHTS.FLHRS` exists (`DECIMAL 4,0`, nullable, text `FLIGHT HOURS`) | ✅ |
| `FLGHT403_FLIGHTSZ.FHRS` available (`DECIMAL 4,0`, buffer pos 234) | ✅ |
| `FLGHT403_FRS021DF.SFLHRS` at row 16 col 35, immediately after `SMILES` row 15 | ✅ |
| `FLGHT403_FRS021DF` contains no `COLHDG` keyword | ✅ |
| `FLGHT403_FRS021` compiled — severity 00, created 2026-10-29 13:30:59 | ✅ |

### End-to-end field mapping (one line)

> **`FLGHT403_FLIGHTS.FLHRS`** (4P 0) → `RENAME` → **`FLGHT403_FLIGHTSZ.FHRS`** → I-spec `P 234-236` → **`FLGHT403_FRS021.FHRS`** → `Z-ADD` ↔ **`FLGHT403_FRS021DF.SFLHRS`** (4D 0B, row 16, ind 61)

---

## 6. Follow-up Work (Out of Scope — Flagged Only)

`DSPDBR FILE(FLGHT403/FLIGHTS)` shows 2 dependent logical files:

| Object | Type | Action Required |
|--------|------|----------------|
| `FLGHT403_FLIGHTSZ` | LF | ✅ Already rebuilt in this change |
| `FLGHT403_FLIGHTSL` | LF | ⚠️ Logical file over FLIGHTS — verify access path; recompile if needed |

Programs to review and recompile before production deployment:

| Program | Reason |
|---------|--------|
| `FLGHT403_BFLGHT` | Populates FLIGHTS with test data — must include `FLHRS` write |
| `FLGHT403_FRS413` | Called by `FRS021/@F04SR`; receives `SMILES` parm; may read FLIGHTSZ directly |
| Any program opening FLIGHTSZ | OPM RPG programs do not register DSPDBR dependencies — use `DSPPGMREF` or Understand to enumerate |

> **Command to identify all programs referencing FLIGHTS or FLIGHTSZ:**  
> `QSYS/DSPPGMREF PGM(FLGHT403/*ALL) OUTPUT(*OUTFILE) OUTFILE(QTEMP/PGMREFS)`  
> Then query for `FILENAME IN ('FLIGHTS', 'FLIGHTSZ')`.
