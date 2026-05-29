# 04 - LAB28 autonomo: portale academy front-end

## Obiettivo

Realizzare una pagina front-end per gestire localmente attività formative di un portale academy.

Il laboratorio richiede di applicare quanto visto nel guidato, aggiungendo:

- più campi nel form;
- una tabella più completa;
- statistiche riepilogative;
- validazione più articolata;
- codice JavaScript organizzato in funzioni.

Non è richiesto alcun backend. I dati devono rimanere in memoria nel browser.

## Scenario

La pagina deve consentire di inserire attività formative con queste informazioni:

- titolo;
- area;
- docente;
- durata in ore;
- modalità: aula, online, ibrida;
- stato: pianificata, attiva, conclusa.

Ogni attività valida deve essere aggiunta a una tabella. La pagina deve mostrare anche tre statistiche:

- numero totale di attività;
- ore complessive;
- numero di attività online.

## Struttura richiesta

Creare un progetto con questa struttura:

```text
UD28_portale_academy_frontend/
  index.html
  css/
    style.css
  js/
    app.js
  docs/
    evidence_UD28_autonomo.md
```

## Requisiti tecnici

Usare:

- HTML semantico;
- Bootstrap tramite CDN;
- un file CSS personalizzato;
- JavaScript con jQuery;
- almeno un breve confronto nell'evidence tra una istruzione jQuery usata e l'equivalente JavaScript nativo;
- funzioni separate per lettura dati, validazione, aggiornamento tabella, statistiche e messaggi.

Non usare backend, database, Maven, Node.js o framework front-end.



## Nota su jQuery e JavaScript nativo

Nel laboratorio autonomo è possibile usare jQuery, come nel laboratorio guidato. Non è richiesto riscrivere tutto in JavaScript nativo.

È però richiesto di riconoscere almeno un equivalente moderno. Nell'evidence dovrà essere riportato un piccolo confronto, ad esempio:

```javascript
// jQuery
const titolo = $("#titolo").val().trim();

// JavaScript nativo equivalente
const titolo = document.querySelector("#titolo").value.trim();
```

Questa richiesta serve a chiarire che jQuery è usato come supporto didattico, mentre molte operazioni possono essere svolte oggi anche con API native del browser.

## Parte autonoma richiesta

A differenza del laboratorio guidato, qui non viene fornito tutto il codice sorgente.

È necessario progettare e completare autonomamente:

1. la struttura HTML del form;
2. la tabella delle attività;
3. le card o aree statistiche;
4. il codice JavaScript di lettura dei dati;
5. la validazione;
6. l'aggiornamento dinamico della tabella;
7. il calcolo delle statistiche;
8. l'evidence con i test eseguiti.

## Campi minimi del form

Il form deve contenere almeno:

| Campo | Tipo consigliato | Note |
|---|---|---|
| Titolo | `input type="text"` | obbligatorio, almeno 3 caratteri |
| Area | `select` | obbligatoria |
| Docente | `input type="text"` | obbligatorio, almeno 3 caratteri |
| Durata ore | `input type="number"` | obbligatoria, maggiore di zero |
| Modalità | `select` | aula, online, ibrida |
| Stato | `select` | pianificata, attiva, conclusa |

## Regole di validazione

La pagina deve rifiutare l'inserimento se:

- il titolo ha meno di 3 caratteri;
- l'area non è selezionata;
- il docente ha meno di 3 caratteri;
- la durata non è un numero maggiore di zero;
- la modalità non è selezionata.

Lo stato può avere un valore predefinito, ad esempio `pianificata`.

## Oggetto JavaScript atteso

Quando il form contiene dati validi, il codice JavaScript deve costruire un oggetto simile a questo:

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

L'oggetto deve essere aggiunto a un array:

```javascript
attivitaFormative.push(attivita);
```

Il nome dell'array può essere diverso, ma deve essere chiaro.

## Funzioni consigliate

Conviene organizzare il codice JavaScript con funzioni simili a queste:

```javascript
leggiDatiForm()
validaAttivita(attivita)
aggiungiAttivita(attivita)
aggiornaTabella()
aggiornaStatistiche()
mostraMessaggio(testo, tipo)
```

Se si usa un nome diverso, la responsabilità deve rimanere chiara.

Esempio di responsabilità attese:

| Funzione | Responsabilità |
|---|---|
| `leggiDatiForm()` | legge i valori dei campi e crea un oggetto |
| `validaAttivita(attivita)` | restituisce errore o `null` |
| `aggiungiAttivita(attivita)` | aggiunge l'oggetto all'array |
| `aggiornaTabella()` | ricostruisce il corpo della tabella |
| `aggiornaStatistiche()` | aggiorna totali e ore |
| `mostraMessaggio(testo, tipo)` | mostra feedback all'utente |

## Comportamento atteso

Quando il form contiene dati validi:

1. viene creato un oggetto JavaScript che rappresenta l'attività;
2. l'oggetto viene aggiunto a un array;
3. la tabella viene aggiornata;
4. le statistiche vengono ricalcolate;
5. viene mostrato un messaggio di conferma;
6. il form viene pulito.

Quando i dati non sono validi:

1. viene mostrato un messaggio di errore;
2. non viene aggiunta alcuna riga alla tabella;
3. le statistiche non devono cambiare.

## Suggerimenti operativi

### Lettura dei dati

Per leggere i dati dal form si può seguire lo stesso schema del laboratorio guidato:

```javascript
const titolo = $("#titolo").val().trim();
const durataOre = Number($("#durataOre").val());
```

Equivalente JavaScript nativo:

```javascript
const titolo = document.querySelector("#titolo").value.trim();
const durataOre = Number(document.querySelector("#durataOre").value);
```

### Validazione

La funzione di validazione può restituire una stringa di errore oppure `null`.

Esempio:

```javascript
if (attivita.titolo.length < 3) {
    return "Il titolo deve contenere almeno 3 caratteri.";
}

return null;
```

### Statistiche

Le statistiche possono essere calcolate partendo dall'array.

Esempio:

```javascript
const totaleAttivita = attivitaFormative.length;
```

Per calcolare le ore totali:

```javascript
let oreTotali = 0;

for (const attivita of attivitaFormative) {
    oreTotali += attivita.durataOre;
}
```

Per contare le attività online:

```javascript
let attivitaOnline = 0;

for (const attivita of attivitaFormative) {
    if (attivita.modalita === "online") {
        attivitaOnline++;
    }
}
```

## Collegamento con API REST

Al termine del laboratorio, indicare nell'evidence quale JSON potrebbe essere inviato a una futura API.

Esempio:

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

Nella UD29 questo tipo di struttura verrà collegato a DTO, JSON e richieste HTTP.

## Test minimi richiesti

Eseguire almeno questi test:

| Test | Risultato atteso |
|---|---|
| Form vuoto | messaggio di errore |
| Titolo troppo corto | messaggio di errore |
| Docente troppo corto | messaggio di errore |
| Durata pari a zero | messaggio di errore |
| Attività valida online | riga aggiunta e contatore online aggiornato |
| Attività valida in aula | riga aggiunta e ore totali aggiornate |
| Più inserimenti consecutivi | tabella e statistiche coerenti |

## Evidence richiesta

Nel file `docs/evidence_UD28_autonomo.md` inserire:

```markdown
# Evidence UD28 autonomo

## Struttura dei file

Indicare la struttura realizzata.

## Funzionalità implementate

Descrivere form, tabella, statistiche e messaggi.

## Regole di validazione

Elencare le regole implementate.

## Test eseguiti

Riportare almeno cinque test, includendo casi validi e non validi.

## Funzioni JavaScript principali

Elencare le funzioni realizzate e indicare brevemente la responsabilità di ciascuna.

## Collegamento con API REST

Indicare quale oggetto JSON potrebbe essere inviato a una futura API per creare una nuova attività formativa.

## Confronto jQuery / JavaScript nativo

Riportare un esempio del codice jQuery usato e indicare l'equivalente in JavaScript nativo. Non serve riscrivere tutta l'applicazione: basta un esempio significativo, come lettura di un campo, gestione del submit o aggiornamento di un messaggio.
```

## Criterio di completamento

Il laboratorio è completo quando:

- la pagina si apre correttamente nel browser;
- il form consente l'inserimento di attività valide;
- i dati non validi vengono rifiutati;
- la tabella viene aggiornata;
- le statistiche sono coerenti con gli inserimenti;
- il codice JavaScript è organizzato in funzioni;
- l'evidence documenta struttura, test e collegamento verso API REST.
