# JPA/Hibernate operativo: annotazioni, persistence unit ed EntityManager

## Collegamento con i file precedenti

Finora abbiamo chiarito tre passaggi:

```text
ORM
serve a collegare oggetti Java e tabelle relazionali.

JPA
è la specifica standard per la persistenza degli oggetti Java.

Hibernate
è il provider che useremo per eseguire concretamente il mapping e le operazioni sul database.
```

Ora vediamo gli elementi operativi che compariranno nel laboratorio:

- annotazioni JPA;
- `persistence.xml`;
- persistence unit;
- `EntityManagerFactory`;
- `EntityManager`;
- transazione locale JPA.

---

## Entity JPA completa: esempio di partenza

```java
package corso.ud30.guidato.model;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "corsi")
public class Corso {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 120)
    private String titolo;

    @Column(nullable = false)
    private double prezzo;

    protected Corso() {
        // Costruttore richiesto da JPA.
    }

    public Corso(String titolo, double prezzo) {
        this.titolo = titolo;
        this.prezzo = prezzo;
    }

    public Long getId() {
        return id;
    }

    public String getTitolo() {
        return titolo;
    }

    public double getPrezzo() {
        return prezzo;
    }
}
```

---

## `@Entity`

```java
@Entity
public class Corso {
```

`@Entity` indica che la classe è una entity JPA.

Senza `@Entity`, Hibernate considera `Corso` una classe Java normale e non la gestisce come oggetto persistente.

---

## `@Table(name = "corsi")`

```java
@Table(name = "corsi")
```

`@Table` collega la entity a una tabella precisa del database.

In questo caso:

```text
classe Corso
↓
tabella corsi
```

Se non usassimo `@Table`, Hibernate potrebbe derivare il nome della tabella dal nome della classe. Rendere il nome esplicito evita ambiguità.

---

## `@Id`

```java
@Id
private Long id;
```

`@Id` indica la chiave primaria della entity.

Ogni entity JPA deve avere un identificativo.
Senza `@Id`, Hibernate non sa come distinguere un oggetto persistente da un altro.

---

## `@GeneratedValue(strategy = GenerationType.IDENTITY)`

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

`@GeneratedValue` indica che l'id viene generato automaticamente.

`GenerationType.IDENTITY` indica che la generazione è demandata al database, tipicamente tramite `AUTO_INCREMENT` in MariaDB/MySQL.

Schema SQL equivalente:

```sql
id BIGINT PRIMARY KEY AUTO_INCREMENT
```

---

## Perché usare `Long` per l'id

```java
private Long id;
```

Si usa spesso `Long` e non `long` perché `Long` può essere `null`.

Quando un oggetto viene creato in memoria ma non è ancora stato salvato, l'id non esiste ancora.

```text
Nuovo oggetto non salvato
id = null

Oggetto salvato
id = valore generato dal database
```

Con `long`, il valore iniziale sarebbe `0`, che può essere ambiguo.

---

## `@Column`

```java
@Column(nullable = false, length = 120)
private String titolo;
```

`@Column` permette di specificare caratteristiche della colonna.

| Attributo | Significato |
|---|---|
| `nullable = false` | la colonna non può essere `NULL` |
| `length = 120` | lunghezza massima per colonne testuali |
| `name = "..."` | nome esplicito della colonna |
| `unique = true` | valore univoco |

Esempio:

```java
@Column(name = "data_inizio", nullable = false)
private LocalDate dataInizio;
```

Qui il campo Java `dataInizio` viene collegato alla colonna `data_inizio`.

---

## Relazione `@ManyToOne`

In Java una edizione corso può contenere un riferimento a un `Corso`:

```java
private Corso corso;
```

Nel database questa relazione viene rappresentata con una chiave esterna.

Con JPA scriviamo:

```java
@ManyToOne
@JoinColumn(name = "corso_id", nullable = false)
private Corso corso;
```

`@ManyToOne` significa:

```text
molte edizioni possono essere associate allo stesso corso.
```

Esempio:

```text
Corso Java Backend
↓
Edizione giugno
Edizione luglio
Edizione settembre
```

---

## `@JoinColumn`

```java
@JoinColumn(name = "corso_id", nullable = false)
private Corso corso;
```

`@JoinColumn` indica quale colonna della tabella corrente contiene la chiave esterna.

In questo caso, nella tabella `edizioni_corso` ci sarà una colonna:

```sql
corso_id BIGINT NOT NULL
```

che punta alla tabella `corsi`.

---

## `@Enumerated(EnumType.STRING)`

Se una entity contiene un enum:

```java
public enum StatoEdizione {
    PIANIFICATA,
    ATTIVA,
    CHIUSA
}
```

possiamo mapparlo così:

```java
@Enumerated(EnumType.STRING)
@Column(nullable = false, length = 30)
private StatoEdizione stato;
```

`EnumType.STRING` salva il nome dell'enum come testo:

```text
PIANIFICATA
ATTIVA
CHIUSA
```

È più leggibile rispetto a salvare la posizione numerica dell'enum.

---

## Costruttore vuoto richiesto da JPA

```java
protected Corso() {
    // Costruttore richiesto da JPA.
}
```

JPA/Hibernate deve poter creare oggetti entity quando legge righe dal database.
Per farlo ha bisogno di un costruttore senza parametri.

Lo possiamo dichiarare `protected` per indicare che non è il costruttore pensato per il normale codice applicativo.

---

## `persistence.xml`

Il file `persistence.xml` si trova in:

```text
src/main/resources/META-INF/persistence.xml
```

Esempio:

```xml
<persistence xmlns="https://jakarta.ee/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence https://jakarta.ee/xml/ns/persistence/persistence_3_2.xsd"
             version="3.2">

    <persistence-unit name="academyPU" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

        <class>corso.ud30.guidato.model.Corso</class>
        <class>corso.ud30.guidato.model.Docente</class>
        <class>corso.ud30.guidato.model.EdizioneCorso</class>

        <properties>
            <property name="jakarta.persistence.jdbc.driver" value="org.mariadb.jdbc.Driver" />
            <property name="jakarta.persistence.jdbc.url" value="jdbc:mariadb://localhost:3306/academy_ud30" />
            <property name="jakarta.persistence.jdbc.user" value="academy_user" />
            <property name="jakarta.persistence.jdbc.password" value="academy_pwd" />
            <property name="hibernate.hbm2ddl.auto" value="update" />
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
        </properties>
    </persistence-unit>
</persistence>
```

---

## Elementi principali di `persistence.xml`

| Elemento | Significato |
|---|---|
| `persistence-unit name="academyPU"` | nome della configurazione JPA usata dal codice Java |
| `transaction-type="RESOURCE_LOCAL"` | transazioni gestite localmente dall'applicazione, senza application server |
| `provider` | implementazione JPA usata, in questo caso Hibernate |
| `class` | entity gestite dalla persistence unit |
| `jakarta.persistence.jdbc.url` | URL JDBC del database |
| `jakarta.persistence.jdbc.user` | utente del database |
| `jakarta.persistence.jdbc.password` | password del database |
| `hibernate.hbm2ddl.auto` | strategia con cui Hibernate gestisce lo schema |
| `hibernate.show_sql` | stampa SQL generato |
| `hibernate.format_sql` | rende SQL più leggibile nei log |

---

## Attenzione a `hibernate.hbm2ddl.auto`

Nel laboratorio useremo spesso:

```xml
<property name="hibernate.hbm2ddl.auto" value="update" />
```

Significa che Hibernate prova ad aggiornare lo schema in base alle entity.

È comodo per un laboratorio, ma in un contesto professionale bisogna gestire lo schema con strumenti controllati, script SQL o migration tool.

---

## Flusso operativo completo

```java
EntityManagerFactory emf =
        Persistence.createEntityManagerFactory("academyPU");

EntityManager em = emf.createEntityManager();

EntityTransaction tx = em.getTransaction();

try {
    tx.begin();

    Corso corso = new Corso("Java Backend", 690.0);
    em.persist(corso);

    tx.commit();
} catch (RuntimeException ex) {
    if (tx.isActive()) {
        tx.rollback();
    }
    throw ex;
} finally {
    em.close();
    emf.close();
}
```

---

## Spiegazione del flusso

| Istruzione | Significato |
|---|---|
| `Persistence.createEntityManagerFactory("academyPU")` | legge `persistence.xml` e inizializza JPA/Hibernate |
| `emf.createEntityManager()` | crea un contesto operativo per lavorare con le entity |
| `em.getTransaction()` | recupera la transazione locale |
| `tx.begin()` | apre la transazione |
| `em.persist(corso)` | rende persistente una nuova entity |
| `tx.commit()` | conferma le modifiche |
| `tx.rollback()` | annulla le modifiche in caso di errore |
| `em.close()` | chiude il contesto operativo |
| `emf.close()` | chiude la factory a fine applicazione |

---

## Dove si colloca nel progetto

Nel laboratorio non useremo `EntityManager` direttamente nel `main` o nel service.

Lo useremo dentro il repository JPA manuale:

```text
Classe applicativa
↓
Service
↓
Repository JPA
↓
EntityManager
↓
Database
```

Questa scelta prepara la UD31, dove Spring Data JPA nasconderà gran parte del codice repository, ma non cambierà la logica architetturale.
