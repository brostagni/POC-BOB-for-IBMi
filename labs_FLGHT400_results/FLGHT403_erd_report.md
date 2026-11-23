# FLGHT403 — Rapport de session `/erd`

> Sortie complète de la commande `/erd FLGHT403` — analyse du catalogue Db2 for i.

---

## Périmètre analysé

Bibliothèque : **FLGHT403**
Requêtes catalogue exécutées : `QSYS2.SYSTABLES`, `QSYS2.SYSCOLUMNS2`, `QSYS2.SYSCST`, `QSYS2.SYSREFCST`

---

## Tables découvertes (TABLE_TYPE = 'P')

| Nom | Type | Commentaire |
|---|---|---|
| AGENTS | PF | Agents de réservation |
| AIRLINE | PF | Table de validation des compagnies aériennes |
| CUSTOMERS | PF | Clients de la compagnie |
| EVFTEMPF01 | PF | Fichier temporaire RDi/Bob (feedback compilateur) |
| FCITY | PF | Table villes (ancien nom — voir FRCITY/TOCITY) |
| FLIGHTS | PF | Planning des vols |
| FNAME | PF | Prénoms (génération données de test) |
| FNEXTO | PF | Compteur prochain numéro de commande |
| FRCITY | PF | Villes de départ |
| LNAME | PF | Noms de famille (génération données de test) |
| ORDERS | PF | Commandes de billets |
| TOCITY | PF | Villes d'arrivée |
| QBOBSRC | PF | Source Bob AI (créé par /init) |
| QCLPSRC | PF | Source CLP |
| QCMDSRC | PF | Source CMD |
| QDDSSRCD | PF | Source DDS Display Files |
| QDDSSRCF | PF | Source DDS PF/LF |
| QMNUSRC | PF | Source menus |
| QRPGLESRC | PF | Source RPGLE ILE |
| QRPGSRC | PF | Source RPG OPM |

---

## Contraintes SQL formelles

**Aucune** — les tables sont des fichiers physiques DDS (`TABLE_TYPE = 'P'`).
`QSYS2.SYSCST` et `QSYS2.SYSREFCST` ne retournent aucune ligne pour FLGHT403.

Les clés et relations sont définies par :
- Le mot-clé `UNIQUE` et `K` (key) dans les sources DDS
- Les RENAME dans les LF (accès alternatifs)
- Les joins dans les logical files (JFILE/JFLD)

---

## Détail des colonnes — tables applicatives

### AGENTS
| # | Colonne | Type | Null |
|---|---|---|---|
| 1 | AGENT_NO | INTEGER(9) | Y |
| 2 | AGENT_NAME | VARCHAR(64) | Y |
| 3 | AGENT00001 | VARCHAR(64) | Y |

> `AGENT00001` = nom système auto-tronqué (3e colonne, probablement mot de passe agent)

### AIRLINE
| # | Colonne | Type | Null |
|---|---|---|---|
| 1 | AIRLNM | CHAR(16) | N |
| 2 | AIRLIN | CHAR(3) | N |

### CUSTOMERS
| # | Colonne | Type | Null |
|---|---|---|---|
| 1 | CUSTOMER_NO | INTEGER(9) | Y |
| 2 | CUSTOMER_NAME | VARCHAR(64) | Y |

### FLIGHTS
| # | Colonne | Type | Null |
|---|---|---|---|
| 1 | FLIGHT_NUMBER | INTEGER(9) | Y |
| 2 | DEPARTURE_INITIALS | VARCHAR(16) | Y |
| 3 | DEPARTURE | VARCHAR(16) | Y |
| 4 | DAY_OF_WEEK | VARCHAR(16) | Y |
| 5 | ARRIVAL_INITIALS | VARCHAR(16) | Y |
| 6 | ARRIVAL | VARCHAR(16) | Y |
| 7 | DEPARTURE_TIME | VARCHAR(32) | Y |
| 8 | ARRIVAL_TIME | VARCHAR(32) | Y |
| 9 | AIRLINES | VARCHAR(32) | Y |
| 10 | SEATS_AVAILABLE | INTEGER(9) | Y |
| 11 | TICKET_PRICE | VARCHAR(22) | Y |
| 12 | MILEAGE | INTEGER(9) | Y |

### ORDERS
| # | Colonne | Type | Null |
|---|---|---|---|
| 1 | ORDER_NUMBER | INTEGER(9) | Y |
| 2 | AGENT_NO | INTEGER(9) | Y |
| 3 | CUSTOMER_NO | INTEGER(9) | Y |
| 4 | FLIGHT_NUMBER | INTEGER(9) | Y |
| 5 | DEPARTURE_DATE | TIMESTAMP | Y |
| 6 | TICKETS_ORDERED | INTEGER(9) | Y |
| 7 | CLASS | VARCHAR(1) | Y |
| 8 | SEND_SIGNATURE_WITH_ORDER | VARCHAR(16) | Y |

### FRCITY
| # | Colonne | Type | Null |
|---|---|---|---|
| 1 | FRCNAM | CHAR(16) | N |
| 2 | FRCINT | CHAR(3) | N |
| 3 | FRCALN | CHAR(3) | N |
| 4 | FRCNBR | NUMERIC(3) | N |

### TOCITY
| # | Colonne | Type | Null |
|---|---|---|---|
| 1 | TOCNAM | CHAR(16) | N |
| 2 | TOCINT | CHAR(3) | N |

### FNEXTO
| # | Colonne | Type | Null |
|---|---|---|---|
| 1 | NEXT_ORDER | NUMERIC(9) | N |

### FNAME / LNAME
Tables de référence pour la génération de données de test (`BCUST.RPG`).

---

## Diagramme ERD (Mermaid)

```mermaid
erDiagram

    AGENTS {
        INTEGER(9) AGENT_NO PK
        VARCHAR(64) AGENT_NAME
        VARCHAR(64) AGENT00001
    }

    CUSTOMERS {
        INTEGER(9) CUSTOMER_NO PK
        VARCHAR(64) CUSTOMER_NAME
    }

    AIRLINE {
        CHAR(16) AIRLNM PK
        CHAR(3) AIRLIN UQ
    }

    FRCITY {
        CHAR(16) FRCNAM PK
        CHAR(3) FRCINT
        CHAR(3) FRCALN
        NUMERIC(3) FRCNBR
    }

    TOCITY {
        CHAR(16) TOCNAM PK
        CHAR(3) TOCINT
    }

    FLIGHTS {
        INTEGER(9) FLIGHT_NUMBER PK
        VARCHAR(16) DEPARTURE_INITIALS FK
        VARCHAR(16) DEPARTURE
        VARCHAR(16) DAY_OF_WEEK
        VARCHAR(16) ARRIVAL_INITIALS FK
        VARCHAR(16) ARRIVAL
        VARCHAR(32) DEPARTURE_TIME
        VARCHAR(32) ARRIVAL_TIME
        VARCHAR(32) AIRLINES FK
        INTEGER(9) SEATS_AVAILABLE
        VARCHAR(22) TICKET_PRICE
        INTEGER(9) MILEAGE
    }

    ORDERS {
        INTEGER(9) ORDER_NUMBER PK
        INTEGER(9) AGENT_NO FK
        INTEGER(9) CUSTOMER_NO FK
        INTEGER(9) FLIGHT_NUMBER FK
        TIMESTAMP DEPARTURE_DATE
        INTEGER(9) TICKETS_ORDERED
        VARCHAR(1) CLASS
        VARCHAR(16) SEND_SIGNATURE_WITH_ORDER
    }

    FNEXTO {
        NUMERIC(9) NEXT_ORDER
    }

    FNAME {
        CHAR(10) FRSTNM
    }

    LNAME {
        CHAR(15) LASTNM
    }

    AGENTS ||--o{ ORDERS : "places"
    CUSTOMERS ||--o{ ORDERS : "books"
    FLIGHTS ||--o{ ORDERS : "reserved_on"
    FRCITY ||--o{ FLIGHTS : "departs_from"
    TOCITY ||--o{ FLIGHTS : "arrives_at"
    AIRLINE ||--o{ FLIGHTS : "operated_by"
```

---

## Fichiers produits

| Fichier | Contenu |
|---|---|
| `/home/ITZUSER/FLGHT403_erd.md` | Diagramme ERD Mermaid seul |
| `/home/ITZUSER/FLGHT403_erd_report.md` | Ce rapport complet |
