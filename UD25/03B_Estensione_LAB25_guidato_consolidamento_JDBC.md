# Estensione LAB25 guidato - Consolidamento JDBC, mapping e query parametrica

## Collocazione dell'estensione

Questa estensione si svolge dopo il completamento del laboratorio guidato `03_LAB25_guidato_connessione_select_corsi.md`.

Il progetto di partenza deve già contenere:

```text
UD25_catalogo_corsi_jdbc_senza_maven/
  lib/
  src/
  sql/
  docs/
```

Devono inoltre essere già presenti le classi:

```text
AppConfig.java
TestConnessioneJdbc.java
Corso.java
CorsoRepository.java
EseguiCatalogoCorsiJdbc.java
```

L'obiettivo non è introdurre il CRUD completo. In questa UD consolidiamo il primo utilizzo di JDBC: connessione, query, `PreparedStatement`, `ResultSet`, mapping verso oggetti Java e verifica dei dati letti dal database.

## Obiettivo dell'estensione

Al termine di questa estensione saremo in grado di:

- confrontare il risultato di una query eseguita nel client SQL con il risultato letto da Java;
- isolare il codice che trasforma una riga del `ResultSet` in un oggetto `Corso`;
- eseguire una query parametrica con `PreparedStatement`;
- distinguere un errore di connessione da un errore di query o di classpath;
- completare una breve evidence tecnica.

## Passo 1 - Verificare lo stesso dato da SQL e da Java

Prima di modificare il codice, verifichiamo direttamente il contenuto della tabella `corso`.

Nel client SQL eseguire:

```sql
USE academy_ud25;

SELECT id, codice, titolo, categoria, posti_disponibili, attivo
FROM corso
WHERE attivo = TRUE
ORDER BY titolo;
```

Eseguire poi il programma Java già realizzato nel laboratorio guidato.

Linux/macOS:

```bash
java -cp "out:lib/mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiCatalogoCorsiJdbc
```

Windows PowerShell:

```powershell
java -cp "out;lib\mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiCatalogoCorsiJdbc
```

Il risultato non deve essere identico nel formato, perché SQL mostra righe e colonne mentre Java stampa oggetti `Corso`, ma i dati devono rappresentare lo stesso stato del database.

Annotare nell'evidence:

```text
La query SQL legge direttamente le righe della tabella corso.
Il programma Java legge le stesse righe tramite JDBC e le trasforma in oggetti Corso.
```

## Passo 2 - Estrarre il mapping del ResultSet

Nel laboratorio guidato il mapping da `ResultSet` a `Corso` è stato scritto direttamente dentro il ciclo `while`.

Ora rendiamo più leggibile il repository creando un metodo privato dedicato al mapping.

Aprire `CorsoRepository.java` e aggiungere questo metodo dentro la classe:

```java
private Corso mappaCorso(ResultSet rs) throws SQLException {
    return new Corso(
        rs.getInt("id"),
        rs.getString("codice"),
        rs.getString("titolo"),
        rs.getString("categoria"),
        rs.getInt("durata_ore"),
        rs.getDouble("prezzo"),
        rs.getInt("posti_disponibili"),
        rs.getBoolean("attivo")
    );
}
```

Poi, nel metodo `trovaCorsiAttivi()`, sostituire la costruzione diretta dell'oggetto con:

```java
corsi.add(mappaCorso(rs));
```

Il ciclo diventa quindi:

```java
while (rs.next()) {
    corsi.add(mappaCorso(rs));
}
```

Questa modifica non cambia il comportamento del programma, ma rende più chiaro un passaggio importante: una riga del database non è ancora un oggetto Java. Il mapping è il punto in cui i valori letti dal database vengono copiati dentro un oggetto del programma.

Ricompilare ed eseguire.

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

## Passo 3 - Aggiungere una query parametrica per categoria

Ora aggiungiamo una lettura filtrata.

Dal punto di vista dell'utente, la richiesta è:

```text
Mostrare solo i corsi appartenenti a una determinata categoria.
```

Nel repository non dobbiamo costruire la query concatenando stringhe. Usiamo `PreparedStatement` con un parametro.

Aggiungere in `CorsoRepository.java` il metodo:

```java
public List<Corso> trovaCorsiPerCategoria(String categoria) {
    List<Corso> corsi = new ArrayList<>();

    String sql = """
            SELECT id, codice, titolo, categoria, durata_ore, prezzo, posti_disponibili, attivo
            FROM corso
            WHERE attivo = TRUE
              AND categoria = ?
            ORDER BY titolo
            """;

    try (Connection conn = DriverManager.getConnection(
                AppConfig.DB_URL,
                AppConfig.DB_USER,
                AppConfig.DB_PASSWORD);
         PreparedStatement ps = conn.prepareStatement(sql)) {

        ps.setString(1, categoria);

        try (ResultSet rs = ps.executeQuery()) {
            while (rs.next()) {
                corsi.add(mappaCorso(rs));
            }
        }
    } catch (SQLException e) {
        System.out.println("Errore durante la ricerca per categoria: " + e.getMessage());
    }

    return corsi;
}
```

Il simbolo `?` è un segnaposto. Il valore reale viene impostato con:

```java
ps.setString(1, categoria);
```

Il numero `1` indica il primo parametro della query.

## Passo 4 - Creare una classe di prova per la ricerca per categoria

Creare il file `EseguiRicercaCategoriaJdbc.java` nella stessa cartella delle altre classi:

```java
package corso.ud25.corsi;

import java.util.List;

public class EseguiRicercaCategoriaJdbc {
    public static void main(String[] args) {
        String categoria = args.length > 0 ? args[0] : "Java";

        CorsoRepository repository = new CorsoRepository();
        List<Corso> corsi = repository.trovaCorsiPerCategoria(categoria);

        System.out.println("Categoria ricercata: " + categoria);
        System.out.println("Corsi trovati: " + corsi.size());

        for (Corso corso : corsi) {
            System.out.println(corso);
        }
    }
}
```

Ricompilare tutto il progetto.

Linux/macOS:

```bash
javac -encoding UTF-8 -cp "lib/mariadb-java-client-3.5.3.jar" -d out $(find src -name "*.java")
```

Windows PowerShell:

```powershell
$sources = Get-ChildItem -Recurse src -Filter *.java | ForEach-Object { $_.FullName }
javac -encoding UTF-8 -cp "lib\mariadb-java-client-3.5.3.jar" -d out $sources
```

Eseguire la ricerca per categoria.

Linux/macOS:

```bash
java -cp "out:lib/mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiRicercaCategoriaJdbc Java
java -cp "out:lib/mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiRicercaCategoriaJdbc Database
```

Windows PowerShell:

```powershell
java -cp "out;lib\mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiRicercaCategoriaJdbc Java
java -cp "out;lib\mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiRicercaCategoriaJdbc Database
```

## Passo 5 - Controllare il risultato direttamente in SQL

Per verificare la correttezza della ricerca, eseguire nel client SQL:

```sql
SELECT codice, titolo, categoria
FROM corso
WHERE attivo = TRUE
  AND categoria = 'Java'
ORDER BY titolo;
```

Poi ripetere con:

```sql
SELECT codice, titolo, categoria
FROM corso
WHERE attivo = TRUE
  AND categoria = 'Database'
ORDER BY titolo;
```

Il numero di righe restituito dal client SQL deve corrispondere al numero di oggetti stampati dal programma Java.

## Passo 6 - Diagnosi rapida degli errori più frequenti

Quando il programma non funziona, prima di modificare il codice è utile individuare il tipo di errore.

| Sintomo | Possibile causa | Controllo consigliato |
|---|---|---|
| `No suitable driver` | Driver non trovato o classpath errato | Verificare che il `.jar` sia in `lib/` e che il comando `java -cp` lo includa |
| `Access denied for user` | Utente o password errati | Controllare `DB_USER` e `DB_PASSWORD` in `AppConfig` |
| `Unknown database` | Database non creato o nome errato | Verificare `academy_ud25` e l'URL JDBC |
| `Unknown column` | Nome colonna errato nella query | Confrontare la query con lo script `00_schema_corsi.sql` |
| Nessun risultato | Filtro troppo restrittivo o dato assente | Verificare la stessa condizione nel client SQL |

Annotare nell'evidence almeno un controllo eseguito e il relativo esito.

## Evidence integrativa

Aggiungere al file `docs/evidence_UD25_guidato.md` una sezione finale chiamata:

```markdown
## Estensione guidata - Query parametrica e mapping
```

Inserire:

1. query SQL usata per verificare i corsi per categoria;
2. comando Java usato per eseguire `EseguiRicercaCategoriaJdbc`;
3. output ottenuto per almeno due categorie;
4. breve spiegazione del metodo `mappaCorso`;
5. breve spiegazione del vantaggio di `PreparedStatement` rispetto alla concatenazione di stringhe;
6. un errore o controllo diagnostico analizzato durante l'attività.
