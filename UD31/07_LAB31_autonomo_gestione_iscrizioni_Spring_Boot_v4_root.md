# LAB31 autonomo - Gestione iscrizioni con Spring Boot

## Scenario

Partendo dal laboratorio guidato, estendere l'applicazione Spring Boot Academy aggiungendo alcune funzionalità di gestione delle edizioni e delle iscrizioni.

Il laboratorio autonomo serve a consolidare:

- controller REST;
- DTO;
- service;
- repository Spring Data JPA;
- mapper;
- transazioni con `@Transactional`;
- gestione errori.

L'obiettivo non è aggiungere molte classi, ma applicare correttamente la struttura a livelli.

---

## Configurazione database utilizzata

Il laboratorio autonomo parte dalla stessa configurazione database del laboratorio guidato.

La connessione deve usare l'utente amministrativo locale `root` con password vuota:

```properties
spring.datasource.url=jdbc:mariadb://localhost:3306/academy_ud31_guidato
spring.datasource.username=root
spring.datasource.password=
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
```

Se occorre ricreare il database prima di ripetere il laboratorio, usare lo script seguente da terminale o da phpMyAdmin:

```sql
DROP DATABASE IF EXISTS academy_ud31_guidato;

CREATE DATABASE academy_ud31_guidato
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

USE academy_ud31_guidato;
```

Non è richiesta la creazione di un utente applicativo dedicato per questo laboratorio.


---

## Funzionalità richieste

| Funzionalità | Metodo | Endpoint suggerito |
|---|---|---|
| dettaglio edizione | `GET` | `/api/edizioni/{id}` |
| chiusura edizione | `PATCH` | `/api/edizioni/{id}/chiudi` |
| creazione iscrizione | `POST` | `/api/iscrizioni` |
| elenco iscrizioni | `GET` | `/api/iscrizioni` |

La creazione iscrizione è già presente nel guidato. In questo laboratorio deve essere verificata, rifinita e collegata al nuovo comportamento di chiusura edizione.

---

## Regole applicative

Il service deve garantire che:

1. non sia possibile iscriversi a una edizione inesistente;
2. non sia possibile iscriversi a una edizione chiusa;
3. non sia possibile iscriversi a una edizione senza posti disponibili;
4. la chiusura di una edizione inesistente produca errore;
5. la chiusura di una edizione già chiusa non produca duplicazioni di stato;
6. i controller non contengano regole applicative.

---

## Nuovi DTO suggeriti


I DTO aggiunti servono solo a modellare le nuove risposte dell'API. Non devono contenere logica applicativa e non devono sostituire le entity JPA.

### `EdizioneDettaglioResponseDto`

```java
package corso.ud31.academy.dto;

public record EdizioneDettaglioResponseDto(
        Long edizioneId,
        Long corsoId,
        String titoloCorso,
        String docente,
        String dataInizio,
        int postiTotali,
        int postiDisponibili,
        String stato
) {
}
```

**Spiegazione essenziale**

Questo DTO restituisce il dettaglio completo di una edizione. È più ricco di `EdizioneDisponibileResponseDto` perché include anche `postiTotali` e `stato`.


### `EdizioneChiusaResponseDto`

```java
package corso.ud31.academy.dto;

public record EdizioneChiusaResponseDto(
        Long edizioneId,
        String titoloCorso,
        String stato
) {
}
```

**Spiegazione essenziale**

Questo DTO viene restituito dopo la chiusura di una edizione. Contiene solo i dati necessari a confermare al client quale edizione è stata chiusa e con quale stato finale.


---

## Estensione del mapper


Il mapper deve occuparsi solo della conversione da entity a DTO. I controlli sullo stato dell'edizione restano nel service.

Nel mapper delle edizioni aggiungere metodi per dettaglio e chiusura.

```java
public EdizioneDettaglioResponseDto toDettaglioDto(EdizioneCorso edizione) {
    // completare la conversione entity → DTO
}

public EdizioneChiusaResponseDto toChiusaDto(EdizioneCorso edizione) {
    // completare la conversione entity → DTO
}
```

**Spiegazione essenziale**

`toDettaglioDto(...)` prepara la risposta per il dettaglio edizione.

`toChiusaDto(...)` prepara la risposta dopo la chiusura.

Entrambi i metodi ricevono una `EdizioneCorso` già recuperata dal service e restituiscono un DTO pronto per il controller.


Suggerimento: il DTO deve ricevere solo dati utili al client, non l'intera entity.

---

## Estensione del service


Il service è il punto in cui devono stare le regole applicative. Il controller riceve l'id dalla richiesta HTTP, ma non deve decidere se una edizione esiste, se può essere chiusa o se può accettare iscrizioni.

Aggiungere nel service un metodo per il dettaglio edizione.

```java
@Transactional(readOnly = true)
public EdizioneDettaglioResponseDto dettaglioEdizione(Long id) {
    // 1. validare id
    // 2. cercare l'edizione
    // 3. se non esiste, lanciare IllegalArgumentException
    // 4. convertire in DTO
}
```

**Spiegazione essenziale**

`dettaglioEdizione(...)` è un metodo di sola lettura. Per questo usa `@Transactional(readOnly = true)`.

Il metodo deve recuperare l'entity dal repository e trasformarla in DTO tramite il mapper.


Aggiungere un metodo per chiudere l'edizione.

```java
@Transactional
public EdizioneChiusaResponseDto chiudiEdizione(Long id) {
    // 1. validare id
    // 2. cercare l'edizione
    // 3. se non esiste, lanciare IllegalArgumentException
    // 4. chiamare edizione.chiudi()
    // 5. restituire DTO
}
```

**Spiegazione essenziale**

`chiudiEdizione(...)` modifica lo stato della entity. Per questo usa `@Transactional` senza `readOnly = true`.

La modifica avviene chiamando un metodo della entity, `edizione.chiudi()`, mentre il service coordina il caso d'uso.

Nota: non è necessario chiamare `save` se l'entity è managed dentro una transazione. In questa fase è però accettabile usare `save` in modo esplicito se aiuta la leggibilità didattica.

---

## Estensione del controller


Il controller aggiunge solo i nuovi endpoint HTTP. Riceve l'id con `@PathVariable`, chiama il service e restituisce il DTO ricevuto.

Nel controller delle edizioni aggiungere:

```java
@GetMapping("/edizioni/{id}")
public EdizioneDettaglioResponseDto dettaglioEdizione(@PathVariable Long id) {
    return catalogoService.dettaglioEdizione(id);
}
```

**Spiegazione essenziale**

`@PathVariable` prende il valore `{id}` presente nell'URL e lo passa al parametro `Long id` del metodo.


Per la chiusura:

```java
@PatchMapping("/edizioni/{id}/chiudi")
public EdizioneChiusaResponseDto chiudiEdizione(@PathVariable Long id) {
    return catalogoService.chiudiEdizione(id);
}
```

**Spiegazione essenziale**

`@PatchMapping` è adatto quando l'endpoint modifica parzialmente una risorsa esistente. In questo caso non sostituiamo tutta l'edizione: cambiamo solo il suo stato.

Se la gestione delle edizioni viene spostata in un service dedicato, usare un controller dedicato `EdizioneController`.

---

## Test richiesti

### Dettaglio edizione

```bash
curl http://localhost:8080/api/edizioni/1
```

PowerShell:

```powershell
Invoke-RestMethod -Uri "http://localhost:8080/api/edizioni/1"
```

### Chiusura edizione

```bash
curl -X PATCH http://localhost:8080/api/edizioni/1/chiudi
```

PowerShell:

```powershell
Invoke-RestMethod -Method Patch -Uri "http://localhost:8080/api/edizioni/1/chiudi"
```

### Tentativo di iscrizione a edizione chiusa

```bash
curl -X POST http://localhost:8080/api/iscrizioni \
  -H "Content-Type: application/json" \
  -d '{"edizioneId":1,"nomePartecipante":"Mario Rossi","emailPartecipante":"mario.rossi@example.com"}'
```

Risultato atteso: errore `400` con messaggio coerente.

---

## Evidence richieste

Creare:

```text
docs/evidence_UD31_autonomo.md
```

Inserire:

1. elenco delle funzionalità implementate;
2. endpoint realizzati;
3. spiegazione di dove è stato usato `@Transactional`;
4. spiegazione del motivo per cui la chiusura edizione deve stare nel service;
5. confronto tra repository Spring Data e repository manuale UD30;
6. output dei test principali;
7. schema del flusso `PATCH /api/edizioni/{id}/chiudi`;
8. schema del flusso `POST /api/iscrizioni` dopo la chiusura di una edizione.

---

## Criteri di accettazione

Il laboratorio è completo quando:

- il progetto compila;
- l'applicazione parte;
- il dettaglio edizione funziona;
- la chiusura edizione funziona;
- una iscrizione su edizione chiusa viene rifiutata;
- gli errori applicativi producono risposte JSON;
- la logica non è stata inserita nei controller;
- le operazioni di modifica sono protette da transazione;
- DTO ed entity restano separati.
