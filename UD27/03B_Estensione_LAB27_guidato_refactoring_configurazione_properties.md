# Estensione LAB27 guidato - Refactoring della configurazione con `database.properties`

## Collocazione dell'estensione

Questa estensione si svolge dopo il completamento del laboratorio guidato:

```text
03_LAB27_guidato_refactoring_dao_corsi.md
```

Nel laboratorio guidato abbiamo costruito una prima applicazione Maven con DAO e Service. La configurazione della connessione al database è stata gestita dalla classe `AppConfig`, che legge i valori da variabili d'ambiente e, se non le trova, usa valori predefiniti.

In questa estensione non cambiamo la logica DAO e non modifichiamo il comportamento del Service. Lavoriamo solo sul modo in cui l'applicazione legge i parametri di connessione.

L'obiettivo è trasformare questa configurazione:

```text
valori letti da variabili d'ambiente
```

in questa configurazione:

```text
valori letti da un file esterno database.properties
```

Il refactoring è utile perché rende più chiaro il ruolo della cartella `src/main/resources` nei progetti Maven e prepara il passaggio a configurazioni più evolute che incontreremo nelle applicazioni Spring.

---

## Obiettivi

Al termine dell'estensione saremo in grado di:

- creare un file `database.properties` in `src/main/resources`;
- leggere un file di configurazione usando `Properties`;
- usare `InputStream` per caricare il file;
- usare il `ClassLoader` per trovare il file nel classpath;
- modificare `AppConfig` senza cambiare `DbConnectionFactory`;
- verificare che il resto dell'applicazione continui a funzionare;
- distinguere configurazione, connessione, DAO e Service.

---

## Punto di partenza

Nel laboratorio guidato abbiamo una classe simile a questa:

```java
package corso.ud27.guidato.config;

public class AppConfig {
    public String getJdbcUrl() {
        return readEnv("UD27_GUIDATO_JDBC_URL", "jdbc:mariadb://localhost:3306/academy_ud27_guidato");
    }

    public String getUsername() {
        return readEnv("UD27_GUIDATO_DB_USER", "root");
    }

    public String getPassword() {
        return readEnv("UD27_GUIDATO_DB_PASSWORD", "");
    }

    private String readEnv(String name, String defaultValue) {
        String value = System.getenv(name);
        if (value == null || value.isBlank()) {
            return defaultValue;
        }
        return value;
    }
}
```

Questa versione funziona. Il problema non è tecnico, ma progettuale: i valori della configurazione sono ancora dentro il codice Java come valori predefiniti.

Nel refactoring spostiamo questi valori in un file esterno al codice sorgente.

---

# Parte 1 - Creare la cartella `resources`

Nei progetti Maven, la cartella:

```text
src/main/resources
```

contiene file non Java che devono essere disponibili durante l'esecuzione dell'applicazione.

Creare la cartella dalla radice del progetto.

## Windows PowerShell

```powershell
New-Item -ItemType Directory -Force src/main/resources
```

## Linux/macOS

```bash
mkdir -p src/main/resources
```

---

# Parte 2 - Creare il file `database.properties`

Creare il file:

```text
src/main/resources/database.properties
```

Inserire il seguente contenuto:

```properties
db.url=jdbc:mariadb://localhost:3306/academy_ud27_guidato
db.user=root
db.password=
```

Se il database usa una password diversa, modificare il valore di `db.password`.

Esempio:

```properties
db.password=password
```

## Significato delle proprietà

| Proprietà | Significato |
|---|---|
| `db.url` | URL JDBC del database |
| `db.user` | utente usato per collegarsi al database |
| `db.password` | password dell'utente |

Il file `database.properties` permette di modificare i parametri di connessione senza cambiare la classe Java.

---

# Parte 3 - Commentare temporaneamente il codice precedente

Aprire il file:

```text
src/main/java/corso/ud27/guidato/config/AppConfig.java
```

Prima di sostituire il codice, commentare temporaneamente la versione precedente.

Esempio:

```java
/*
Versione precedente basata su variabili d'ambiente.

public class AppConfig {
    public String getJdbcUrl() {
        return readEnv("UD27_GUIDATO_JDBC_URL", "jdbc:mariadb://localhost:3306/academy_ud27_guidato");
    }

    public String getUsername() {
        return readEnv("UD27_GUIDATO_DB_USER", "root");
    }

    public String getPassword() {
        return readEnv("UD27_GUIDATO_DB_PASSWORD", "");
    }

    private String readEnv(String name, String defaultValue) {
        String value = System.getenv(name);
        if (value == null || value.isBlank()) {
            return defaultValue;
        }
        return value;
    }
}
*/
```

Questa operazione serve solo durante il refactoring, per confrontare la vecchia versione con la nuova.

A fine esercizio è possibile rimuovere il codice commentato per mantenere il file pulito.

---

# Parte 4 - Sostituire `AppConfig` con la nuova versione

Sostituire il contenuto effettivo di `AppConfig.java` con il codice seguente:

```java
package corso.ud27.guidato.config;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class AppConfig {

    private final Properties properties = new Properties();

    public AppConfig() {
        try (InputStream input = AppConfig.class
                .getClassLoader()
                .getResourceAsStream("database.properties")) {

            if (input == null) {
                throw new IllegalStateException("File database.properties non trovato");
            }

            properties.load(input);

        } catch (IOException e) {
            throw new RuntimeException("Errore durante la lettura della configurazione", e);
        }
    }

    public String getJdbcUrl() {
        return properties.getProperty("db.url");
    }

    public String getUsername() {
        return properties.getProperty("db.user");
    }

    public String getPassword() {
        return properties.getProperty("db.password");
    }
}
```

---

# Parte 5 - Comprendere il nuovo codice

## `Properties`

La classe `Properties` permette di leggere file formati da coppie chiave-valore.

**È una Map?**

Tecnicamente, Properties deriva da Hashtable, quindi ha una parentela con le mappe Java.


```text
Properties è una classe Java usata per leggere configurazioni in formato chiave-valore.

È simile a una mappa: a ogni chiave corrisponde un valore.

Nel nostro caso la usiamo per leggere dal file database.properties i parametri di connessione al database.
```


```text
Properties non apre da sola il file: legge il contenuto da uno stream già aperto, cioè dall’InputStream.
```

Quindi la sequenza è:

````text
ClassLoader trova il file
↓
InputStream lo apre come flusso di lettura
↓
Properties.load(input) legge le coppie chiave-valore
↓
getProperty("db.url") recupera un valore
``` recupera un valore
````

## `InputStream`

`InputStream` rappresenta un flusso di lettura.

In questo caso viene usato per leggere il contenuto del file `database.properties`.

```java
try (InputStream input = AppConfig.class
        .getClassLoader()
        .getResourceAsStream("database.properties")) {
    ...
}
```

**questo codice equivale concettualmente a:**

- Class<AppConfig> classeDiRiferimento = AppConfig.class;
- ClassLoader caricatoreClassi = classeDiRiferimento.getClassLoader();
- String nomeFileConfigurazione = "database.properties";
- InputStream flussoLettura = caricatoreClassi.getResourceAsStream(nomeFileConfigurazione);

**Si potrebbe riscrivere anche così nel costruttore completo una versione più didattica:**

```java
public AppConfig() {

    // crea una variabile chiamata classeAppConfig che contiene il riferimento alla classe AppConfig, non a un oggetto AppConfig.
    Class<AppConfig> classeAppConfig = AppConfig.class;

    // prendi dalla classe AppConfig il ClassLoader che l'ha caricata e salvalo nella variabile classLoader
    ClassLoader classLoader = classeAppConfig.getClassLoader();

    String nomeFile = "database.properties";

    // usa il ClassLoader per cercare nel classpath una risorsa chiamata come il valore della variabile nomeFile e, se la trova, aprila come flusso di lettura.
    try (InputStream input = classLoader.getResourceAsStream(nomeFile)) {

        if (input == null) {
            throw new IllegalStateException("File database.properties non trovato");
        }

        properties.load(input);

    } catch (IOException e) {
        throw new RuntimeException("Errore durante la lettura della configurazione", e);
    }
}
```

Il blocco usa `try-with-resources`, quindi lo stream viene chiuso automaticamente alla fine del blocco.

## `ClassLoader`

Il `ClassLoader` viene usato per cercare il file nel classpath dell'applicazione.

Quando Maven compila il progetto, i file presenti in:

```text
src/main/resources
```

vengono copiati nel classpath di esecuzione.

Per questo motivo possiamo leggere il file con:

```java
getResourceAsStream("database.properties")
```

senza indicare un percorso assoluto del filesystem.

## Controllo sul file mancante

Questa parte:

```java
if (input == null) {
    throw new IllegalStateException("File database.properties non trovato");
}
```

serve a intercettare subito un errore frequente: il file non esiste, ha un nome diverso oppure non si trova nella cartella corretta.

---

# Parte 6 - Verificare `DbConnectionFactory`

Aprire il file:

```text
src/main/java/corso/ud27/guidato/config/DbConnectionFactory.java
```

La classe può rimanere uguale:

```java
package corso.ud27.guidato.config;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DbConnectionFactory {
    private final AppConfig config;

    public DbConnectionFactory(AppConfig config) {
        this.config = config;
    }

    public Connection openConnection() throws SQLException {
        return DriverManager.getConnection(
                config.getJdbcUrl(),
                config.getUsername(),
                config.getPassword()
        );
    }
}
```

Il punto importante è che `DbConnectionFactory` continua a usare i metodi pubblici di `AppConfig`:

```java
config.getJdbcUrl()
config.getUsername()
config.getPassword()
```

Quindi il modo in cui `AppConfig` ottiene i valori è cambiato, ma il resto dell'applicazione non deve essere modificato.

Questo è un esempio utile di incapsulamento: abbiamo cambiato l'implementazione interna di una classe senza cambiare il modo in cui le altre classi la usano.

---

# Parte 7 - Verificare il `main`

Aprire:

```text
src/main/java/corso/ud27/guidato/app/EseguiCatalogoCorsiDao.java
```

Il codice di avvio dovrebbe rimanere invariato:

```java
AppConfig config = new AppConfig();
DbConnectionFactory connectionFactory = new DbConnectionFactory(config);
CorsoDao corsoDao = new JdbcCorsoDao(connectionFactory);
CatalogoCorsiService service = new CatalogoCorsiService(corsoDao);
```

Anche qui si vede il vantaggio del refactoring: cambiamo la lettura della configurazione senza cambiare il codice che avvia l'applicazione.

---

# Parte 8 - Compilare ed eseguire

Eseguire:

```bash
mvn clean compile
mvn exec:java
```

I comandi Maven sono uguali su Windows, Linux e macOS.

Se l'applicazione mostra l'elenco corsi e la ricerca per id, il refactoring è riuscito.

---

# Parte 9 - Verificare che Maven copi le risorse

Dopo la compilazione, Maven copia i file di `src/main/resources` nella cartella `target/classes`.

Verificare la presenza del file.

## Windows PowerShell

```powershell
Get-ChildItem target\classes
```

## Linux/macOS

```bash
ls -l target/classes
```

Dovrebbe comparire:

```text
database.properties
```

Questa verifica aiuta a capire perché il file viene trovato dal `ClassLoader`.

---

# Parte 10 - Errore guidato: file non trovato

Per verificare il comportamento in caso di errore, rinominare temporaneamente il file:

```text
src/main/resources/database.properties
```

in:

```text
src/main/resources/database.properties.bak
```

Eseguire di nuovo:

```bash
mvn clean compile
mvn exec:java
```

L'applicazione dovrebbe generare un errore simile:

```text
File database.properties non trovato
```

Ripristinare poi il nome corretto:

```text
database.properties
```

e rieseguire:

```bash
mvn clean compile
mvn exec:java
```

---

# Parte 11 - Evidence richiesta

Aggiornare o creare il file:

```text
docs/evidence_UD27_refactoring_configurazione.md
```

Inserire le seguenti sezioni.

````md
# Evidence - Refactoring configurazione UD27

## 1. File `database.properties`

Incollare il contenuto del file, evitando di riportare password reali se diverse da quelle didattiche.

```properties

```

## 2. Nuova classe `AppConfig`

Incollare la nuova versione della classe o descrivere le modifiche principali.

```java

```

## 3. Verifica compilazione

Incollare l'output di:

```bash
mvn clean compile
```

```text

```

## 4. Verifica esecuzione

Incollare l'output di:

```bash
mvn exec:java
```

```text

```

## 5. Verifica risorsa copiata da Maven

Indicare se `database.properties` è presente in `target/classes`.

```text

```

## 6. Errore guidato

Descrivere cosa accade quando il file `database.properties` viene rinominato o rimosso temporaneamente.

```text

```

## 7. Domande

1. Perché `database.properties` si trova in `src/main/resources` e non in `src/main/java`?
2. Quale ruolo ha `Properties`?
3. Quale ruolo ha `InputStream`?
4. Perché usiamo il `ClassLoader`?
5. Perché `DbConnectionFactory` non ha dovuto cambiare?
6. In che modo questo refactoring migliora la separazione delle responsabilità?
````

---

# Parte 12 - Domande di controllo

## Domanda 1

Perché spostare i parametri di connessione fuori dal codice Java?

### Risposta attesa

Perché i parametri di connessione sono configurazione, non logica applicativa. Tenendoli in un file esterno possiamo modificarli senza cambiare e ricompilare la classe Java.

## Domanda 2

Perché il file viene letto con il `ClassLoader`?

### Risposta attesa

Perché `database.properties` si trova in `src/main/resources` e Maven lo rende disponibile nel classpath. Il `ClassLoader` permette di recuperare il file dal classpath senza usare un percorso assoluto del computer.

## Domanda 3

Cosa succede se il file `database.properties` non viene trovato?

### Risposta attesa

`getResourceAsStream` restituisce `null`. Il codice controlla questa condizione e genera una `IllegalStateException` con un messaggio esplicito.

## Domanda 4

Perché `DbConnectionFactory` può rimanere invariata?

### Risposta attesa

Perché `DbConnectionFactory` usa i metodi pubblici di `AppConfig`, non conosce il modo in cui `AppConfig` legge i dati. Questo permette di cambiare l'implementazione interna di `AppConfig` senza modificare le classi che la usano.

## Domanda 5

Quale responsabilità ha ora `AppConfig`?

### Risposta attesa

`AppConfig` ha la responsabilità di leggere i parametri di configurazione e renderli disponibili al resto dell'applicazione tramite metodi pubblici.

---

# Conclusione

Con questo refactoring abbiamo migliorato la separazione delle responsabilità:

```text
database.properties
    contiene i valori di configurazione

AppConfig
    legge la configurazione

DbConnectionFactory
    crea connessioni JDBC

JdbcCorsoDao
    usa connessioni e query SQL

CatalogoCorsiService
    applica logica applicativa

EseguiCatalogoCorsiDao
    avvia la demo
```

Il comportamento dell'applicazione non cambia, ma la struttura è più pulita. Questo passaggio prepara meglio il lavoro successivo su DAO, Service e separazione dei livelli applicativi.
