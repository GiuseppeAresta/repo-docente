# Maven, file POM e dipendenze JDBC

## 1. Introduzione a Maven

Nella UD25 abbiamo collegato Java a MariaDB/MySQL gestendo manualmente il driver JDBC, il classpath, la compilazione e l'esecuzione. Questo passaggio è stato utile perché ha mostrato cosa serve davvero a un programma Java per comunicare con un database.

In un progetto reale, però, quella modalità diventa presto fragile. Ogni libreria esterna deve essere scaricata a mano, ogni comando deve contenere il classpath corretto e ogni ambiente di lavoro rischia di avere una configurazione diversa.

Maven nasce per risolvere questo problema: fornisce una struttura standard al progetto, dichiara le librerie necessarie e permette di compilare ed eseguire il codice con comandi ripetibili.

Il punto importante è questo:

```text
Maven non sostituisce JDBC.
Maven organizza il progetto e gestisce le dipendenze necessarie per usare JDBC.
```

Nel nostro caso Maven ci permette di non copiare manualmente il driver MariaDB/MySQL nella cartella `lib`, ma di dichiararlo nel file `pom.xml`.

---

## 2. Che cos'è il file POM

Il file `pom.xml` è il file centrale di un progetto Maven. Il nome POM significa **Project Object Model**.

In pratica, il `pom.xml` descrive il progetto e dice a Maven:

- come si chiama il progetto;
- quale versione del progetto stiamo costruendo;
- quali librerie esterne servono;
- quale versione di Java usare;
- quali plugin usare per compilare o avviare l'applicazione.

Il `pom.xml` non contiene codice Java. Contiene la configurazione con cui Maven costruisce il progetto.

Una struttura minima può essere questa:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>corso.ud26</groupId>
    <artifactId>academy-jdbc-maven</artifactId>
    <version>1.0.0</version>

</project>
```

Questa configurazione non aggiunge ancora librerie esterne, ma definisce l'identità minima del progetto.

---

## 3. Coordinate Maven: groupId, artifactId e version

Ogni progetto Maven viene identificato da tre informazioni principali.

| Elemento | Significato | Esempio |
|---|---|---|
| `groupId` | Identifica il gruppo, l'organizzazione o il dominio logico del progetto | `corso.ud26` |
| `artifactId` | Identifica il nome tecnico del progetto o modulo | `academy-jdbc-maven` |
| `version` | Identifica la versione del progetto | `1.0.0` |

Queste tre informazioni vengono spesso chiamate **coordinate Maven**.

Non sono solo etichette. Maven le usa per identificare il progetto in modo univoco, soprattutto quando un progetto diventa a sua volta una libreria usata da altri progetti.

Per il nostro percorso formativo è sufficiente usare nomi semplici e coerenti con l'unità didattica.

Esempio:

```xml
<groupId>corso.ud26</groupId>
<artifactId>academy-jdbc-maven</artifactId>
<version>1.0.0</version>
```

---

## 4. Struttura standard di un progetto Maven

Maven si basa su una struttura di cartelle convenzionale. Questo evita di dover specificare ogni volta dove si trovano i sorgenti, le risorse e i test.

La struttura minima è:

```text
academy-jdbc-maven/
  pom.xml
  src/
    main/
      java/
        corso/
          ud26/
            Main.java
      resources/
    test/
      java/
```

I percorsi principali hanno un significato preciso.

| Percorso | Significato |
|---|---|
| `pom.xml` | Configurazione Maven del progetto |
| `src/main/java` | Codice Java dell'applicazione |
| `src/main/resources` | File di configurazione e risorse usate dall'applicazione |
| `src/test/java` | Codice dei test |
| `target/` | Cartella generata da Maven durante compilazione e build |

La cartella `target/` non va normalmente modificata a mano. Viene creata da Maven e può essere eliminata e rigenerata.

---

## 5. La versione di Java nel POM

Per evitare comportamenti diversi tra ambienti, è opportuno indicare nel `pom.xml` la versione di Java da usare per compilare.

Esempio:

```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

Significato:

| Proprietà | Significato |
|---|---|
| `maven.compiler.source` | Versione del linguaggio Java usata nei sorgenti |
| `maven.compiler.target` | Versione del bytecode generato |
| `project.build.sourceEncoding` | Codifica dei file sorgente |

Nel nostro percorso useremo Java 17 o superiore. Dichiararlo nel POM rende il progetto più prevedibile.

---

## 6. Le dipendenze Maven

Una dipendenza è una libreria esterna di cui il progetto ha bisogno.

Nel caso JDBC con MariaDB/MySQL, il progetto ha bisogno del driver JDBC. Nella UD25 il driver era un file `.jar` scaricato e inserito manualmente nel classpath. In Maven, invece, lo dichiariamo nella sezione `<dependencies>`.

Esempio con MariaDB Connector/J:

```xml
<dependencies>
    <dependency>
        <groupId>org.mariadb.jdbc</groupId>
        <artifactId>mariadb-java-client</artifactId>
        <version>3.5.3</version>
    </dependency>
</dependencies>
```

La dipendenza ha anch'essa coordinate Maven:

| Elemento | Significato |
|---|---|
| `groupId` | Gruppo che pubblica la libreria |
| `artifactId` | Nome della libreria |
| `version` | Versione della libreria |

Quando eseguiamo un comando Maven, Maven legge il `pom.xml`, scarica la libreria richiesta e la rende disponibile al progetto.

---

## 7. Dove Maven scarica le librerie

Maven scarica le librerie da repository remoti, come Maven Central, e le salva in una repository locale sul computer.

La repository locale si trova normalmente in:

```text
~/.m2/repository
```

Su Windows il percorso corrisponde in genere a:

```text
C:\Users\<nome-utente>\.m2\repository
```

Non è necessario modificare manualmente questa cartella durante il laboratorio. È utile però sapere che Maven non scarica le librerie dentro il progetto, ma le conserva in un'area locale condivisa tra più progetti.

Questo significa che due progetti diversi possono usare la stessa libreria senza doverne conservare una copia in ogni cartella.

---

## 8. Dipendenze dirette e transitive

Una dipendenza diretta è una libreria che dichiariamo esplicitamente nel `pom.xml`.

Una dipendenza transitiva è una libreria richiesta da una delle nostre dipendenze.

Esempio concettuale:

```text
Il nostro progetto
  ↓ dipendenza diretta
MariaDB JDBC Driver
  ↓ eventuali dipendenze transitive
altre librerie necessarie al driver
```

Maven calcola automaticamente l'insieme delle librerie necessarie.

Per visualizzare l'albero delle dipendenze si può usare:

```bash
mvn dependency:tree
```

Questo comando è utile quando il progetto cresce e vogliamo capire quali librerie sono entrate nel classpath.

---

## 9. Classpath manuale e classpath gestito da Maven

Nella UD25 abbiamo gestito il classpath manualmente. Il comando poteva avere una forma simile a questa.

Windows PowerShell:

```powershell
java -cp "out;lib\mariadb-java-client-3.5.3.jar" corso.ud25.Main
```

Linux/macOS:

```bash
java -cp "out:lib/mariadb-java-client-3.5.3.jar" corso.ud25.Main
```

La differenza tra `;` e `:` è solo uno dei motivi per cui il classpath manuale diventa scomodo.

Con Maven non scriviamo normalmente il classpath a mano. Dichiariamo la dipendenza nel `pom.xml` e usiamo comandi Maven.

Esempi:

```bash
mvn compile
```

```bash
mvn exec:java
```

Il vantaggio non è solo scrivere meno comandi. Il vantaggio è avere un progetto più ripetibile: la stessa configurazione può essere usata da persone diverse e su sistemi operativi diversi.

---

## 10. Comandi Maven essenziali

In questa UD useremo pochi comandi.

| Comando | Significato |
|---|---|
| `mvn validate` | Verifica che il progetto sia formalmente corretto |
| `mvn compile` | Compila il codice sorgente |
| `mvn test` | Esegue i test, se presenti |
| `mvn package` | Crea il pacchetto finale, ad esempio un `.jar` |
| `mvn clean` | Elimina la cartella `target/` generata in precedenza |
| `mvn clean compile` | Pulisce e compila da zero |
| `mvn exec:java` | Avvia una classe Java tramite plugin configurato |

Durante il laboratorio useremo soprattutto:

```bash
mvn clean compile
```

per compilare il progetto, e:

```bash
mvn exec:java
```

per eseguire la classe principale quando il plugin sarà configurato.

---

## 11. Plugin Maven e dipendenze: differenza importante

Nel `pom.xml` possiamo trovare sia dipendenze sia plugin. Non sono la stessa cosa.

Una **dipendenza** è una libreria usata dal codice Java.

Un **plugin** è uno strumento usato da Maven durante la build o l'esecuzione.

Esempio:

| Elemento | A cosa serve |
|---|---|
| Driver MariaDB JDBC | Serve al codice Java per collegarsi al database |
| `maven-compiler-plugin` | Serve a Maven per compilare il codice |
| `exec-maven-plugin` | Serve a Maven per avviare una classe `main` |

Quindi:

```text
Le dipendenze servono all'applicazione.
I plugin servono a Maven per costruire o avviare l'applicazione.
```

---

## 12. Esempio di POM completo per JDBC

Un POM adatto al nostro laboratorio può avere questa forma:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>corso.ud26</groupId>
    <artifactId>academy-jdbc-maven</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.mariadb.jdbc</groupId>
            <artifactId>mariadb-java-client</artifactId>
            <version>3.5.3</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>3.5.0</version>
                <configuration>
                    <mainClass>corso.ud26.corsi.EseguiCatalogoCorsi</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

Questo file consente di:

- compilare il progetto con Java 17;
- usare il driver MariaDB JDBC;
- avviare la classe principale con Maven.

Nel laboratorio il nome della classe principale dovrà corrispondere alla classe realmente creata nel progetto.

---

## 13. Cosa succede quando eseguiamo `mvn compile`

Quando eseguiamo:

```bash
mvn compile
```

Maven esegue una serie di operazioni:

1. legge il `pom.xml`;
2. verifica la struttura del progetto;
3. scarica le dipendenze mancanti;
4. compila i file presenti in `src/main/java`;
5. scrive il risultato nella cartella `target/classes`.

Il codice sorgente resta in `src/main/java`. Il risultato della compilazione non va scritto manualmente: viene generato da Maven.

---

## 14. Cosa succede quando eseguiamo `mvn exec:java`

Quando eseguiamo:

```bash
mvn exec:java
```

Maven usa il plugin `exec-maven-plugin` per avviare una classe Java.

Per funzionare correttamente, il plugin deve sapere quale classe contiene il metodo `main`.

Esempio:

```xml
<mainClass>corso.ud26.corsi.EseguiCatalogoCorsi</mainClass>
```

Se il nome del package o della classe non coincide, l'esecuzione fallisce.

Errore tipico:

```text
ClassNotFoundException
```

In quel caso bisogna verificare:

- il package dichiarato nella classe Java;
- il percorso del file dentro `src/main/java`;
- il valore indicato in `<mainClass>` nel `pom.xml`.

---

## 15. Relazione tra Maven, JDBC e database

È importante distinguere i tre livelli.

| Livello | Responsabilità |
|---|---|
| Maven | Organizza progetto, compilazione, dipendenze |
| JDBC | Permette al codice Java di inviare SQL al database |
| Database | Conserva dati, tabelle, vincoli e risultati delle query |

Maven non apre connessioni al database. Maven rende disponibile il driver.

JDBC non crea automaticamente una struttura pulita del progetto. JDBC è l'API usata dal codice.

Il database non conosce Maven. Riceve query SQL attraverso la connessione aperta dall'applicazione Java.

Il flusso complessivo è:

```text
pom.xml
  ↓ dichiara il driver JDBC
Maven
  ↓ scarica e rende disponibile la libreria
Codice Java
  ↓ usa JDBC
Driver JDBC
  ↓ comunica con il DBMS
MariaDB/MySQL
```

---

## 16. Errori frequenti sul POM

Quando il progetto non compila o non si avvia, il problema può trovarsi nel POM.

| Problema | Possibile causa | Controllo |
|---|---|---|
| Maven non trova la dipendenza | Coordinate errate o rete non disponibile | Verificare `groupId`, `artifactId`, `version` |
| Il progetto compila ma non si avvia | Classe `main` errata nel plugin | Verificare `<mainClass>` |
| Versione Java non supportata | JDK installato troppo vecchio | Verificare `java -version` |
| Caratteri accentati visualizzati male | Codifica non dichiarata | Verificare `project.build.sourceEncoding` |
| Errore su tag XML | POM non ben formato | Verificare apertura e chiusura dei tag |

Un errore nel `pom.xml` blocca Maven prima ancora di arrivare al codice Java.

---

## 17. Cosa dobbiamo saper spiegare al termine di questa parte

Al termine di questa parte dobbiamo essere in grado di spiegare:

- perché Maven semplifica la gestione delle librerie;
- che cosa rappresenta il file `pom.xml`;
- dove si dichiarano le dipendenze;
- perché il driver JDBC è una dipendenza;
- perché Maven non sostituisce JDBC;
- che differenza c'è tra dipendenza e plugin;
- che cosa succede quando eseguiamo `mvn compile`.

Questa base serve per affrontare il laboratorio guidato: creeremo un progetto Maven, aggiungeremo il driver JDBC e useremo Java per leggere dati da MariaDB/MySQL in modo più ordinato rispetto alla UD25.
