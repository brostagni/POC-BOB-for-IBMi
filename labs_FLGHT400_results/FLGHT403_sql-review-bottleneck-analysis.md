# FLGHT403 — SQL Review & Bottleneck Analysis
**Session :** 20261029-14-03  
**Bibliothèque cible :** FLGHT403  
**Tables analysées :** FLIGHTS, ORDERS, AGENTS, CUSTOMERS  

---

## Partie 4b — SQL Review

### Requête originale (corrigée FLGHT400 → FLGHT403)

```sql
SELECT
    f.FLIGH00001                                    AS FLIGHT_NUMBER,
    f.DEPARTURE                                     AS FROM_CITY,
    f.ARRIVAL                                       AS TO_CITY,
    f.AIRLINES                                      AS AIRLINE,
    f.DAY_O00001                                    AS DAY_OF_WEEK,
    f.DEPAR00002                                    AS DEPARTURE_TIME,
    f.ARRIV00002                                    AS ARRIVAL_TIME,
    f.MILEAGE,
    f.TICKE00001                                    AS TICKET_PRICE,
    f.SEATS00001                                    AS SEATS_AVAILABLE,
    ag.AGENT_NAME,
    COUNT(DISTINCT o.CUSTO00001)                    AS UNIQUE_CUSTOMERS,
    SUM(o.TICKE00001)                               AS TOTAL_TICKETS_SOLD,
    SUM(CASE WHEN o.CLASS = 'F' THEN o.TICKE00001 ELSE 0 END) AS FIRST_CLASS_TICKETS,
    SUM(CASE WHEN o.CLASS = 'B' THEN o.TICKE00001 ELSE 0 END) AS BUSINESS_TICKETS,
    SUM(CASE WHEN o.CLASS = 'E' THEN o.TICKE00001 ELSE 0 END) AS ECONOMY_TICKETS,
    MIN(o.DEPAR00001)                               AS EARLIEST_BOOKING_DATE,
    MAX(o.DEPAR00001)                               AS LATEST_BOOKING_DATE
FROM FLGHT400/FLIGHTS       f
JOIN FLGHT400/ORDERS        o  ON o.FLIGH00001  = f.FLIGH00001
JOIN FLGHT400/AGENTS        ag ON ag.AGENT_NO   = o.AGENT_NO
LEFT JOIN FLGHT400/CUSTOMERS c  ON c.CUSTO00001  = o.CUSTO00001
WHERE o.DEPAR00001 >= TIMESTAMP('2004-02-08-00.00.00')
  AND o.DEPAR00001 <  TIMESTAMP('2004-02-11-00.00.00')
GROUP BY
    f.FLIGH00001, f.DEPARTURE, f.ARRIVAL, f.AIRLINES,
    f.DAY_O00001, f.DEPAR00002, f.ARRIV00002,
    f.MILEAGE, f.TICKE00001, f.SEATS00001, ag.AGENT_NAME
ORDER BY
    o.DEPAR00001,
    f.FLIGH00001
FETCH FIRST 100 ROWS ONLY;
```

---

### 1. Summary

Requête de reporting multi-tables correctement structurée dans l'ensemble. Quatre problèmes identifiés, dont un bloquant à l'exécution.

---

### 2. Confirmed Findings

#### F1 — `ORDER BY o.DEPAR00001` invalide dans un contexte agrégé ⛔ BLOQUANT
- **Catégorie :** Correctness
- `o.DEPAR00001` (DEPARTURE_DATE) n'est pas dans le `GROUP BY`. En contexte `GROUP BY`, seules les colonnes groupées et les agrégats sont autorisés dans `ORDER BY`. **Erreur SQL SQLSTATE 42803 à l'exécution.**
- **Correction :** `ORDER BY MIN(o.DEPARTURE_DATE), f.FLIGHT_NUMBER`

#### F2 — `LEFT JOIN CUSTOMERS` sans usage dans le SELECT ⚠️ MORT-CODE
- **Catégorie :** Correctness / Simplification
- Aucune colonne de `c` (CUSTOMERS) n'est référencée dans le SELECT, WHERE, GROUP BY ou ORDER BY.
- Génère des I/O inutiles ; la jointure LEFT ne filtre rien non plus.
- **Correction :** supprimer la jointure.

#### F3 — Noms système auto-générés comme identifiants de colonne 📖 LISIBILITÉ
- **Catégorie :** Readability
- `FLIGH00001`, `DEPAR00001`, `DEPAR00002`, `TICKE00001`, etc. sont des noms système IBM i 10-car. auto-générés, illisibles.
- Les noms SQL longs sont disponibles : `FLIGHT_NUMBER`, `DEPARTURE_DATE`, `DEPARTURE_TIME`, `TICKETS_ORDERED`, etc.
- **Correction :** utiliser les noms SQL longs dans le corps de la requête.

#### F4 — `TIMESTAMP('2004-02-08-00.00.00')` — syntaxe verbeuse 📝 STYLE
- **Catégorie :** Best Practice
- Db2 for i accepte directement `'2004-02-08 00:00:00'` comme littéral timestamp implicite.
- **Correction :** `WHERE o.DEPARTURE_DATE >= '2004-02-08 00:00:00'`

---

### 3. Conditional Recommendations

| # | Recommandation | Condition |
|---|---|---|
| C1 | Index sur `ORDERS.DEPARTURE_DATE` | À confirmer si scan via Visual Explain |
| C2 | Index composite `ORDERS(DEPARTURE_DATE, FLIGHT_NUMBER, AGENT_NO)` | Si requête fréquente à fort volume |
| C3 | `OPTIMIZE FOR 100 ROWS` | Si livraison rapide des premières lignes est prioritaire |
| C4 | `COUNT(DISTINCT …)` — coût de déduplication | À évaluer selon cardinalité clients/vol |
| C5 | Ajouter `WITH UR` (reporting read-only) | Évite les verrous inutiles |

---

### 4. Requête révisée

```sql
SELECT
    f.FLIGHT_NUMBER,
    f.DEPARTURE                                         AS FROM_CITY,
    f.ARRIVAL                                           AS TO_CITY,
    f.AIRLINES                                          AS AIRLINE,
    f.DAY_OF_WEEK,
    f.DEPARTURE_TIME,
    f.ARRIVAL_TIME,
    f.MILEAGE,
    f.TICKET_PRICE,
    f.SEATS_AVAILABLE,
    ag.AGENT_NAME,
    COUNT(DISTINCT o.CUSTOMER_NO)                       AS UNIQUE_CUSTOMERS,
    SUM(o.TICKETS_ORDERED)                              AS TOTAL_TICKETS_SOLD,
    -- Ventilation par classe de service
    SUM(CASE WHEN o.CLASS = 'F' THEN o.TICKETS_ORDERED ELSE 0 END) AS FIRST_CLASS_TICKETS,
    SUM(CASE WHEN o.CLASS = 'B' THEN o.TICKETS_ORDERED ELSE 0 END) AS BUSINESS_TICKETS,
    SUM(CASE WHEN o.CLASS = 'E' THEN o.TICKETS_ORDERED ELSE 0 END) AS ECONOMY_TICKETS,
    MIN(o.DEPARTURE_DATE)                               AS EARLIEST_BOOKING_DATE,
    MAX(o.DEPARTURE_DATE)                               AS LATEST_BOOKING_DATE
FROM FLGHT403/FLIGHTS       f
JOIN FLGHT403/ORDERS        o  ON o.FLIGHT_NUMBER = f.FLIGHT_NUMBER
JOIN FLGHT403/AGENTS        ag ON ag.AGENT_NO      = o.AGENT_NO
WHERE o.DEPARTURE_DATE >= '2004-02-08 00:00:00'
  AND o.DEPARTURE_DATE <  '2004-02-11 00:00:00'
GROUP BY
    f.FLIGHT_NUMBER, f.DEPARTURE, f.ARRIVAL, f.AIRLINES,
    f.DAY_OF_WEEK, f.DEPARTURE_TIME, f.ARRIVAL_TIME,
    f.MILEAGE, f.TICKET_PRICE, f.SEATS_AVAILABLE, ag.AGENT_NAME
ORDER BY
    MIN(o.DEPARTURE_DATE),
    f.FLIGHT_NUMBER
FETCH FIRST 100 ROWS ONLY
WITH UR;
```

### 5. Explication des changements

| # | Modification | Raison |
|---|---|---|
| 1 | `FLGHT400` → `FLGHT403` | Directive de session |
| 2 | `ORDER BY o.DEPAR00001` → `ORDER BY MIN(o.DEPARTURE_DATE)` | F1 — erreur SQLSTATE 42803 corrigée |
| 3 | `LEFT JOIN FLGHT403/CUSTOMERS` supprimé | F2 — jointure morte |
| 4 | Noms système → noms SQL longs | F3 — lisibilité |
| 5 | `TIMESTAMP('…')` → littéral implicite | F4 — style idiomatique Db2 for i |
| 6 | `WITH UR` ajouté | C5 — reporting lecture seule |

---

## Partie 4c — Bottleneck Analysis

### Volumétrie des tables (QSYS2.SYSTABLESTAT)

| Table | Lignes | Taille données | Lectures logiques | Lectures physiques | Lignes supprimées |
|---|---|---|---|---|---|
| **FLIGHTS** | 482 331 | 129 Mo | 2 889 899 | **2 323** | 1 |
| **ORDERS** | 701 512 | 45 Mo | 2 797 908 | 1 430 | 11 |
| CUSTOMERS | 10 022 | 0,4 Mo | 70 159 | 136 | 1 |
| AGENTS | 110 | 12 Ko | 443 | 0 | 1 |

### Index SQL présents

`QSYS2.SYSINDEXES` ne retourne **aucun index SQL** sur FLIGHTS, ORDERS, AGENTS ou CUSTOMERS dans FLGHT403.

Les seules access paths disponibles sont des **fichiers logiques DDS** :

| Fichier logique | Table de base | Taille | Probable clé |
|---|---|---|---|
| `ORDDATE.LF` | ORDERS | ~11 Mo | DEPARTURE_DATE |
| `ORDERSZ.LF` | ORDERS | ~14 Mo | Conversion binary→zoned |
| `FLIGHTSZ.LF` | FLIGHTS | ~10 Mo | Conversion binary→zoned |
| `FLIGHTSL.LF` | FLIGHTS | ~12 Mo | Clé de liste |
| `CUSTORD.LF` | ORDERS+CUSTOMERS | ~11 Mo | Join LF |

> ⚠️ L'utilisation de ces LF par SQE n'est **pas garantie** — seul Visual Explain le confirme.

### Verdict : ORDERS est le goulot principal

**Raisons cumulatives :**

1. **Filtre de plage sans index SQL natif.** `WHERE o.DEPARTURE_DATE >= … AND < …` sur 701 512 lignes — scan complet probable si `ORDDATE.LF` n'est pas exploité par SQE.
2. **Clé de jointure `FLIGHT_NUMBER` sans index SQL.** La jonction vers FLIGHTS traverse toute la table sans access path confirmé.
3. **Agrégation `GROUP BY` sur 11 colonnes.** Nécessite tri ou hashing en mémoire temporaire.

**FLIGHTS est le goulot secondaire** :

- Taille physique 129 Mo — la plus élevée des 4 tables.
- 2 323 lectures physiques (disque réel, pas buffer).
- Jointure `FLIGHT_NUMBER` sans index SQL confirmé.

**AGENTS et CUSTOMERS : négligeables** (respectivement 110 lignes / 0 physiques, et déjà supprimé de la requête révisée).

### Diagramme

```
ORDERS (701 512 lignes)          FLIGHTS (482 331 lignes)
  Filtre DEPARTURE_DATE            129 Mo physiques
  ⚠️ Scan probable           <-->  ⚠️ Physiques élevées
         |
         +--AGENT_NO--> AGENTS (110 lignes ✅ négligeable)
```

### Actions de vérification recommandées

| Action | Outil | Objectif |
|---|---|---|
| Confirmer scan vs index sur ORDERS | Visual Explain (ACS) | Méthode d'accès sur DEPARTURE_DATE |
| Confirmer utilisation FLIGHTSZ/FLIGHTSL | Visual Explain (ACS) | Méthode d'accès sur FLIGHTS |
| Vérifier MTI auto-créés par SQE | `QSYS2.MTI_INFO` | Indexes temporaires actifs |
| Conseils d'index récents | `QSYS2.SYSIXADV` | Recommandations optimizer |

```sql
-- MTI actifs sur ORDERS et FLIGHTS
SELECT *
FROM TABLE(QSYS2.MTI_INFO(TABLE_SCHEMA => 'FLGHT403'))
WHERE TABLE_NAME IN ('FLIGHTS', 'ORDERS');

-- Conseils d'index récents
SELECT TABLE_NAME, KEY_COLUMNS_ADVISED, TIMES_ADVISED, LAST_ADVISED
FROM QSYS2.SYSIXADV
WHERE TABLE_SCHEMA = 'FLGHT403'
ORDER BY TIMES_ADVISED DESC, LAST_ADVISED DESC;
```

### Index recommandé si scan confirmé

```sql
CREATE INDEX FLGHT403/ORDIDX01
    ON FLGHT403/ORDERS (DEPARTURE_DATE, FLIGHT_NUMBER, AGENT_NO)
    FOR SYSTEM NAME ORDIDX01;
```

Couvre le filtre de plage, la jointure FLIGHTS et la jointure AGENTS en une seule access path.

---

*Rapport généré le 2026-10-29 14:03 UTC — bibliothèque FLGHT403 — IBM i 7.6 / Db2 for i*
