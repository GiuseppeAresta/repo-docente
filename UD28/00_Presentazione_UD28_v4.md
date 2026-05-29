# UD28 - Web base essenziale per applicazioni Java

## Collocazione nel percorso

Nelle UD precedenti abbiamo lavorato sul lato applicativo Java: classi, oggetti, JDBC, Maven, DAO e Service. Abbiamo visto come il codice può leggere dati, organizzarli in oggetti e separare responsabilità tra livelli diversi.

In questa UD spostiamo l'attenzione sul lato browser. Una web application non è composta solo da codice Java: l'utente interagisce con pagine, form, pulsanti, messaggi, tabelle e dati visualizzati a video. Prima di introdurre API REST e Spring Boot è utile capire come nasce una semplice interfaccia web e come i dati inseriti dall'utente possono essere raccolti e preparati.

L'obiettivo non è diventare sviluppatori front-end completi. L'obiettivo è comprendere il minimo indispensabile per leggere e costruire una pagina web semplice che, nelle UD successive, potrà comunicare con un backend Java tramite HTTP e JSON.

## Scenario della UD

Costruiremo una piccola dashboard front-end per un contesto academy. La pagina permetterà di inserire corsi o attività formative tramite un form, validarli lato browser e visualizzarli in una tabella.

In questa UD i dati restano nella memoria del browser. Non vengono ancora inviati a un server e non vengono salvati in un database.

Il flusso sarà questo:

```text
utente compila il form
↓
JavaScript intercetta l'invio
↓
JavaScript legge i valori dei campi
↓
JavaScript valida i dati
↓
viene creato un oggetto JavaScript
↓
l'oggetto viene aggiunto a un array
↓
la tabella HTML viene aggiornata
```

Nella UD29 lo stesso ragionamento verrà esteso a HTTP, JSON, DTO e API REST.

## Concetti introdotti

| Concetto | Significato operativo |
|---|---|
| HTML | Definisce la struttura della pagina |
| CSS | Personalizza l'aspetto grafico |
| Bootstrap | Fornisce classi pronte per layout, form, tabelle e pulsanti |
| JavaScript | Aggiunge comportamento alla pagina |
| jQuery | Semplifica selezione elementi, eventi e aggiornamento del DOM |
| DOM | Rappresentazione della pagina HTML modificabile da JavaScript |
| Form | Raccoglie dati inseriti dall'utente |
| Validazione client-side | Controlla i dati prima di usarli nella pagina |
| Oggetto JavaScript | Rappresenta in memoria i dati raccolti |
| Array JavaScript | Contiene più oggetti inseriti durante l'uso della pagina |

## Cosa facciamo in questa UD

Realizzeremo una pagina che consente di:

- visualizzare un layout ordinato;
- inserire dati tramite un form;
- validare campi obbligatori;
- mostrare messaggi di errore o conferma;
- aggiungere righe a una tabella;
- mantenere temporaneamente i dati in un array JavaScript;
- collegare il lavoro svolto alla futura comunicazione con API REST.

## jQuery e JavaScript moderno

jQuery viene usato in modo essenziale per rendere più compatte alcune operazioni sul DOM:

- selezionare elementi HTML;
- intercettare eventi;
- leggere valori dai campi;
- aggiungere righe alla tabella;
- mostrare messaggi nella pagina.

Non studiamo jQuery come tecnologia principale per nuovi progetti front-end. Lo usiamo come strumento didattico di transizione perché permette di concentrarci sul flusso applicativo:

```text
form
↓
lettura dei dati
↓
validazione
↓
aggiornamento del DOM
↓
preparazione di un oggetto JavaScript e di un payload JSON
```

Per evitare equivoci, in questa versione della UD vengono introdotti piccoli box di confronto con JavaScript nativo moderno. Lo scopo non è riscrivere due volte il laboratorio, ma riconoscere che molte operazioni oggi possono essere svolte anche senza jQuery, usando API native del browser come:

- `document.querySelector(...)`;
- `addEventListener(...)`;
- `textContent`;
- `insertAdjacentHTML(...)`;
- `classList`.

Nel laboratorio il codice principale rimane basato su jQuery. I box servono a leggere lo stesso comportamento anche con la sintassi moderna del browser.

## Risultati attesi

Al termine della UD saremo in grado di:

1. distinguere il ruolo di HTML, CSS, Bootstrap, JavaScript e jQuery;
2. costruire una pagina con form e tabella;
3. collegare file CSS e JavaScript esterni;
4. capire perché l'ordine degli script è importante;
5. intercettare l'invio di un form;
6. usare `event.preventDefault()` per evitare il comportamento standard del browser;
7. leggere i valori inseriti dall'utente;
8. validare dati obbligatori;
9. creare un oggetto JavaScript a partire dal form;
10. aggiornare dinamicamente il contenuto della pagina;
11. spiegare quali dati potranno diventare il corpo JSON di una futura richiesta HTTP.



