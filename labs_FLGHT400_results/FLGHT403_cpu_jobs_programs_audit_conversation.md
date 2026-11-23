# Conversation — CPU actifs & audit des programmes FLGHT400

---

## Question 1 : Jobs actifs avec le plus de CPU time

**Utilisateur :** Which active jobs have accumulated the most CPU time? For the top jobs, distinguish cumulative CPU time from their current elapsed CPU percentage.

**Bob :**

Requête exécutée :
```sql
SELECT
    JOB_NAME, JOB_TYPE, JOB_STATUS, AUTHORIZATION_NAME,
    CPU_TIME, ELAPSED_CPU_PERCENTAGE, ELAPSED_TIME,
    FUNCTION_TYPE, FUNCTION
FROM TABLE(QSYS2.ACTIVE_JOB_INFO(RESET_STATISTICS => 'NO'))
ORDER BY CPU_TIME DESC
FETCH FIRST 20 ROWS ONLY
```

Résultats (top 20) :

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

**Points clés :**
- `CPU_TIME` = CPU cumulé en ms depuis le démarrage du job.
- `ELAPSED_CPU_PERCENTAGE` = % CPU sur la dernière période de mesure = **0 pour tous** → système au repos au moment de la capture.
- Jobs `QWEBADMIN/ADMIN*` (JVM) et `QLWISVR/ADMIN*` dominent le CPU cumulé — typique des serveurs IWS/JVM.
- `Q1ACPDST` (QBRMS — Backup Recovery) en 2e position.
- `QZDASOINIT` lié à `ITZUSER` (session ODBC/DDM) : 11 985 ms.
- Pour des % CPU en temps réel, relancer avec `RESET_STATISTICS => 'YES'` puis réexécuter après quelques secondes.

---

## Question 2 : Programmes FLGHT400 non recompilés depuis 5 ans

**Utilisateur :** Which programs in the FLGHT400 library have not been recompiled in the last 5 years?

**Bob :**

Requête exécutée :
```sql
SELECT
    OBJNAME, OBJTYPE, OBJATTRIBUTE, OBJCREATED,
    LAST_USED_TIMESTAMP, OBJOWNER, OBJSIZE
FROM TABLE(QSYS2.OBJECT_STATISTICS('FLGHT400', '*PGM'))
WHERE OBJCREATED < CURRENT_DATE - 5 YEARS
ORDER BY OBJCREATED ASC
```

Résultat : **34/34 programmes** — 100% de la bibliothèque FLGHT400 non recompilés depuis plus de 5 ans.

### Lot 1997 — OPM RPG (cœur applicatif original, ~28 ans)

`RTSEED`, `BLDCST`, `RTRAND`, `GTOJCV`, `JTOGCV`, `BCUST`, `FRS406`, `FRS407`, `FRS409`, `FRS010`, `RTDOFW`, `FRS410`, `FRS009`, `FAXDLY`, `FRS411CL`, `FRS403`, `FRS000`, `FRS413`, `FRS404`, `FRS005`, `FRS022`, `FRS405`, `FRS023`, `FRS002`, `BFLGHT`

### Lot 2003-2004 — maintenance ultérieure

`AGENTDFU`, `FRS001CL`, `USECLEAR`, `USERESET`, `FRS001T`, `MSGPRGH`, `MSGPRG`, `FRS001`, `FRS001U`

**Observations :**
- 25 programmes OPM RPG de 1997 — candidats prioritaires à la modernisation vers RPGLE free-format.
- `FLGHT400` = bibliothèque de production/runtime figée.
- Tous actifs (dernière utilisation : 2026-10-28).
- `AGENTDFU` = objet DFU, pas un programme source.

---

## Question 3 : Mémorisation et rapport

**Utilisateur :** Fait un rapport MD de la session et copie le résultat sur /home/ITZUSER. Reprends toute cette discussion pour la mémoriser dans un MD dans le même répertoire. Préfixe FLGHT403_, nom explicite de session, sans date/heure dans les noms.

**Bob :** Deux fichiers créés dans `/home/ITZUSER` :
- `FLGHT403_cpu_jobs_programs_audit.md` — rapport de session structuré.
- `FLGHT403_cpu_jobs_programs_audit_conversation.md` — ce fichier, mémorisant l'intégralité de la conversation.
