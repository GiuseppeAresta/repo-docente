# JDBC base: connessione, query e PreparedStatement

## 1. Che cos'è JDBC

JDBC significa Java Database Connectivity.

È l'API standard Java che permette a un'applicazione di comunicare con un database relazionale.

JDBC non è un DBMS e non è un ORM.

JDBC è il livello base attraverso cui Java invia comandi SQL al database e riceve risultati.

## 2. Componenti principali

| Componente | Ruolo |
|---|---|
| Driver JDBC | Libreria che conosce il protocollo del DBMS specifico |
| `DriverManager` | Punto di accesso per ottenere una connessione |
| `Connection` | Connessione aperta verso il database |
| `PreparedStatement` | Comando SQL parametrico |
| `ResultSet` | Risultato di una query `SELECT` |
| `SQLException` | Eccezione legata al database |

## 3. URL JDBC

Una connessione richiede un URL.

Esempio per MariaDB:

```text
jdbc:mariadb://localhost:3307/academy_ud26
```

Parti principali:

| Parte | Significato |
|---|---|
| `jdbc` | Protocollo generale JDBC |
| `mariadb` | Driver/subprotocollo |
| `localhost` | Host del DBMS |
| `3307` | Porta del DBMS |
| `academy_ud26` | Database di destinazione |

Se il database usa la porta standard MariaDB/MySQL, la porta potrebbe essere `3306` invece di `3307`.

## 4. Apertura di una connessione

Esempio base:

```java
String url = "jdbc:mariadb://localhost:3307/academy_ud26";
String user = "academy";
String password = "academy_pwd";

try (Connection conn = DriverManager.getConnection(url, user, password)) {
    System.out.println("Connessione aperta");
} catch (SQLException e) {
    System.out.println("Errore di connessione: " + e.getMessage());
}
```

Il blocco `try-with-resources` chiude automaticamente la connessione.

## 5. Perché usare `PreparedStatement`

Una query costruita concatenando stringhe è fragile e pericolosa.

Esempio da evitare:

```java
String livello = "intermedio";
String sql = "SELECT * FROM corso WHERE livello = '" + livello + "'";
```

La forma corretta usa un parametro:

```java
String livello = "intermedio";
String sql = "SELECT * FROM corso WHERE livello = ?";

try (PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.setString(1, livello);
}
```

Vantaggi:

- il codice è più leggibile;
- i valori sono separati dalla struttura SQL;
- si riduce il rischio di errori di sintassi;
- si evita la concatenazione diretta di input dentro SQL.

## 6. `executeQuery()` ed `executeUpdate()`

| Metodo | Uso principale | Risultato |
|---|---|---|
| `executeQuery()` | Query `SELECT` | Restituisce un `ResultSet` |
| `executeUpdate()` | `INSERT`, `UPDATE`, `DELETE`, `CREATE TABLE` | Restituisce il numero di righe modificate |

Esempio con `SELECT`:

```java
String sql = "SELECT id, titolo FROM corso";

try (PreparedStatement ps = conn.prepareStatement(sql);
     ResultSet rs = ps.executeQuery()) {

    while (rs.next()) {
        System.out.println(rs.getInt("id") + " - " + rs.getString("titolo"));
    }
}
```

Esempio con `INSERT`:

```java
String sql = "INSERT INTO corso(codice, titolo, durata_ore, livello) VALUES (?, ?, ?, ?)";

try (PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.setString(1, "JAVA-ADV");
    ps.setString(2, "Java Advanced");
    ps.setInt(3, 40);
    ps.setString(4, "avanzato");

    int righe = ps.executeUpdate();
    System.out.println("Righe inserite: " + righe);
}
```

## 7. Mapping manuale da riga DB a oggetto Java

Quando leggiamo un `ResultSet`, non otteniamo direttamente oggetti del nostro dominio.

Dobbiamo trasformare ogni riga in un oggetto Java.

Esempio:

```java
Corso corso = new Corso(
    rs.getInt("id"),
    rs.getString("codice"),
    rs.getString("titolo"),
    rs.getInt("durata_ore"),
    rs.getString("livello")
);
```

Questo passaggio si chiama mapping manuale.

Nelle UD successive vedremo che DAO, JPA e Spring Data renderanno questa parte più ordinata o più automatica.

## 8. Gestione delle risorse

Le risorse JDBC devono essere chiuse:

- `Connection`;
- `PreparedStatement`;
- `ResultSet`.

La forma consigliata è `try-with-resources`:

```java
try (Connection conn = DriverManager.getConnection(url, user, password);
     PreparedStatement ps = conn.prepareStatement(sql);
     ResultSet rs = ps.executeQuery()) {

    while (rs.next()) {
        System.out.println(rs.getString("titolo"));
    }
}
```

Questa forma evita di dimenticare connessioni aperte.

## 9. Dove mettere il codice JDBC

In questa UD iniziamo a separare il codice:

| Elemento | Responsabilità |
|---|---|
| Classe di configurazione | URL, utente, password |
| Factory di connessione | Apertura connessioni |
| Modello Java | Rappresentazione dei dati |
| Repository semplice | Query SQL e mapping |
| Classe di avvio | Esecuzione della demo |

Questa separazione prepara il passaggio alla UD27, dove il repository verrà riorganizzato come DAO più strutturato.

## 10. Collegamento con la UD25

Nella UD25 abbiamo gestito manualmente il driver `.jar` e il classpath.

In questa UD Maven scarica il driver e lo rende disponibile automaticamente.

Il codice JDBC resta simile, ma la struttura del progetto diventa più ordinata e più vicina alle applicazioni che svilupperemo con Spring.
