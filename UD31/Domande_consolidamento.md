# UD31 - Domande di consolidamento

## Scopo dell'attività

Queste domande servono a riflettere sui concetti principali della UD31, collegandoli in modo logico ad alcuni argomenti già affrontati nelle UD precedenti.

```text
Java OOP
↓
SQL e database relazionali
↓
JPA/Hibernate
↓
Spring Boot
↓
API REST
↓
Controller → Service → Repository
↓
Transazioni con @Transactional
```

Le risposte saranno poi inserite nel file evidence, copiando anche il testo della domanda.

---

# Domande

## 1. Dal repository manuale a Spring Data JPA

Nella UD30 abbiamo usato repository manuali basati su `EntityManager`.

Nella UD31 usiamo invece interfacce come:

```java
public interface CorsoRepository extends JpaRepository<Corso, Long> {
}
```

Quale problema risolve Spring Data JPA rispetto al repository manuale della UD30?

---

## 2. Controller, Service e Repository

Nel laboratorio UD31 una richiesta `POST /api/iscrizioni` attraversa più livelli.

Spiega, in modo sintetico, il ruolo di:

- Controller;
- Service;
- Repository.

Perché non sarebbe corretto mettere tutta la logica direttamente nel controller?

---

## 3. DTO ed entity

Nel laboratorio UD31 il controller riceve un `CreaIscrizioneRequestDto` e restituisce un `IscrizioneResponseDto`.

Perché non usiamo direttamente la entity `Iscrizione` come oggetto di input e output della API?

---

## 4. OOP e applicazione concreta in Spring

Nel percorso abbiamo studiato concetti OOP come incapsulamento, astrazione e polimorfismo.

Indica almeno un punto della UD31 in cui questi concetti diventano concreti.

Puoi fare riferimento, ad esempio, a:

- repository come interfacce;
- dependency injection;
- service che lavora con componenti ricevuti tramite costruttore;
- entity che contengono piccoli metodi di dominio.

---

## 5. Query SQL: edizioni e corsi

Dato questo schema semplificato:

```sql
corsi(
    id,
    titolo,
    categoria,
    pubblicabile
)

edizioni_corso(
    id,
    corso_id,
    docente,
    data_inizio,
    posti_totali,
    posti_disponibili,
    stato
)
```

Scrivi una query SQL che mostri le edizioni aperte con il titolo del corso associato.

La query deve mostrare almeno:

- id edizione;
- titolo corso;
- docente;
- data inizio;
- posti disponibili.

---

## 6. Query SQL sul conteggio iscrizioni

Dato questo schema semplificato:

```sql
corsi(
    id,
    titolo,
    categoria,
    pubblicabile
)

edizioni_corso(
    id,
    corso_id,
    docente,
    data_inizio,
    posti_totali,
    posti_disponibili,
    stato
)

iscrizioni(
    id,
    edizione_id,
    nome_partecipante,
    email_partecipante,
    data_iscrizione
)
```

Scrivi una query SQL che mostri quante iscrizioni sono presenti per ogni edizione.

La query deve mostrare almeno:

- id edizione;
- titolo corso;
- numero iscrizioni.

---

## 7. Transazione nel database e `@Transactional` in Spring

Qual è la differenza tra una transazione gestita dal database e l'annotazione `@Transactional` usata in Spring?

Spiega la differenza usando il caso della creazione di una iscrizione.

---

## 8. Perché `creaIscrizione(...)` deve essere transazionale?

Nel metodo `creaIscrizione(...)` avvengono due operazioni collegate:

```text
1. decrementare i posti disponibili dell'edizione;
2. salvare la nuova iscrizione.
```

Perché queste due operazioni devono stare nella stessa transazione?

---

## 9. Gestione degli errori

Nel laboratorio UD31 abbiamo usato una classe come `ApiExceptionHandler` con `@RestControllerAdvice`.

Perché è preferibile gestire gli errori in modo centralizzato invece di scrivere blocchi `try/catch` in ogni metodo del controller?

---

## 10. Ragionamento pratico sul progetto

Se l'applicazione compila correttamente, ma non parte perché non riesce a collegarsi al database, quali controlli faresti?

Indica almeno quattro elementi da verificare.
