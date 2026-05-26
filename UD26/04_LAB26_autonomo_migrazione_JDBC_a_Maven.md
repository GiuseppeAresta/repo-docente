# LAB26 autonomo - Migrazione del progetto JDBC UD25 a Maven

## Scenario

Nella UD25 abbiamo realizzato un primo progetto JDBC senza Maven. Il progetto usava il driver MariaDB scaricato manualmente nella cartella `lib/`, comandi `javac` e `java -cp`, una struttura di cartelle costruita a mano e alcune classi dedicate alla configurazione, alla connessione, al modello dati e all'accesso alla tabella `docente`.

In questo laboratorio non riscriviamo da zero l'anagrafica docenti. L'obiettivo è trasformare il lavoro svolto nella UD25 in un progetto Maven più ordinato, più ripetibile e più vicino a una struttura professionale.

La novità principale non è aggiungere altre operazioni JDBC, ma capire come cambia l'organizzazione del progetto quando si introduce Maven.

---

## Obiettivo del laboratorio

Al termine del laboratorio saremo in grado di:

1. creare una struttura Maven standard;
2. dichiarare il driver JDBC come dipendenza nel `pom.xml`;
3. eliminare la gestione manuale del file `.jar` nella cartella `lib/`;
4. compilare il progetto con `mvn clean compile`;
5. eseguire il programma con `mvn exec:java`;
6. spostare la configurazione del database in `src/main/resources/database.properties`;
7. adattare le classi già viste nella UD25 alla nuova struttura Maven;
8. documentare le differenze tra progetto JDBC manuale e progetto Maven;
9. diagnosticare almeno un errore Maven o un errore di esecuzione.

---

## Punto di partenza

Il punto di partenza è il progetto realizzato nella UD25, in particolare le classi:

```text
AppConfig.java
DbConnectionFactory.java
Docente.java
DocenteRepository.java
EseguiDemoDocentiJdbc.java
```

Nel progetto UD25 la struttura era simile a questa:

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
  docs/
```

In questo laboratorio dovremo ottenere una struttura Maven:

```text
UD26_docenti_jdbc_maven/
  pom.xml
  src/
    main/
      java/
        corso/
          ud26/
            docenti/
              AppConfig.java
              DbConnectionFactory.java
              Docente.java
              DocenteRepository.java
              EseguiDemoDocentiJdbc.java
      resources/
        database.properties
  sql/
  docs/
```

---

## Indicazione importante

Questo è un laboratorio autonomo. Alcuni frammenti vengono forniti come guida, ma il completamento delle classi e l'adattamento del codice fanno parte dell'attività.

L'intento non è copoiare intere classi già pronte. È richiesto partire dal codice della UD25, adattarlo alla struttura Maven e verificare che il progetto compili ed esegua correttamente.

---

## Requisiti software

| Strumento | Uso nel laboratorio |
|---|---|
| JDK 17 o superiore | Compilazione ed esecuzione Java |
| Maven | Build, gestione dipendenze ed esecuzione |
| MariaDB/MySQL | Database relazionale |
| Terminale Windows PowerShell, Linux o macOS | Esecuzione comandi |
| Editor Java | Modifica sorgenti |



---

# Parte 1 - Analizzare il progetto UD25

Prima di creare il progetto Maven, rispondiamo a queste domande nel file di evidence.

1. In quale cartella si trovava il driver JDBC nella UD25?
2. Quale opzione dei comandi `javac` e `java` indicava il classpath?
3. Quale classe conteneva i parametri di connessione?
4. Quale classe apriva fisicamente la connessione al database?
5. Quale classe conteneva le query SQL?
6. Quale classe si occupava di avviare la demo?
7. Quale parte della UD25 dovrebbe essere semplificata da Maven?

Questa analisi serve a evitare una migrazione meccanica. Prima di cambiare struttura, dobbiamo capire che cosa stiamo spostando e perché.

---

# Parte 2 - Creare il progetto Maven

## Passo 1 - Creare la cartella principale

### Windows PowerShell

```powershell
New-Item -ItemType Directory -Force UD26_docenti_jdbc_maven
Set-Location UD26_docenti_jdbc_maven
```

### Linux/macOS

```bash
mkdir -p UD26_docenti_jdbc_maven
cd UD26_docenti_jdbc_maven
```

## Passo 2 - Creare la struttura Maven

### Windows PowerShell

```powershell
New-Item -ItemType Directory -Force src\main\java\corso\ud26\docenti
New-Item -ItemType Directory -Force src\main\resources
New-Item -ItemType Directory -Force sql
New-Item -ItemType Directory -Force docs
```

### Linux/macOS

```bash
mkdir -p src/main/java/corso/ud26/docenti
mkdir -p src/main/resources
mkdir -p sql
mkdir -p docs
```

## Passo 3 - Verificare la struttura

### Windows PowerShell

```powershell
tree /F
```

### Linux/macOS

```bash
find . -maxdepth 6 -type d
```

---

# Parte 3 - Creare il file `pom.xml`

Creare il file nella root del progetto:

```text
pom.xml
```

Il file deve dichiarare:

- coordinate del progetto;
- versione Java 17;
- codifica UTF-8;
- dipendenza MariaDB JDBC;
- plugin per eseguire la classe `main`.

## Struttura da completare

Completare il file seguente nei punti indicati.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>corso.ud26</groupId>
    <artifactId>docenti-jdbc-maven</artifactId>
    <version>1.0.0</version>

    <properties>
        <!-- completare con Java 17 -->
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- completare con la dipendenza MariaDB Connector/J -->
    </dependencies>

    <build>
        <plugins>
            <!-- completare con exec-maven-plugin -->
        </plugins>
    </build>

</project>
```

## Suggerimento per la dipendenza MariaDB

La dipendenza deve usare:

```text
groupId: org.mariadb.jdbc
artifactId: mariadb-java-client
version: 3.5.3
```

## Suggerimento per la classe principale

La classe principale sarà:

```text
corso.ud26.docenti.EseguiDemoDocentiJdbc
```

---

# Parte 4 - Preparare la configurazione esterna

Nella UD25 i parametri di connessione erano gestiti direttamente nel codice o tramite variabili d'ambiente.

In questa UD spostiamo la configurazione in un file:

```text
src/main/resources/database.properties
```

Creare il file e inserire valori coerenti con il proprio ambiente.

Esempio:

```properties
db.url=jdbc:mariadb://localhost:3306/academy_ud25
db.user=root
db.password=
```

Se il database usa una porta diversa, modificare l'URL.

Esempio con porta `3307`:

```properties
db.url=jdbc:mariadb://localhost:3307/academy_ud25
db.user=academy
db.password=academy
```

## Domande di controllo

Rispondere nel file di evidence.

1. Perché `database.properties` viene inserito in `src/main/resources`?
2. Perché è preferibile non scrivere sempre URL, utente e password direttamente nei metodi JDBC?
3. Quale differenza c'è tra un errore nel `pom.xml` e un errore nei parametri del database?

---

# Parte 5 - Migrare le classi Java

Copiare e adattare le classi della UD25 dentro:

```text
src/main/java/corso/ud26/docenti/
```

Le classi richieste sono:

```text
AppConfig.java
DbConnectionFactory.java
Docente.java
DocenteRepository.java
EseguiDemoDocentiJdbc.java
```

## Modifiche obbligatorie

Durante la migrazione ricordare di modificare il package.

Da:

```java
package corso.ud25.docenti;
```

A:

```java
package corso.ud26.docenti;
```

## Attenzione

Non copiare la cartella `lib/` della UD25. In UD26 il driver JDBC deve essere gestito da Maven tramite il `pom.xml`.

---

# Parte 6 - Adattare `AppConfig`

La classe `AppConfig` deve leggere il file `database.properties`.

Non viene fornita la soluzione completa. Completare la classe usando i suggerimenti.

## Struttura richiesta

```java
package corso.ud26.docenti;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class AppConfig {

    private final Properties properties = new Properties();

    public AppConfig() {
        // TODO:
        // 1. aprire database.properties usando il ClassLoader
        // 2. verificare che il file sia stato trovato
        // 3. caricare le proprietà con properties.load(input)
        // 4. gestire IOException con RuntimeException
    }

    public String getDbUrl() {
        // TODO: restituire la proprietà db.url
        return null;
    }

    public String getDbUser() {
        // TODO: restituire la proprietà db.user
        return null;
    }

    public String getDbPassword() {
        // TODO: restituire la proprietà db.password
        return null;
    }
}
```

## Suggerimenti

Per leggere un file presente in `src/main/resources` si può usare:

```java
InputStream input = AppConfig.class
        .getClassLoader()
        .getResourceAsStream("database.properties");
```

Per leggere una proprietà:

```java
properties.getProperty("db.url")
```

Per gestire il file non trovato:

```java
if (input == null) {
    throw new IllegalStateException("File database.properties non trovato");
}
```

---

# Parte 7 - Adattare `DbConnectionFactory`

La factory deve usare `AppConfig` per creare la connessione.

## Struttura richiesta

```java
package corso.ud26.docenti;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DbConnectionFactory {

    private final AppConfig config;

    public DbConnectionFactory(AppConfig config) {
        this.config = config;
    }

    public Connection getConnection() throws SQLException {
        // TODO:
        // usare DriverManager.getConnection(...)
        // con config.getDbUrl(), config.getDbUser(), config.getDbPassword()
        return null;
    }
}
```

## Domanda di controllo

Perché `DbConnectionFactory` riceve `AppConfig` dal costruttore invece di creare direttamente i valori di connessione al proprio interno?

Rispondere nel file di evidence.

---

# Parte 8 - Adattare `DocenteRepository`

In questo laboratorio non è necessario riscrivere tutto il CRUD.

Il repository deve contenere almeno:

```java
List<Docente> findAll();
Optional<Docente> findById(int id);
int countActive();
```

Se si vuole riutilizzare altro codice dalla UD25, è possibile mantenere anche metodi come `insert`, `updateEmail` o `deactivateById`, ma non sono il centro di questa attività.

## Regole tecniche obbligatorie

- usare `PreparedStatement` per le query;
- usare `try-with-resources`;
- non stampare risultati dentro il repository;
- restituire oggetti Java o valori semplici;
- mantenere il mapping in un metodo privato dedicato.

## Struttura suggerita

```java
public class DocenteRepository {

    private final DbConnectionFactory connectionFactory;

    public DocenteRepository(DbConnectionFactory connectionFactory) {
        this.connectionFactory = connectionFactory;
    }

    public List<Docente> findAll() {
        // TODO:
        // 1. scrivere la SELECT
        // 2. aprire Connection con try-with-resources
        // 3. creare PreparedStatement
        // 4. eseguire executeQuery()
        // 5. usare mapRow(rs)
        // 6. restituire la lista
        return null;
    }

    public Optional<Docente> findById(int id) {
        // TODO:
        // usare WHERE id = ?
        return Optional.empty();
    }

    public int countActive() {
        // TODO:
        // usare SELECT COUNT(*)
        return 0;
    }

    private Docente mapRow(ResultSet rs) throws SQLException {
        // TODO:
        // trasformare una riga del ResultSet in un oggetto Docente
        return null;
    }
}
```

## Query utili

Per leggere tutti i docenti:

```sql
SELECT id, nome, cognome, email, specializzazione, attivo
FROM docente
ORDER BY cognome, nome
```

Per cercare un docente per id:

```sql
SELECT id, nome, cognome, email, specializzazione, attivo
FROM docente
WHERE id = ?
```

Per contare i docenti attivi:

```sql
SELECT COUNT(*) AS totale
FROM docente
WHERE attivo = TRUE
```

---

# Parte 9 - Adattare la classe principale

La classe `EseguiDemoDocentiJdbc` deve:

1. creare `AppConfig`;
2. creare `DbConnectionFactory`;
3. creare `DocenteRepository`;
4. stampare tutti i docenti;
5. cercare un docente per id;
6. stampare il numero dei docenti attivi.

## Struttura suggerita

```java
public class EseguiDemoDocentiJdbc {

    public static void main(String[] args) {
        AppConfig config = new AppConfig();
        DbConnectionFactory connectionFactory = new DbConnectionFactory(config);
        DocenteRepository repository = new DocenteRepository(connectionFactory);

        // TODO:
        // stampare tutti i docenti
        // cercare un docente per id
        // stampare il numero di docenti attivi
    }
}
```

Il `main` può stampare a video, ma non deve contenere:

- SQL;
- `Connection`;
- `PreparedStatement`;
- `ResultSet`;
- mapping da colonne SQL a oggetti Java.

---

# Parte 10 - Compilare ed eseguire con Maven

## Verificare Maven

```bash
mvn -version
```

## Compilare

```bash
mvn clean compile
```

## Eseguire

Se nel `pom.xml` è stato configurato `exec-maven-plugin` con la classe principale:

```bash
mvn exec:java
```

In alternativa:

```bash
mvn exec:java -Dexec.mainClass=corso.ud26.docenti.EseguiDemoDocentiJdbc
```

---

# Parte 11 - Diagnostica obbligatoria

Per completare il laboratorio, bisogna provocare e correggere almeno un errore controllato.

Scegliere uno dei casi seguenti.

## Caso A - Classe principale errata nel `pom.xml`

Modificare temporaneamente il nome della classe principale nel plugin Maven.

Esempio:

```xml
<mainClass>corso.ud26.docenti.ClasseInesistente</mainClass>
```

Eseguire:

```bash
mvn exec:java
```

Annotare:

- messaggio di errore;
- causa;
- correzione applicata.

## Caso B - File `database.properties` mancante

Rinominare temporaneamente:

```text
src/main/resources/database.properties
```

in:

```text
src/main/resources/database.properties.bak
```

Eseguire il programma e annotare:

- messaggio di errore;
- perché la compilazione può riuscire comunque;
- perché l'errore compare a runtime.

Ripristinare poi il nome corretto.

## Caso C - Password o database errati

Modificare temporaneamente un valore nel file `database.properties`.

Annotare:

- messaggio di errore;
- differenza tra errore Maven ed errore di connessione al database;
- correzione applicata.

---

# Parte 12 - Evidence richiesta

Creare il file:

```text
docs/evidence_UD26_autonomo.md
```

Inserire le seguenti sezioni.

````md
# Evidence - LAB26 autonomo - Migrazione JDBC a Maven

## 1. Struttura finale del progetto

Incollare l'output del comando usato per mostrare la struttura.

```text

```

## 2. Contenuto essenziale del pom.xml

Incollare le sezioni principali:

- properties;
- dependency MariaDB;
- exec-maven-plugin.

```xml

```

## 3. Contenuto di database.properties

Non inserire password reali se usate credenziali personali.

```properties

```

## 4. Output di mvn -version

```text

```

## 5. Output di mvn clean compile

```text

```

## 6. Output di mvn exec:java

```text

```

## 7. Confronto UD25 / UD26

Compilare la tabella.

| Aspetto | UD25 senza Maven | UD26 con Maven |
|---|---|---|
| Driver JDBC |  |  |
| Compilazione |  |  |
| Esecuzione |  |  |
| Struttura sorgenti |  |  |
| Configurazione database |  |  |

## 8. Diagnostica errore

Descrivere l'errore provocato e corretto.

- Errore scelto:
- Messaggio osservato:
- Causa:
- Correzione:

## 9. Domande finali

1. Quale problema risolve Maven rispetto al classpath manuale?
2. Perché il driver MariaDB è una dipendenza Maven?
3. Perché `database.properties` viene inserito in `src/main/resources`?
4. Perché il repository non deve stampare risultati a video?
5. Quale parte del codice sarà migliorata nella UD27 con DAO e Service?
````

---

# Parte 13 - Criteri di completamento

Il laboratorio è completato se:

- il progetto ha una struttura Maven corretta;
- il file `pom.xml` contiene la dipendenza MariaDB;
- il progetto compila con `mvn clean compile`;
- il programma esegue con `mvn exec:java`;
- la configurazione viene letta da `database.properties`;
- il repository usa `PreparedStatement` e `try-with-resources`;
- il `main` non contiene SQL o codice JDBC diretto;
- almeno un errore è stato provocato, compreso e corretto;
- l'evidence documenta il confronto tra UD25 e UD26.

---

# Parte 14 - Collegamento alla UD27

In questa UD abbiamo mantenuto ancora un repository concreto, scritto direttamente con JDBC.

Nella UD27 il progetto verrà ulteriormente migliorato introducendo in modo più esplicito:

- DAO o repository come contratto;
- implementazione JDBC separata;
- service per le regole applicative;
- maggiore separazione tra accesso ai dati e logica dell'applicazione.

La UD26 serve quindi a rendere il progetto più ordinato dal punto di vista della struttura e della gestione delle dipendenze. La UD27 servirà a migliorare l'architettura applicativa.
