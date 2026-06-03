# 02A - Relazioni JPA `@ManyToMany`

## Obiettivo del documento

In questa sezione introduciamo la relazione `@ManyToMany` in JPA.

L'obiettivo è capire:

- quando si usa una relazione molti-a-molti;
- come viene rappresentata nel database;
- come si scrive nelle entity Java;
- cosa significano `@ManyToMany`, `@JoinTable` e `mappedBy`;
- quando è meglio evitare `@ManyToMany` e usare invece una entity intermedia.

Non svolgiamo esercizi operativi in questa parte. Usiamo solo esempi brevi per chiarire il concetto.

---

## 1. Che cos'è una relazione molti-a-molti

Una relazione molti-a-molti si verifica quando più record di una tabella possono essere collegati a più record di un'altra tabella.

Esempio:

- uno studente può seguire più corsi;
- un corso può essere seguito da più studenti.

Quindi abbiamo:

```text
Studente  N : N  Corso
```

Nel modello a oggetti Java questo rapporto sembra naturale:

```text
Studente contiene una lista di corsi
Corso contiene una lista di studenti
```

Nel database relazionale, però, una relazione molti-a-molti non si rappresenta direttamente. Serve una tabella intermedia.

---

## 2. La tabella intermedia

Per collegare studenti e corsi servono tre tabelle:

```text
studenti
- id
- nome

corsi
- id
- titolo

studenti_corsi
- studente_id
- corso_id
```

La tabella `studenti_corsi` è la tabella di collegamento.

Contiene:

- la chiave dello studente;
- la chiave del corso.

Esempio di contenuto:

```text
studente_id | corso_id
------------|---------
1           | 10
1           | 11
2           | 10
```

Questo significa:

- lo studente 1 segue i corsi 10 e 11;
- lo studente 2 segue il corso 10.

---

## 3. Le entity coinvolte

Supponiamo di avere due entity:

```text
Studente
Corso
```

La relazione può essere rappresentata così:

```text
Studente ----< studenti_corsi >---- Corso
```

In Java, ogni `Studente` può avere un insieme di `Corso`.
Ogni `Corso` può avere un insieme di `Studente`.

Di solito si usa `Set` invece di `List`, perché una relazione molti-a-molti non dovrebbe contenere duplicati.

---

## 4. Entity `Studente`

```java
import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "studenti")
public class Studente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @ManyToMany
    @JoinTable(
        name = "studenti_corsi",
        joinColumns = @JoinColumn(name = "studente_id"),
        inverseJoinColumns = @JoinColumn(name = "corso_id")
    )
    private Set<Corso> corsi = new HashSet<>();

    public Studente() {
    }

    public Studente(String nome) {
        this.nome = nome;
    }

    public Long getId() {
        return id;
    }

    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }

    public Set<Corso> getCorsi() {
        return corsi;
    }
}
```

Questa è la parte che definisce fisicamente la tabella di collegamento.

Il campo:

```java
private Set<Corso> corsi = new HashSet<>();
```

indica che uno studente può essere associato a più corsi.

---

## 5. Significato di `@ManyToMany`

L'annotazione:

```java
@ManyToMany
```

indica a JPA che il campo rappresenta una relazione molti-a-molti.

Nel nostro esempio significa:

```text
uno Studente può essere collegato a molti Corso
un Corso può essere collegato a molti Studente
```

Da sola, però, `@ManyToMany` non basta sempre a rendere chiara la struttura della tabella intermedia.
Per questo si usa anche `@JoinTable`.

---

## 6. Significato di `@JoinTable`

L'annotazione:

```java
@JoinTable(
    name = "studenti_corsi",
    joinColumns = @JoinColumn(name = "studente_id"),
    inverseJoinColumns = @JoinColumn(name = "corso_id")
)
```

spiega a JPA quale tabella usare per collegare le due entity.

Analizziamola in modo semplice.

```java
name = "studenti_corsi"
```

indica il nome della tabella intermedia.

```java
joinColumns = @JoinColumn(name = "studente_id")
```

indica la colonna che punta alla entity in cui ci troviamo, cioè `Studente`.

```java
inverseJoinColumns = @JoinColumn(name = "corso_id")
```

indica la colonna che punta all'altra entity, cioè `Corso`.

Quindi JPA interpreta la relazione in questo modo:

```text
studenti.id  <-- studenti_corsi.studente_id
corsi.id     <-- studenti_corsi.corso_id
```

---

## 7. Entity `Corso`

La entity `Corso` può essere scritta così:

```java
import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "corsi")
public class Corso {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String titolo;

    @ManyToMany(mappedBy = "corsi")
    private Set<Studente> studenti = new HashSet<>();

    public Corso() {
    }

    public Corso(String titolo) {
        this.titolo = titolo;
    }

    public Long getId() {
        return id;
    }

    public String getTitolo() {
        return titolo;
    }

    public void setTitolo(String titolo) {
        this.titolo = titolo;
    }

    public Set<Studente> getStudenti() {
        return studenti;
    }
}
```

In questa classe troviamo:

```java
@ManyToMany(mappedBy = "corsi")
private Set<Studente> studenti = new HashSet<>();
```

Questo significa che la relazione è già stata definita nella classe `Studente`, nel campo chiamato `corsi`.

---

## 8. Significato di `mappedBy`

In una relazione bidirezionale, entrambe le classi conoscono la relazione:

```text
Studente conosce i suoi corsi
Corso conosce i suoi studenti
```

Tuttavia, JPA deve sapere quale lato comanda la relazione.

Nel nostro esempio il lato proprietario è `Studente`, perché contiene `@JoinTable`.

La classe `Corso`, invece, usa:

```java
mappedBy = "corsi"
```

Questa indicazione dice a JPA:

```text
non creare una seconda tabella di collegamento;
la relazione è già gestita dal campo corsi della classe Studente.
```

Il valore `"corsi"` deve corrispondere esattamente al nome del campo presente nella classe `Studente`:

```java
private Set<Corso> corsi = new HashSet<>();
```

---

## 9. Lato proprietario e lato inverso

In una relazione bidirezionale esistono due lati:

| Lato | Significato | Nel nostro esempio |
|---|---|---|
| Lato proprietario | Definisce la tabella di collegamento | `Studente` |
| Lato inverso | Si collega alla relazione già definita | `Corso` |

Il lato proprietario contiene:

```java
@JoinTable(...)
```

Il lato inverso contiene:

```java
mappedBy = "nomeDelCampo"
```

È importante non definire `@JoinTable` da entrambe le parti, altrimenti JPA può interpretare la relazione in modo errato o tentare di creare strutture duplicate.

---

## 10. Perché inizializzare il `Set`

Nei due esempi abbiamo scritto:

```java
private Set<Corso> corsi = new HashSet<>();
```

e:

```java
private Set<Studente> studenti = new HashSet<>();
```

Questo evita errori quando proviamo ad aggiungere elementi alla relazione.

Senza inizializzazione, il campo sarebbe `null` fino a quando non viene valorizzato.

Con `new HashSet<>()`, invece, la collezione è già pronta per essere usata.

---

## 11. Metodi di supporto per gestire la relazione

In una relazione bidirezionale può essere utile aggiungere metodi di supporto.

Per esempio, nella classe `Studente`:

```java
public void aggiungiCorso(Corso corso) {
    corsi.add(corso);
    corso.getStudenti().add(this);
}
```

Questo metodo aggiorna entrambi i lati della relazione:

- aggiunge il corso allo studente;
- aggiunge lo studente al corso.

In questo modo il modello Java rimane coerente.

Possiamo anche aggiungere un metodo per rimuovere il collegamento:

```java
public void rimuoviCorso(Corso corso) {
    corsi.remove(corso);
    corso.getStudenti().remove(this);
}
```

Questi metodi non sono obbligatori, ma aiutano a evitare incoerenze nel codice.

---

## 12. Persistenza dei dati

Quando salviamo entity collegate da una relazione molti-a-molti, JPA deve gestire tre cose:

- il record nella tabella `studenti`;
- il record nella tabella `corsi`;
- il record nella tabella `studenti_corsi`.

Esempio semplificato:

```java
Studente studente = new Studente("Mario Rossi");
Corso corso = new Corso("Java Base");

studente.aggiungiCorso(corso);

em.persist(corso);
em.persist(studente);
```

Dopo il commit della transazione, JPA può inserire anche il collegamento nella tabella intermedia.

La relazione viene salvata correttamente solo dentro una transazione.

---

## 13. Attenzione al `cascade`

In molti esempi online si trova:

```java
@ManyToMany(cascade = CascadeType.ALL)
```

Non è sempre una buona scelta.

`CascadeType.ALL` significa che le operazioni fatte su una entity possono essere propagate anche alle entity collegate.

Per esempio, eliminare uno studente potrebbe avere effetti anche sui corsi collegati, se il cascade è configurato male.

Nel nostro caso, un corso può essere condiviso da molti studenti.
Quindi non vogliamo che la cancellazione di uno studente cancelli anche il corso.

Per questo, nelle relazioni molti-a-molti bisogna usare il cascade con attenzione.

In fase iniziale è meglio evitare `CascadeType.ALL`, a meno che il comportamento non sia stato progettato con precisione.

---

## 14. Quando `@ManyToMany` va bene

`@ManyToMany` è adatta quando la tabella intermedia serve solo a collegare due entity.

Esempio:

```text
studenti_corsi
- studente_id
- corso_id
```

In questo caso la tabella intermedia non ha dati propri.
Serve solo a dire quali studenti sono collegati a quali corsi.

---

## 15. Quando è meglio evitare `@ManyToMany`

In un'applicazione reale, spesso la tabella intermedia contiene anche altri dati.

Esempio:

```text
iscrizioni
- id
- studente_id
- corso_id
- data_iscrizione
- stato
- voto_finale
```

In questo caso la tabella intermedia non è più un semplice collegamento.
Rappresenta un concetto autonomo: l'iscrizione.

Quando la tabella intermedia contiene dati propri, è meglio creare una entity dedicata.

Il modello diventa:

```text
Studente 1 : N Iscrizione
Corso    1 : N Iscrizione
```

In Java avremo quindi tre entity:

```text
Studente
Corso
Iscrizione
```

Questo modello è più adatto quando dobbiamo gestire informazioni come data, stato, voto, pagamento o presenza.

---

## 16. Confronto tra i due approcci

| Caso | Soluzione consigliata |
|---|---|
| La tabella intermedia contiene solo le due chiavi esterne | `@ManyToMany` |
| La tabella intermedia contiene dati aggiuntivi | Entity intermedia |
| La relazione è solo un collegamento tecnico | `@ManyToMany` |
| La relazione rappresenta un concetto del dominio | Entity intermedia |

Nel dominio dei corsi, per esempio, una relazione semplice tra `Studente` e `Corso` può essere rappresentata con `@ManyToMany`.

Se però vogliamo gestire iscrizioni, stati, date o valutazioni, è meglio usare una entity `Iscrizione`.

---

## 17. Riepilogo

`@ManyToMany` permette di rappresentare in JPA una relazione molti-a-molti tra due entity.

Nel database questa relazione richiede una tabella intermedia.

Nel codice Java:

- `@ManyToMany` indica il tipo di relazione;
- `@JoinTable` definisce la tabella di collegamento;
- `joinColumns` indica la colonna riferita alla entity corrente;
- `inverseJoinColumns` indica la colonna riferita all'altra entity;
- `mappedBy` indica il lato inverso della relazione;
- il lato proprietario è quello che contiene `@JoinTable`.

La relazione `@ManyToMany` è utile per esempi semplici, ma nei progetti reali va usata con attenzione.

Quando il collegamento tra due entity contiene dati propri, la soluzione migliore è creare una entity intermedia.
