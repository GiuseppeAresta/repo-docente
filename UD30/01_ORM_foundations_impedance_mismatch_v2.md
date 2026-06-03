# ORM foundations: dal JDBC al mapping oggetto-relazionale

## Punto di partenza: cosa abbiamo fatto con JDBC

Con JDBC abbiamo scritto codice che lavora direttamente con:

- `Connection`;
- `PreparedStatement`;
- `ResultSet`;
- query SQL;
- mapping manuale da righe del database a oggetti Java.

Esempio semplificato:

```java
String sql = "SELECT id, titolo, prezzo FROM corsi WHERE id = ?";

try (Connection conn = connectionFactory.getConnection();
     PreparedStatement ps = conn.prepareStatement(sql)) {

    ps.setLong(1, id);

    try (ResultSet rs = ps.executeQuery()) {
        if (rs.next()) {
            Corso corso = new Corso(
                    rs.getLong("id"),
                    rs.getString("titolo"),
                    rs.getDouble("prezzo")
            );
            return Optional.of(corso);
        }
    }
}
```

Questo codice è esplicito e molto istruttivo, ma richiede di gestire manualmente molti dettagli:

- apertura e chiusura della connessione;
- SQL scritto nel codice;
- parametri della query;
- lettura del `ResultSet`;
- conversione delle colonne in attributi Java;
- gestione degli errori SQL.

Con JPA/Hibernate non eliminiamo il database. Cambiamo però il modo in cui descriviamo il collegamento tra oggetti Java e tabelle.

---

## Il problema: Java e database relazionale ragionano in modo diverso

Java lavora con oggetti.
Un database relazionale lavora con tabelle.

| Java | Database relazionale |
|---|---|
| classe | tabella |
| oggetto | riga |
| attributo | colonna |
| riferimento a un oggetto | chiave esterna |
| lista di oggetti | relazione uno-a-molti o molti-a-molti |
| metodo | non ha equivalente diretto |
| incapsulamento | non ha equivalente diretto |

Questa distanza viene spesso chiamata **object-relational impedance mismatch**.

Non serve memorizzare il termine per moda. Serve capire il problema: gli oggetti Java e le tabelle SQL rappresentano gli stessi dati con modelli diversi.

---

## Esempio: una edizione corso in Java

In Java possiamo rappresentare una edizione così:

```java
public class EdizioneCorso {
    private Long id;
    private Corso corso;
    private Docente docente;
    private LocalDate dataInizio;
    private LocalDate dataFine;
    private int postiMassimi;
}
```

Qui `corso` e `docente` sono riferimenti a oggetti Java.

Dal codice possiamo scrivere:

```java
String titolo = edizione.getCorso().getTitolo();
String docente = edizione.getDocente().getNomeCompleto();
```

Per Java questa è una normale navigazione tra oggetti.

---

## Esempio: la stessa edizione nel database

Nel database relazionale, invece, i dati sono distribuiti in più tabelle:

```sql
CREATE TABLE corsi (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    titolo VARCHAR(120) NOT NULL,
    prezzo DECIMAL(10, 2) NOT NULL
);

CREATE TABLE docenti (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(80) NOT NULL,
    cognome VARCHAR(80) NOT NULL,
    email VARCHAR(120) NOT NULL UNIQUE
);

CREATE TABLE edizioni_corso (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    corso_id BIGINT NOT NULL,
    docente_id BIGINT NOT NULL,
    data_inizio DATE NOT NULL,
    data_fine DATE NOT NULL,
    posti_massimi INT NOT NULL,
    FOREIGN KEY (corso_id) REFERENCES corsi(id),
    FOREIGN KEY (docente_id) REFERENCES docenti(id)
);
```

Nel database `edizioni_corso` non contiene direttamente un oggetto `Corso` o un oggetto `Docente`.
Contiene invece due chiavi esterne:

```text
corso_id
 docente_id
```

Queste colonne permettono di collegare la riga dell'edizione alle righe delle tabelle `corsi` e `docenti`.

---

## Cosa fa un ORM

ORM significa **Object-Relational Mapping**.

Un ORM aiuta a descrivere il collegamento tra:

```text
classi Java
↓
tabelle del database
```

Con un ORM possiamo dichiarare, ad esempio:

- questa classe è una entity persistente;
- questa classe corrisponde a una certa tabella;
- questo campo è la chiave primaria;
- questo campo è una colonna;
- questo riferimento Java corrisponde a una chiave esterna;
- questa relazione è molti-a-uno, uno-a-molti o molti-a-molti.

Esempio JPA/Hibernate:

```java
@Entity
@Table(name = "edizioni_corso")
public class EdizioneCorso {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "corso_id", nullable = false)
    private Corso corso;

    @ManyToOne
    @JoinColumn(name = "docente_id", nullable = false)
    private Docente docente;
}
```

Qui non stiamo eliminando il database. Stiamo descrivendo a JPA/Hibernate come collegare gli oggetti Java alle strutture relazionali.

---

## Confronto JDBC/JPA

### Lettura con JDBC

```java
String sql = "SELECT id, titolo, prezzo FROM corsi WHERE id = ?";

PreparedStatement ps = conn.prepareStatement(sql);
ps.setLong(1, id);
ResultSet rs = ps.executeQuery();
```

Poi leggiamo manualmente le colonne e costruiamo l'oggetto.

### Lettura con JPA

```java
Corso corso = entityManager.find(Corso.class, id);
```

Qui chiediamo a JPA di trovare una entity `Corso` con un certo identificativo.
JPA/Hibernate userà il mapping definito nelle annotazioni per costruire l'oggetto.

---

## Cosa non fa un ORM

Un ORM non sostituisce:

- la progettazione del database;
- la conoscenza di SQL;
- la comprensione di chiavi primarie e chiavi esterne;
- la separazione tra repository e service;
- la distinzione tra entity e DTO;
- la gestione consapevole delle transazioni;
- l'analisi delle query generate.

Un ORM semplifica parte del mapping e della persistenza, ma non rende inutile il ragionamento progettuale.

---

## Entity non significa DTO

Una **entity JPA** rappresenta un oggetto persistente collegato al database.

Un **DTO** rappresenta dati usati da uno specifico caso d'uso.

Esempio:

```text
Entity EdizioneCorso
- id
- corso
- docente
- dataInizio
- dataFine
- postiMassimi
- postiOccupati
- stato

DTO EdizioneCorsoResponseDto
- id
- titoloCorso
- nomeDocente
- dataInizio
- dataFine
- postiMassimi
- stato
```

Il DTO può aggregare informazioni provenienti da più entity.
Il DTO può anche nascondere campi tecnici o dettagli interni.

---

## Regola pratica

```text
Entity
serve alla persistenza.

DTO
serve allo scambio dati.

Mapper
converte entity in DTO.

Repository
usa JPA/Hibernate per accedere ai dati.

Service
coordina il caso d'uso e applica regole applicative.
```

Questa distinzione sarà mantenuta anche nella UD31 con Spring Boot e Spring Data JPA.
