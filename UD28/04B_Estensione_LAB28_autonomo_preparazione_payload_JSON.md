# 04B - Estensione LAB28 autonomo: preparare un payload JSON per una futura API

## Collocazione dell'estensione

Questa estensione si svolge dopo il laboratorio autonomo:

```text
04_LAB28_autonomo_portale_academy_frontend_v2.md
```

Nel laboratorio autonomo abbiamo lavorato su una pagina front-end che raccoglie dati nel browser. In questa estensione prepariamo quei dati in una forma più vicina a ciò che useremo nella UD29: un payload JSON.

Non inviamo ancora dati a una API reale. Non usiamo `fetch`, backend Java, Spring o database. L'obiettivo è capire come un oggetto JavaScript può essere trasformato in una stringa JSON pronta per essere inviata in una futura richiesta HTTP.

## Obiettivo dell'estensione

Al termine dell'attività saremo in grado di:

- distinguere un oggetto JavaScript da una stringa JSON;
- costruire un oggetto coerente con i dati del form;
- trasformare un oggetto JavaScript in JSON con `JSON.stringify()`;
- visualizzare il payload JSON nella pagina;
- preparare il collegamento concettuale con DTO e API REST della UD29;
- documentare il payload prodotto nell'evidence.

## Concetto principale

Quando compiliamo un form, JavaScript può costruire un oggetto in memoria:

```javascript
const attivita = {
    titolo: "Java Web",
    area: "Java",
    docente: "Mario Rossi",
    durataOre: 24,
    modalita: "online",
    stato: "pianificata"
};
```

Questo oggetto non è ancora JSON. È un oggetto JavaScript.

Per ottenere una stringa JSON possiamo usare:

```javascript
const json = JSON.stringify(attivita, null, 2);
```

Il risultato sarà un testo simile:

```json
{
  "titolo": "Java Web",
  "area": "Java",
  "docente": "Mario Rossi",
  "durataOre": 24,
  "modalita": "online",
  "stato": "pianificata"
}
```

Nella UD29 vedremo che una API REST riceve e restituisce JSON. Questa estensione prepara quel passaggio.

## Parte 1 - Aggiungere un riquadro per il payload JSON

Nel file `index.html`, individuare un punto sotto il form o sotto la tabella.

Aggiungere una sezione dedicata al payload JSON.

```html
<section class="mt-4">
    <h2 class="h5">Payload JSON preparato per una futura API</h2>
    <p class="text-muted">
        Il riquadro mostra come i dati inseriti nel form potrebbero essere rappresentati
        nel body di una richiesta HTTP verso una API REST.
    </p>
    <pre id="payloadJson" class="bg-light border rounded p-3">Nessun payload generato.</pre>
</section>
```

### Spiegazione

| Elemento | Ruolo |
|---|---|
| `<section>` | Raggruppa una parte della pagina |
| `<h2>` | Titolo della sezione |
| `<p>` | Spiegazione testuale per chi usa la pagina |
| `<pre>` | Mostra testo preformattato, utile per visualizzare JSON indentato |
| `id="payloadJson"` | Permette a JavaScript di aggiornare il contenuto del riquadro |

## Parte 2 - Costruire un oggetto JavaScript dai dati del form

Nel file `app.js`, individuare il punto in cui vengono letti i valori del form.

Creare o adattare un oggetto JavaScript con una struttura simile:

```javascript
const attivita = {
    titolo: titolo,
    area: area,
    docente: docente,
    durataOre: durataOre,
    modalita: modalita,
    stato: "pianificata"
};
```

I nomi delle proprietà possono variare in base al laboratorio autonomo già svolto. L'importante è che l'oggetto rappresenti in modo chiaro l'attività formativa inserita.

### Nota sui nomi delle proprietà

Usare nomi leggibili e coerenti:

| Nome consigliato | Significato |
|---|---|
| `titolo` | titolo dell'attività o del corso |
| `area` | area didattica o tecnologia principale |
| `docente` | docente assegnato |
| `durataOre` | durata numerica in ore |
| `modalita` | aula, online, blended |
| `stato` | stato operativo, ad esempio `pianificata` |

Questi nomi somigliano ai nomi che potremmo trovare in un DTO di richiesta nella UD29.

## Parte 3 - Creare una funzione dedicata al payload

Aggiungere in `app.js` una funzione dedicata.

```javascript
function creaPayloadApi(attivita) {
    return {
        titolo: attivita.titolo,
        area: attivita.area,
        docente: attivita.docente,
        durataOre: attivita.durataOre,
        modalita: attivita.modalita,
        stato: attivita.stato
    };
}
```

### Perché creare una funzione separata

La funzione rende esplicito un passaggio importante:

```text
oggetto usato dalla pagina
↓
oggetto preparato per una futura API
```

In questa UD i due oggetti possono essere quasi identici. Nelle applicazioni reali, però, il dato usato dalla pagina e il dato inviato a una API non sempre coincidono.

Questo anticipa il concetto di DTO che useremo nella UD29.

## Parte 4 - Convertire l'oggetto in JSON

Aggiungere una funzione per mostrare il payload nel riquadro.

```javascript
function aggiornaPayloadJson(attivita) {
    const payload = creaPayloadApi(attivita);
    const json = JSON.stringify(payload, null, 2);
    $("#payloadJson").text(json);
}
```

### Spiegazione riga per riga

```javascript
const payload = creaPayloadApi(attivita);
```

Prepara l'oggetto che rappresenta ciò che potremmo inviare a una API.

```javascript
const json = JSON.stringify(payload, null, 2);
```

Converte l'oggetto JavaScript in una stringa JSON.

Il parametro `null` indica che non vogliamo trasformare o filtrare le proprietà.

Il parametro `2` indica che il JSON deve essere indentato con due spazi, così è più leggibile.

```javascript
$("#payloadJson").text(json);
```

Scrive la stringa JSON dentro l'elemento `<pre>`.

Usiamo `.text()` e non `.html()` perché vogliamo mostrare il JSON come testo, senza interpretarlo come HTML.

## Parte 5 - Collegare il payload all'inserimento nel form

Dopo aver creato l'oggetto dell'attività e dopo aver superato la validazione, chiamare:

```javascript
aggiornaPayloadJson(attivita);
```

Esempio di posizione nel flusso:

```javascript
$("#formAttivita").on("submit", function (event) {
    event.preventDefault();

    // 1. leggere i valori del form
    // 2. validare i dati
    // 3. creare l'oggetto attivita
    // 4. aggiungerlo all'array in memoria
    // 5. aggiornare la tabella
    // 6. mostrare il payload JSON
    aggiornaPayloadJson(attivita);
});
```

Adattare il nome del form e il nome della variabile `attivita` alla soluzione realizzata nel laboratorio autonomo.

## Parte 6 - Verifica nella pagina

Aprire la pagina nel browser e inserire una nuova attività.

Il riquadro JSON deve mostrare un contenuto simile:

```json
{
  "titolo": "Java Web",
  "area": "Java",
  "docente": "Mario Rossi",
  "durataOre": 24,
  "modalita": "online",
  "stato": "pianificata"
}
```

Verificare che:

- il JSON cambi quando vengono inseriti dati diversi;
- i numeri rimangano numeri e non stringhe, quando possibile;
- il testo sia leggibile e indentato;
- il riquadro venga aggiornato dopo una validazione corretta.

## Parte 7 - Controllo in console

Aggiungere temporaneamente:

```javascript
console.log("Payload API:", creaPayloadApi(attivita));
console.log("Payload JSON:", JSON.stringify(creaPayloadApi(attivita)));
```

La prima riga mostra l'oggetto JavaScript.

La seconda riga mostra la stringa JSON compatta.

Questa distinzione è importante:

| Forma | Esempio | Tipo |
|---|---|---|
| Oggetto JavaScript | `{ titolo: "Java Web" }` | struttura dati in memoria |
| JSON | `{"titolo":"Java Web"}` | testo |

## Parte 8 - Domande di ragionamento

Rispondere nell'evidence.

1. Qual è la differenza tra oggetto JavaScript e JSON?
2. A cosa serve `JSON.stringify()`?
3. Perché usiamo un elemento `<pre>` per mostrare il JSON?
4. Perché `.text()` è più adatto di `.html()` in questo caso?
5. Quali proprietà del payload potrebbero diventare campi di un DTO nella UD29?
6. Perché questa estensione prepara il concetto di API REST?

## Evidence richiesta

Nel file:

```text
docs/evidence_LAB28_autonomo.md
```

aggiungere una sezione:

```markdown
## Estensione autonoma - Payload JSON

### Codice HTML aggiunto
Incollare la sezione con il riquadro `<pre id="payloadJson">`.

### Funzioni JavaScript aggiunte
Incollare `creaPayloadApi` e `aggiornaPayloadJson`.

### Esempio di payload prodotto
Incollare un JSON generato dalla pagina.

### Confronto oggetto JavaScript / JSON
Scrivere una breve spiegazione.

### Domande
1. Qual è la differenza tra oggetto JavaScript e JSON?
2. A cosa serve `JSON.stringify()`?
3. Quale collegamento esiste tra questo payload e i DTO della UD29?
```

## Criteri di completamento

L'estensione è completata se:

- la pagina mostra un riquadro JSON;
- il payload viene generato dopo un inserimento valido;
- il JSON contiene dati coerenti con il form;
- `JSON.stringify()` viene usato correttamente;
- l'evidence contiene codice, output e risposte di ragionamento.

## Collegamento con UD29

Nella UD29 useremo il concetto di DTO e API REST.

La relazione può essere letta così:

```text
UD28
oggetto JavaScript nel browser
↓
JSON preparato con JSON.stringify()

UD29
DTO Java
↓
JSON prodotto o letto da una API
```

Questa estensione non realizza ancora la comunicazione HTTP, ma prepara il modo corretto di pensare ai dati che viaggeranno tra browser e backend.


## Nota sul confronto con JavaScript moderno

Questa estensione usa `JSON.stringify()`, che è già JavaScript nativo. Non dipende da jQuery.

Questo è un punto importante: jQuery può semplificare la lettura dei campi e l'aggiornamento del DOM, ma la trasformazione di un oggetto JavaScript in JSON viene eseguita direttamente dal linguaggio tramite `JSON.stringify()`.
