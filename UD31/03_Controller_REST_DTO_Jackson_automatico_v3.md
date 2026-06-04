# Controller REST, DTO e Jackson automatico

## Obiettivo del file

In UD29 abbiamo realizzato una API HTTP usando `HttpServer`, `HttpHandler`, `HttpExchange` e Jackson esplicito.

In UD31 vediamo come Spring Boot semplifica la stessa logica usando controller REST.

Il punto centrale è questo:

```text
UD29: leggiamo manualmente HTTP e JSON
UD31: Spring gestisce HTTP e Jackson automaticamente
```

---

## Da `HttpHandler` a `@RestController`

In UD29 un endpoint era gestito da una classe che implementava `HttpHandler`.

```java
public class CorsiHandler implements HttpHandler {

    @Override
    public void handle(HttpExchange exchange) throws IOException {
        send(exchange, 200, catalogoService.elencoCorsiPubblicabili());
    }
}
```

In Spring Boot un endpoint viene gestito da un controller.

```java
@RestController
@RequestMapping("/api/corsi")
public class CorsoController {

    private final CatalogoService catalogoService;

    public CorsoController(CatalogoService catalogoService) {
        this.catalogoService = catalogoService;
    }

    @GetMapping
    public List<CorsoResponseDto> elencoCorsi() {
        return catalogoService.elencoCorsiPubblicabili();
    }
}
```

---

## `@RestController`

`@RestController` indica che la classe gestisce richieste HTTP e restituisce dati, non pagine HTML.

Quando un metodo restituisce un oggetto Java, Spring usa Jackson per convertirlo automaticamente in JSON.

```java
@RestController
public class CorsoController {
}
```

Con questa annotazione il controller può restituire direttamente DTO.

---

## `@RequestMapping`

`@RequestMapping` definisce un prefisso comune per gli endpoint della classe.

```java
@RequestMapping("/api/corsi")
public class CorsoController {
}
```

Tutti i metodi del controller partiranno da:

```text
/api/corsi
```

---

## `@GetMapping`

`@GetMapping` associa un metodo Java a una richiesta HTTP `GET`.

```java
@GetMapping
public List<CorsoResponseDto> elencoCorsi() {
    return catalogoService.elencoCorsiPubblicabili();
}
```

Questo metodo risponde a:

```http
GET /api/corsi
```

Spring:

1. riceve la richiesta HTTP;
2. chiama il metodo Java;
3. prende la lista di DTO restituita;
4. usa Jackson per convertirla in JSON;
5. invia la risposta HTTP.

---

## `@PostMapping`

`@PostMapping` associa un metodo Java a una richiesta HTTP `POST`.

```java
@PostMapping
public ResponseEntity<IscrizioneResponseDto> creaIscrizione(
        @RequestBody CreaIscrizioneRequestDto request
) {
    IscrizioneResponseDto response = service.creaIscrizione(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

Questo metodo risponde a:

```http
POST /api/iscrizioni
```

---

## `@RequestBody`

`@RequestBody` indica che il body JSON della richiesta deve essere convertito in un oggetto Java.

```java
public ResponseEntity<IscrizioneResponseDto> creaIscrizione(
        @RequestBody CreaIscrizioneRequestDto request
) {
    ...
}
```

Se il client invia:

```json
{
  "edizioneId": 1,
  "nomePartecipante": "Mario Rossi",
  "emailPartecipante": "mario.rossi@example.com"
}
```

Spring usa Jackson per creare un oggetto `CreaIscrizioneRequestDto`.

In UD29 scrivevamo manualmente:

```java
CreaIscrizioneRequestDto request = JsonRequestReader.fromJson(
        body,
        CreaIscrizioneRequestDto.class
);
```

In UD31 questa conversione è gestita automaticamente da Spring.

---

## Jackson automatico

Spring Boot include e configura Jackson quando nel progetto è presente lo starter web.

Per questo possiamo scrivere:

```java
return catalogoService.elencoCorsiPubblicabili();
```

e ottenere una risposta JSON.

Non dobbiamo più chiamare manualmente:

```java
objectMapper.writeValueAsString(...)
```

Non perché Jackson sia sparito, ma perché Spring Boot lo usa dietro le quinte.

---

## `ResponseEntity`

`ResponseEntity` permette di controllare la risposta HTTP in modo esplicito.

Con `ResponseEntity` possiamo decidere:

- status code;
- body;
- eventuali header.

Esempio:

```java
return ResponseEntity.status(HttpStatus.CREATED).body(response);
```

Significa:

```text
status code: 201 Created
body: DTO di risposta convertito in JSON
```

Per un elenco semplice possiamo anche restituire direttamente una lista:

```java
@GetMapping
public List<CorsoResponseDto> elencoCorsi() {
    return service.elencoCorsiPubblicabili();
}
```

Spring risponderà con `200 OK`.

---

## `@PathVariable`

`@PathVariable` permette di leggere un valore presente nel percorso dell'URL.

Esempio:

```java
@GetMapping("/{id}")
public EdizioneDettaglioResponseDto dettaglio(@PathVariable Long id) {
    return service.dettaglioEdizione(id);
}
```

La richiesta:

```http
GET /api/edizioni/3
```

produce:

```java
id = 3L
```

---

## Gestione degli errori

In una API REST non basta restituire dati corretti. Bisogna anche restituire errori coerenti.

Per esempio:

- edizione non trovata;
- posti esauriti;
- email non valida;
- richiesta non corretta.

Un approccio semplice è usare eccezioni nel service e gestirle con un controller advice.

```java
@RestControllerAdvice
public class ApiExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErroreResponseDto> handleIllegalArgument(IllegalArgumentException e) {
        return ResponseEntity
                .badRequest()
                .body(new ErroreResponseDto(e.getMessage()));
    }
}
```

In questo modo il service può segnalare errori applicativi e il controller advice li trasforma in risposte HTTP.

---
```java
package corso.ud31.academy.exception;

import corso.ud31.academy.dto.ErroreResponseDto;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class ApiExceptionHandler {

    /*
     * Questo metodo viene eseguito automaticamente da Spring
     * quando, durante la gestione di una richiesta HTTP,
     * viene lanciata una IllegalArgumentException.
     *
     * Esempi di casi possibili:
     * - edizione non trovata;
     * - email non valida;
     * - posti esauriti;
     * - dati obbligatori mancanti.
     */
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErroreResponseDto> handleIllegalArgument(IllegalArgumentException e) {

        /*
         * Creiamo un DTO di errore da restituire al client.
         * Il messaggio dell'eccezione diventa il contenuto della risposta.
         */
        ErroreResponseDto errore = new ErroreResponseDto(e.getMessage());

        /*
         * ResponseEntity.badRequest() produce una risposta HTTP 400.
         *
         * Il codice 400 Bad Request indica che la richiesta ricevuta
         * non è valida dal punto di vista applicativo.
         */
        return ResponseEntity
                .badRequest()
                .body(errore);
    }
}
```


## Flusso di una POST in Spring Boot

```text
Client HTTP
↓
POST /api/iscrizioni con JSON body
↓
Controller Spring
↓
@RequestBody
↓
Jackson crea CreaIscrizioneRequestDto
↓
Service
↓
Repository
↓
Mapper
↓
IscrizioneResponseDto
↓
Jackson produce JSON
↓
HTTP 201 Created
```

---

## Sintesi

| In UD29 | In UD31 |
|---|---|
| `HttpHandler` | `@RestController` |
| `HttpExchange` | parametri del metodo controller |
| lettura manuale body | `@RequestBody` |
| `JsonRequestReader` | Jackson automatico in input |
| `JsonResponseWriter` | Jackson automatico in output |
| status code manuale | `ResponseEntity` |
| handler di errore manuale | `@RestControllerAdvice` |

Spring Boot semplifica il codice HTTP, ma la separazione tra controller, service, repository, DTO e mapper resta necessaria.
