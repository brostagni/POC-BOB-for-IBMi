# FLGHT403 — Entity Relationship Diagram

> Généré par `/erd FLGHT403` — tables applicatives uniquement (PF DDS).
> Aucune contrainte SQL formelle (FK/PK) n'est définie dans le catalogue.
> Les clés primaires et relations sont déduites de la structure DDS et du code RPG.

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

## Notes

- **Pas de contraintes SQL formelles** : toutes les tables sont des fichiers physiques DDS (`TABLE_TYPE = 'P'`).
  Les relations ci-dessus sont déduites du code RPG (champs partagés, joins dans les LF).
- **FNEXTO** : compteur du prochain numéro de commande (un seul enregistrement, pas de clé).
- **FNAME / LNAME** : tables de référence pour la génération de données de test (`BCUST.RPG`).
- **Logical files (LF) exclus du diagramme** :
  - `FLIGHTSZ`, `ORDERSZ`, `CUSTOMRZ`, `AGENTSZ` — wrappers binaire→zoné pour OPM RPG
  - `FLIGHTSL` — tri FROM/TO/DAY_OF_WEEK
  - `FRCITYL` — tri ville départ
  - `CUSTNAME`, `CUSTOMER`, `CUSTORD`, `ORDNAME`, `ORDDATE` — join/accès alternatifs
  - `AIRLINEL` — accès par initiales compagnie
- **AGENTS.AGENT00001** : nom système auto-tronqué (colonne 3, probablement `AGENT_PASSWORD`).
