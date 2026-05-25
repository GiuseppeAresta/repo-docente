# JDBC: concetti base

## 1. Che cos'è JDBC

JDBC significa **Java Database Connectivity**.

È l'API standard Java che permette a un'applicazione di comunicare con un database relazionale.

JDBC non è un DBMS, non è un linguaggio SQL, non è un ORM e non è Maven. JDBC è il livello base con cui Java invia SQL al database e riceve risultati.

## 2. Perché non basta il JDK

Il JDK contiene classi e interfacce del package `java.sql`, ad esempio:

```java
Connection
DriverManager
PreparedStatement
ResultSet
SQLException
```

Queste classi definiscono il modo standard con cui Java lavora con i database relazionali.

Per collegarsi a un DBMS specifico serve però anche un driver. Nel nostro caso useremo MariaDB Connector/J:

```text
mariadb-java-client-3.5.3.jar
```

## 3. API JDBC e driver JDBC

La relazione tra codice Java, JDBC, driver e DBMS è questa:

```text
Codice Java
    ↓
API JDBC del JDK
    ↓
Driver JDBC MariaDB
    ↓
DBMS MariaDB/MySQL
```

L'API JDBC definisce le operazioni. Il driver contiene l'implementazione specifica necessaria per comunicare con MariaDB/MySQL.

## 4. URL JDBC

Per collegarsi al database serve un URL JDBC.

Esempio:

```text
jdbc:mariadb://localhost:3306/academy_ud25
```

| Parte | Significato |
|---|---|
| `jdbc` | indica l'uso del protocollo JDBC |
| `mariadb` | indica il driver da usare |
| `localhost` | indica che il DBMS è sulla macchina locale |
| `3306` | porta del DBMS |
| `academy_ud25` | database da usare |

## 5. Connessione JDBC

Esempio minimo:

```java
String url = "jdbc:mariadb://localhost:3306/academy_ud25";
String user = "academy";
String password = "academy_pwd";

try (Connection conn = DriverManager.getConnection(url, user, password)) {
    System.out.println("Connessione aperta");
} catch (SQLException e) {
    System.out.println("Errore JDBC: " + e.getMessage());
}
```

Il blocco `try-with-resources` chiude automaticamente la connessione quando il blocco termina.

## 6. PreparedStatement

`PreparedStatement` permette di eseguire una query SQL con parametri.

Esempio:

```java
String sql = "SELECT id, titolo, durata_ore FROM corso WHERE id = ?";

try (PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.setInt(1, idCorso);

    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) {
            String titolo = rs.getString("titolo");
            int durata = rs.getInt("durata_ore");
        }
    }
}
```

Il simbolo `?` rappresenta un parametro. Il valore viene impostato con metodi come `setInt`, `setString`, `setDouble`.

In questa UD le query devono usare `PreparedStatement`, non concatenazione manuale di stringhe.

## 7. ResultSet

`ResultSet` rappresenta il risultato di una query `SELECT`.

Ogni chiamata a:

```java
rs.next()
```

sposta il cursore sulla riga successiva.

Dalla riga corrente si leggono i valori:

```java
rs.getInt("id")
rs.getString("titolo")
rs.getInt("durata_ore")
```

Questi valori vengono poi trasformati in oggetti Java.

## 8. Questa UD non usa Maven

In questa UD il driver viene gestito manualmente per rendere visibile il problema delle dipendenze.

In questa UD verifichiamo concretamente che:

- il driver è un file esterno;
- il programma non funziona se il `.jar` non è nel classpath;
- il classpath cambia tra Windows e Linux/macOS;
- la gestione manuale è fragile in progetti più grandi.

La UD26 introdurrà Maven per superare questi limiti.
