# 03 - LAB28 guidato: dashboard corsi front-end

## Obiettivo

In questo laboratorio costruiamo una piccola dashboard front-end per inserire corsi e visualizzarli in una tabella.

Il laboratorio non usa backend e non salva dati su database. I dati restano nella memoria del browser, dentro un array JavaScript.

Il risultato serve a comprendere il flusso che ritroveremo nelle API REST:

```text
form HTML
↓
dati letti con JavaScript
↓
validazione
↓
oggetto JavaScript
↓
aggiornamento tabella
↓
possibile futuro JSON verso API REST
```

## Requisiti

- browser moderno;
- VS Code o editor equivalente;
- connessione Internet per Bootstrap e jQuery da CDN.

Non servono Maven, Docker, Node.js o backend Java.

## Risultato atteso

Alla fine del laboratorio avremo una pagina con:

- intestazione;
- form per inserire corsi;
- messaggio di errore o conferma;
- tabella aggiornata dinamicamente;
- script JavaScript separato dal file HTML;
- CSS minimo separato.

## Passo 1 - Creare la struttura del progetto

Creare una cartella chiamata `LAB28_dashboard_corsi_frontend` con questa struttura:

```text
LAB28_dashboard_corsi_frontend/
  index.html
  css/
    style.css
  js/
    app.js
  docs/
    evidence_LAB28_guidato.md
```

### Windows PowerShell

```powershell
mkdir LAB28_dashboard_corsi_frontend
cd LAB28_dashboard_corsi_frontend
mkdir css, js, docs
New-Item index.html
New-Item css/style.css
New-Item js/app.js
New-Item docs/evidence_LAB28_guidato.md
```

### Linux/macOS

```bash
mkdir -p LAB28_dashboard_corsi_frontend/{css,js,docs}
cd LAB28_dashboard_corsi_frontend
touch index.html css/style.css js/app.js docs/evidence_LAB28_guidato.md
```

## Passo 2 - Creare la pagina HTML

In questo passaggio costruiamo la struttura della pagina. La pagina contiene:

- intestazione;
- form di inserimento;
- area messaggi;
- tabella dei corsi inseriti;
- collegamenti a Bootstrap, CSS locale, jQuery e JavaScript locale.

Inserire in `index.html`:

```html
<!DOCTYPE html>
<html lang="it">
<head>
    <!--
        Imposta la codifica dei caratteri.
        UTF-8 permette di gestire correttamente lettere accentate e simboli comuni.
    -->
    <meta charset="UTF-8">

    <!--
        Rende la pagina più adatta a schermi di dimensioni diverse.
        È particolarmente utile quando usiamo Bootstrap.
    -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Dashboard Corsi</title>

    <!--
        Bootstrap viene caricato da CDN.
        Fornisce classi pronte per layout, form, pulsanti, card, alert e tabelle.
    -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css" rel="stylesheet">

    <!--
        File CSS locale del laboratorio.
        Lo useremo solo per poche personalizzazioni.
    -->
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <!--
        header rappresenta l'intestazione della pagina.
        Le classi Bootstrap impostano sfondo scuro, testo bianco e spaziatura verticale.
    -->
    <header class="bg-dark text-white py-4">
        <div class="container">
            <h1 class="mb-1">Dashboard Corsi</h1>
            <p class="mb-0">Esempio front-end per la gestione locale dei corsi</p>
        </div>
    </header>

    <!--
        main contiene il contenuto principale della pagina.
        container limita la larghezza e my-4 aggiunge margine verticale.
    -->
    <main class="container my-4">
        <!--
            row e col-lg-* fanno parte della griglia Bootstrap.
            Su schermi grandi il form occuperà 4 colonne e la tabella 8 colonne.
        -->
        <section class="row g-4">
            <div class="col-lg-4">
                <div class="card shadow-sm">
                    <div class="card-body">
                        <h2 class="h5">Nuovo corso</h2>

                        <!--
                            Il form raccoglie i dati del corso.
                            id="formCorso" permette a JavaScript di intercettare il submit.
                            novalidate disattiva la validazione automatica del browser.
                        -->
                        <form id="formCorso" novalidate>
                            <div class="mb-3">
                                <label for="titolo" class="form-label">Titolo</label>

                                <!--
                                    id="titolo" sarà usato da JavaScript con il selettore #titolo.
                                -->
                                <input type="text" id="titolo" class="form-control">
                            </div>

                            <div class="mb-3">
                                <label for="area" class="form-label">Area</label>

                                <!--
                                    Il primo valore è vuoto. Serve per riconoscere il caso in cui l'utente
                                    non ha ancora scelto un'area valida.
                                -->
                                <select id="area" class="form-select">
                                    <option value="">Seleziona area</option>
                                    <option value="Java">Java</option>
                                    <option value="Database">Database</option>
                                    <option value="Web">Web</option>
                                </select>
                            </div>

                            <div class="mb-3">
                                <label for="durata" class="form-label">Durata ore</label>

                                <!--
                                    type="number" aiuta il browser a mostrare un campo numerico,
                                    ma il valore sarà comunque letto da JavaScript come stringa.
                                -->
                                <input type="number" id="durata" class="form-control" min="1">
                            </div>

                            <button type="submit" class="btn btn-primary w-100">Aggiungi corso</button>
                        </form>
                    </div>
                </div>
            </div>

            <div class="col-lg-8">
                <!--
                    Area usata da JavaScript per mostrare messaggi.
                    d-none la tiene nascosta all'avvio.
                    role="alert" aiuta anche strumenti di accessibilità.
                -->
                <div id="messaggio" class="alert d-none" role="alert"></div>

                <div class="card shadow-sm">
                    <div class="card-body">
                        <h2 class="h5">Corsi inseriti</h2>
                        <div class="table-responsive">
                            <!--
                                La tabella ha un id perché JavaScript dovrà individuare il tbody
                                e aggiungere righe in modo dinamico.
                            -->
                            <table id="tabellaCorsi" class="table table-striped align-middle">
                                <thead>
                                    <tr>
                                        <th>Titolo</th>
                                        <th>Area</th>
                                        <th>Ore</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <!-- Le righe verranno aggiunte da JavaScript. -->
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </main>

    <!--
        jQuery deve essere caricato prima di app.js.
        app.js userà il simbolo $, che viene definito da jQuery.
    -->
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>

    <!-- Bootstrap JavaScript. In questo laboratorio non è centrale, ma lo includiamo per completezza. -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/js/bootstrap.bundle.min.js"></script>

    <!-- Script locale del laboratorio. -->
    <script src="js/app.js"></script>
</body>
</html>
```

### Spiegazione del file HTML

Il file HTML definisce la struttura della pagina, non la logica applicativa.

I punti più importanti sono:

| Parte | Perché è importante |
|---|---|
| `id="formCorso"` | permette di intercettare il submit del form |
| `id="titolo"`, `id="area"`, `id="durata"` | permettono di leggere i valori dei campi |
| `id="messaggio"` | permette di mostrare messaggi di errore o conferma |
| `id="tabellaCorsi"` | permette di aggiornare la tabella con JavaScript |
| `jQuery` prima di `app.js` | evita l'errore `$ is not defined` |

## Passo 3 - Aggiungere un CSS minimo

In `css/style.css` inserire:

```css
/*
    Imposta un colore di sfondo chiaro per distinguere meglio le card bianche.
*/
body {
    background-color: #f8f9fa;
}

/*
    Rimuove il bordo predefinito delle card Bootstrap.
    La card resta visibile grazie all'ombra shadow-sm usata nell'HTML.
*/
.card {
    border: 0;
}

/*
    Allinea verticalmente il contenuto delle celle della tabella.
*/
#tabellaCorsi th,
#tabellaCorsi td {
    vertical-align: middle;
}
```

Il CSS è volutamente ridotto. Bootstrap gestisce già gran parte del layout.

## Passo 4 - Gestire il form con JavaScript e jQuery


> **Nota didattica su jQuery e JavaScript nativo**  
> Il codice principale del laboratorio usa jQuery per mantenere compatte le operazioni sul DOM. Dopo il blocco principale sono presenti alcuni box con equivalenti JavaScript nativi. Non è necessario riscrivere il laboratorio due volte: il confronto serve a riconoscere la stessa operazione anche senza jQuery.

In `js/app.js` inserire:

```javascript
/*
    Array usato come memoria temporanea lato browser.
    Ogni corso valido verrà aggiunto a questo array.
    I dati vengono persi quando la pagina viene aggiornata o chiusa.
*/
const corsi = [];

/*
    Questa funzione viene eseguita quando il DOM è pronto.
    È una forma breve di jQuery per dire:
    "esegui questo codice quando la pagina è stata caricata".
*/
$(function() {
    /*
        Seleziona il form con id formCorso e collega una funzione
        all'evento submit.
    */
    $("#formCorso").on("submit", function(event) {
        /*
            Evita il comportamento predefinito del form.
            Senza questa istruzione il browser ricaricherebbe la pagina
            e perderemmo i dati inseriti nell'array.
        */
        event.preventDefault();

        /*
            Legge i valori dai campi e costruisce un oggetto JavaScript.
        */
        const corso = leggiDatiForm();

        /*
            Valida l'oggetto corso.
            Se i dati sono corretti, la funzione restituisce null.
            Se c'è un problema, restituisce un messaggio di errore.
        */
        const errore = validaCorso(corso);

        if (errore !== null) {
            mostraMessaggio(errore, "danger");
            return;
        }

        /*
            Aggiunge il corso valido all'array.
        */
        corsi.push(corso);

        /*
            Ricostruisce la tabella leggendo il contenuto aggiornato dell'array.
        */
        aggiornaTabella();

        mostraMessaggio("Corso aggiunto correttamente.", "success");

        /*
            Dentro questa funzione, this indica il form che ha generato l'evento.
            reset() svuota i campi del form dopo un inserimento valido.
        */
        this.reset();
    });
});

/*
    Legge i valori del form e restituisce un oggetto JavaScript.
    Questo oggetto rappresenta il corso inserito dall'utente.
*/
function leggiDatiForm() {
    return {
        titolo: $("#titolo").val().trim(),
        area: $("#area").val(),
        durataOre: Number($("#durata").val())
    };
}

/*
    Controlla se il corso rispetta le regole minime.
    Restituisce:
    - una stringa con il messaggio di errore, se il dato non è valido;
    - null, se il dato è valido.
*/
function validaCorso(corso) {
    if (corso.titolo.length < 3) {
        return "Il titolo deve contenere almeno 3 caratteri.";
    }

    if (corso.area === "") {
        return "Selezionare un'area.";
    }

    if (!Number.isFinite(corso.durataOre) || corso.durataOre <= 0) {
        return "La durata deve essere un numero maggiore di zero.";
    }

    return null;
}

/*
    Aggiorna il corpo della tabella.
    Prima svuota il tbody, poi aggiunge una riga per ogni corso presente nell'array.
*/
function aggiornaTabella() {
    const corpoTabella = $("#tabellaCorsi tbody");

    /*
        empty() rimuove tutte le righe presenti nel tbody.
        La tabella viene poi ricostruita a partire dall'array corsi.
    */
    corpoTabella.empty();

    for (const corso of corsi) {
        /*
            Template literal: stringa delimitata da backtick.
            Permette di inserire valori con ${...}.
        */
        const riga = `
            <tr>
                <td>${corso.titolo}</td>
                <td>${corso.area}</td>
                <td>${corso.durataOre}</td>
            </tr>
        `;

        corpoTabella.append(riga);
    }
}

/*
    Mostra un messaggio nella pagina.
    tipo può valere, per esempio:
    - "success" per messaggi positivi;
    - "danger" per messaggi di errore.

    Questi nomi corrispondono alle classi Bootstrap alert-success e alert-danger.
*/
function mostraMessaggio(testo, tipo) {
    $("#messaggio")
        .removeClass("d-none alert-success alert-danger")
        .addClass("alert-" + tipo)
        .text(testo);
}
```

### Spiegazione del codice JavaScript

Il codice è diviso in funzioni con responsabilità diverse.

| Funzione / variabile | Responsabilità |
|---|---|
| `const corsi = []` | mantiene i corsi inseriti durante l'uso della pagina |
| `$(function() { ... })` | esegue il codice quando il DOM è pronto |
| `event.preventDefault()` | impedisce al form di ricaricare la pagina |
| `leggiDatiForm()` | legge i campi e costruisce un oggetto JavaScript |
| `validaCorso(corso)` | controlla le regole minime |
| `aggiornaTabella()` | ricostruisce la tabella |
| `mostraMessaggio(testo, tipo)` | mostra feedback all'utente |



### Box di confronto - jQuery e JavaScript nativo

Nel codice del laboratorio abbiamo usato jQuery. Le operazioni principali possono essere lette anche con JavaScript nativo moderno.

#### DOM pronto

Con jQuery:

```javascript
$(function() {
    // codice eseguito quando il DOM è pronto
});
```

Con JavaScript nativo:

```javascript
document.addEventListener("DOMContentLoaded", function() {
    // codice eseguito quando il DOM è pronto
});
```

#### Gestione del submit

Con jQuery:

```javascript
$("#formCorso").on("submit", function(event) {
    event.preventDefault();
});
```

Con JavaScript nativo:

```javascript
document.querySelector("#formCorso").addEventListener("submit", function(event) {
    event.preventDefault();
});
```

#### Lettura dei campi

Con jQuery:

```javascript
const titolo = $("#titolo").val().trim();
```

Con JavaScript nativo:

```javascript
const titolo = document.querySelector("#titolo").value.trim();
```

#### Aggiornamento della tabella

Con jQuery:

```javascript
corpoTabella.append(riga);
```

Con JavaScript nativo:

```javascript
document
    .querySelector("#tabellaCorsi tbody")
    .insertAdjacentHTML("beforeend", riga);
```

Il laboratorio continua a usare jQuery, ma il confronto mostra che il concetto centrale non è la libreria: il concetto centrale è leggere dati dal form, reagire a un evento e aggiornare il DOM.

### Perché ricostruiamo la tabella ogni volta

Nel laboratorio scegliamo un approccio semplice:

```text
array corsi
↓
svuota tabella
↓
ricrea una riga per ogni corso
```

Questo rende il codice più facile da capire. Per pochi dati è adeguato.

In applicazioni più grandi si possono usare strategie più efficienti, ma qui il focus è il flusso logico.

### Collegamento con API REST

La funzione `leggiDatiForm()` crea un oggetto simile a quello che in futuro potremmo inviare a una API:

```javascript
{
    titolo: "Java Web",
    area: "Java",
    durataOre: 24
}
```

Nella UD29 questo oggetto potrà diventare JSON nel corpo di una richiesta HTTP.

## Passo 5 - Eseguire la pagina

È possibile aprire direttamente `index.html` dal file manager oppure usare il terminale.

### Windows PowerShell

```powershell
Start-Process .\index.html
```

### macOS

```bash
open index.html
```

### Linux

```bash
xdg-open index.html
```

Se Python è disponibile, è possibile avviare un piccolo server locale.

### Windows

```powershell
py -m http.server 5500
```

### Linux/macOS

```bash
python3 -m http.server 5500
```

Aprire poi il browser su:

```text
http://localhost:5500
```

## Passo 6 - Verificare il funzionamento

Eseguire questi test:

| Test | Risultato atteso |
|---|---|
| Invio form vuoto | Messaggio di errore |
| Titolo con meno di 3 caratteri | Messaggio di errore |
| Area non selezionata | Messaggio di errore |
| Durata pari a 0 | Messaggio di errore |
| Corso valido | Riga aggiunta alla tabella |
| Più corsi validi | La tabella mostra tutte le righe inserite |

## Passo 7 - Aprire la console del browser

La console del browser è utile per diagnosticare errori JavaScript.

Aprire gli strumenti sviluppatore:

| Browser | Scorciatoia comune |
|---|---|
| Chrome / Edge / Firefox su Windows | `F12` oppure `Ctrl + Shift + I` |
| Chrome / Edge / Firefox su macOS | `Cmd + Option + I` |

Controllare che non compaiano errori come:

| Errore | Causa probabile |
|---|---|
| `$ is not defined` | jQuery non caricato o caricato dopo `app.js` |
| `Cannot read properties of undefined` | selettore errato o elemento non trovato |
| Nessuna reazione al submit | id del form errato o script non caricato |

## Evidence

Nel file `docs/evidence_LAB28_guidato.md` riportare:

```markdown
# Evidence LAB28 guidato

## Struttura realizzata

Descrivere i file creati e il ruolo di `index.html`, `css/style.css` e `js/app.js`.

## Test eseguiti

Riportare almeno tre test, includendo un caso valido e un caso non valido.

## Spiegazione del codice JavaScript

Rispondere brevemente:

1. A cosa serve `event.preventDefault()`?
2. Perché i dati vengono salvati nell'array `corsi`?
3. Quale funzione legge i dati del form?
4. Quale funzione aggiorna la tabella?
5. Perché `Number(...)` viene usato sul campo durata?

## Confronto jQuery / JavaScript nativo

Riportare almeno un esempio di codice jQuery usato nel laboratorio e scrivere sotto l'equivalente in JavaScript nativo.

Esempio:

```javascript
// jQuery
const titolo = $("#titolo").val().trim();

// JavaScript nativo
const titolo = document.querySelector("#titolo").value.trim();
```

## Collegamento con il backend

Indicare quali dati del form potrebbero essere inviati a una futura API REST.

Esempio:

```json
{
  "titolo": "Java Web",
  "area": "Java",
  "durataOre": 24
}
```
```
