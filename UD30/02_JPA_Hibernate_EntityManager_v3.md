# JPA, Hibernate, EntityManager e ciclo di vita delle entity

## Collegamento con il file precedente

Nel file precedente abbiamo visto perché nasce il problema del mapping tra oggetti Java e database relazionale.
Java lavora con classi, oggetti e riferimenti tra oggetti; il database relazionale lavora con tabelle, righe, colonne, chiavi primarie e chiavi esterne.

In questo file non ripetiamo tutta la teoria dell'ORM, ma collochiamo con precisione i tre elementi che useremo nella UD30:

```text
ORM
JPA
Hibernate
```

e poi arriviamo agli oggetti operativi che useremo nel codice:

```text
EntityManagerFactory
EntityManager
EntityTransaction
```

---

## ORM, JPA e Hibernate: distinzione fondamentale

La distinzione da tenere presente è questa:

```text
ORM è l'idea.
JPA è lo standard Java/Jakarta.
Hibernate è lo strumento concreto che lo esegue.
```

| Concetto | Che cos'è | Ruolo nella UD30 |
|---|---|---|
| ORM | Approccio generale per collegare oggetti e tabelle | Spiega perché una classe Java può essere collegata a una tabella |
| JPA / Jakarta Persistence | Specifica standard per la persistenza ORM in Java | Definisce annotazioni e API come `@Entity`, `@Id`, `EntityManager` |
| Hibernate | Implementazione concreta della specifica JPA | Legge le annotazioni, gestisce le entity, genera SQL e usa JDBC |

ORM non è una libreria specifica.
È una tecnica di mapping.

JPA non è Hibernate.
È una specifica.

Hibernate non è la specifica.
È un provider concreto che implementa JPA.

In una frase:

```text
Scriviamo codice usando lo standard JPA.
A runtime Hibernate esegue concretamente il lavoro ORM.
```

---

## ORM ha una sua implementazione Java?

Non esiste una sola implementazione Java chiamata genericamente “ORM”.

ORM è la categoria generale. Nel mondo Java esistono diversi strumenti che realizzano questa idea.

Esempi:

| Strumento | Descrizione |
|---|---|
| Hibernate | Provider ORM molto diffuso e usato in questa UD |
| EclipseLink | Provider JPA alternativo |
| OpenJPA | Altro provider JPA |

Quindi è corretto dire:

```text
Hibernate è una implementazione ORM per Java.
```

Non è corretto dire:

```text
ORM è una libreria Java.
```

Nel nostro percorso useremo Hibernate perché è molto diffuso e perché prepara bene il passaggio successivo a Spring Data JPA.

---

## Relazione tra JPA e Hibernate

JPA definisce un contratto.
Hibernate lo realizza.

Un parallelo con Java può aiutare:

```java
public interface CorsoRepository {
    Corso findById(Long id);
}
```

L'interfaccia stabilisce cosa deve essere disponibile, ma non contiene l'implementazione concreta.

Poi una classe può implementarla:

```java
public class JdbcCorsoRepository implements CorsoRepository {
    @Override
    public Corso findById(Long id) {
        // codice concreto
    }
}
```

In modo analogo:

```text
JPA
specifica il contratto

Hibernate
fornisce l'implementazione concreta
```

Quando scriviamo:

```java
@Entity
@Table(name = "corsi")
public class Corso {
    ...
}
```

stiamo usando annotazioni standard JPA.

Quando il programma parte, Hibernate legge quelle annotazioni e le usa per capire come collegare la classe `Corso` alla tabella `corsi`.

---

## Cosa succede sotto il cofano

Quando scriviamo:

```java
Corso corso = entityManager.find(Corso.class, 1L);
```

il codice sembra molto più semplice rispetto a JDBC.

Con JDBC avremmo scritto manualmente:

```java
String sql = "SELECT id, titolo, prezzo FROM corsi WHERE id = ?";
```

poi avremmo usato `PreparedStatement`, `ResultSet` e mapping manuale.

Con JPA/Hibernate, invece, il flusso concettuale è questo:

```text
1. Il codice chiama EntityManager.
2. EntityManager fa parte della API JPA.
3. Hibernate è il provider concreto configurato nel progetto.
4. Hibernate sa che Corso è una entity.
5. Hibernate sa che Corso è collegata alla tabella corsi.
6. Hibernate genera SQL.
7. Hibernate usa JDBC per comunicare con il database.
8. Il database restituisce righe.
9. Hibernate costruisce oggetti Java.
10. Il codice riceve un oggetto Corso.
```

Quindi JDBC non scompare.
Viene usato da Hibernate a un livello più basso.

Per questo JPA non elimina il database e non elimina SQL.
Semplicemente sposta una parte del lavoro ripetitivo dal nostro codice applicativo al provider ORM.

---

## Che cos'è JPA

JPA significa oggi **Jakarta Persistence**.

È una specifica standard per gestire la persistenza degli oggetti Java in un contesto ORM.

JPA definisce concetti come:

- `@Entity`;
- `@Table`;
- `@Id`;
- `@GeneratedValue`;
- `@Column`;
- `@ManyToOne`;
- `@JoinColumn`;
- `EntityManager`;
- `EntityTransaction`;
- `persistence.xml`;
- JPQL.

JPA da sola non salva i dati.
Definisce le regole e le API.

Per eseguire realmente il lavoro serve un provider, cioè una implementazione concreta.
In questa UD il provider sarà Hibernate.

---

## Che cos'è Hibernate

Hibernate è il provider JPA che useremo nella UD30.

Si occupa di:

- leggere le annotazioni JPA presenti nelle entity;
- collegare classi Java e tabelle;
- collegare attributi Java e colonne;
- gestire le relazioni tra entity;
- generare SQL quando necessario;
- usare JDBC per comunicare con il database;
- mantenere un persistence context;
- sincronizzare le entity gestite con il database.

Nel file `persistence.xml` indicheremo Hibernate come provider.

Esempio indicativo:

```xml
<provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
```

Questa riga dice alla configurazione JPA quale implementazione concreta usare.

---

## Perché usiamo Hibernate senza Spring

Nella UD31 useremo Spring Boot e Spring Data JPA.

Prima, però, è utile vedere cosa accade senza Spring:

- come si configura una persistence unit;
- come si crea un `EntityManagerFactory`;
- come si apre un `EntityManager`;
- dove si apre una transazione;
- dove si usa `persist`, `find`, `createQuery`, `commit` e `rollback`.

Questo passaggio è importante perché in Spring molte parti verranno semplificate.

Se prima vediamo il funzionamento manuale, dopo sarà più chiaro cosa Spring sta facendo per noi.

---

## Che cos'è una entity

Una **entity** è una classe Java persistente.

Significa che JPA/Hibernate può collegarla a una tabella del database.

Esempio:

```java
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
}
```

Questa classe non è un DTO.

Un DTO è progettato per trasferire dati verso l'esterno o ricevere dati dall'esterno.

Una entity, invece, rappresenta lo stato persistente dell'applicazione.

Nel nostro percorso la distinzione resta questa:

| Tipo | Ruolo |
|---|---|
| Entity | Oggetto persistente collegato al database |
| DTO | Oggetto usato per input/output dell'applicazione o delle API |
| Mapper | Classe che converte entity in DTO o DTO in entity quando serve |

---

## EntityManagerFactory

`EntityManagerFactory` rappresenta la configurazione JPA inizializzata.

Si crea usando il nome della persistence unit:

```java
EntityManagerFactory factory =
        Persistence.createEntityManagerFactory("academyPU");
```

Il nome `academyPU` deve corrispondere al nome dichiarato nel file `persistence.xml`.

Esempio:

```xml
<persistence-unit name="academyPU">
    ...
</persistence-unit>
```

`EntityManagerFactory` è un oggetto costoso da creare.
Normalmente viene creato una sola volta e riutilizzato.

Nel laboratorio useremo una classe di supporto, ad esempio `JpaUtil`, per centralizzare questa creazione.

---

## EntityManager

`EntityManager` è l'oggetto operativo usato per lavorare con le entity.

Con `EntityManager` possiamo:

- salvare una nuova entity con `persist`;
- cercare una entity per chiave primaria con `find`;
- scrivere query JPQL con `createQuery`;
- aggiornare entity gestite;
- eliminare entity con `remove`;
- ottenere una transazione locale con `getTransaction`.

Esempio:

```java
EntityManager em = factory.createEntityManager();

Corso corso = em.find(Corso.class, 1L);
```

La chiamata:

```java
em.find(Corso.class, 1L)
```

significa:

```text
cerca una entity Corso con chiave primaria 1.
```

Il parametro:

```java
Corso.class
```

indica a JPA il tipo di entity da cercare.
Non crea una classe.
Serve a comunicare il tipo Java di destinazione dell'operazione.

---

## Persistence context

Quando un `EntityManager` carica o salva entity, lavora dentro un contesto chiamato **persistence context**.

Il persistence context è l'area in cui JPA tiene traccia degli oggetti gestiti.

Esempio:

```java
Corso corso = em.find(Corso.class, 1L);
```

Dopo questa istruzione, se il corso viene trovato, l'oggetto `corso` è gestito dall'`EntityManager`.

Finché l'oggetto è gestito, JPA può rilevare alcune modifiche e sincronizzarle con il database al momento opportuno, di solito al commit della transazione.

Esempio concettuale:

```java
Corso corso = em.find(Corso.class, 1L);
corso.setPrezzo(750.0);
```

Se `corso` è managed e la modifica avviene dentro una transazione, Hibernate può rilevare il cambiamento e generare un `UPDATE` al commit.

Questo meccanismo viene spesso chiamato **dirty checking**.

---

## Ciclo di vita minimo di una entity

Una entity può attraversare stati diversi.

| Stato | Significato |
|---|---|
| Transient | Oggetto Java appena creato, non ancora noto a JPA |
| Managed | Oggetto gestito dall'EntityManager |
| Detached | Oggetto non più collegato a un EntityManager attivo |
| Removed | Oggetto marcato per eliminazione |

Esempio:

```java
Corso corso = new Corso("Java Backend", 690.0);
```

In questa fase `corso` è transient.
È solo un oggetto Java in memoria.

```java
em.persist(corso);
```

Dopo `persist`, l'oggetto diventa managed.
JPA lo conosce e potrà inserirlo nel database.

```java
em.close();
```

Dopo la chiusura dell'`EntityManager`, l'oggetto non è più gestito da quel contesto.
Diventa detached.

---

## EntityTransaction

Quando usiamo JPA senza Spring, le operazioni di scrittura devono essere racchiuse in una transazione locale.

Esempio:

```java
EntityTransaction tx = em.getTransaction();

try {
    tx.begin();

    em.persist(corso);

    tx.commit();
} catch (RuntimeException ex) {
    if (tx.isActive()) {
        tx.rollback();
    }
    throw ex;
}
```

Significato:

| Istruzione | Significato |
|---|---|
| `em.getTransaction()` | recupera la transazione locale associata all'EntityManager |
| `tx.begin()` | avvia la transazione |
| `em.persist(corso)` | registra una nuova entity da salvare |
| `tx.commit()` | conferma le modifiche sul database |
| `tx.rollback()` | annulla le modifiche se avviene un errore |

Questo collega JPA a quanto già visto su transazioni SQL e JDBC.

```text
SQL/JDBC
START TRANSACTION / COMMIT / ROLLBACK

JPA manuale
EntityTransaction begin / commit / rollback

Spring
@Transactional
```

In UD31 il confine transazionale sarà spesso dichiarato con `@Transactional`.

---

## Repository JPA manuale

In questa UD il repository usa direttamente `EntityManager`.

Esempio:

```java
public Optional<Corso> findCorsoById(Long id) {
    try (EntityManager em = JpaUtil.createEntityManager()) {
        return Optional.ofNullable(em.find(Corso.class, id));
    }
}
```

Il service non deve usare direttamente `EntityManager`.

La separazione corretta è:

```text
Service
coordina il caso d'uso e applica regole applicative

Repository JPA
usa EntityManager per lavorare con le entity

EntityManager
interagisce con persistence context, provider JPA e database
```

Questa separazione mantiene continuità con la UD27, dove avevamo separato service e DAO JDBC.

---

## Progressione verso UD31

La progressione del percorso è questa:

```text
UD27
DAO/Repository con JDBC manuale

UD30
Repository JPA manuale con EntityManager

UD31
Repository Spring Data JPA
```

Spring Data JPA ridurrà molto il codice repository, ma non eliminerà il bisogno di capire:

- entity;
- mapping;
- persistence context;
- transazioni;
- DTO;
- mapper;
- confini tra repository e service.

Se questi concetti sono chiari in UD30, Spring Data JPA in UD31 non sembrerà magia, ma una semplificazione di meccanismi già incontrati.

---

## Sintesi finale

| Concetto | Da ricordare |
|---|---|
| ORM | Approccio generale per mappare oggetti e tabelle |
| JPA | Specifica Java/Jakarta per persistenza ORM |
| Hibernate | Implementazione concreta di JPA usata nel laboratorio |
| Entity | Classe Java persistente collegata a una tabella |
| EntityManagerFactory | Oggetto pesante che inizializza la configurazione JPA |
| EntityManager | Oggetto operativo per lavorare con le entity |
| Persistence context | Area in cui JPA tiene traccia degli oggetti gestiti |
| EntityTransaction | Transazione manuale usata senza Spring |
| Repository JPA | Classe che incapsula l'uso di EntityManager |

Frase chiave:

```text
Con JDBC scriviamo direttamente SQL e mappiamo manualmente i ResultSet.
Con JPA descriviamo il mapping e lavoriamo con entity.
Hibernate realizza concretamente quel mapping usando anche JDBC sotto il cofano.
```
