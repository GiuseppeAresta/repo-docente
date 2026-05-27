# 03 - LAB27 guidato: refactoring DAO per la gestione corsi

## Obiettivo

Trasformare un accesso JDBC scritto in modo diretto in una struttura più ordinata basata su DAO e service.

Il laboratorio guida la costruzione di:

- modello `Corso`;
- interfaccia `CorsoDao`;
- implementazione `JdbcCorsoDao`;
- factory di connessione;
- service applicativo;
- classe di avvio.

## Requisiti software

| Software/tool | Uso |
|---|---|
| JDK | Compilazione ed esecuzione Java |
| Maven | Gestione progetto e dipendenza JDBC |
| MariaDB/MySQL | Database del laboratorio |
| Client SQL | Esecuzione script SQL |
| Terminale | Comandi di verifica |

I comandi Maven sono uguali su Windows, Linux e macOS. Cambiano invece alcuni dettagli dei percorsi e l'eventuale comando usato per eseguire script SQL dal terminale.

## Verifica ambiente

Da terminale, verificare Java e Maven:

```bash
java -version
mvn -version
```

Verificare poi che MariaDB/MySQL sia avviato e accessibile. Gli esempi usano la porta `3306`, tipica di un'installazione locale. Se nel proprio ambiente il database usa un'altra porta, aggiornare l'URL JDBC.

## Scenario

L'academy gestisce un catalogo corsi. Ogni corso ha:

- id;
- codice;
- titolo;
- durata in ore;
- livello.

Partiamo da una situazione in cui il codice JDBC potrebbe essere scritto direttamente nel `main`. L'obiettivo è spostare l'accesso ai dati in una classe DAO e lasciare al service la logica applicativa.

## Passo 1 - Creazione del progetto Maven

Creare una cartella di lavoro, ad esempio `UD27_catalogo_corsi_dao`, e inizializzare la struttura del progetto.

### Windows PowerShell

```powershell
New-Item -ItemType Directory -Force UD27_catalogo_corsi_dao
Set-Location UD27_catalogo_corsi_dao
New-Item -ItemType Directory -Force src/main/java/corso/ud27/guidato/app
New-Item -ItemType Directory -Force src/main/java/corso/ud27/guidato/config
New-Item -ItemType Directory -Force src/main/java/corso/ud27/guidato/dao/jdbc
New-Item -ItemType Directory -Force src/main/java/corso/ud27/guidato/model
New-Item -ItemType Directory -Force src/main/java/corso/ud27/guidato/service
New-Item -ItemType Directory -Force sql docs
```

### Linux/macOS

```bash
mkdir -p UD27_catalogo_corsi_dao
cd UD27_catalogo_corsi_dao
mkdir -p src/main/java/corso/ud27/guidato/app
mkdir -p src/main/java/corso/ud27/guidato/config
mkdir -p src/main/java/corso/ud27/guidato/dao/jdbc
mkdir -p src/main/java/corso/ud27/guidato/model
mkdir -p src/main/java/corso/ud27/guidato/service
mkdir -p sql docs
```

Creare il file `pom.xml` nella radice del progetto.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>corso.ud27</groupId>
    <artifactId>catalogo-corsi-dao</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.release>17</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.mariadb.jdbc</groupId>
            <artifactId>mariadb-java-client</artifactId>
            <version>3.5.8</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>3.5.0</version>
                <configuration>
                    <mainClass>corso.ud27.guidato.app.EseguiCatalogoCorsiDao</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## Passo 2 - Script SQL minimo

Creare il file `sql/00_schema_corsi.sql`.

```sql
CREATE DATABASE IF NOT EXISTS academy_ud27_guidato;
USE academy_ud27_guidato;

DROP TABLE IF EXISTS corsi;

CREATE TABLE corsi (
    id INT PRIMARY KEY AUTO_INCREMENT,
    codice VARCHAR(20) NOT NULL UNIQUE,
    titolo VARCHAR(120) NOT NULL,
    durata_ore INT NOT NULL,
    livello VARCHAR(30) NOT NULL
);

INSERT INTO corsi (codice, titolo, durata_ore, livello) VALUES
('JAVA-BASE', 'Java Base', 40, 'base'),
('JAVA-OOP', 'Java Object Oriented', 48, 'intermedio'),
('JDBC-DAO', 'JDBC e DAO', 32, 'intermedio');
```

Eseguire lo script dal client SQL preferito. Se si usa il terminale, si possono usare questi comandi.

### Windows PowerShell

```powershell
Get-Content .\sql\00_schema_corsi.sql | mysql -u root -p
```

### Linux/macOS

```bash
mysql -u root -p < sql/00_schema_corsi.sql
```

Se il comando `mysql` non è disponibile nel terminale, eseguire lo script da dbForge, MySQL Workbench, DBeaver o dal client già utilizzato nelle UD precedenti.

## Passo 3 - Modello `Corso`

Creare il file `src/main/java/corso/ud27/guidato/model/Corso.java`.

```java
package corso.ud27.guidato.model;

public class Corso {
    private final int id;
    private final String codice;
    private final String titolo;
    private final int durataOre;
    private final String livello;

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

## Passo 4 - Configurazione connessione

Creare `src/main/java/corso/ud27/guidato/config/AppConfig.java`.

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

Se il database richiede utente e password diversi, configurare le variabili d'ambiente prima di avviare Maven.

### Windows PowerShell

```powershell
$env:UD27_GUIDATO_JDBC_URL="jdbc:mariadb://localhost:3306/academy_ud27_guidato"
$env:UD27_GUIDATO_DB_USER="root"
$env:UD27_GUIDATO_DB_PASSWORD="password"
```

### Linux/macOS

```bash
export UD27_GUIDATO_JDBC_URL="jdbc:mariadb://localhost:3306/academy_ud27_guidato"
export UD27_GUIDATO_DB_USER="root"
export UD27_GUIDATO_DB_PASSWORD="password"
```

## Passo 5 - Factory di connessione

Creare `src/main/java/corso/ud27/guidato/config/DbConnectionFactory.java`.

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

Questa classe centralizza la creazione delle connessioni. Il DAO userà questa factory invece di conoscere direttamente tutti i parametri.

## Passo 6 - Interfaccia DAO ed eccezione

Creare `src/main/java/corso/ud27/guidato/dao/CorsoDao.java`.

```java
package corso.ud27.guidato.dao;

import corso.ud27.guidato.model.Corso;

import java.util.List;
import java.util.Optional;

public interface CorsoDao {
    List<Corso> findAll();
    Optional<Corso> findById(int id);
}
```

Creare `src/main/java/corso/ud27/guidato/dao/DaoException.java`.

```java
package corso.ud27.guidato.dao;

public class DaoException extends RuntimeException {
    public DaoException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

## Passo 7 - Implementazione JDBC

Creare `src/main/java/corso/ud27/guidato/dao/jdbc/JdbcCorsoDao.java`.

```java
package corso.ud27.guidato.dao.jdbc;

import corso.ud27.guidato.config.DbConnectionFactory;
import corso.ud27.guidato.dao.CorsoDao;
import corso.ud27.guidato.dao.DaoException;
import corso.ud27.guidato.model.Corso;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class JdbcCorsoDao implements CorsoDao {
    private final DbConnectionFactory connectionFactory;

    public JdbcCorsoDao(DbConnectionFactory connectionFactory) {
        this.connectionFactory = connectionFactory;
    }

    @Override
    public List<Corso> findAll() {
        String sql = "SELECT id, codice, titolo, durata_ore, livello FROM corsi ORDER BY titolo";
        List<Corso> corsi = new ArrayList<>();

        try (Connection conn = connectionFactory.openConnection();
             PreparedStatement ps = conn.prepareStatement(sql);
             ResultSet rs = ps.executeQuery()) {

            while (rs.next()) {
                corsi.add(mapRow(rs));
            }

            return corsi;
        } catch (SQLException e) {
            throw new DaoException("Errore durante la lettura dei corsi", e);
        }
    }

    @Override
    public Optional<Corso> findById(int id) {
        String sql = "SELECT id, codice, titolo, durata_ore, livello FROM corsi WHERE id = ?";

        try (Connection conn = connectionFactory.openConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {

            ps.setInt(1, id);

            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    return Optional.of(mapRow(rs));
                }
                return Optional.empty();
            }
        } catch (SQLException e) {
            throw new DaoException("Errore durante la ricerca del corso con id " + id, e);
        }
    }

    private Corso mapRow(ResultSet rs) throws SQLException {
        return new Corso(
                rs.getInt("id"),
                rs.getString("codice"),
                rs.getString("titolo"),
                rs.getInt("durata_ore"),
                rs.getString("livello")
        );
    }
}
```

## Passo 8 - Service

Creare `src/main/java/corso/ud27/guidato/service/CatalogoCorsiService.java`.

```java
package corso.ud27.guidato.service;

import corso.ud27.guidato.dao.CorsoDao;
import corso.ud27.guidato.model.Corso;

import java.util.List;
import java.util.Optional;

public class CatalogoCorsiService {
    private final CorsoDao corsoDao;

    public CatalogoCorsiService(CorsoDao corsoDao) {
        this.corsoDao = corsoDao;
    }

    public List<Corso> elencoCorsi() {
        return corsoDao.findAll();
    }

    public Optional<Corso> cercaCorso(int id) {
        if (id <= 0) {
            throw new IllegalArgumentException("L'id del corso deve essere positivo");
        }
        return corsoDao.findById(id);
    }
}
```

Il service riceve un `CorsoDao`. Non crea direttamente `JdbcCorsoDao` e non conosce `PreparedStatement` o `ResultSet`.

## Passo 9 - Classe di avvio

Creare `src/main/java/corso/ud27/guidato/app/EseguiCatalogoCorsiDao.java`.

```java
package corso.ud27.guidato.app;

import corso.ud27.guidato.config.AppConfig;
import corso.ud27.guidato.config.DbConnectionFactory;
import corso.ud27.guidato.dao.CorsoDao;
import corso.ud27.guidato.dao.jdbc.JdbcCorsoDao;
import corso.ud27.guidato.model.Corso;
import corso.ud27.guidato.service.CatalogoCorsiService;

public class EseguiCatalogoCorsiDao {
    public static void main(String[] args) {
        AppConfig config = new AppConfig();
        DbConnectionFactory connectionFactory = new DbConnectionFactory(config);
        CorsoDao corsoDao = new JdbcCorsoDao(connectionFactory);
        CatalogoCorsiService service = new CatalogoCorsiService(corsoDao);

        System.out.println("Elenco corsi");
        for (Corso corso : service.elencoCorsi()) {
            System.out.println(corso);
        }

        System.out.println("\nRicerca corso con id 1");
        service.cercaCorso(1).ifPresentOrElse(
                System.out::println,
                () -> System.out.println("Corso non trovato")
        );
    }
}
```

## Passo 10 - Compilazione ed esecuzione

Da terminale, nella cartella del progetto:

```bash
mvn clean package
mvn exec:java
```

Se il database usa credenziali diverse, impostare prima le variabili d'ambiente come indicato nel Passo 4.

## Evidenza richiesta

Creare `docs/evidence_UD27_guidato.md` con:

1. struttura dei package;
2. differenza tra `CorsoDao` e `JdbcCorsoDao`;
3. motivo per cui il service non deve usare direttamente JDBC;
4. output dell'elenco corsi;
5. eventuali errori incontrati e relativa correzione.

## Domande di controllo

1. Cosa cambia se domani sostituiamo MariaDB con un repository in memoria?
2. Quale classe conosce `PreparedStatement`?
3. Quale classe applica le regole applicative?
4. Perché il mapping non deve stare nel `main`?
5. Dove viene applicato il polimorfismo in questo laboratorio?
