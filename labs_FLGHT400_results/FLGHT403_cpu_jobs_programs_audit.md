# Rapport de session — CPU actifs & audit des programmes FLGHT400

## 1. Jobs actifs avec le plus de CPU cumulé

Requête : `QSYS2.ACTIVE_JOB_INFO` — top 20 par `CPU_TIME` décroissant.

| Rang | Job | Type | Statut | User | CPU cumulé (ms) | % CPU elapsed |
|------|-----|------|--------|------|----------------:|:-------------:|
| 1 | `076982/QWEBADMIN/ADMIN1` | BCI | THDW | QWEBADMIN | 39 587 | 0 |
| 2 | `077020/QBRMS/Q1ACPDST` | PJ | DLYW | QBRMS | 35 792 | 0 |
| 3 | `076981/QLWISVR/ADMIN5` | BCI | THDW | QLWISVR | 31 202 | 0 |
| 4 | `076985/QLWISVR/ADMIN2` | BCI | THDW | QLWISVR | 29 795 | 0 |
| 5 | `077011/QTCP/QTMSSMTPD` | BCI | SELW | QTCP | 23 269 | 0 |
| 6 | `076984/QWEBADMIN/ADMIN3` | BCI | THDW | QWEBADMIN | 21 365 | 0 |
| 7 | `077129/QUSER_NC/QZDASOINIT` | PJ | TIMW | ITZUSER | 11 985 | 0 |
| 8 | `076983/QWEBADMIN/ADMIN4` | BCI | THDW | QWEBADMIN | 11 729 | 0 |
| 9 | `077323/QSECOFR_NC/QP0ZSPWT` | BCI | SELW | QSECOFR_NC | 10 895 | 0 |
| 10 | `076842/QSYS/QFILESYS1` | SYS | TIMW | QSYS | 8 624 | 0 |
| 11 | `077012/QTCP/QTSMTPCLTD` | BCI | SELW | QTCP | 7 745 | 0 |
| 12 | `076963/QSYS/QRMCCTCASD` | BCI | SELW | QSYS | 7 719 | 0 |
| 13 | `076979/QSRVAGT/QTOCRUNPRX` | BCH | THDW | QSRVAGT | 7 512 | 0 |
| 14 | `077280/QWEBADMIN/QNAVMNSRV` | BCI | THDW | QWEBADMIN | 7 268 | 0 |
| 15 | `077368/QSECOFR_NC/QP0ZSPWT` | BCI | DEQW | ITZUSER | 5 878 | 0 |
| 16 | `076977/QTCP/QTSMTPSRVD` | BCH | SELW | QTCP | 4 443 | 0 |
| 17 | `077291/QUSER/QSQSRVR` | PJ | CNDW | QWEBADMIN | 4 244 | 0 |
| 18 | `077701/QSECOFR_NC/QP0ZSPWT` | BCI | DEQA | ITZUSER | 3 716 | 0 |
| 19 | `076968/QSYS/QSLPSVR` | BCH | THDW | QSYS | 3 268 | 0 |
| 20 | `076944/QSYS/QSRVMON` | BCI | DEQW | QSECOFR | 3 235 | 0 |

### Observations

- **`CPU_TIME`** = CPU cumulé en millisecondes depuis le démarrage du job.
- **`ELAPSED_CPU_PERCENTAGE`** = % CPU sur la dernière période de mesure = **0 pour tous** → système au repos au moment de la capture.
- Les jobs `QWEBADMIN/ADMIN*` (JVM) et `QLWISVR/ADMIN*` dominent — comportement normal pour des serveurs IWS/JVM qui accumulent du CPU depuis leur démarrage.
- `Q1ACPDST` (QBRMS — Backup Recovery) en 2e position.
- `QZDASOINIT` lié à `ITZUSER` (session ODBC/DDM de développement) : 11 985 ms.

---

## 2. Programmes FLGHT400 non recompilés depuis 5 ans

Requête : `QSYS2.OBJECT_STATISTICS('FLGHT400', '*PGM')` — filtre `OBJCREATED < CURRENT_DATE - 5 YEARS`.

**Résultat : 34/34 programmes — 100% de la bibliothèque.**

### Lot 1997 — OPM RPG (cœur applicatif original)

| Programme | Attr | Date compilation |
|-----------|------|-----------------|
| `RTSEED` | CLP | 1997-07-31 |
| `BLDCST` | CLP | 1997-07-31 |
| `RTRAND` | RPG | 1997-07-31 |
| `GTOJCV` | RPG | 1997-08-01 |
| `JTOGCV` | RPG | 1997-08-01 |
| `BCUST` | RPG | 1997-08-02 |
| `FRS406` | RPG | 1997-08-06 |
| `FRS407` | RPG | 1997-08-07 |
| `FRS409` | RPG | 1997-08-07 |
| `FRS010` | RPG | 1997-08-07 |
| `RTDOFW` | RPG | 1997-08-08 |
| `FRS410` | RPG | 1997-08-08 |
| `FRS009` | RPG | 1997-08-08 |
| `FAXDLY` | CLP | 1997-08-09 |
| `FRS411CL` | CLP | 1997-08-11 |
| `FRS403` | RPG | 1997-08-11 |
| `FRS000` | RPG | 1997-08-11 |
| `FRS413` | RPG | 1997-08-12 |
| `FRS404` | RPG | 1997-08-12 |
| `FRS005` | RPG | 1997-08-12 |
| `FRS022` | RPG | 1997-08-12 |
| `FRS405` | RPG | 1997-08-13 |
| `FRS023` | RPG | 1997-08-13 |
| `FRS002` | RPG | 1997-08-15 |
| `BFLGHT` | RPG | 1997-08-15 |

### Lot 2003-2004 — maintenance ultérieure

| Programme | Attr | Date compilation |
|-----------|------|-----------------|
| `AGENTDFU` | DFU | 2003-04-01 |
| `FRS001CL` | CLP | 2003-12-01 |
| `USECLEAR` | CLP | 2003-12-02 |
| `USERESET` | CLP | 2003-12-02 |
| `FRS001T` | RPG | 2003-12-12 |
| `MSGPRGH` | CLP | 2003-12-17 |
| `MSGPRG` | CLP | 2003-12-19 |
| `FRS001` | RPG | 2004-01-07 |
| `FRS001U` | RPG | 2004-01-08 |

### Observations

- **25 programmes OPM RPG** datant de **1997** (~28 ans sans recompilation) — candidats prioritaires à la modernisation vers RPGLE free-format.
- `FLGHT400` joue le rôle de bibliothèque de **production/runtime figée** — cohérent avec l'architecture du projet.
- Tous ces programmes restent **actifs** (dernière utilisation : 2026-10-28).
- `AGENTDFU` est un objet DFU (Data File Utility), pas un programme source.
