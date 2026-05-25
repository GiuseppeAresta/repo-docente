# LAB25 autonomo - Anagrafica docenti con JDBC

## Scenario

L'academy vuole gestire una piccola anagrafica dei docenti su database.

In questo laboratorio si realizza una piccola applicazione Java che usa JDBC per eseguire operazioni di base sulla tabella `docente`.

La struttura deve rimanere semplice, ma deve già preparare il passaggio alla UD26 e alla successiva introduzione di DAO e Service.

## Requisiti software

| Strumento | Uso nel laboratorio |
|---|---|
| JDK 17 o superiore | Compilazione ed esecuzione Java |
| MariaDB/MySQL | Database relazionale |
| MariaDB Connector/J `.jar` | Driver JDBC |
| Terminale | Esecuzione comandi |
| Editor Java | Modifica sorgenti |

## Obiettivo tecnico

Realizzare un'applicazione Java che:

1. si connette al database;
2. legge tutti i docenti;
3. cerca un docente per id;
4. inserisce un nuovo docente;
5. aggiorna l'email di un docente;
6. disattiva un docente;
7. stampa un report finale.

## Struttura richiesta

```text
UD25_anagrafica_docenti_jdbc/
  lib/
    mariadb-java-client-3.5.3.jar
  src/
    corso/
      ud25/
        docenti/
          AppConfig.java
          DbConnectionFactory.java
          Docente.java
          DocenteRepository.java
          EseguiDemoDocentiJdbc.java
  sql/
    00_schema_docenti.sql
    01_seed_docenti.sql
  docs/
    evidence_UD25_autonomo.md
```

## Responsabilità delle classi

| Classe | Responsabilità |
|---|---|
| `AppConfig` | Centralizza URL, utente e password |
| `DbConnectionFactory` | Crea connessioni JDBC |
| `Docente` | Rappresenta il dato docente in memoria |
| `DocenteRepository` | Contiene metodi JDBC per la tabella `docente` |
| `EseguiDemoDocentiJdbc` | Avvia una demo automatica dell'applicazione |

## Database richiesto

Database:

```text
academy_ud25
```

Tabella:

```sql
CREATE TABLE docente (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(50) NOT NULL,
    cognome VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    area_competenza VARCHAR(80) NOT NULL,
    attivo BOOLEAN NOT NULL DEFAULT TRUE
);
```

## Metodi minimi del repository

Il repository deve contenere almeno:

```java
List<Docente> trovaTutti();
Docente cercaPerId(int id);
void inserisci(Docente docente);
void aggiornaEmail(int id, String nuovaEmail);
void disattiva(int id);
```

I metodi devono usare `PreparedStatement` e `try-with-resources`.


## Preparazione del driver JDBC

Il file del driver deve essere disponibile nella cartella `lib/`:

```text
lib/mariadb-java-client-3.5.3.jar
```

Su Windows PowerShell, dalla root del progetto:

```powershell
New-Item -ItemType Directory -Force lib
Invoke-WebRequest `
  -Uri "https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/3.5.3/mariadb-java-client-3.5.3.jar" `
  -OutFile ".\lib\mariadb-java-client-3.5.3.jar"
```

Su Linux/macOS, dalla root del progetto:

```bash
mkdir -p lib
curl -L \
  "https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/3.5.3/mariadb-java-client-3.5.3.jar" \
  -o "lib/mariadb-java-client-3.5.3.jar"
```

Se `curl` non è disponibile, usare `wget`:

```bash
mkdir -p lib
wget \
  "https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/3.5.3/mariadb-java-client-3.5.3.jar" \
  -O "lib/mariadb-java-client-3.5.3.jar"
```

## Comandi di compilazione ed esecuzione

### Linux/macOS

```bash
mkdir -p out
javac -encoding UTF-8 -cp "lib/mariadb-java-client-3.5.3.jar" -d out $(find src -name "*.java")
java -cp "out:lib/mariadb-java-client-3.5.3.jar" corso.ud25.docenti.EseguiDemoDocentiJdbc
```

### Windows PowerShell

```powershell
New-Item -ItemType Directory -Force out
$sources = Get-ChildItem -Recurse src -Filter *.java | ForEach-Object { $_.FullName }
javac -encoding UTF-8 -cp "lib\mariadb-java-client-3.5.3.jar" -d out $sources
java -cp "out;lib\mariadb-java-client-3.5.3.jar" corso.ud25.docenti.EseguiDemoDocentiJdbc
```

## Evidence richiesta

Nel file `docs/evidence_UD25_autonomo.md` inserire:

1. struttura finale del progetto;
2. comandi usati per compilare;
3. comandi usati per eseguire;
4. output della demo;
5. spiegazione del ruolo di `DbConnectionFactory`;
6. spiegazione del ruolo del driver `.jar`;
7. risposta alla domanda: quale parte di questo progetto verrà semplificata da Maven nella UD26?

## Criteri minimi di accettazione

Il laboratorio è accettabile se:

- il codice compila;
- la connessione al database funziona;
- le query usano `PreparedStatement`;
- le risorse JDBC sono chiuse con `try-with-resources`;
- la struttura separa configurazione, model, repository e programma principale;
- l'evidence documenta i comandi realmente usati.
