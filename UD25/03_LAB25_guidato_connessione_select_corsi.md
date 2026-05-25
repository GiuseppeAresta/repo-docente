# LAB25 guidato - Connessione JDBC, lettura corsi e verifica di COMMIT/ROLLBACK

## Scenario

L'academy ha un database MariaDB/MySQL con una tabella `corso` e una tabella `iscrizione`.

Il laboratorio costruisce un'applicazione Java senza Maven che:

1. si collega al database;
2. esegue una `SELECT`;
3. legge i dati con `ResultSet`;
4. trasforma le righe in oggetti Java;
5. stampa un report testuale;
6. verifica l'effetto di una modifica composta annullata con `ROLLBACK` e poi confermata con `COMMIT`.

La parte su `COMMIT` e `ROLLBACK` non serve ancora a scrivere codice transazionale in Java. Serve a capire il significato funzionale di una transazione: più modifiche collegate devono essere confermate insieme oppure annullate insieme.

## Requisiti software

| Strumento | Uso nel laboratorio |
|---|---|
| JDK 17 o superiore | Compilazione ed esecuzione Java |
| MariaDB/MySQL | Database relazionale |
| MariaDB Connector/J `.jar` | Driver JDBC |
| Terminale | Esecuzione comandi |
| Editor Java | Modifica sorgenti |

## Struttura del progetto

Creare la seguente struttura:

```text
UD25_catalogo_corsi_jdbc_senza_maven/
  lib/
    mariadb-java-client-3.5.3.jar
  src/
    corso/
      ud25/
        corsi/
          AppConfig.java
          TestConnessioneJdbc.java
          Corso.java
          CorsoRepository.java
          EseguiCatalogoCorsiJdbc.java
  sql/
    00_schema_corsi.sql
    01_seed_corsi.sql
  docs/
```

## Parte 1 - Preparazione database

Creare il file `sql/00_schema_corsi.sql`:

```sql
DROP DATABASE IF EXISTS academy_ud25;
CREATE DATABASE academy_ud25 CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE academy_ud25;

CREATE TABLE corso (
    id INT PRIMARY KEY AUTO_INCREMENT,
    codice VARCHAR(20) NOT NULL UNIQUE,
    titolo VARCHAR(100) NOT NULL,
    categoria VARCHAR(50) NOT NULL,
    durata_ore INT NOT NULL,
    prezzo DECIMAL(10,2) NOT NULL,
    posti_disponibili INT NOT NULL,
    attivo BOOLEAN NOT NULL DEFAULT TRUE
);

CREATE TABLE iscrizione (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome_partecipante VARCHAR(100) NOT NULL,
    corso_id INT NOT NULL,
    data_iscrizione DATE NOT NULL,
    CONSTRAINT fk_iscrizione_corso
        FOREIGN KEY (corso_id) REFERENCES corso(id)
);
```

Creare il file `sql/01_seed_corsi.sql`:

```sql
USE academy_ud25;

INSERT INTO corso (codice, titolo, categoria, durata_ore, prezzo, posti_disponibili, attivo) VALUES
('JAVA-BASE', 'Java Base', 'Java', 40, 900.00, 3, TRUE),
('SQL-BASE', 'SQL Operativo', 'Database', 24, 600.00, 4, TRUE),
('WEB-BASE', 'HTML CSS JavaScript', 'Web', 32, 700.00, 2, TRUE),
('OLD-01', 'Corso dismesso', 'Archivio', 16, 300.00, 0, FALSE);
```

Eseguire gli script sul database prima di avviare il codice Java.

Gli script possono essere eseguiti da uno strumento grafico, ad esempio dbForge Studio, MySQL Workbench o phpMyAdmin, oppure da terminale se il client `mysql` o `mariadb` è disponibile nel sistema.

Esempio da terminale, valido sia su Linux/macOS sia su Windows se il client è nel `PATH`:

```bash
mariadb -u academy -p < sql/00_schema_corsi.sql
mariadb -u academy -p < sql/01_seed_corsi.sql
```

In alternativa, se si usa il client MySQL:

```bash
mysql -u academy -p < sql/00_schema_corsi.sql
mysql -u academy -p < sql/01_seed_corsi.sql
```

## Parte 2 - Configurazione Java

Creare `AppConfig.java`:

```java
package corso.ud25.corsi;

public class AppConfig {
    public static final String DB_URL = "jdbc:mariadb://localhost:3306/academy_ud25";
    public static final String DB_USER = "academy";
    public static final String DB_PASSWORD = "academy_pwd";

    private AppConfig() {
    }
}
```

Se si usa XAMPP con utente `root` senza password, adattare temporaneamente:

```java
public static final String DB_USER = "root";
public static final String DB_PASSWORD = "";
```

## Parte 3 - Test connessione

Creare `TestConnessioneJdbc.java`:

```java
package corso.ud25.corsi;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class TestConnessioneJdbc {
    public static void main(String[] args) {
        try (Connection conn = DriverManager.getConnection(
                AppConfig.DB_URL,
                AppConfig.DB_USER,
                AppConfig.DB_PASSWORD)) {

            System.out.println("Connessione aperta correttamente");
            System.out.println("Database: " + conn.getCatalog());
        } catch (SQLException e) {
            System.out.println("Errore JDBC: " + e.getMessage());
        }
    }
}
```

Se il driver non è ancora nella cartella `lib`, scaricarlo dalla root del progetto.

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force lib
Invoke-WebRequest `
  -Uri "https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/3.5.3/mariadb-java-client-3.5.3.jar" `
  -OutFile ".\lib\mariadb-java-client-3.5.3.jar"
```

Linux/macOS:

```bash
mkdir -p lib
curl -L \
  "https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/3.5.3/mariadb-java-client-3.5.3.jar" \
  -o "lib/mariadb-java-client-3.5.3.jar"
```

Se `curl` non è disponibile, su molte distribuzioni Linux può essere usato anche `wget`:

```bash
mkdir -p lib
wget -O "lib/mariadb-java-client-3.5.3.jar" \
  "https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/3.5.3/mariadb-java-client-3.5.3.jar"
```

Verificare che il file sia presente in `lib/` prima di compilare.

### Compilazione ed esecuzione Linux/macOS

```bash
mkdir -p out
javac -encoding UTF-8 -cp "lib/mariadb-java-client-3.5.3.jar" -d out $(find src -name "*.java")
java -cp "out:lib/mariadb-java-client-3.5.3.jar" corso.ud25.corsi.TestConnessioneJdbc
```

### Compilazione ed esecuzione Windows PowerShell

```powershell
New-Item -ItemType Directory -Force out
$sources = Get-ChildItem -Recurse src -Filter *.java | ForEach-Object { $_.FullName }
javac -encoding UTF-8 -cp "lib\mariadb-java-client-3.5.3.jar" -d out $sources
java -cp "out;lib\mariadb-java-client-3.5.3.jar" corso.ud25.corsi.TestConnessioneJdbc
```

## Parte 4 - Classe model

Creare `Corso.java`:

```java
package corso.ud25.corsi;

public class Corso {
    private int id;
    private String codice;
    private String titolo;
    private String categoria;
    private int durataOre;
    private double prezzo;
    private int postiDisponibili;
    private boolean attivo;

    public Corso(int id, String codice, String titolo, String categoria,
            int durataOre, double prezzo, int postiDisponibili, boolean attivo) {
        this.id = id;
        this.codice = codice;
        this.titolo = titolo;
        this.categoria = categoria;
        this.durataOre = durataOre;
        this.prezzo = prezzo;
        this.postiDisponibili = postiDisponibili;
        this.attivo = attivo;
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

    public String getCategoria() {
        return categoria;
    }

    public int getDurataOre() {
        return durataOre;
    }

    public double getPrezzo() {
        return prezzo;
    }

    public int getPostiDisponibili() {
        return postiDisponibili;
    }

    public boolean isAttivo() {
        return attivo;
    }

    @Override
    public String toString() {
        return codice + " - " + titolo + " (" + categoria + ", "
                + durataOre + " ore, " + prezzo + " euro, posti disponibili: "
                + postiDisponibili + ")";
    }
}
```

## Parte 5 - Repository JDBC minimale

Creare `CorsoRepository.java`:

```java
package corso.ud25.corsi;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

public class CorsoRepository {

    public List<Corso> trovaCorsiAttivi() {
        List<Corso> corsi = new ArrayList<>();

        String sql = """
                SELECT id, codice, titolo, categoria, durata_ore, prezzo, posti_disponibili, attivo
                FROM corso
                WHERE attivo = TRUE
                ORDER BY titolo
                """;

        try (Connection conn = DriverManager.getConnection(
                    AppConfig.DB_URL,
                    AppConfig.DB_USER,
                    AppConfig.DB_PASSWORD);
             PreparedStatement ps = conn.prepareStatement(sql);
             ResultSet rs = ps.executeQuery()) {

            while (rs.next()) {
                Corso corso = new Corso(
                    rs.getInt("id"),
                    rs.getString("codice"),
                    rs.getString("titolo"),
                    rs.getString("categoria"),
                    rs.getInt("durata_ore"),
                    rs.getDouble("prezzo"),
                    rs.getInt("posti_disponibili"),
                    rs.getBoolean("attivo")
                );
                corsi.add(corso);
            }
        } catch (SQLException e) {
            System.out.println("Errore durante la lettura dei corsi: " + e.getMessage());
        }

        return corsi;
    }
}
```

Il repository contiene il codice JDBC necessario per accedere alla tabella `corso`. Nelle UD successive questa separazione verrà sviluppata in modo più completo con DAO, Service e Repository strutturati.

## Parte 6 - Programma principale

Creare `EseguiCatalogoCorsiJdbc.java`:

```java
package corso.ud25.corsi;

import java.util.List;

public class EseguiCatalogoCorsiJdbc {
    public static void main(String[] args) {
        CorsoRepository repository = new CorsoRepository();
        List<Corso> corsi = repository.trovaCorsiAttivi();

        System.out.println("Corsi attivi trovati: " + corsi.size());

        for (Corso corso : corsi) {
            System.out.println(corso);
        }
    }
}
```

Se sono stati creati nuovi file dopo il primo test di connessione, ricompilare il progetto.

Linux/macOS:

```bash
javac -encoding UTF-8 -cp "lib/mariadb-java-client-3.5.3.jar" -d out $(find src -name "*.java")
java -cp "out:lib/mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiCatalogoCorsiJdbc
```

Windows PowerShell:

```powershell
$sources = Get-ChildItem -Recurse src -Filter *.java | ForEach-Object { $_.FullName }
javac -encoding UTF-8 -cp "lib\mariadb-java-client-3.5.3.jar" -d out $sources
java -cp "out;lib\mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiCatalogoCorsiJdbc
```

Output atteso iniziale, con valori indicativi:

```text
Corsi attivi trovati: 3
JAVA-BASE - Java Base (Java, 40 ore, 900.0 euro, posti disponibili: 3)
SQL-BASE - SQL Operativo (Database, 24 ore, 600.0 euro, posti disponibili: 4)
WEB-BASE - HTML CSS JavaScript (Web, 32 ore, 700.0 euro, posti disponibili: 2)
```

## Parte 7 - Confermare o annullare una modifica composta sul database

In questa parte simuliamo l'iscrizione di un partecipante a un corso.

Dal punto di vista applicativo, iscrivere una persona non significa eseguire una sola modifica. L'operazione completa richiede almeno due modifiche collegate:

1. inserire una nuova riga nella tabella `iscrizione`;
2. diminuire i posti disponibili nella tabella `corso`.

Queste due modifiche devono essere trattate come una sola operazione logica. Non sarebbe corretto salvare l'iscrizione senza aggiornare i posti disponibili, né aggiornare i posti senza salvare l'iscrizione.

In questa UD eseguiamo la transazione dalla console SQL. Il programma Java viene usato per verificare lo stato letto dal database dopo `ROLLBACK` e dopo `COMMIT`.

### Caso A - Modifica annullata con ROLLBACK

Eseguire nel client SQL:

```sql
USE academy_ud25;

START TRANSACTION;

INSERT INTO iscrizione (nome_partecipante, corso_id, data_iscrizione)
VALUES ('Mario Rossi', 1, CURRENT_DATE);

UPDATE corso
SET posti_disponibili = posti_disponibili - 1
WHERE id = 1;

ROLLBACK;
```

Dopo il `ROLLBACK`, rieseguire il programma Java.

Linux/macOS:

```bash
java -cp "out:lib/mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiCatalogoCorsiJdbc
```

Windows PowerShell:

```powershell
java -cp "out;lib\mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiCatalogoCorsiJdbc
```

Il corso `JAVA-BASE` deve mostrare ancora lo stesso numero di posti disponibili iniziale.

Per verificare anche la tabella delle iscrizioni, eseguire:

```sql
SELECT * FROM iscrizione;
```

Non deve comparire l'iscrizione di Mario Rossi.

### Caso B - Modifica confermata con COMMIT

Eseguire nel client SQL:

```sql
USE academy_ud25;

START TRANSACTION;

INSERT INTO iscrizione (nome_partecipante, corso_id, data_iscrizione)
VALUES ('Mario Rossi', 1, CURRENT_DATE);

UPDATE corso
SET posti_disponibili = posti_disponibili - 1
WHERE id = 1;

COMMIT;
```

Dopo il `COMMIT`, rieseguire il programma Java.

Linux/macOS:

```bash
java -cp "out:lib/mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiCatalogoCorsiJdbc
```

Windows PowerShell:

```powershell
java -cp "out;lib\mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiCatalogoCorsiJdbc
```

Il corso `JAVA-BASE` deve mostrare un posto disponibile in meno.

Per verificare anche la tabella delle iscrizioni, eseguire:

```sql
SELECT * FROM iscrizione;
```

Questa volta deve comparire l'iscrizione di Mario Rossi.

## Significato del passaggio

Una transazione non è una singola query. È un confine entro cui più istruzioni possono essere confermate o annullate insieme.

Nel caso visto:

```text
iscrivere un partecipante
=
inserire una iscrizione
+
aggiornare i posti disponibili
```

Con `ROLLBACK` l'operazione viene annullata. Con `COMMIT` l'operazione diventa definitiva.

Nella UD31 riprenderemo lo stesso concetto in Spring: non scriveremo direttamente `START TRANSACTION`, `COMMIT` e `ROLLBACK`, ma useremo `@Transactional` sul metodo del service che rappresenta l'operazione applicativa completa.

## Evidence richiesta

Nel file `docs/evidence_UD25_guidato.md` inserire:

1. comando di compilazione usato;
2. comando di esecuzione del test connessione;
3. output del test connessione;
4. comando di esecuzione dell'elenco corsi;
5. output dell'elenco corsi prima della transazione;
6. script SQL eseguito con `ROLLBACK`;
7. output dell'elenco corsi dopo il `ROLLBACK`;
8. script SQL eseguito con `COMMIT`;
9. output dell'elenco corsi dopo il `COMMIT`;
10. spiegazione del ruolo del file `.jar` del driver;
11. risposta alla domanda: perché l'iscrizione e l'aggiornamento dei posti devono essere confermati o annullati insieme?
12. risposta alla domanda: quale problema risolverà Maven nella UD26?
