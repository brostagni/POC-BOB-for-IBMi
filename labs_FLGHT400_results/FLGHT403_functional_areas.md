# FLGHT403 — High-Level Functional Areas

> **Source**: Understand scan of libraries `FLGHT403` and `FLGHT400`, last scanned 2026-10-29.

---

## 1. 🛒 Order / Reservation Management

The most coupled programs — `FRS001`, `FRS002` — handle new and updated flight reservations. Core tables are `ORDERS` (10 inbound deps) and `CUSTOMERS` (12 inbound deps).

- **New order entry**: `FRS001`, `FRS001T`, `FRS001U`
- **Order update**: `FRS002`
- **Order reporting**: `FRS003`–`FRS009` range

---

## 2. ✈️ Flight Schedule Maintenance

`FRS021` (10 outbound deps) manages the flight schedule. Key files: `FLIGHTS` PF accessed via `FLIGHTSZ` LF (binary→zoned conversion for OPM programs), and `FRCITY` / `TOCITY` city tables (11–10 inbound deps each).

---

## 3. 👤 Customer Management

`CUSTOMERS` PF is the most-depended-upon file (12 deps). Programs in the `FRS01x` range handle customer lookup, creation, and maintenance.

---

## 4. 📊 Reporting

The `QRPGSRC` member range includes report programs (e.g., `FRS011`–`FRS020` pattern). The presence of `FRSREPT` menu (via `FLGHT400`) indicates a dedicated reporting menu.

---

## 5. 🔧 System Administration / Utilities

- `FRSMANT` menu — system maintenance entry point
- `USECLEAR` / `USERESET` CLP programs — manage the `FLDS` data area in `FLGHT400`
- `BFLGHT.RPG`, `BCUST.RPG` — build/populate test data
- `RTRAND`, `RTDOFW`, `JTOGCV`, `GTOJCV` — standalone date/random utilities

---

## 6. 🏗️ Build / Data Population

`B`-prefixed programs (`BFLGHT`, `BCUST`) populate master files with test data — not production logic, but part of the demo setup.

---

## Key Signals

| Signal | Detail |
|---|---|
| OPM programs | 68 of 92 — legacy RPG/400, modernization underway |
| ILE programs | 22 — free-format RPGLE |
| Programs with no inbound deps | 40 — entry points or dead-code candidates |
| Foreign keys / triggers / journaling | None recorded on application files |

---

*Generated from Understand scan — FLGHT403 / FLGHT400 — 2026-10-29*
