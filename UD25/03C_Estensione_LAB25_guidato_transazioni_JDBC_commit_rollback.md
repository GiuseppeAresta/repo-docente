# Estensione LAB25 guidato - Gestire COMMIT e ROLLBACK da Java con JDBC

## Collocazione dell'estensione

Questa estensione si svolge dopo:

```text
03_LAB25_guidato_connessione_select_corsi.md
03B_Estensione_LAB25_guidato_consolidamento_JDBC.md
```

Nel laboratorio guidato abbiamo già visto come leggere dati dal database usando JDBC. In questa estensione facciamo un passo ulteriore: gestiamo da Java una piccola operazione composta, confermandola con `commit()` oppure annullandola con `rollback()`.

L'obiettivo non è ancora costruire un CRUD completo e non è ancora introdurre DAO o Service. In questa fase usiamo una classe Java semplice per capire il comportamento della transazione.

## Scenario funzionale

Nel database `academy_ud25` sono presenti le tabelle:

```text
corso
iscrizione
```

Dal punto di vista applicativo, iscrivere un partecipante a un corso non significa eseguire una sola modifica.

L'operazione completa richiede due azioni collegate:

1. inserire una riga nella tabella `iscrizione`;
2. diminuire di 1 il valore `posti_disponibili` nella tabella `corso`.

Queste due modifiche devono essere trattate come un'unica operazione logica. Se una parte riesce e l'altra fallisce, il database potrebbe restare in uno stato incoerente.

Esempio di stato incoerente:

```text
l'iscrizione risulta salvata
ma i posti disponibili non sono diminuiti
```

oppure:

```text
i posti disponibili sono diminuiti
ma l'iscrizione non risulta salvata
```

La transazione serve a evitare questi stati intermedi confermati.

## Collegamento con il passaggio precedente

Nel laboratorio precedente abbiamo visto la transazione manuale da SQL:

```sql
START TRANSACTION;
-- istruzioni SQL
COMMIT;
```

oppure:

```sql
START TRANSACTION;
-- istruzioni SQL
ROLLBACK;
```

Con JDBC il concetto è lo stesso, ma i comandi vengono gestiti tramite l'oggetto `Connection`.

| SQL manuale | JDBC |
|---|---|
| `START TRANSACTION` | `connection.setAutoCommit(false)` |
| `COMMIT` | `connection.commit()` |
| `ROLLBACK` | `connection.rollback()` |

Per impostazione predefinita, JDBC lavora in modalità `autoCommit = true`. Questo significa che ogni istruzione viene confermata automaticamente subito dopo l'esecuzione.

Per gestire una transazione manuale da Java dobbiamo disattivare temporaneamente l'autocommit:

```java
connection.setAutoCommit(false);
```

Da quel momento, le modifiche non diventano definitive fino a quando non viene chiamato:

```java
connection.commit();
```

Se invece vogliamo annullare le modifiche eseguite nella transazione, usiamo:

```java
connection.rollback();
```

## Passo 1 - Verificare lo stato iniziale del database

Prima di eseguire il codice Java, controlliamo lo stato del corso con `id = 1`.

Nel client SQL eseguire:

```sql
USE academy_ud25;

SELECT id, codice, titolo, posti_disponibili
FROM corso
WHERE id = 1;

SELECT id, nome_partecipante, corso_id, data_iscrizione
FROM iscrizione
WHERE corso_id = 1
ORDER BY id;
```

Se il laboratorio è stato eseguito più volte e si vuole ripartire da una situazione pulita, è possibile usare questo reset opzionale:

```sql
USE academy_ud25;

DELETE FROM iscrizione
WHERE nome_partecipante IN ('Laura Bianchi', 'Giulia Verdi');

UPDATE corso
SET posti_disponibili = 3
WHERE id = 1;
```

Il reset serve solo per ripetere l'esercizio in modo controllato.

## Passo 2 - Creare la classe Java per la transazione

Creare il file:

```text
src/corso/ud25/corsi/EseguiTransazioneIscrizioneJdbc.java
```

Inserire il seguente codice:

```java
package corso.ud25.corsi;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class EseguiTransazioneIscrizioneJdbc {

    public static void main(String[] args) {
        stampaStato("Stato iniziale");

        iscriviConRollbackSimulato("Laura Bianchi", 1);
        stampaStato("Dopo rollback simulato");

        iscriviConCommit("Giulia Verdi", 1);
        stampaStato("Dopo commit");
    }

    private static void iscriviConRollbackSimulato(String nomePartecipante, int corsoId) {
        System.out.println("\n--- Caso 1: iscrizione annullata con rollback ---");

        String inserisciIscrizione = """
                INSERT INTO iscrizione (nome_partecipante, corso_id, data_iscrizione)
                VALUES (?, ?, CURRENT_DATE())
                """;

        String aggiornaPosti = """
                UPDATE corso
                SET posti_disponibili = posti_disponibili - 1
                WHERE id = ?
                  AND posti_disponibili > 0
                """;

        try (Connection connection = DriverManager.getConnection(
                AppConfig.DB_URL,
                AppConfig.DB_USER,
                AppConfig.DB_PASSWORD)) {

            connection.setAutoCommit(false);

            try (
                    PreparedStatement psIscrizione = connection.prepareStatement(inserisciIscrizione);
                    PreparedStatement psPosti = connection.prepareStatement(aggiornaPosti)) {

                psIscrizione.setString(1, nomePartecipante);
                psIscrizione.setInt(2, corsoId);
                psIscrizione.executeUpdate();

                psPosti.setInt(1, corsoId);
                int righeAggiornate = psPosti.executeUpdate();

                if (righeAggiornate != 1) {
                    throw new SQLException("Nessun posto disponibile per il corso indicato");
                }

                throw new SQLException("Errore simulato prima del commit");

            } catch (SQLException errore) {
                connection.rollback();
                System.out.println("Rollback eseguito: iscrizione annullata.");
                System.out.println("Motivo: " + errore.getMessage());
            } finally {
                connection.setAutoCommit(true);
            }

        } catch (SQLException erroreConnessione) {
            System.out.println("Errore di connessione o gestione transazione: " + erroreConnessione.getMessage());
        }
    }

    private static void iscriviConCommit(String nomePartecipante, int corsoId) {
        System.out.println("\n--- Caso 2: iscrizione confermata con commit ---");

        String inserisciIscrizione = """
                INSERT INTO iscrizione (nome_partecipante, corso_id, data_iscrizione)
                VALUES (?, ?, CURRENT_DATE())
                """;

        String aggiornaPosti = """
                UPDATE corso
                SET posti_disponibili = posti_disponibili - 1
                WHERE id = ?
                  AND posti_disponibili > 0
                """;

        try (Connection connection = DriverManager.getConnection(
                AppConfig.DB_URL,
                AppConfig.DB_USER,
                AppConfig.DB_PASSWORD)) {

            connection.setAutoCommit(false);

            try (
                    PreparedStatement psIscrizione = connection.prepareStatement(inserisciIscrizione);
                    PreparedStatement psPosti = connection.prepareStatement(aggiornaPosti)) {

                psIscrizione.setString(1, nomePartecipante);
                psIscrizione.setInt(2, corsoId);
                psIscrizione.executeUpdate();

                psPosti.setInt(1, corsoId);
                int righeAggiornate = psPosti.executeUpdate();

                if (righeAggiornate != 1) {
                    throw new SQLException("Nessun posto disponibile per il corso indicato");
                }

                connection.commit();
                System.out.println("Commit eseguito: iscrizione confermata.");

            } catch (SQLException errore) {
                connection.rollback();
                System.out.println("Rollback eseguito: iscrizione annullata.");
                System.out.println("Motivo: " + errore.getMessage());
            } finally {
                connection.setAutoCommit(true);
            }

        } catch (SQLException erroreConnessione) {
            System.out.println("Errore di connessione o gestione transazione: " + erroreConnessione.getMessage());
        }
    }

    private static void stampaStato(String titolo) {
        System.out.println("\n=== " + titolo + " ===");

        String sqlCorso = """
                SELECT id, codice, titolo, posti_disponibili
                FROM corso
                WHERE id = 1
                """;

        String sqlIscrizioni = """
                SELECT id, nome_partecipante, corso_id, data_iscrizione
                FROM iscrizione
                WHERE corso_id = 1
                ORDER BY id
                """;

        try (Connection connection = DriverManager.getConnection(
                AppConfig.DB_URL,
                AppConfig.DB_USER,
                AppConfig.DB_PASSWORD);
             PreparedStatement psCorso = connection.prepareStatement(sqlCorso);
             PreparedStatement psIscrizioni = connection.prepareStatement(sqlIscrizioni);
             ResultSet rsCorso = psCorso.executeQuery()) {

            if (rsCorso.next()) {
                System.out.println("Corso: " + rsCorso.getString("codice")
                        + " - " + rsCorso.getString("titolo")
                        + " | posti disponibili: " + rsCorso.getInt("posti_disponibili"));
            }

            try (ResultSet rsIscrizioni = psIscrizioni.executeQuery()) {
                System.out.println("Iscrizioni registrate:");

                boolean presenti = false;
                while (rsIscrizioni.next()) {
                    presenti = true;
                    System.out.println("- " + rsIscrizioni.getInt("id")
                            + " | " + rsIscrizioni.getString("nome_partecipante")
                            + " | corso_id=" + rsIscrizioni.getInt("corso_id")
                            + " | data=" + rsIscrizioni.getDate("data_iscrizione"));
                }

                if (!presenti) {
                    System.out.println("Nessuna iscrizione presente per il corso indicato.");
                }
            }

        } catch (SQLException errore) {
            System.out.println("Errore durante la lettura dello stato: " + errore.getMessage());
        }
    }
}
```

## Passo 3 - Compilare il progetto

### Linux/macOS

Dalla root del progetto eseguire:

```bash
javac -encoding UTF-8 -cp "lib/mariadb-java-client-3.5.3.jar" -d out $(find src -name "*.java")
```

### Windows PowerShell

Dalla root del progetto eseguire:

```powershell
$sources = Get-ChildItem -Recurse src -Filter *.java | ForEach-Object { $_.FullName }
javac -encoding UTF-8 -cp "lib\mariadb-java-client-3.5.3.jar" -d out $sources
```

## Passo 4 - Eseguire la classe

### Linux/macOS

```bash
java -cp "out:lib/mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiTransazioneIscrizioneJdbc
```

### Windows PowerShell

```powershell
java -cp "out;lib\mariadb-java-client-3.5.3.jar" corso.ud25.corsi.EseguiTransazioneIscrizioneJdbc
```

## Passo 5 - Interpretare l'output

L'output deve mostrare tre momenti:

```text
Stato iniziale
Dopo rollback simulato
Dopo commit
```

Nel primo caso viene simulato un errore prima del `commit()`.

Il codice esegue:

```java
connection.rollback();
```

Dopo il rollback dobbiamo osservare che:

```text
Laura Bianchi non risulta iscritta
i posti disponibili non sono diminuiti
```

Nel secondo caso non viene simulato alcun errore.

Il codice esegue:

```java
connection.commit();
```

Dopo il commit dobbiamo osservare che:

```text
Giulia Verdi risulta iscritta
i posti disponibili sono diminuiti di 1
```

## Passo 6 - Verificare direttamente dal client SQL

Dopo l'esecuzione del programma Java, verificare il database anche dal client SQL:

```sql
USE academy_ud25;

SELECT id, codice, titolo, posti_disponibili
FROM corso
WHERE id = 1;

SELECT id, nome_partecipante, corso_id, data_iscrizione
FROM iscrizione
WHERE corso_id = 1
ORDER BY id;
```

Il risultato deve confermare quanto osservato dall'applicazione Java.

## Passo 7 - Cosa abbiamo imparato

Con questa estensione abbiamo visto che una transazione può essere gestita anche dal codice Java.

Il passaggio concettuale è questo:

| SQL manuale | JDBC |
|---|---|
| `START TRANSACTION` | `connection.setAutoCommit(false)` |
| `COMMIT` | `connection.commit()` |
| `ROLLBACK` | `connection.rollback()` |

Il codice Java non si limita più a leggere il database. In questa estensione esegue due modifiche collegate e decide se renderle definitive o annullarle.

Questa gestione resta comunque a livello JDBC. Nelle unità successive il codice verrà organizzato meglio usando classi dedicate all'accesso ai dati e, più avanti, con Spring il confine transazionale verrà dichiarato sul metodo del service tramite `@Transactional`.

## Evidence richiesta

Creare o aggiornare il file:

```text
docs/evidence_LAB25_guidato_transazioni_jdbc.md
```

Inserire:

1. output della classe `EseguiTransazioneIscrizioneJdbc`;
2. risultato della query SQL finale su `corso`;
3. risultato della query SQL finale su `iscrizione`;
4. breve risposta alle domande seguenti.

Domande:

```text
1. Perché è stato necessario disattivare l'autocommit?
2. Quali istruzioni vengono annullate nel caso con rollback?
3. Quali istruzioni vengono confermate nel caso con commit?
4. Perché l'iscrizione e l'aggiornamento dei posti devono essere trattati come un'unica operazione?
5. Quale differenza c'è tra eseguire COMMIT in SQL ed eseguire connection.commit() da Java?
```
