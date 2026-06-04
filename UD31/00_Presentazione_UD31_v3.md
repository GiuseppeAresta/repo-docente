# UD31 - Spring Boot REST, Spring Data JPA e transazioni

## Presentazione dell'unità

In questa unità portiamo a sintesi il lavoro svolto nelle unità precedenti. Finora abbiamo costruito manualmente diversi pezzi di una applicazione Java backend:

- accesso al database con JDBC;
- organizzazione in DAO, repository e service;
- API HTTP con handler manuali;
- DTO di richiesta e risposta;
- conversione JSON con Jackson;
- persistenza JPA/Hibernate con `EntityManager`;
- transazioni manuali con JDBC e JPA.

Con Spring Boot vediamo come questi concetti vengono integrati in una applicazione più vicina a un contesto professionale.

L'obiettivo non è imparare annotazioni a memoria, ma capire come Spring Boot prende in carico molte attività che finora abbiamo gestito manualmente.

---

## Collegamento con le unità precedenti

| Unità precedente | Cosa abbiamo fatto manualmente | Evoluzione in UD31 |
|---|---|---|
| UD25 | JDBC, `Connection`, `PreparedStatement`, `commit`, `rollback` | transazioni dichiarative con `@Transactional` |
| UD26 | Maven, dipendenze, configurazione esterna | progetto Spring Boot Maven con `application.properties` |
| UD27 | DAO, Service, CRUD JDBC | Repository Spring Data e Service Spring |
| UD28 | form, dati lato browser, payload JSON | client che può comunicare con API REST |
| UD29 | API HTTP manuale, DTO, Jackson esplicito | Controller REST e Jackson automatico |
| UD30 | Entity JPA, `persistence.xml`, `EntityManager`, repository JPA manuale | Spring Data JPA e gestione automatica della persistenza |

La UD31 va letta come una trasformazione: non si riparte da zero, ma si riorganizzano concetti già studiati in un contesto Spring Boot.

---

## Obiettivi formativi

Al termine della UD31 saremo in grado di:

1. comprendere il ruolo di Spring Boot in una applicazione backend;
2. distinguere container Spring, bean e dependency injection;
3. usare controller REST con DTO di input e output;
4. comprendere come Spring Boot usa Jackson per convertire automaticamente JSON e oggetti Java;
5. configurare l'accesso al database tramite `application.properties`;
6. usare entity JPA con repository Spring Data;
7. distinguere entity, DTO, mapper, service e repository;
8. applicare regole applicative nel service;
9. usare `@Transactional` per gestire transazioni dichiarative;
10. testare endpoint REST con `curl`, PowerShell o strumenti analoghi;
11. spiegare il passaggio da JPA manuale a Spring Data JPA.

---


## Risultato atteso

Alla fine della UD31 non avremo solo una applicazione che risponde a richieste HTTP. Avremo una applicazione organizzata secondo una struttura a livelli:

```text
Client HTTP
↓
Controller REST
↓
DTO di richiesta
↓
Service
↓
Repository Spring Data
↓
Entity JPA
↓
Database
```

Il flusso di ritorno sarà:

```text
Database
↓
Entity
↓
Mapper
↓
DTO di risposta
↓
Controller REST
↓
JSON HTTP
```

Questo schema rappresenta la sintesi pratica del percorso Java backend.
