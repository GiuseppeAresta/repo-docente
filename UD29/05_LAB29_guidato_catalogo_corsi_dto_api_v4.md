# LAB29 guidato - Catalogo corsi con DTO, API REST e Jackson

## Obiettivo

Costruire passo passo una piccola API REST locale per pubblicare un catalogo corsi senza esporre direttamente il model interno.

Il laboratorio guida nella progettazione di:

- model interno;
- DTO di risposta;
- mapper manuale;
- repository in memoria;
- service;
- handler HTTP;
- endpoint REST;
- serializzazione JSON con Jackson.

## Idea centrale

Una API REST restituisce una rappresentazione dei dati. In questo laboratorio la rappresentazione è JSON.

Il JSON non viene costruito concatenando stringhe: partiamo da DTO Java e usiamo Jackson per convertirli in JSON.

```text
Model interno
↓
Mapper
↓
DTO di risposta
↓
Jackson
↓
JSON
↓
Risposta HTTP
```

## Requisiti software

| Software | Utilizzo |
|---|---|
| JDK 17 o superiore | Compilazione ed esecuzione |
| Maven | Gestione progetto e dipendenze |
| Terminale | Comandi Maven, `curl` o `Invoke-RestMethod` |
| Editor Java | Scrittura del codice |

Docker e database non sono richiesti in questo laboratorio.

## Struttura progetto

```text
UD29_catalogo_corsi_api/
  pom.xml
  src/main/java/corso/ud29/catalogo/
    app/
    controller/
    dto/
    mapper/
    model/
    repository/
    service/
    util/
  docs/
```

## Passo 1 - Creare il progetto Maven

Creare la cartella del progetto.

### Windows PowerShell

```powershell
New-Item -ItemType Directory -Force UD29_catalogo_corsi_api
Set-Location UD29_catalogo_corsi_api
New-Item -ItemType Directory -Force src/main/java/corso/ud29/catalogo/app
New-Item -ItemType Directory -Force src/main/java/corso/ud29/catalogo/controller
New-Item -ItemType Directory -Force src/main/java/corso/ud29/catalogo/dto
New-Item -ItemType Directory -Force src/main/java/corso/ud29/catalogo/mapper
New-Item -ItemType Directory -Force src/main/java/corso/ud29/catalogo/model
New-Item -ItemType Directory -Force src/main/java/corso/ud29/catalogo/repository
New-Item -ItemType Directory -Force src/main/java/corso/ud29/catalogo/service
New-Item -ItemType Directory -Force src/main/java/corso/ud29/catalogo/util
New-Item -ItemType Directory -Force docs
```

### Linux/macOS

```bash
mkdir -p UD29_catalogo_corsi_api
cd UD29_catalogo_corsi_api
mkdir -p src/main/java/corso/ud29/catalogo/app
mkdir -p src/main/java/corso/ud29/catalogo/controller
mkdir -p src/main/java/corso/ud29/catalogo/dto
mkdir -p src/main/java/corso/ud29/catalogo/mapper
mkdir -p src/main/java/corso/ud29/catalogo/model
mkdir -p src/main/java/corso/ud29/catalogo/repository
mkdir -p src/main/java/corso/ud29/catalogo/service
mkdir -p src/main/java/corso/ud29/catalogo/util
mkdir -p docs
```

## Passo 2 - Creare il file `pom.xml`

Creare il file `pom.xml` nella root del progetto.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>corso.ud29</groupId>
    <artifactId>catalogo-corsi-api</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.release>17</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.17.2</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>3.5.0</version>
                <configuration>
                    <mainClass>corso.ud29.catalogo.app.CatalogoApiApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### Spiegazione

La dipendenza `jackson-databind` permette di usare `ObjectMapper`, cioè la classe che useremo per convertire DTO Java in JSON.

L'`exec-maven-plugin` permette di avviare l'applicazione con:

```bash
mvn exec:java
```

## Passo 3 - Creare il model interno

Creare il file:

```text
src/main/java/corso/ud29/catalogo/model/Corso.java
```

```java
package corso.ud29.catalogo.model;

public class Corso {
    private final long id;
    private final String titolo;
    private final String descrizione;
    private final double prezzo;
    private final boolean attivo;

    public Corso(long id, String titolo, String descrizione, double prezzo, boolean attivo) {
        this.id = id;
        this.titolo = titolo;
        this.descrizione = descrizione;
        this.prezzo = prezzo;
        this.attivo = attivo;
    }

    public long getId() {
        return id;
    }

    public String getTitolo() {
        return titolo;
    }

    public String getDescrizione() {
        return descrizione;
    }

    public double getPrezzo() {
        return prezzo;
    }

    public boolean isAttivo() {
        return attivo;
    }
}
```

Il model contiene lo stato interno del corso. Non tutti i campi devono essere esposti al client.

## Passo 4 - Creare il DTO di risposta

Creare il file:

```text
src/main/java/corso/ud29/catalogo/dto/CorsoResponseDto.java
```

```java
package corso.ud29.catalogo.dto;

public class CorsoResponseDto {
    private final long id;
    private final String titolo;
    private final double prezzo;

    public CorsoResponseDto(long id, String titolo, double prezzo) {
        this.id = id;
        this.titolo = titolo;
        this.prezzo = prezzo;
    }

    public long getId() {
        return id;
    }

    public String getTitolo() {
        return titolo;
    }

    public double getPrezzo() {
        return prezzo;
    }
}
```

Il DTO non contiene `descrizione` e `attivo`, perché l'elenco pubblico del catalogo richiede solo una vista sintetica.

## Passo 5 - Creare il mapper

Creare il file:

```text
src/main/java/corso/ud29/catalogo/mapper/CorsoMapper.java
```

```java
package corso.ud29.catalogo.mapper;

import corso.ud29.catalogo.dto.CorsoResponseDto;
import corso.ud29.catalogo.model.Corso;

public class CorsoMapper {

    private CorsoMapper() {
    }

    public static CorsoResponseDto toResponseDto(Corso corso) {
        return new CorsoResponseDto(
                corso.getId(),
                corso.getTitolo(),
                corso.getPrezzo()
        );
    }
}
```

Il mapper centralizza la conversione. L'handler HTTP non deve costruire direttamente il DTO.

## Passo 6 - Creare il repository in memoria

Creare il file:

```text
src/main/java/corso/ud29/catalogo/repository/CorsoRepository.java
```

```java
package corso.ud29.catalogo.repository;

import corso.ud29.catalogo.model.Corso;
import java.util.List;

public class CorsoRepository {
    private final List<Corso> corsi = List.of(
            new Corso(1, "Java Object Oriented", "Classi, oggetti e relazioni", 350.0, true),
            new Corso(2, "Java Web", "HTTP, servlet e web app", 420.0, true),
            new Corso(3, "Spring Boot", "Applicazioni enterprise", 600.0, true),
            new Corso(4, "Corso archiviato", "Non pubblicabile", 200.0, false)
    );

    public List<Corso> findAll() {
        return corsi;
    }
}
```

Il repository isola il recupero dei dati. In questa UD i dati sono in memoria; nelle UD successive torneremo a database e persistenza.

## Passo 7 - Creare il service

Creare il file:

```text
src/main/java/corso/ud29/catalogo/service/CatalogoService.java
```

```java
package corso.ud29.catalogo.service;

import corso.ud29.catalogo.dto.CorsoResponseDto;
import corso.ud29.catalogo.mapper.CorsoMapper;
import corso.ud29.catalogo.repository.CorsoRepository;
import java.util.List;

public class CatalogoService {
    private final CorsoRepository repository;

    public CatalogoService(CorsoRepository repository) {
        this.repository = repository;
    }

    public List<CorsoResponseDto> elencoCorsiPubblici() {
        return repository.findAll().stream()
                .filter(corso -> corso.isAttivo())
                .map(CorsoMapper::toResponseDto)
                .toList();
    }
}
```

Qui il service applica una regola: solo i corsi attivi vengono pubblicati.

## Passo 8 - Serializzare DTO in JSON con Jackson

Creare il file:

```text
src/main/java/corso/ud29/catalogo/util/JsonResponseWriter.java
```

```java
package corso.ud29.catalogo.util;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;

public class JsonResponseWriter {

    private static final ObjectMapper objectMapper = new ObjectMapper();

    private JsonResponseWriter() {
    }

    public static String toJson(Object value) {
        try {
            return objectMapper.writeValueAsString(value);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("Errore durante la serializzazione JSON", e);
        }
    }
}
```

### Cosa succede qui

`ObjectMapper` è la classe principale di Jackson.

Il metodo:

```java
objectMapper.writeValueAsString(value)
```

trasforma un oggetto Java, o una lista di oggetti Java, in una stringa JSON.

Non costruiamo JSON manualmente. Il compito della classe `JsonResponseWriter` è isolare la serializzazione in un punto riconoscibile del progetto.

## Passo 9 - Creare l'handler HTTP

Creare il file:

```text
src/main/java/corso/ud29/catalogo/controller/CorsiHandler.java
```

Prima di scrivere il codice, è utile chiarire il ruolo di questa classe.

`CorsiHandler` è il punto in cui una richiesta HTTP concreta viene collegata alla logica applicativa. In una applicazione Spring questo ruolo sarà svolto da un controller annotato, per esempio con `@RestController` e `@GetMapping`. In questo laboratorio, invece, usiamo il server HTTP minimale del JDK, quindi dobbiamo creare esplicitamente una classe che implementa `HttpHandler`.

Il flusso gestito da `CorsiHandler` è questo:

```text
richiesta HTTP GET /api/corsi
↓
metodo handle(HttpExchange exchange)
↓
verifica del metodo HTTP
↓
chiamata al service
↓
lista di DTO
↓
serializzazione JSON con Jackson
↓
risposta HTTP con status code e Content-Type
```

Nel codice seguente sono presenti commenti didattici sulle istruzioni principali. Non tutti questi commenti sarebbero necessari in un progetto reale, ma in questa fase aiutano a capire il ruolo delle singole righe.

```java
package corso.ud29.catalogo.controller;

import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import corso.ud29.catalogo.dto.CorsoResponseDto;
import corso.ud29.catalogo.service.CatalogoService;
import corso.ud29.catalogo.util.JsonResponseWriter;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.List;

// HttpHandler è l'interfaccia del server HTTP del JDK.
// Una classe che implementa HttpHandler può gestire le richieste
// dirette a uno specifico endpoint registrato nel server.
public class CorsiHandler implements HttpHandler {

    // L'handler non contiene direttamente la logica applicativa.
    // Per ottenere i corsi pubblicabili usa il service.
    private final CatalogoService service;

    // Il service viene ricevuto dall'esterno.
    // Questa è una forma semplice di dependency injection manuale:
    // l'handler non crea il service, ma lo riceve già pronto.
    public CorsiHandler(CatalogoService service) {
        this.service = service;
    }

    // Il metodo handle viene chiamato automaticamente dal server
    // ogni volta che arriva una richiesta sull'endpoint associato
    // a questo handler, cioè /api/corsi.
    @Override
    public void handle(HttpExchange exchange) throws IOException {

        // HttpExchange rappresenta lo scambio HTTP completo:
        // contiene la richiesta ricevuta e permette di costruire la risposta.
        // Qui leggiamo il metodo HTTP usato dal client: GET, POST, PUT, DELETE, ecc.
        if (!"GET".equals(exchange.getRequestMethod())) {
            // L'endpoint supporta solo GET.
            // Se arriva un metodo diverso, restituiamo 405 Method Not Allowed.
            send(exchange, 405, "{\"errore\":\"Metodo non supportato\"}");
            return;
        }

        // Il service applica la regola applicativa:
        // restituisce solo i corsi pubblicabili e già convertiti in DTO.
        List<CorsoResponseDto> corsi = service.elencoCorsiPubblici();

        // JsonResponseWriter usa Jackson per trasformare la lista di DTO
        // in una stringa JSON da inserire nel body della risposta HTTP.
        String json = JsonResponseWriter.toJson(corsi);

        // Inviamo al client una risposta HTTP 200 OK con body JSON.
        send(exchange, 200, json);
    }

    // Metodo di supporto usato per inviare una risposta HTTP.
    // Centralizzare questa parte evita di ripetere le stesse istruzioni
    // ogni volta che dobbiamo restituire un body al client.
    private void send(HttpExchange exchange, int statusCode, String body) throws IOException {

        // Il body della risposta HTTP deve essere inviato come byte.
        // Per questo convertiamo la stringa JSON in byte usando UTF-8.
        byte[] response = body.getBytes(StandardCharsets.UTF_8);

        // Content-Type informa il client che il contenuto della risposta è JSON.
        exchange.getResponseHeaders().set("Content-Type", "application/json; charset=utf-8");

        // Qui inviamo lo status code HTTP e la lunghezza del body.
        // Esempio: 200 con il JSON dei corsi oppure 405 con un messaggio di errore.
        exchange.sendResponseHeaders(statusCode, response.length);

        // Scriviamo materialmente i byte del body nella risposta HTTP.
        exchange.getResponseBody().write(response);

        // Chiudiamo lo scambio HTTP.
        // Dopo questa istruzione la risposta è completata.
        exchange.close();
    }
}
```

### Analisi del codice `CorsiHandler`

La riga:

```java
public class CorsiHandler implements HttpHandler
```

indica che `CorsiHandler` è una classe capace di gestire richieste HTTP ricevute dal server del JDK.

Il campo:

```java
private final CatalogoService service;
```

mostra che l'handler dipende dal service, non dal repository. Questa scelta è importante: l'handler deve occuparsi di HTTP, mentre le regole applicative restano nel service.

Il costruttore:

```java
public CorsiHandler(CatalogoService service)
```

permette di passare il service dall'esterno. In questo modo la classe non decide come costruire il service e non conosce direttamente il repository. È una forma semplice di composizione manuale degli oggetti.

Il metodo:

```java
public void handle(HttpExchange exchange)
```

è il metodo chiamato dal server quando arriva una richiesta. Il parametro `exchange` contiene sia informazioni della richiesta sia strumenti per costruire la risposta.

La verifica:

```java
if (!"GET".equals(exchange.getRequestMethod()))
```

serve a limitare l'endpoint al solo metodo `GET`. Se il client prova a usare `POST`, `PUT` o `DELETE`, il server risponde con `405`.

La chiamata:

```java
List<CorsoResponseDto> corsi = service.elencoCorsiPubblici();
```

è il punto in cui l'handler passa dal livello HTTP al livello applicativo. L'handler non filtra i corsi e non costruisce i DTO: chiede il risultato al service.

La riga:

```java
String json = JsonResponseWriter.toJson(corsi);
```

trasforma la lista di DTO in JSON. Il JSON viene prodotto con Jackson, non con concatenazione manuale di stringhe.

Il metodo `send` prepara la risposta HTTP: imposta header, status code, body e chiusura dello scambio.

### Responsabilità dell'handler

L'handler conosce HTTP. Il service conosce la regola applicativa. Il repository conosce i dati. Jackson conosce la conversione Java → JSON.

## Passo 10 - Creare la classe di avvio

Creare il file:

```text
src/main/java/corso/ud29/catalogo/app/CatalogoApiApplication.java
```

Prima di scrivere la classe di avvio, è utile chiarire che qui non stiamo ancora usando un framework come Spring Boot. Non esiste quindi un contenitore che crea automaticamente repository, service e controller.

In questo laboratorio la composizione dell'applicazione viene fatta manualmente nel `main`.

`CatalogoApiApplication` ha tre responsabilità principali:

1. creare gli oggetti principali dell'applicazione;
2. collegarli tra loro;
3. avviare il server HTTP locale.

```java
package corso.ud29.catalogo.app;

import com.sun.net.httpserver.HttpServer;
import corso.ud29.catalogo.controller.CorsiHandler;
import corso.ud29.catalogo.repository.CorsoRepository;
import corso.ud29.catalogo.service.CatalogoService;
import java.io.IOException;
import java.net.InetSocketAddress;

public class CatalogoApiApplication {

    public static void main(String[] args) throws IOException {

        // 1. Creiamo il repository.
        // In questo laboratorio il repository usa una lista in memoria.
        // In una applicazione successiva potrebbe leggere da database.
        CorsoRepository repository = new CorsoRepository();

        // 2. Creiamo il service e gli passiamo il repository.
        // Il service userà il repository per recuperare i corsi
        // e applicherà la regola: pubblicare solo corsi attivi.
        CatalogoService service = new CatalogoService(repository);

        // 3. Creiamo il server HTTP locale sulla porta 8080.
        // InetSocketAddress indica indirizzo/porta di ascolto.
        // Il secondo parametro è il backlog: 0 lascia il valore predefinito al sistema.
        HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);

        // 4. Registriamo l'endpoint /api/corsi.
        // Quando arriva una richiesta su /api/corsi,
        // il server la consegna a un nuovo CorsiHandler.
        // Anche qui usiamo dependency injection manuale:
        // il CorsiHandler riceve il service già costruito.
        server.createContext("/api/corsi", new CorsiHandler(service));

        // 5. Avviamo il server.
        // Da questo momento il programma resta in ascolto di richieste HTTP.
        server.start();

        // 6. Messaggi informativi per chi esegue l'applicazione.
        // Non fanno parte della API: servono solo a confermare l'avvio.
        System.out.println("Catalogo API avviata su http://localhost:8080");
        System.out.println("Endpoint: GET /api/corsi");
    }
}
```

### Analisi del codice `CatalogoApiApplication`

Questa classe si trova nel package `app` perché rappresenta il punto di avvio dell'applicazione.

Questa riga:

```java
CorsoRepository repository = new CorsoRepository();
```

crea il repository in memoria. In questo laboratorio il repository contiene una lista di corsi già pronta. In una applicazione successiva, questo oggetto potrebbe essere sostituito da un repository che legge da database.

Questa riga:

```java
CatalogoService service = new CatalogoService(repository);
```

crea il service e gli passa il repository. Il service dipende dal repository perché deve recuperare i corsi e applicare la regola sui soli corsi attivi.

Il flusso di dipendenza è quindi:

```text
CatalogoService
↓ usa
CorsoRepository
```

Questa riga crea il server HTTP locale:

```java
HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
```

`new InetSocketAddress(8080)` indica che il server ascolterà sulla porta `8080` della macchina locale. Il secondo parametro, `0`, indica il backlog, cioè la coda delle connessioni in attesa. In questo laboratorio lasciamo `0`, delegando al sistema il valore predefinito.

Questa riga registra l'endpoint:

```java
server.createContext("/api/corsi", new CorsiHandler(service));
```

Significa:

```text
quando arriva una richiesta su /api/corsi,
usa un oggetto CorsiHandler per gestirla
```

Il nuovo `CorsiHandler` riceve il service:

```java
new CorsiHandler(service)
```

Anche qui vediamo una dependency injection manuale: l'handler non crea il service, ma lo riceve dall'applicazione principale.

Questa riga avvia il server:

```java
server.start();
```

Dopo questa istruzione, il programma resta in esecuzione e attende richieste HTTP. Per fermarlo dal terminale si può usare `CTRL + C`.

I messaggi finali:

```java
System.out.println("Catalogo API avviata su http://localhost:8080");
System.out.println("Endpoint: GET /api/corsi");
```

non fanno parte della API. Servono solo a confermare all'utente che il server è stato avviato e quale endpoint può essere testato.

### Perché questa classe non contiene logica applicativa

`CatalogoApiApplication` non deve decidere quali corsi mostrare, non deve convertire DTO in JSON e non deve gestire direttamente le richieste HTTP. Il suo compito è comporre e avviare l'applicazione.

In questa UD stiamo facendo manualmente un lavoro che in Spring Boot verrà svolto in modo più automatico:

| In questa UD | In Spring Boot |
|---|---|
| creiamo manualmente repository e service | Spring crea i bean |
| registriamo manualmente `/api/corsi` | Spring usa `@GetMapping` |
| usiamo `HttpServer` del JDK | Spring avvia un server embedded |
| usiamo direttamente Jackson | Spring usa Jackson automaticamente |

### Errore frequente: porta già occupata

Se la porta `8080` è già usata da un'altra applicazione, l'avvio può fallire. In quel caso è possibile cambiare porta, ad esempio:

```java
HttpServer server = HttpServer.create(new InetSocketAddress(18080), 0);
```

Il test andrà poi eseguito su:

```text
http://localhost:18080/api/corsi
```

## Passo 11 - Compilazione ed esecuzione

Compilare:

```bash
mvn clean compile
```

Eseguire:

```bash
mvn exec:java
```

Il comando è uguale su Windows, Linux e macOS.

## Passo 12 - Test

Aprire un secondo terminale.

### Linux/macOS

```bash
curl http://localhost:8080/api/corsi
```

### Windows PowerShell

```powershell
Invoke-RestMethod -Uri "http://localhost:8080/api/corsi"
```

Il risultato atteso è un array JSON con i corsi attivi.

Esempio:

```json
[
  {"id":1,"titolo":"Java Object Oriented","prezzo":350.0},
  {"id":2,"titolo":"Java Web","prezzo":420.0},
  {"id":3,"titolo":"Spring Boot","prezzo":600.0}
]
```

Il corso archiviato non viene restituito perché il service pubblica solo corsi attivi.

## Evidence richiesta

Creare il file:

```text
docs/evidence_UD29_guidato.md
```

Inserire:

1. struttura finale del progetto;
2. contenuto essenziale del `pom.xml`;
3. output di `mvn clean compile`;
4. output di `mvn exec:java` o messaggio di avvio del server;
5. comando usato per testare `GET /api/corsi`;
6. risposta JSON ottenuta;
7. spiegazione del ruolo di `ObjectMapper`;
8. spiegazione del motivo per cui non costruiamo JSON con concatenazione di stringhe;
9. spiegazione del ruolo di `CorsiHandler`;
10. spiegazione del ruolo di `CatalogoApiApplication`;
11. spiegazione del significato di `server.createContext("/api/corsi", new CorsiHandler(service))`.

## Domande di consolidamento

1. Quali campi sono presenti nel model `Corso` ma non nel DTO `CorsoResponseDto`?
2. Perché il DTO non coincide con il model?
3. In quale punto viene deciso che solo i corsi attivi sono pubblicati?
4. Quale livello conosce HTTP?
5. Quale livello conosce il repository?
6. Quale livello costruisce i DTO?
7. Quale classe trasforma i DTO in JSON?
8. Perché `CorsiHandler` riceve un `CatalogoService` nel costruttore?
9. Che cosa rappresenta `HttpExchange` nel metodo `handle`?
10. Che cosa fa `server.createContext("/api/corsi", new CorsiHandler(service))`?
11. Perché `CatalogoApiApplication` non deve contenere regole applicative?
12. Cosa cambierebbe se il repository usasse un database invece di una lista in memoria?
