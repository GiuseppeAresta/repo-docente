# 01 - DTO, database, memoria e trasferimento dati

## Obiettivo

Comprendere perché un DTO è una classe aggiunta come strumento per separare rappresentazioni diverse dello stesso concetto.

Un corso, una edizione o una iscrizione possono essere rappresentati in modi diversi a seconda del contesto:

- come riga di una tabella;
- come oggetto Java interno;
- come JSON ricevuto o restituito da una API;
- come dato visualizzato in una pagina web;
- come risposta sintetica in un report.

Pretendere che tutte queste forme coincidano produce codice rigido e difficile da modificare.

## Tre rappresentazioni da distinguere

| Rappresentazione | Dove viene usata | Scopo |
|---|---|---|
| Tabella o record | Database | Conservare dati persistenti e coerenti |
| Model / Entity | Memoria Java | Elaborare dati e applicare regole |
| DTO | Confine applicativo | Trasferire solo i dati richiesti dal caso d'uso |
| JSON | HTTP/API | Rappresentare i dati come testo scambiabile tra client e server |

## Frase chiave

```text
Tabella, model, DTO e JSON non sono la stessa cosa.
```

## Esempio

Nel database possiamo avere tre tabelle:

```text
corsi
- id
- titolo
- prezzo

edizioni_corso
- id
- corso_id
- data_inizio
- posti_massimi

iscrizioni
- id
- edizione_id
- nome_partecipante
- email_partecipante
```

Una API però potrebbe dover restituire questa risposta:

```json
{
  "edizioneId": 7,
  "titoloCorso": "Java Backend",
  "dataInizio": "2026-06-10",
  "postiMassimi": 20,
  "postiOccupati": 13,
  "postiDisponibili": 7
}
```

Questa struttura non coincide con una singola tabella. È una risposta costruita per il client, combinando dati e calcoli applicativi.

## Perché serve il DTO

Il DTO permette di:

- esporre solo i campi necessari;
- evitare di rendere pubblico lo stato interno;
- costruire risposte adatte a uno specifico caso d'uso;
- separare il contratto della API dal model interno;
- preparare il passaggio a Spring, dove la distinzione tra Entity e DTO diventa ancora più importante.

## Esempio di model interno

```java
public class Corso {
    private long id;
    private String titolo;
    private String descrizione;
    private double prezzo;
    private boolean attivo;

    // costruttore e getter
}
```

Questo model contiene anche informazioni interne, come `descrizione` e `attivo`.

## Esempio di DTO di risposta

```java
public class CorsoResponseDto {
    private long id;
    private String titolo;
    private double prezzo;

    // costruttore e getter
}
```

Il DTO contiene solo ciò che vogliamo restituire al client nell'elenco pubblico dei corsi.

## Dal DTO al JSON

Il DTO è ancora un oggetto Java. Per inviarlo al client attraverso HTTP, deve essere trasformato in testo JSON.

```text
CorsoResponseDto
↓
serializzazione
↓
JSON
↓
body della risposta HTTP
```

Esempio di DTO in memoria:

```java
new CorsoResponseDto(1, "Java Object Oriented", 350.0);
```

Esempio di JSON restituito dalla API:

```json
{
  "id": 1,
  "titolo": "Java Object Oriented",
  "prezzo": 350.0
}
```

In UD29 useremo Jackson per questa conversione.

## Perché non costruire JSON a mano

Costruire JSON concatenando stringhe può sembrare semplice nei casi minimi, ma diventa fragile quando compaiono:

- virgolette nei testi;
- caratteri speciali;
- liste annidate;
- date;
- oggetti composti;
- errori di formato.

Per questo useremo una libreria dedicata.

## Collegamento con Spring Boot

In UD29 useremo direttamente Jackson.

In UD31, quando useremo Spring Boot, il controller potrà restituire direttamente DTO Java. Spring userà Jackson dietro le quinte per produrre JSON.

```java
@GetMapping("/api/corsi")
public List<CorsoResponseDto> elencoCorsi() {
    return service.elencoCorsiPubblici();
}
```

Il passaggio Java → JSON verrà gestito automaticamente dal framework.

## Domande di controllo

1. Perché il DTO non deve coincidere necessariamente con il model?
2. Perché il JSON non è la stessa cosa del DTO?
3. Quale parte dell'applicazione decide quali campi esporre?
4. In quali casi un DTO di risposta può contenere dati calcolati?
5. Che problema può nascere se una API restituisce direttamente il model interno?
