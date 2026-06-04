# Spring Data JPA e Repository

## Obiettivo del file

In UD30 abbiamo usato JPA/Hibernate in modo manuale.

Abbiamo creato repository che usavano direttamente `EntityManager`.

In UD31 introduciamo Spring Data JPA, che riduce molto il codice necessario per implementare repository basati su JPA.

Il punto da ricordare è questo:

```text
Spring Data JPA non sostituisce JPA.
Spring Data JPA usa JPA/Hibernate sotto il cofano.
```

---

## Repository JPA manuale in UD30

In UD30 un repository poteva contenere codice simile:

```java
public Optional<Corso> findById(Long id) {
    EntityManager em = JpaUtil.getEntityManager();
    try {
        return Optional.ofNullable(em.find(Corso.class, id));
    } finally {
        em.close();
    }
}
```

Questo codice ci ha permesso di capire:

- cos'è un `EntityManager`;
- come si cerca una entity per id;
- quando aprire e chiudere il contesto operativo;
- come JPA lavora con le entity.

---

## Repository Spring Data JPA

Con Spring Data JPA possiamo scrivere:

```java
public interface CorsoRepository extends JpaRepository<Corso, Long> {
}
```

Questa interfaccia sembra vuota, ma non lo è dal punto di vista funzionale.

Ereditando da `JpaRepository`, ottiene già molti metodi:

| Metodo | Funzione |
|---|---|
| `findAll()` | restituisce tutte le entity |
| `findById(id)` | cerca una entity per chiave primaria |
| `save(entity)` | inserisce o aggiorna una entity |
| `deleteById(id)` | elimina una entity per id |
| `existsById(id)` | verifica se una entity esiste |
| `count()` | conta le righe |

Spring Data genera automaticamente l'implementazione concreta.

---

## I parametri di `JpaRepository`

```java
JpaRepository<Corso, Long>
```

contiene due informazioni:

| Parte | Significato |
|---|---|
| `Corso` | tipo della entity gestita |
| `Long` | tipo della chiave primaria |

Quindi:

```java
public interface CorsoRepository extends JpaRepository<Corso, Long> {
}
```

significa:

```text
Repository Spring Data per la entity Corso, identificata da una chiave primaria Long.
```

---

## Entity JPA e Repository

Il repository lavora sulle **entity**, non sui DTO.

Esempio:

```java
@Entity
@Table(name = "corsi")
public class Corso {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String titolo;
}
```

Il repository sarà:

```java
public interface CorsoRepository extends JpaRepository<Corso, Long> {
}
```

Il service userà il repository per recuperare entity, applicare regole e produrre DTO tramite mapper.

---

## Query derivate dal nome del metodo

Spring Data può generare query partendo dal nome del metodo.

Esempio:

```java
List<EdizioneCorso> findByStato(StatoEdizione stato);
```

Spring interpreta il metodo come:

```text
Trova tutte le edizioni corso con il campo stato uguale al valore indicato.
```

Altro esempio:

```java
List<EdizioneCorso> findByPostiDisponibiliGreaterThan(int postiDisponibili);
```

Significa:

```text
Trova tutte le edizioni corso con postiDisponibili maggiore del valore indicato.
```

Nel laboratorio useremo solo query derivate semplici.

---

## Quando usare query derivate e quando no

Le query derivate sono utili per casi semplici.

Esempi adeguati:

```java
findByStato(...)
findByEmailPartecipante(...)
findByPostiDisponibiliGreaterThan(...)
```

Se il nome diventa troppo lungo o poco leggibile, è meglio usare una query esplicita con `@Query` oppure spostare il ragionamento in un service.

In questa UD non è necessario approfondire `@Query`, salvo eventuali estensioni.

---

## `application.properties` al posto di `persistence.xml`

In UD30 configuravamo JPA con `persistence.xml`.

In Spring Boot useremo `application.properties`.

Esempio:

```properties
spring.datasource.url=jdbc:mariadb://localhost:3306/academy_ud31_guidato
spring.datasource.username=academy_user
spring.datasource.password=academy_pwd
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

Spring Boot usa queste proprietà per configurare:

- datasource;
- Hibernate;
- JPA;
- repository Spring Data.

---

## Relazione tra Spring Data JPA, JPA, Hibernate e JDBC

Il flusso reale è:

```text
Service
↓
Repository Spring Data JPA
↓
JPA
↓
Hibernate
↓
JDBC
↓
Database
```

JDBC non scompare. Viene usato a un livello più basso.

Hibernate non scompare. È il provider JPA.

JPA non scompare. È la specifica su cui Spring Data lavora.

Spring Data JPA aggiunge un livello di astrazione per ridurre il codice ripetitivo dei repository.

---

## Dove mettere le regole applicative

Il repository non deve contenere regole applicative complesse.

Esempio di regola applicativa:

```text
Non creare una iscrizione se l'edizione è chiusa.
```

Questa regola appartiene al service.

Il repository deve occuparsi di accesso ai dati:

```text
cerca edizione
salva iscrizione
verifica esistenza
elenca dati
```

Il service coordina il caso d'uso.

---

## Sintesi

| Concetto | Significato |
|---|---|
| `JpaRepository` | interfaccia Spring Data con metodi CRUD già pronti |
| Repository Spring Data | interfaccia che lavora sulle entity |
| Entity | oggetto persistente JPA |
| DTO | oggetto di input/output API |
| Service | livello che usa repository e applica regole |
| Hibernate | provider JPA usato sotto Spring Data |
| JDBC | tecnologia usata più in basso per comunicare con il database |

Spring Data JPA semplifica i repository, ma non elimina la necessità di distinguere chiaramente entity, DTO, mapper e service.
