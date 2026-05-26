# LAB26 guidato - Progetto Maven e accesso JDBC a un catalogo corsi

## Scenario

Creiamo una piccola applicazione Java che legge dati da una tabella `corso` presente in MariaDB/MySQL.

L'obiettivo del laboratorio non è costruire un CRUD completo, ma rendere chiaro il percorso:

```text
Maven -> dipendenza driver -> connessione JDBC -> SELECT -> ResultSet -> oggetti Java
```

Nella UD25 abbiamo gestito manualmente driver e classpath. In questo laboratorio usiamo Maven per organizzare il progetto e gestire la dipendenza JDBC.

## Requisiti software

| Strumento | Uso nel laboratorio |
|---|---|
| JDK 17 o superiore | Compilazione ed esecuzione Java |
| Maven | Build e gestione dipendenze |
| MariaDB/MySQL | DBMS relazionale già disponibile localmente o su server condiviso |
| Terminale | Esecuzione comandi |
| Editor Java | Modifica file sorgente |

> Nota: Docker non è richiesto per questo laboratorio. Se nel proprio ambiente è già disponibile, può essere usato solo come alternativa per avviare il DBMS, ma le istruzioni principali assumono MariaDB/MySQL già disponibile.

## Parametri database usati negli esempi

Gli esempi usano questi parametri:

| Parametro | Valore |
|---|---|
| Host | `localhost` |
| Porta | `3307` |
| Database | `academy_ud26` |
| Utente | `academy` |
| Password | `academy_pwd` |

Se il proprio DBMS usa valori diversi, bisogna aggiornare la classe `DbConfig`.

## Parte 1 - Verifica ambiente

### Verifica JDK

Windows PowerShell, Linux e macOS:

```bash
java -version
javac -version
```

### Verifica Maven

Windows PowerShell, Linux e macOS:

```bash
mvn -version
```

Se il comando non viene riconosciuto, installare Maven prima di procedere.

### Installazione Maven su Windows

Da browser:
```bash
https://maven.apache.org/download.cgi
```
Con Chocolatey:
Installare Chocolatey se non presente
In PowerShell amministratore, incolla questo comando:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```
Questo è il comando indicato dalla documentazione ufficiale di Chocolatey per installare Chocolatey da PowerShell amministratore.

Al termine, chiudi PowerShell e riaprilo come amministratore.

Poi verifica:

```powershell
choco --version
```
Se risponde con un numero di versione, Chocolatey è installato.

A quel punto installi Maven:
```powershell
choco install maven -y
```
Apache Maven indica Chocolatey come uno dei metodi supportati su Windows.

Poi chiudi e riapri PowerShell e verifica:

```powershell
mvn -version
```


### Installazione Maven su Linux Ubuntu/Debian

```bash
sudo apt update
sudo apt install maven
mvn -version
```

### Installazione Maven su macOS

Con Homebrew:

```bash
brew install maven
mvn -version
```

## Parte 2 - Preparazione database

Creare il database e la tabella `corso` usando dbForge Studio, MySQL Workbench, phpMyAdmin oppure il client da terminale.

Script SQL:

```sql
CREATE DATABASE IF NOT EXISTS academy_ud26;
USE academy_ud26;

DROP TABLE IF EXISTS corso;

CREATE TABLE corso (
    id INT AUTO_INCREMENT PRIMARY KEY,
    codice VARCHAR(20) NOT NULL UNIQUE,
    titolo VARCHAR(120) NOT NULL,
    durata_ore INT NOT NULL,
    livello VARCHAR(30) NOT NULL
);

INSERT INTO corso(codice, titolo, durata_ore, livello) VALUES
('JAVA-BASE', 'Java Base', 40, 'base'),
('JAVA-OOP', 'Java Object Oriented', 48, 'intermedio'),
('JAVA-WEB', 'Java Web', 56, 'intermedio'),
('SPRING-BASE', 'Spring Boot Foundations', 40, 'avanzato');
```

### Esecuzione da terminale opzionale

Se il client `mariadb` è installato, si può salvare lo script in `sql/00_schema_corsi.sql` ed eseguirlo da terminale.

Windows PowerShell:

```powershell
cmd /c "mariadb -h localhost -P 3307 -u academy -p academy_ud26 < sql\00_schema_corsi.sql"
```

Linux/macOS:

```bash
mariadb -h localhost -P 3307 -u academy -p academy_ud26 < sql/00_schema_corsi.sql
```

Se il database non esiste ancora, eseguire prima lo script da uno strumento grafico oppure collegarsi come utente amministratore e creare database/utente secondo la configurazione del proprio ambiente.

## Parte 3 - Creazione progetto Maven

Creare una cartella di lavoro.

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force UD26_lab_guidato_jdbc_corsi
Set-Location UD26_lab_guidato_jdbc_corsi
New-Item -ItemType Directory -Force src/main/java/corso/ud26/corsi
New-Item -ItemType Directory -Force src/main/resources
New-Item -ItemType Directory -Force docs
New-Item -ItemType Directory -Force sql
```

Linux/macOS:

```bash
mkdir -p UD26_lab_guidato_jdbc_corsi
cd UD26_lab_guidato_jdbc_corsi
mkdir -p src/main/java/corso/ud26/corsi
mkdir -p src/main/resources
mkdir -p docs
mkdir -p sql
```

Creare il file `pom.xml` nella radice del progetto:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>corso.java</groupId>
    <artifactId>ud26-lab-guidato-jdbc-corsi</artifactId>
    <version>1.0.0</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.release>17</maven.compiler.release>
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
            </plugin>
        </plugins>
    </build>
</project>
```

Verificare che Maven riesca a leggere il progetto:

```bash
mvn validate
```

## Parte 4 - Classe di configurazione

Creare `src/main/java/corso/ud26/corsi/DbConfig.java`:

```java
package corso.ud26.corsi;

public class DbConfig {
    public static final String URL = readEnv("DB_URL", "jdbc:mariadb://localhost:3307/academy_ud26");
    public static final String USER = readEnv("DB_USER", "academy");
    public static final String PASSWORD = readEnv("DB_PASSWORD", "academy_pwd");

    private DbConfig() {
    }

    private static String readEnv(String name, String defaultValue) {
        String value = System.getenv(name);
        if (value == null || value.isBlank()) {
            return defaultValue;
        }
        return value;
    }
}
```

Questa classe centralizza i parametri di connessione. Se il database usa porta o credenziali diverse, si può modificare il valore predefinito oppure impostare variabili d'ambiente.


````md
## Nota sul metodo `readEnv`

Il metodo:

```java
private static String readEnv(String name, String defaultValue)
````

serve a leggere una variabile d’ambiente del sistema operativo.

* `name` indica il nome della variabile da cercare, ad esempio `DB_URL`, `DB_USER`, `DB_PASSWORD`.
* `defaultValue` indica il valore da usare se la variabile non esiste o è vuota.

La riga:

```java
String value = System.getenv(name);
```

chiede al sistema operativo il valore della variabile indicata.

Se la variabile non esiste oppure contiene solo spazi, viene restituito il valore predefinito:

```java
if (value == null || value.isBlank()) {
    return defaultValue;
}
```

Se invece la variabile esiste ed è valorizzata, viene restituito il valore trovato:

```java
return value;
```

In sintesi, il metodo permette di usare valori predefiniti nel laboratorio, ma consente anche di cambiare configurazione senza modificare il codice Java.

```
```
## Parte 5 - Test di connessione

Creare `src/main/java/corso/ud26/corsi/TestConnessione.java`:

```java
package corso.ud26.corsi;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class TestConnessione {
    public static void main(String[] args) {
        try (Connection conn = DriverManager.getConnection(
                DbConfig.URL,
                DbConfig.USER,
                DbConfig.PASSWORD)) {

            System.out.println("Connessione riuscita");
            System.out.println("Database: " + conn.getCatalog());

        } catch (SQLException e) {
            System.out.println("Errore di connessione");
            System.out.println(e.getMessage());
        }
    }
}
```

Eseguire:

```bash
mvn clean compile exec:java -Dexec.mainClass=corso.ud26.corsi.TestConnessione
```

## Parte 6 - Classe modello

Creare `src/main/java/corso/ud26/corsi/Corso.java`:

```java
package corso.ud26.corsi;

public class Corso {
    private int id;
    private String codice;
    private String titolo;
    private int durataOre;
    private String livello;

    public Corso(int id, String codice, String titolo, int durataOre, String livello) {
        this.id = id;
        this.codice = codice;
        this.titolo = titolo;
        this.durataOre = durataOre;
        this.livello = livello;
    }

    public int getId() {
        return id;
    }

    public String getCodice() {
        return codice;
    }

    public String getTitolo() {
        return titolo;
    }

    public int getDurataOre() {
        return durataOre;
    }

    public String getLivello() {
        return livello;
    }

    @Override
    public String toString() {
        return id + " - " + codice + " - " + titolo + " (" + durataOre + " ore, " + livello + ")";
    }
}
```

## Parte 7 - Lettura dati con SELECT

Creare `src/main/java/corso/ud26/corsi/ElencoCorsi.java`:

```java
package corso.ud26.corsi;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

public class ElencoCorsi {
    public static void main(String[] args) {
        List<Corso> corsi = findAll();

        for (Corso corso : corsi) {
            System.out.println(corso);
        }
    }

    private static List<Corso> findAll() {
        List<Corso> corsi = new ArrayList<>();
        String sql = "SELECT id, codice, titolo, durata_ore, livello FROM corso ORDER BY titolo";

        try (Connection conn = DriverManager.getConnection(DbConfig.URL, DbConfig.USER, DbConfig.PASSWORD);
             PreparedStatement ps = conn.prepareStatement(sql);
             ResultSet rs = ps.executeQuery()) {

            while (rs.next()) {
                Corso corso = new Corso(
                    rs.getInt("id"),
                    rs.getString("codice"),
                    rs.getString("titolo"),
                    rs.getInt("durata_ore"),
                    rs.getString("livello")
                );

                corsi.add(corso);
            }

        } catch (SQLException e) {
            System.out.println("Errore durante la lettura dei corsi");
            System.out.println(e.getMessage());
        }

        return corsi;
    }
}
```

Eseguire:
con Powershell:
```bash
mvn compile exec:java "-Dexec.mainClass=corso.ud26.corsi.ElencoCorsi"
```
con Linux/Mac:

```bash
mvn compile exec:java -Dexec.mainClass=corso.ud26.corsi.ElencoCorsi
```
## Parte 8 - Query parametrica

Creare `src/main/java/corso/ud26/corsi/CercaCorsiPerLivello.java`:

```java
package corso.ud26.corsi;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class CercaCorsiPerLivello {
    public static void main(String[] args) {
        String livello = "intermedio";
        String sql = "SELECT id, codice, titolo, durata_ore, livello FROM corso WHERE livello = ? ORDER BY titolo";

        try (Connection conn = DriverManager.getConnection(DbConfig.URL, DbConfig.USER, DbConfig.PASSWORD);
             PreparedStatement ps = conn.prepareStatement(sql)) {

            ps.setString(1, livello);

            try (ResultSet rs = ps.executeQuery()) {
                while (rs.next()) {
                    System.out.println(rs.getString("codice") + " - " + rs.getString("titolo"));
                }
            }

        } catch (SQLException e) {
            System.out.println("Errore durante la ricerca per livello");
            System.out.println(e.getMessage());
        }
    }
}
```

Eseguire:

```bash
mvn compile exec:java -Dexec.mainClass=corso.ud26.corsi.CercaCorsiPerLivello
```

## Parte 9 - Evidence richiesta

Creare `docs/evidence_LAB26_guidato.md`.

Contenuto richiesto:

```md
# Evidence LAB26 guidato

## Verifica Maven

Incollare output di:

```bash
mvn -version
```

## Verifica database

Indicare:

- host;
- porta;
- database;
- utente usato.

## Output test connessione

Incollare output di `TestConnessione`.

## Output elenco corsi

Incollare output di `ElencoCorsi`.

## Output ricerca per livello

Incollare output di `CercaCorsiPerLivello`.

## Domande

1. Quale file Maven contiene la dipendenza del driver JDBC?
2. Che cosa rappresenta un URL JDBC?
3. Perché si usa `PreparedStatement` invece della concatenazione di stringhe?
4. Quale parte del codice trasforma le righe del database in oggetti Java?
5. Quale problema pratico risolve Maven rispetto alla UD25?
```

## Criterio di successo

Il laboratorio è completato quando:

- Maven compila il progetto;
- la dipendenza JDBC viene scaricata correttamente;
- l'applicazione apre una connessione al database;
- almeno una `SELECT` restituisce dati;
- la query parametrica funziona;
- il flusso da query SQL a oggetto Java è chiaro.
