# 01 - Web base, form e dati nel browser

## Il browser come ambiente di esecuzione

Quando apriamo una pagina web, il browser interpreta HTML, CSS e JavaScript.

Questi tre elementi hanno responsabilità diverse:

| Tecnologia | Responsabilità principale |
|---|---|
| HTML | Definisce la struttura della pagina |
| CSS | Definisce l'aspetto grafico |
| JavaScript | Definisce il comportamento dinamico |

Nel percorso Java abbiamo lavorato soprattutto sul backend. Qui osserviamo il punto di ingresso dell'utente: la pagina web.

Una pagina web può essere vista come un'interfaccia che raccoglie dati e li mostra. In una web application completa quei dati possono poi essere inviati a un backend, ma in questa UD resteranno nel browser.

## HTML: struttura della pagina

HTML descrive il contenuto visualizzato dal browser. Non contiene logica applicativa Java e non accede al database.

Un documento HTML minimo contiene:

```html
<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <title>Portale Academy</title>
</head>
<body>
    <h1>Portale Academy</h1>
    <p>Elenco delle attività formative.</p>
</body>
</html>
```

Le parti principali sono:

| Elemento | Significato |
|---|---|
| `<!DOCTYPE html>` | Indica al browser che il documento usa HTML moderno |
| `<html>` | Elemento radice della pagina |
| `<head>` | Contiene metadati, titolo e collegamenti a CSS/script |
| `<body>` | Contiene ciò che viene visualizzato nella pagina |
| `<meta charset="UTF-8">` | Permette di gestire correttamente caratteri accentati |
| `<title>` | Titolo mostrato nella scheda del browser |

## Elementi semantici utili

Nel laboratorio useremo una struttura semplice ma ordinata.

| Elemento | Uso |
|---|---|
| `header` | Intestazione della pagina |
| `main` | Contenuto principale |
| `section` | Blocco tematico della pagina |
| `form` | Raccolta dei dati inseriti dall'utente |
| `input` | Campo di input testuale o numerico |
| `select` | Campo con valori selezionabili |
| `button` | Pulsante di azione |
| `table` | Visualizzazione di dati tabellari |
| `div` | Contenitore generico usato per layout e messaggi |

Usare elementi coerenti aiuta a leggere meglio il file HTML e prepara il lavoro successivo con JavaScript.

## Form: ingresso dei dati

Un form permette di raccogliere valori inseriti dall'utente.

Esempio:

```html
<form id="formCorso" novalidate>
    <label for="titolo">Titolo corso</label>
    <input type="text" id="titolo">

    <label for="durata">Durata ore</label>
    <input type="number" id="durata">

    <button type="submit">Aggiungi</button>
</form>
```

### Perché usiamo `id`

L'attributo `id` identifica un elemento in modo univoco nella pagina.

Nel laboratorio sarà fondamentale perché JavaScript userà questi identificativi per leggere i campi:

```javascript
const titolo = $("#titolo").val();
```

Il selettore `#titolo` significa: cerca l'elemento HTML con `id="titolo"`.

### Perché usiamo `for` nelle label

La label:

```html
<label for="titolo">Titolo corso</label>
```

è collegata al campo:

```html
<input type="text" id="titolo">
```

Il valore di `for` deve corrispondere all'`id` del campo. Questo migliora leggibilità e accessibilità della pagina.

### Perché usiamo `novalidate`

L'attributo:

```html
<form id="formCorso" novalidate>
```

disattiva la validazione automatica del browser.

In questo laboratorio vogliamo gestire noi la validazione con JavaScript, così possiamo capire meglio il flusso:

```text
invio del form
↓
JavaScript intercetta l'evento
↓
JavaScript legge i valori
↓
JavaScript controlla le regole
↓
la pagina viene aggiornata
```

## Tabella: visualizzazione dei dati

Una tabella HTML mostra dati organizzati in righe e colonne.

```html
<table id="tabellaCorsi">
    <thead>
        <tr>
            <th>Titolo</th>
            <th>Area</th>
            <th>Ore</th>
        </tr>
    </thead>
    <tbody>
    </tbody>
</table>
```

La tabella contiene due parti importanti:

| Sezione | Significato |
|---|---|
| `thead` | intestazione della tabella |
| `tbody` | corpo della tabella, dove verranno inserite le righe |

Nel laboratorio il `tbody` sarà inizialmente vuoto. JavaScript aggiungerà righe quando il form contiene dati validi.

## Bootstrap: layout senza scrivere troppo CSS

Bootstrap fornisce classi già pronte per layout, form, tabelle, card, pulsanti e messaggi.

Esempio di collegamento tramite CDN:

```html
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css" rel="stylesheet">
```

Il CDN permette di caricare Bootstrap da Internet. Per questo laboratorio è sufficiente e riduce il lavoro di configurazione.

Alcune classi Bootstrap che useremo:

| Classe | Effetto |
|---|---|
| `container` | centra e limita la larghezza del contenuto |
| `row` | crea una riga del sistema a griglia |
| `col-lg-4` | colonna larga 4 unità su schermi grandi |
| `card` | contenitore grafico con stile predefinito |
| `form-control` | stile per input testuali e numerici |
| `form-select` | stile per select |
| `btn btn-primary` | pulsante primario |
| `alert` | messaggio informativo o di errore |
| `table table-striped` | tabella leggibile con righe alternate |

## Dati nel browser

In questa UD i dati inseriti non vengono salvati nel database.

Vengono mantenuti in memoria dentro una variabile JavaScript, per esempio:

```javascript
const corsi = [];
```

Quando viene inserito un corso valido, possiamo creare un oggetto:

```javascript
const corso = {
    titolo: "Java Web",
    area: "Java",
    durataOre: 24
};
```

Poi possiamo aggiungerlo all'array:

```javascript
corsi.push(corso);
```

Il dato esiste solo finché la pagina resta aperta. Se il browser viene aggiornato, l'array torna vuoto.

## Collegamento con il backend

Anche se in questa UD non usiamo backend, possiamo già ragionare sui dati come faremo nelle API REST.

L'oggetto JavaScript:

```javascript
const corso = {
    titolo: "Java Web",
    area: "Java",
    durataOre: 24
};
```

può essere rappresentato in JSON così:

```json
{
  "titolo": "Java Web",
  "area": "Java",
  "durataOre": 24
}
```

Nella UD29 questo tipo di struttura verrà collegato a richieste HTTP, DTO e API REST.

## Sintesi

In questa UD il browser svolge tre compiti:

1. mostra una pagina HTML;
2. raccoglie dati tramite un form;
3. usa JavaScript per validare e visualizzare i dati.

Il backend non è ancora presente, ma il modo in cui organizziamo form, oggetti e tabella prepara il passaggio successivo verso API REST.
