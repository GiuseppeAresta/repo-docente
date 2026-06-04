# Da JPA manuale a Spring Boot

## Obiettivo del file

Questo file collega il lavoro fatto nelle unità precedenti con la nuova struttura Spring Boot.

Il punto centrale è il seguente: Spring Boot non sostituisce i concetti studiati, ma li integra e automatizza.

In UD29 abbiamo costruito una API manuale con `HttpServer`, handler, DTO e Jackson esplicito.

In UD30 abbiamo costruito la persistenza con JPA/Hibernate, `persistence.xml`, `EntityManager`, repository manuali e transazioni JPA locali.

In UD31 uniamo questi due mondi:

```text
API REST
+
JPA/Hibernate
+
Spring Boot
+
Spring Data JPA
+
transazioni dichiarative
```

---

## Prima di Spring Boot: cosa facevamo manualmente

### API HTTP manuale

Nella UD29 il flusso di una richiesta era gestito manualmente:

```text
Client HTTP
↓
HttpServer
↓
HttpHandler
↓
HttpExchange
↓
lettura metodo e body
↓
DTO di richiesta
↓
Service
↓
DTO di risposta
↓
ObjectMapper
↓
JSON
```

Questo ci ha permesso di capire cosa succede dietro una API: una richiesta HTTP arriva, viene interpretata, viene trasformata in un oggetto Java, passa al service, produce una risposta e viene trasformata in JSON.

### Persistenza JPA manuale

Nella UD30 il flusso di persistenza era gestito manualmente:

```text
main
↓
Service
↓
Repository JPA manuale
↓
EntityManager
↓
EntityTransaction
↓
Hibernate
↓
Database
```

Abbiamo visto che:

- `persistence.xml` definisce la persistence unit;
- `EntityManagerFactory` crea gli `EntityManager`;
- `EntityManager` lavora con le entity;
- `EntityTransaction` gestisce `begin`, `commit` e `rollback`;
- Hibernate è il provider che implementa concretamente JPA.

---

## Con Spring Boot: cosa cambia

Spring Boot introduce un modello applicativo più integrato.

| Prima | Con Spring Boot |
|---|---|
| `HttpServer` creato manualmente | server web embedded avviato automaticamente |
| `HttpHandler` | `@RestController` |
| `HttpExchange` | parametri del metodo controller, `@RequestBody`, `@PathVariable` |
| `JsonRequestReader` | Jackson automatico con `@RequestBody` |
| `JsonResponseWriter` | Jackson automatico sul valore restituito |
| `EntityManagerFactory` manuale | gestito da Spring Boot |
| `persistence.xml` | `application.properties` |
| Repository JPA manuale | interfacce Spring Data JPA |
| `EntityTransaction` manuale | `@Transactional` |
| creazione manuale degli oggetti nel `main` | container Spring e dependency injection |

---

## Cosa resta uguale

Spring Boot semplifica molto codice, ma non elimina le responsabilità applicative.

| Concetto | Resta necessario? | Motivo |
|---|---|---|
| Entity | Sì | rappresentano dati persistenti gestiti da JPA |
| DTO | Sì | rappresentano input/output della API |
| Mapper | Sì | separano entity e DTO |
| Service | Sì | contiene regole applicative e casi d'uso |
| Repository | Sì | incapsula accesso ai dati |
| Transazione | Sì | garantisce coerenza tra operazioni collegate |
| Validazione | Sì | protegge il dominio da dati non validi |

Quindi Spring Boot riduce codice ripetitivo, ma non elimina la necessità di progettare bene i livelli.

---

## Esempio di confronto: API manuale e controller Spring

### UD29: handler manuale

```java
public class IscrizioniHandler implements HttpHandler {

    @Override
    public void handle(HttpExchange exchange) throws IOException {
        String body = readRequestBody(exchange);

        CreaIscrizioneRequestDto request = JsonRequestReader.fromJson(
                body,
                CreaIscrizioneRequestDto.class
        );

        IscrizioneResponseDto response = service.creaIscrizione(request);

        send(exchange, 201, response);
    }
}
```

### UD31: controller Spring

```java
@PostMapping
public ResponseEntity<IscrizioneResponseDto> creaIscrizione(
        @RequestBody CreaIscrizioneRequestDto request
) {
    IscrizioneResponseDto response = service.creaIscrizione(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

Nel secondo esempio non leggiamo manualmente il body HTTP. Spring legge il JSON, usa Jackson e crea automaticamente il DTO di richiesta.

---

## Esempio di confronto: repository JPA manuale e Spring Data

### UD30: repository manuale

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

### UD31: repository Spring Data

```java
public interface CorsoRepository extends JpaRepository<Corso, Long> {
}
```

Il metodo `findById` non scompare. Viene ereditato da `JpaRepository`.

Spring Data JPA genera l'implementazione a runtime usando JPA/Hibernate.

---

## Esempio di confronto: transazione manuale e `@Transactional`

### UD30: transazione JPA manuale

```java
EntityTransaction tx = em.getTransaction();

try {
    tx.begin();
    em.persist(edizione);
    tx.commit();
} catch (RuntimeException e) {
    if (tx.isActive()) {
        tx.rollback();
    }
    throw e;
}
```

### UD31: transazione dichiarativa

```java
@Transactional
public IscrizioneResponseDto creaIscrizione(CreaIscrizioneRequestDto request) {
    // operazioni collegate sul database
}
```

Con `@Transactional`, Spring apre una transazione all'inizio del metodo e la conclude alla fine:

- se il metodo termina correttamente, esegue il commit;
- se si verifica un'eccezione non gestita, esegue il rollback.

---

## Sintesi

La UD31 deve essere letta come trasformazione finale:

```text
UD29: API REST manuale
UD30: JPA/Hibernate manuale
UD31: API REST + JPA/Hibernate con Spring Boot
```

La competenza richiesta non è solo usare annotazioni, ma riconoscere quali responsabilità sono gestite da Spring e quali restano a carico dell'applicazione.
