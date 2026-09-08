# Neniku's DB

Gestionale per negozi fisici — sviluppato da zero per risolvere problemi reali di magazzino, vendite e gestione clienti che i software commerciali generalisti gestiscono male o a caro prezzo.

Stack: **FastAPI** (Python) · **Next.js** (TypeScript) · **PostgreSQL** · **Docker Compose**

![Backend](https://img.shields.io/badge/Backend-FastAPI-009688?style=flat-square&logo=fastapi)
![Frontend](https://img.shields.io/badge/Frontend-Next.js-000000?style=flat-square&logo=nextdotjs)
![Database](https://img.shields.io/badge/Database-PostgreSQL-4169E1?style=flat-square&logo=postgresql)
![Containerized](https://img.shields.io/badge/Containerized-Docker-2496ED?style=flat-square&logo=docker)
![License](https://img.shields.io/badge/License-Proprietary-lightgrey?style=flat-square)

---

## Anteprima

<!-- SCREENSHOT DASHBOARD PRINCIPALE -->
> _screenshot dashboard_

<!-- SCREENSHOT SCHEDA PRODOTTO -->
> _screenshot scheda prodotto / inventario_

<!-- SCREENSHOT IMPORT PDF -->
> _screenshot flusso import PDF_

---

## Cos'è

Neniku's DB nasce per gestire un negozio fisico in modo completo, senza dipendere da SaaS esterni o abbonamenti mensili. Gira interamente in locale (o su un server self-hosted) tramite Docker Compose: nessun dato fuori dalla tua rete.

Il sistema copre tutto il ciclo operativo di un negozio:

- **Catalogo prodotti** con barcode, SKU, prezzi differenziati (acquisto / consigliato / vendita), soglie di riordino e supporto nativo ai bundle (es. un box che contiene N bustine: la giacenza si calcola automaticamente)
- **Clienti** con store credit spendibile, storico acquisti e un sistema di fiducia/strike per gestire i clienti abituali
- **Vendite** multi-articolo con utilizzo del credito, possibilità di annullamento e ripristino automatico della giacenza
- **Prenotazioni** con stati (attiva / evasa / annullata / scaduta) e blocco automatico della disponibilità
- **Fornitori** con anagrafica completa e collegamento diretto ai prodotti
- **Categorie** ad albero gerarchico
- **Report** mensili e annuali generati in PDF direttamente dall'interfaccia
- **Ricerca full-text** su prodotti, clienti, barcode e codici fornitore
- **Audit log** immutabile su ogni operazione critica (chi ha fatto cosa, quando, con quali valori prima e dopo)

---

## Il modulo Import PDF

<!-- SCREENSHOT FLUSSO IMPORT -->
> _screenshot interfaccia import e anteprima_

Questo è probabilmente il modulo più specifico del progetto. I fornitori mandano i listini in PDF, ognuno con un formato diverso. Invece di reinserire tutto a mano, il sistema include un parser dedicato per ogni fornitore, costruito su un'architettura comune.

Il flusso è a due fasi intenzionalmente:

1. **Anteprima** — il PDF viene caricato e analizzato, il sistema mostra tutti i prodotti rilevati, i campi estratti e i warning per le righe ambigue. In questa fase non viene scritto nulla nel database.
2. **Conferma** — solo dopo l'approvazione dell'operatore i dati vengono applicati: prodotti creati o aggiornati, giacenza movimentata, sessione registrata con il conteggio di creati / aggiornati / saltati / errori.

Il parser riconosce automaticamente i box (bundle) e li collega al prodotto figlio corretto, impostando il moltiplicatore. Il PDF originale viene archiviato su disco.

---

## Architettura

```
Browser
  └── Next.js (TypeScript)
        └── REST API
              └── FastAPI (Python)
                    ├── Parser PDF
                    ├── Generatore PDF report
                    └── PostgreSQL
                          ├── Rete Docker isolata (db-net)
                          └── Migrazioni via Alembic
```

Il database gira su una rete Docker dedicata, separata dal resto dello stack. Non è raggiungibile da container esterni, anche se qualcos'altro nello stesso host venisse compromesso.

---

## Struttura del progetto

```
Neniku-s_DB/
├── backend/
│   ├── app/
│   │   ├── models/       # Modelli SQLAlchemy
│   │   ├── schemas/      # Schemi Pydantic
│   │   ├── routers/      # Endpoint per modulo
│   │   ├── parsers/      # Parser PDF per fornitore
│   │   ├── services/     # Generatore report PDF
│   │   ├── audit.py
│   │   ├── config.py
│   │   ├── database.py
│   │   └── main.py
│   ├── alembic/
│   ├── Dockerfile
│   └── requirements.txt
├── frontend/
│   ├── src/app/
│   │   ├── dashboard/
│   │   ├── prodotti/
│   │   ├── clienti/
│   │   ├── vendite/
│   │   ├── prenotazioni/
│   │   ├── fornitori/
│   │   ├── categorie/
│   │   ├── import-pdf/
│   │   ├── report/
│   │   ├── giacenza-bassa/
│   │   └── audit/
│   ├── Dockerfile
│   └── package.json
└── docker-compose.yml
```

---

## Alcune scelte tecniche

**Soft delete ovunque.** Prodotti, clienti e vendite non vengono mai eliminati fisicamente dal database. Questo garantisce integrità storica, possibilità di ripristino e coerenza nei report anche a distanza di mesi.

**Integrità a livello di database, non solo applicativo.** Vincoli, check constraint e indici parziali su PostgreSQL. Se il backend ha un bug e manda dati inconsistenti, il DB li rifiuta comunque.

**Nessun secret nel codice.** Tutto passa per variabili d'ambiente. Il `docker-compose.yml` pubblico usa solo placeholder.

**Timestamp timezone-aware** su tutte le tabelle (UTC). I report per data funzionano correttamente anche su fusi orari diversi.

---

## Stack

| | Tecnologia |
|---|---|
| Backend | Python 3.11 + FastAPI 0.115 |
| ORM / Migrations | SQLAlchemy 2.x + Alembic |
| Validazione | Pydantic v2 |
| Database | PostgreSQL 15 |
| PDF parsing | pdfplumber |
| PDF generation | ReportLab |
| Frontend | Next.js 14 (TypeScript) |
| Deploy | Docker + Compose |

---

## Note

Questa repository è una vetrina tecnica. Il codice sorgente completo, la documentazione operativa e i dettagli di deployment non sono pubblici.

Per una demo, per discutere di licensing o per qualsiasi altra cosa, contattami tramite i canali nel profilo GitHub.

---

*Tutti i diritti riservati. Non è consentita la copia, la distribuzione o la modifica senza autorizzazione scritta.*
