## Strategie di caricamento nelle relazioni JPA: `LAZY` ed `EAGER`

Quando in una entity JPA definiamo una relazione verso un'altra entity, possiamo indicare a Hibernate **quando** deve caricare l'oggetto collegato.

Nel nostro esempio, una `EdizioneCorso` è associata a un `Corso`:

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "corso_id", nullable = false)
private Corso corso;
```

La relazione tra le tabelle è questa:

```text
corsi
  ↑
  │ corso_id
  │
edizioni_corso
```

La colonna `corso_id` della tabella `edizioni_corso` contiene la chiave esterna verso la tabella `corsi`.

La scelta tra `LAZY` ed `EAGER` non cambia la relazione tra le tabelle. Cambia solo **quando Hibernate carica l'oggetto `Corso` associato all'oggetto `EdizioneCorso`**.

---

### `FetchType.LAZY`

Con `FetchType.LAZY`, Hibernate non carica subito l'oggetto collegato.

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "corso_id", nullable = false)
private Corso corso;
```

Questo significa:

```text
Carica l'edizione.
Non caricare subito il corso associato.
Carica il corso solo se il codice lo richiede davvero.
```

Esempio:

```java
EdizioneCorso edizione = edizioneRepository.findById(1L).orElseThrow();
```

In questa fase Hibernate può caricare solo i dati dell'edizione.

Il corso associato viene caricato quando il codice accede alla relazione, ad esempio:

```java
String titoloCorso = edizione.getCorso().getTitolo();
```

In quel momento Hibernate capisce che il codice ha bisogno del `Corso` collegato e prova a caricarlo dal database.

---

### Quando usare `LAZY`

`LAZY` è consigliabile quando l'oggetto collegato **non serve sempre**.

Nel nostro laboratorio è una buona scelta perché potremmo voler caricare un'elenco di edizioni usando solo alcuni dati:

```text
id edizione
docente
data inizio
posti disponibili
stato
```

Non sempre abbiamo bisogno di caricare subito anche tutti i dati del corso associato.

Per questo motivo, `LAZY` evita caricamenti inutili e rende più controllato l'accesso ai dati.

---

### Attenzione con `LAZY`

Con `LAZY`, l'oggetto collegato deve essere caricato mentre il contesto JPA è ancora attivo.

Nel nostro percorso, questo significa che la conversione da entity a DTO deve avvenire nel service, all'interno di un metodo transazionale.

Esempio:

```java
@Transactional(readOnly = true)
public List<EdizioneDisponibileResponseDto> elencoEdizioniDisponibili() {
    return edizioneCorsoRepository
            .findByStatoAndPostiDisponibiliGreaterThan(StatoEdizione.APERTA, 0)
            .stream()
            .map(edizioneMapper::toDto)
            .toList();
}
```

Nel mapper possiamo poi accedere al corso associato:

```java
public EdizioneDisponibileResponseDto toDto(EdizioneCorso edizione) {
    return new EdizioneDisponibileResponseDto(
            edizione.getId(),
            edizione.getCorso().getId(),
            edizione.getCorso().getTitolo(),
            edizione.getDocente(),
            edizione.getDataInizio(),
            edizione.getPostiDisponibili()
    );
}
```

Qui `edizione.getCorso().getTitolo()` richiede il corso associato. Se la transazione è ancora attiva, Hibernate può caricarlo correttamente.

---

### `FetchType.EAGER`

Con `FetchType.EAGER`, Hibernate carica subito l'oggetto collegato insieme all'entity principale.

```java
@ManyToOne(fetch = FetchType.EAGER)
@JoinColumn(name = "corso_id", nullable = false)
private Corso corso;
```

Questo significa:

```text
Carica l'edizione.
Carica subito anche il corso associato.
```

Quindi, quando otteniamo una `EdizioneCorso`, il relativo `Corso` viene preparato immediatamente.

---

### Quando usare `EAGER`

`EAGER` può essere utile quando l'oggetto collegato serve **sempre o quasi sempre**.

Nel nostro caso, potremmo considerare `EAGER` se ogni volta che carichiamo una `EdizioneCorso` dobbiamo mostrare anche:

```text
titolo del corso
descrizione del corso
dati principali del corso
```

In uno scenario molto semplice, questa scelta può sembrare comoda perché il corso è già disponibile senza ulteriori accessi.

---

### Attenzione con `EAGER`

`EAGER` può però caricare più dati del necessario.

Se carichiamo molte edizioni, Hibernate può caricare automaticamente anche i corsi associati, anche quando non servono davvero.

Per questo motivo, `EAGER` non va considerato automaticamente migliore o più veloce.

È più corretto usarlo solo quando siamo sicuri che la relazione serva sempre.

---

### Confronto sintetico

| Strategia | Comportamento                                          | Caso d'uso                                      |
| --------- | ------------------------------------------------------ | ----------------------------------------------- |
| `LAZY`    | Carica l'oggetto collegato solo quando viene richiesto | Scelta consigliata nella maggior parte dei casi |
| `EAGER`   | Carica subito anche l'oggetto collegato                | Utile solo se il dato collegato serve sempre    |

---

### Scelta adottata nel laboratorio

Nel laboratorio usiamo:

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "corso_id", nullable = false)
private Corso corso;
```

Questa scelta è preferibile perché:

* evita caricamenti non necessari;
* rende più controllato l'accesso ai dati collegati;
* abitua a ragionare su quali dati servono davvero;
* prepara a un uso più corretto di JPA e Spring Data JPA.

Quando il DTO richiede dati del corso, il mapper li legge esplicitamente:

```java
edizione.getCorso().getTitolo()
```

In questo modo il caricamento del corso avviene solo quando il codice ne ha bisogno.

---

### Nota finale

`LAZY` ed `EAGER` non indicano due tipi diversi di relazione.

La relazione resta sempre:

```text
molte edizioni possono essere associate a un corso
```

La differenza riguarda solo il momento in cui Hibernate carica l'oggetto collegato:

```text
LAZY  → carica dopo, solo se serve
EAGER → carica subito
```

Nel lavoro quotidiano, una buona regola è:

```text
usare LAZY come scelta principale
e caricare esplicitamente le relazioni quando servono davvero
```
