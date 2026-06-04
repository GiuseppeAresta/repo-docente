# Transazioni Spring con `@Transactional`

## Obiettivo del file

Questo file collega le transazioni viste nelle unità precedenti con il modello dichiarativo di Spring.

Abbiamo già incontrato le transazioni in due forme:

- transazioni JDBC manuali;
- transazioni JPA manuali.

In UD31 vediamo la forma Spring:

```java
@Transactional
public void metodoApplicativo() {
    ...
}
```

Il punto centrale è questo:

```text
@Transactional non è una query.
@Transactional non è un commit scritto a mano.
@Transactional indica a Spring di gestire una transazione attorno al metodo.
```

---

## Transazione JDBC manuale

In UD25 abbiamo visto una forma simile:

```java
connection.setAutoCommit(false);

try {
    // operazione SQL 1
    // operazione SQL 2

    connection.commit();
} catch (SQLException e) {
    connection.rollback();
    throw e;
}
```

Qui il codice gestisce direttamente:

- apertura della transazione;
- commit;
- rollback.

---

## Transazione JPA manuale

In UD30 abbiamo visto una forma simile:

```java
EntityTransaction tx = entityManager.getTransaction();

try {
    tx.begin();

    entityManager.persist(edizione);

    tx.commit();
} catch (RuntimeException e) {
    if (tx.isActive()) {
        tx.rollback();
    }
    throw e;
}
```

Anche qui il codice gestisce manualmente la transazione.

---

## Transazione Spring dichiarativa

Con Spring il codice applicativo può diventare:

```java
@Transactional
public IscrizioneResponseDto creaIscrizione(CreaIscrizioneRequestDto request) {
    // operazioni collegate sul database
}
```

Spring apre una transazione prima di eseguire il metodo.

Se il metodo termina correttamente, Spring esegue il commit.

Se il metodo termina con un'eccezione che causa rollback, Spring esegue il rollback.

---

## Dove mettere `@Transactional`

Nel percorso useremo `@Transactional` soprattutto sui metodi del **service**.

Motivo: il service rappresenta il caso d'uso applicativo.

Esempio:

```java
@Service
public class IscrizioneService {

    @Transactional
    public IscrizioneResponseDto creaIscrizione(CreaIscrizioneRequestDto request) {
        // 1. cerca edizione
        // 2. controlla disponibilità
        // 3. decrementa posti
        // 4. salva iscrizione
        // 5. restituisce DTO
    }
}
```

Queste operazioni fanno parte dello stesso caso d'uso. Devono riuscire insieme o fallire insieme.

---

## Esempio applicativo

```java
@Transactional
public IscrizioneResponseDto creaIscrizione(CreaIscrizioneRequestDto request) {
    validaRequest(request);

    EdizioneCorso edizione = edizioneRepository.findById(request.getEdizioneId())
            .orElseThrow(() -> new IllegalArgumentException("Edizione non trovata"));

    if (!edizione.hasPostiDisponibili()) {
        throw new IllegalArgumentException("Posti esauriti");
    }

    edizione.decrementaPosti();

    Iscrizione iscrizione = new Iscrizione(
            edizione,
            request.getNomePartecipante().trim(),
            request.getEmailPartecipante().trim()
    );

    Iscrizione salvata = iscrizioneRepository.save(iscrizione);

    return iscrizioneMapper.toDto(salvata);
}
```

In questo metodo avvengono più operazioni collegate:

1. lettura dell'edizione;
2. controllo dei posti;
3. decremento dei posti disponibili;
4. creazione dell'iscrizione;
5. salvataggio dell'iscrizione.

Se una di queste operazioni fallisce, la transazione deve evitare uno stato parziale incoerente.

---

## Commit e rollback

| Situazione | Effetto |
|---|---|
| il metodo termina correttamente | commit |
| viene lanciata una `RuntimeException` non gestita | rollback |
| viene lanciata una eccezione gestita internamente | Spring potrebbe non eseguire rollback se l'eccezione non esce dal metodo |

Nel laboratorio useremo eccezioni applicative semplici, come `IllegalArgumentException`, per rappresentare errori di richiesta.

---

## `@Transactional(readOnly = true)`

Per metodi di sola lettura è possibile usare:

```java
@Transactional(readOnly = true)
public List<EdizioneDisponibileResponseDto> elencoEdizioniDisponibili() {
    ...
}
```

Questa forma comunica che il metodo non modifica dati.

Nel laboratorio può essere usata sui metodi di consultazione, ma il punto più importante resta il metodo che crea l'iscrizione.

---

## Errori comuni

### Mettere `@Transactional` nel controller

Il controller dovrebbe gestire HTTP, non il caso d'uso transazionale.

La transazione va normalmente nel service.

### Pensare che `@Transactional` sostituisca la validazione

`@Transactional` gestisce la coerenza delle operazioni sul database.

Non valida automaticamente email, posti disponibili o dati obbligatori.

La validazione resta nel service.

### Pensare che `@Transactional` sia sempre necessario

Non tutti i metodi richiedono una transazione esplicita.

È particolarmente importante quando un caso d'uso modifica più dati collegati.

---

## Confronto finale

| Fase | Come gestiamo la transazione |
|---|---|
| UD25 JDBC | `setAutoCommit(false)`, `commit`, `rollback` |
| UD30 JPA manuale | `EntityTransaction`, `begin`, `commit`, `rollback` |
| UD31 Spring | `@Transactional` sul service |

---

## Sintesi

`@Transactional` serve a dichiarare che un metodo deve essere eseguito dentro una transazione.

Nel laboratorio lo useremo soprattutto nei metodi del service che modificano dati collegati.

La transazione non elimina le regole applicative: rende coerenti le modifiche quando il caso d'uso coinvolge più operazioni sul database.
