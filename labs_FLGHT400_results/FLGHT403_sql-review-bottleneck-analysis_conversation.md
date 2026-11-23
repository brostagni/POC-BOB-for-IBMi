# FLGHT403 — SQL Review & Bottleneck Analysis — Conversation
**Session :** 20261029-14-03  
**Bibliothèque cible :** FLGHT403 (remplace FLGHT400 pour cette session)  
**Système :** IBM i 7.6 — Flight Reservation System (FRS)  
**Utilisateur :** ITZUSER  

---

## Contexte de la session

Session de revue SQL sur la base de données de démonstration Flight Reservation System.  
La requête originale ciblait `FLGHT400` ; l'utilisateur a imposé le remplacement systématique par `FLGHT403` pour toute cette session.

Deux exercices ont été réalisés, numérotés 4b et 4c, dans le cadre d'un rapport final à construire.

---

## Échange 4b — SQL Review

### Requête soumise

Requête de reporting multi-tables : vols par agent sur une plage de 3 jours (2004-02-08 au 2004-02-11), avec agrégats par classe de service (F/B/E), comptage de clients distincts, et limitation à 100 résultats.

Tables impliquées : `FLGHT400/FLIGHTS`, `FLGHT400/ORDERS`, `FLGHT400/AGENTS`, `FLGHT400/CUSTOMERS`.

### Validation catalogue effectuée

Les colonnes suivantes ont été vérifiées via `get_table_columns` (QSYS2.SYSCOLUMNS2) :

**FLGHT403/FLIGHTS** — 13 colonnes :
- `FLIGHT_NUMBER` / sys `FLIGH00001` — INTEGER
- `DEPARTURE` / sys `DEPARTURE` — VARCHAR(16)
- `ARRIVAL` / sys `ARRIVAL` — VARCHAR(16)
- `AIRLINES` / sys `AIRLINES` — VARCHAR(32)
- `DAY_OF_WEEK` / sys `DAY_O00001` — VARCHAR(16)
- `DEPARTURE_TIME` / sys `DEPAR00002` — VARCHAR(32)
- `ARRIVAL_TIME` / sys `ARRIV00002` — VARCHAR(32)
- `MILEAGE` / sys `MILEAGE` — INTEGER (BINARY)
- `TICKET_PRICE` / sys `TICKE00001` — VARCHAR(22) ⚠️ pas numérique
- `SEATS_AVAILABLE` / sys `SEATS00001` — INTEGER (BINARY)
- `DEPARTURE_INITIALS` / sys `DEPAR00001` — VARCHAR(16)
- `ARRIVAL_INITIALS` / sys `ARRIV00001` — VARCHAR(16)
- `FLHRS` — DECIMAL(4,0)

**FLGHT403/ORDERS** — 8 colonnes :
- `ORDER_NUMBER` / sys `ORDER00001` — INTEGER (BINARY)
- `AGENT_NO` / sys `AGENT_NO` — INTEGER (BINARY)
- `CUSTOMER_NO` / sys `CUSTO00001` — INTEGER (BINARY)
- `FLIGHT_NUMBER` / sys `FLIGH00001` — INTEGER (BINARY)
- `DEPARTURE_DATE` / sys `DEPAR00001` — TIMESTAMP ← clé du filtre
- `TICKETS_ORDERED` / sys `TICKE00001` — INTEGER (BINARY)
- `CLASS` / sys `CLASS` — VARCHAR(1)
- `SEND_SIGNATURE_WITH_ORDER` / sys `SEND_00001` — VARCHAR(16)

**FLGHT403/AGENTS** — 3 colonnes :
- `AGENT_NO` / sys `AGENT_NO` — INTEGER (BINARY)
- `AGENT_NAME` / sys `AGENT_NAME` — VARCHAR(64)
- `AGENT_PASSWORD` / sys `AGENT00001` — VARCHAR(64) ⚠️ password en clair

**FLGHT403/CUSTOMERS** — 2 colonnes :
- `CUSTOMER_NO` / sys `CUSTO00001` — INTEGER (BINARY)
- `CUSTOMER_NAME` / sys `CUSTO00002` — VARCHAR(64)

### Findings confirmés (F1–F4)

| Code | Catégorie | Problème | Correction |
|---|---|---|---|
| **F1** ⛔ | Correctness | `ORDER BY o.DEPAR00001` — colonne non groupée, SQLSTATE 42803 | `ORDER BY MIN(o.DEPARTURE_DATE), f.FLIGHT_NUMBER` |
| **F2** ⚠️ | Correctness / Simplification | `LEFT JOIN CUSTOMERS` — aucune colonne utilisée, mort-code | Supprimer la jointure |
| **F3** 📖 | Readability | Noms système auto-générés (`FLIGH00001`, etc.) | Utiliser les noms SQL longs |
| **F4** 📝 | Best Practice | `TIMESTAMP('2004-02-08-00.00.00')` verbeux | Littéral implicite `'2004-02-08 00:00:00'` |

### Recommandations conditionnelles (C1–C5)

| Code | Recommandation | Condition |
|---|---|---|
| C1 | Index sur `ORDERS.DEPARTURE_DATE` | Si scan confirmé par Visual Explain |
| C2 | Index composite `(DEPARTURE_DATE, FLIGHT_NUMBER, AGENT_NO)` | Requête fréquente à fort volume |
| C3 | `OPTIMIZE FOR 100 ROWS` | Livraison rapide des premières lignes |
| C4 | Coût `COUNT(DISTINCT CUSTOMER_NO)` | Selon cardinalité clients/vol |
| C5 | `WITH UR` | Reporting lecture seule — ajouté dans la requête révisée |

### Requête révisée (résumé des changements)

1. `FLGHT400` → `FLGHT403` (directive)
2. `ORDER BY` corrigé avec `MIN(o.DEPARTURE_DATE)`
3. `LEFT JOIN CUSTOMERS` supprimé
4. Noms système → noms SQL longs
5. `TIMESTAMP('…')` → littéral implicite
6. `WITH UR` ajouté

---

## Échange 4c — Bottleneck Analysis

### Données SYSTABLESTAT collectées

| Table | Lignes | Taille | Lectures logiques | Lectures physiques |
|---|---|---|---|---|
| FLIGHTS | 482 331 | 129 Mo | 2 889 899 | 2 323 |
| ORDERS | 701 512 | 45 Mo | 2 797 908 | 1 430 |
| CUSTOMERS | 10 022 | 0,4 Mo | 70 159 | 136 |
| AGENTS | 110 | 12 Ko | 443 | 0 |

### Index SQL

`QSYS2.SYSINDEXES` → **aucun index SQL** sur ces 4 tables dans FLGHT403.

Fichiers logiques DDS présents (access paths potentiels) :
- `ORDDATE.LF` (~11 Mo) — probablement keyed sur DEPARTURE_DATE
- `ORDERSZ.LF` (~14 Mo) — conversion binary→zoned pour OPM
- `FLIGHTSZ.LF` (~10 Mo) — idem FLIGHTS
- `FLIGHTSL.LF` (~12 Mo)
- `CUSTORD.LF`, `ORDNAME.LF`, `ORDDATE.LF` — LF de jointure/tri

`QSYS2.SYSIXADV` → aucun conseil d'index retourné (pas encore peuplé pour ces tables).

### Verdict

**Goulot principal : ORDERS**
- 701 512 lignes, filtre de plage sur `DEPARTURE_DATE` sans index SQL natif
- Scan probable si SQE n'exploite pas `ORDDATE.LF`
- Jointure `FLIGHT_NUMBER` sans index SQL confirmé

**Goulot secondaire : FLIGHTS**
- 129 Mo — table la plus lourde physiquement
- 2 323 lectures physiques (la plus haute)
- Jointure clé sans index SQL

**Négligeables :** AGENTS (110 lignes, 0 physiques), CUSTOMERS (supprimé de la requête).

### Index recommandé si scan confirmé

```sql
CREATE INDEX FLGHT403/ORDIDX01
    ON FLGHT403/ORDERS (DEPARTURE_DATE, FLIGHT_NUMBER, AGENT_NO)
    FOR SYSTEM NAME ORDIDX01;
```

### Actions de vérification à réaliser

1. Visual Explain (ACS) sur la requête révisée — méthode d'accès ORDERS et FLIGHTS
2. `QSYS2.MTI_INFO(TABLE_SCHEMA => 'FLGHT403')` — MTI SQE actifs
3. `QSYS2.SYSIXADV` après exécution — conseils d'index générés

---

## Notes de contexte IBM i / FLGHT403

- `TICKET_PRICE` dans FLIGHTS est **VARCHAR(22)**, pas numérique — toute tentative de `SUM(f.TICKET_PRICE)` échouerait.
- `AGENT_PASSWORD` dans AGENTS est stocké en clair (VARCHAR(64)) — risque sécurité à signaler.
- Les fichiers logiques `FLIGHTSZ`, `ORDERSZ`, `CUSTOMRZ` existent pour convertir les champs BINARY en ZONED pour les programmes OPM RPG.
- Aucun service program ni binding directory dans FLGHT403.
- Runtime library : `FLGHT400` pour `FRSMSGF` et certains fichiers de production.

---

## Fichiers produits lors de cette session

| Fichier | Contenu |
|---|---|
| `FLGHT403_sql-review-bottleneck-analysis_20261029-14-03.md` | Rapport structuré 4b + 4c |
| `FLGHT403_sql-review-bottleneck-analysis_conversation_20261029-14-03.md` | Ce fichier — conversation complète |

---

*Généré le 2026-10-29 14:03 UTC*
