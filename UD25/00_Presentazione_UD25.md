# UD25 - JDBC - applicazioni senza utilizzo di Maven

## Contesto

Nella UD24 abbiamo lavorato su SQL e database relazionali. Con la UD25 passiamo dal database al codice Java.

L'obiettivo non è ancora costruire un'applicazione completa a livelli, ma capire quali elementi servono per collegare Java a MariaDB/MySQL:

- un database disponibile;
- un driver JDBC;
- una stringa di connessione;
- credenziali valide;
- codice Java che apre una connessione;
- istruzioni SQL eseguite tramite JDBC;
- lettura dei risultati tramite `ResultSet`;
- verifica pratica di una modifica composta confermata o annullata con `COMMIT` e `ROLLBACK`.

## Problema didattico della UD

Il JDK contiene le API JDBC, ma non contiene automaticamente il driver specifico per MariaDB/MySQL.

Per questo motivo il programma Java deve essere compilato ed eseguito rendendo disponibile il driver JDBC nel classpath.

In questa UD il driver viene gestito manualmente come file `.jar` nella cartella `lib/`.

## Risultati attesi

Al termine della UD saremo in grado di spiegare e usare i seguenti elementi.

| Elemento | Ruolo |
|---|---|
| JDBC | API standard Java per comunicare con database relazionali |
| Driver JDBC | Libreria specifica che permette a JDBC di parlare con il DBMS |
| MariaDB/MySQL | DBMS relazionale usato nei laboratori |
| URL JDBC | Indirizzo logico del database |
| `Connection` | Connessione aperta verso il database |
| `PreparedStatement` | Oggetto usato per eseguire SQL parametrizzato |
| `ResultSet` | Oggetto usato per leggere il risultato di una `SELECT` |

## Collegamento con il percorso

La progressione delle prossime UD sarà la seguente:

```text
UD25 - JDBC senza Maven
  ↓
UD26 - Maven e JDBC strutturato
  ↓
UD27 - DAO, Service e CRUD a livelli
  ↓
UD29/UD31 - API e applicazioni Spring
```

La UD25 serve a rendere visibile il problema tecnico che Maven semplificherà nella UD26: gestione del driver, classpath, struttura del progetto e comandi di compilazione/esecuzione.

## Sistemi operativi gestiti nei laboratori

I comandi dei laboratori sono riportati sia per Windows PowerShell sia per Linux/macOS.

La differenza più importante riguarda il separatore del classpath:

| Sistema operativo | Separatore classpath | Esempio |
|---|---|---|
| Windows | `;` | `out;lib\mariadb-java-client-3.5.3.jar` |
| Linux/macOS | `:` | `out:lib/mariadb-java-client-3.5.3.jar` |

Quando il comando è diverso tra sistemi operativi, il laboratorio riporta entrambe le versioni.

## Sequenza della UD

| Fase | Attività |
|---|---|
| Parte 1 | JDBC, driver, URL di connessione e classpath |
| Parte 2 | Laboratorio guidato: connessione, lettura dati e verifica di `COMMIT`/`ROLLBACK` su una modifica composta |
| Parte 3 | Laboratorio autonomo: anagrafica docenti con JDBC senza Maven |
| Parte 4 | Evidence, verifica e ponte verso Maven |


## Nota sul richiamo a COMMIT e ROLLBACK

Nel laboratorio guidato useremo una breve transazione SQL manuale per osservare quando una modifica composta diventa definitiva e quando viene annullata. Il caso resta semplice: iscrivere una persona a un corso significa registrare l'iscrizione e aggiornare i posti disponibili.

In questa UD Java non gestisce ancora direttamente la transazione: il programma JDBC viene usato per leggere lo stato del database prima e dopo `ROLLBACK` e `COMMIT`. La gestione applicativa più strutturata arriverà nelle UD successive.
