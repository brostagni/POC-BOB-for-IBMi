# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## IBM i Project Overview
- Workspace type: QSYS library file system (member scheme)
- Current library: FLGHT403
- Library list: FLGHT403, QGPL, QTEMP, RPGUNIT, QDEVTOOLS, UNDERSTAND
- Application: Flight Reservation System (FRS) - a demo/training application
- Languages found: OPM RPG, ILE RPGLE (free-format), CLP, DDS (PF/LF/DSPF), MNUDDS/MNUCMD, CMD

## Source Physical Files
```
FLGHT403.LIB
  QRPGSRC.SRCPF  (37 members) - OPM RPG (original) + some modernized members stored here
    RPG x35, *NONE x2 (FRS022.RPG and FRS009.RPG use **FREE despite .RPG extension)
  QRPGLESRC.SRCPF (14 members) - ILE RPGLE modernized versions
    RPGLE x11, TXT x1 (FRS401DOC.TXT), *NONE x2 (FRS021. and FRS021DOC. have no extension)
  QDDSSRCF.SRCPF  (24 members) - PF x10, LF x14
  QDDSSRCD.SRCPF  (28 members) - DSPF x28
  QCLPSRC.SRCPF   (10 members) - CLP x10
  QCMDSRC.SRCPF   (2 members)  - CMD x2
  QMNUSRC.SRCPF   (6 members)  - MNUDDS x3, MNUCMD x3
  EVFTEMPF01.SRCPF (1 member)  - RDi/Bob error feedback temporary file (ignore)
  QBOBSRC.SRCPF   - Bob AI context (this file)
```

## Critical Cross-Library Dependency: FLGHT400
**FLGHT400 is a separate runtime library that must be on the library list.**
Every program hardcodes `FRSMSGF FLGHT400` as the message file (20-char literal in RPG I-specs
and DCL-S initializers). Removing FLGHT400 from *LIBL at runtime will cause message API failures.

Additional FLGHT400 objects used at runtime:
- `FRSMSGF` (*MSGF) - message file used by all FRS programs for error/info messages
- `CUSTOMERS` (*FILE) - `BLDCST.CLP` and `FRS001CL.CLP` use OVRDBF to redirect CUSTOMER to FLGHT400/CUSTOMERS
- `FRS411DF` (*FILE) - `FRS411CL.CLP` uses `DCLF FLGHT400/FRS411DF` directly
- `MSGPANEL` (*FILE) - `MSGPRGH.CLP` uses `DCLF FILE(FLGHT400/MSGPANEL)` (hardcoded, not overridden)
- `FLDS` (*DTAARA) - `USECLEAR.CLP` and `USERESET.CLP` create/update `FLGHT400/FLDS`
- Menus `FRSMANT`, `FRSREPT` run via `GO FLGHT400/FRSMANT` from FRS000 (main logon)

## ILE Architecture
- No service programs (*SRVPGM) or binding directories (*BNDDIR) in FLGHT403
- No /COPY or /INCLUDE copybooks anywhere in the library
- ILE programs compiled with mixed activation groups: *CALLER (3), *NEW (5), *DFTACTGRP (2), QILE (1)
- All modernized RPGLE programs use `CTL-OPT DFTACTGRP(*NO)` - never use DFTACTGRP(*YES)
- 2 ILE programs compiled with `*DFTACTGRP` (legacy OPM-style ILE) - avoid replicating this pattern

## Build and Compile Commands
Documentation members (`FRS021DOC.`, `FRS401DOC.TXT`) specify compile targets as `FLGHT400/xxx`,
not `FLGHT403/xxx`. Compile into FLGHT403 during development; deploy to FLGHT400 for production.

Typical CRTBNDRPG for modernized programs:
```
CRTBNDRPG PGM(FLGHT403/FRS0xx)
          SRCFILE(FLGHT403/QRPGLESRC)
          SRCMBR(FRS0xx)
          DFTACTGRP(*NO)
          ACTGRP(*NEW or *CALLER - see source CTL-OPT)
          DBGVIEW(*SOURCE)
```

FRS022.RPG (in QRPGSRC, .RPG extension) uses **FREE and embedded SQL -
must be compiled with `CRTSQLRPGI` not `CRTBNDRPG`. Same for FRS009.RPG (check source).

## Database Objects (Non-Obvious)
- PF fields use auto-truncated 10-char system names + full ALIAS() for SQL access
  (e.g., `ORDER00001` system name, `ALIAS(ORDER_NUMBER)` SQL name)
- All PF fields have `ALWNULL` - no NOT NULL constraints anywhere
- Variable-length character fields (`VARLEN`) with `CCSID(37)` throughout FLIGHTS and ORDERS PFs
- Binary (B) numeric fields in PFs (e.g., `9B 0`) - LFs provide zoned (S) equivalents for OPM RPG
  - `FLIGHTSZ.LF` over `FLIGHTS.PF` - converts binary fields to zoned for OPM programs
  - `ORDERSZ.LF` over `ORDERS.PF` - same pattern
  - `CUSTOMRZ.LF` over `CUSTOMERS.PF` - same pattern
- Join logical files: `ORDNAME.LF` (ORDERS+CUSTOMERS by order#), `ORDDATE.LF` (by date),
  `CUSTORD.LF` (by customer name)
- No application-level journaling on FLGHT403 files (only system catalog files are journaled)

## Naming Conventions
- OPM programs: `FRS001`-`FRS024` in QRPGSRC (original RPG/400)
- ILE modernized: same base name in QRPGLESRC with .RPGLE extension
- Variants: suffix B = old/no-reposition version (e.g., FRS402B, FRS404B), suffix X/T/U/U2 = 2004 era variants
- X2 prefix = fully modernized with SQL (e.g., `X2FRS024.RPGLE`)
- XX prefix = intermediate ILE conversion (e.g., `XXFRS024.RPGLE`)
- Build programs: `B` prefix (BFLGHT.RPG, BCUST.RPG) - populate master files with test data
- Utility programs: RT prefix (RTRAND, RTDOFW, JTOGCV, GTOJCV) - standalone date/random utilities
- Display file member names: `FRS0xxDF` in QDDSSRCD (matches program number)
- CL drivers: `FRS0xxCL` (e.g., FRS001CL.CLP, FRS411CL.CLP)

## Code Style Patterns (Modernized RPGLE)
- All modernized members start with `**FREE` on line 1
- Function key constants defined as hex literals (DCL-C F03 X'33'; etc.) - no shared copybook
- Indicators replaced by named `IND` variables (DCL-S exitFlag IND) or full INDDS data structures
- PSDS always declared `QUALIFIED` with named subfields for job/pgm info
- Boolean constants: `DCL-C TRUE *ON; DCL-C FALSE *OFF;` (some use `'1'`/`'0'` - inconsistent)
- API calls prototyped inline per program (no shared prototype copybook exists)
- `OPTION(*NODEBUGIO:*SRCSTMT)` standard on modernized CTL-OPT

## Critical Gotchas
- Two members in QRPGLESRC have NO file extension (FRS021. and FRS021DOC.) - Bob handles
  these but some tools may not display them correctly
- FRS022.RPG (QRPGSRC) is **FREE ILE RPGLE with embedded SQL stored in an OPM .RPG member -
  it must be compiled with CRTSQLRPGI, not CRTRPGPGM or CRTBNDRPG
- The EVFTEMPF01 source file is auto-generated by RDi/Bob error feedback; do not edit members in it
- Compilation docs (FRS021DOC., FRS401DOC.TXT) target FLGHT400, not FLGHT403 - adjust library
  when compiling in dev
