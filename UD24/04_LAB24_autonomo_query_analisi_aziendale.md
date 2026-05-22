# LAB24 - Laboratorio autonomo SQL per analisi dati aziendale

## Scopo del laboratorio autonomo

Questo file contiene gli esercizi da svolgere in autonomia a partire dal database `modelli_classici_it`.

Gli esercizi richiedono di costruire query utili all'analisi aziendale su catalogo, vendite, clienti, pagamenti, ritardi e performance commerciali.

## Regole di svolgimento

- Scrivere le query in autonomia.
- Usare i suggerimenti solo come guida, non come soluzione completa.
- Salvare le query e un breve commento sul risultato ottenuto.
- Controllare sempre che le join non moltiplichino righe o importi.

## Durata consigliata

Il laboratorio autonomo può essere distribuito nelle 4 sessioni oppure usato come verifica pratica conclusiva.

## File di evidenza richiesto

Creare un file:

```text
docs/evidence_LAB24_autonomo.md
```

Il file deve contenere:

1. query svolte degli esercizi autonomi;
2. breve commento sul risultato ottenuto;
3. eventuali dubbi o anomalie osservate;
4. risposta sintetica alle domande di verifica.

---
# Sessione 1 - Analisi catalogo prodotti e magazzino

Durata: 2 ore  
Livello: base  
Focus tecnico: `SELECT`, `WHERE`, `ORDER BY`, calcoli, `CASE`, primo uso di `JOIN` con tabella di classificazione.

## Scenario aziendale



La direzione commerciale vuole una prima lettura del catalogo prodotti: linee prodotto disponibili, prezzi di listino, margini teorici e prodotti con scorte basse.

## Obiettivi della sessione



Al termine della sessione il partecipante deve saper:

- leggere dati da una tabella;
- selezionare solo le colonne utili;
- filtrare e ordinare i risultati;
- creare colonne calcolate;
- classificare i dati con `CASE`;
- usare una join semplice tra `prodotti` e `linee_prodotto`.

---

## Esercizio autonomo 1.A - Linee prodotto più ricche



### Richiesta aziendale

Per ogni linea prodotto, calcolare:

- numero di prodotti;
- prezzo medio di listino;
- prezzo minimo;
- prezzo massimo.

Ordinare dalla linea con più prodotti a quella con meno prodotti.

### Suggerimenti

- Usare la tabella `prodotti`.
- Usare `COUNT`, `AVG`, `MIN`, `MAX`.
- Raggruppare per `linea_prodotto`.
- Arrotondare il prezzo medio con `ROUND`.

### Evidenza richiesta

Salvare la query e indicare quale linea contiene più prodotti.

---

## Esercizio autonomo 1.B - Catalogo ad alto margine



### Richiesta aziendale

Estrarre i prodotti con percentuale di margine superiore al 90%.

Il report deve mostrare:

- codice prodotto;
- nome prodotto;
- linea prodotto;
- prezzo acquisto;
- prezzo listino;
- percentuale di margine.

### Suggerimenti

- La formula è la stessa del laboratorio guidato 1.3.
- Il filtro deve essere applicato sul calcolo della percentuale.
- Ordinare dai margini più alti ai più bassi.

### Evidenza richiesta

Salvare la query e riportare i primi 5 prodotti del risultato.

---

## Domande di verifica della sessione 1



1. Perché è utile collegare `prodotti` e `linee_prodotto`?
2. Che differenza c'è tra margine unitario e percentuale di margine?
3. Perché un prodotto con scorta bassa non è automaticamente un problema commerciale?
4. Quando è utile usare `CASE` in una query di analisi?

---

---

# Sessione 2 - Analisi ordini, vendite e aggregazioni

Durata: 2 ore  
Livello: base/intermedio  
Focus tecnico: `JOIN`, `GROUP BY`, `HAVING`, aggregazioni su dati di vendita.

## Scenario aziendale



La direzione vendite vuole capire quali clienti, prodotti e linee prodotto generano più valore. Le informazioni sono distribuite tra ordini, dettagli ordine, prodotti e clienti.

## Obiettivi della sessione



Al termine della sessione il partecipante deve saper:

- collegare ordini, clienti e dettagli ordine;
- calcolare il valore di una riga ordine;
- calcolare il totale di un ordine;
- aggregare vendite per cliente, prodotto, linea e paese;
- filtrare gruppi con `HAVING`.

---

## Esercizio autonomo 2.A - Prodotti più venduti per quantità



### Richiesta aziendale

Individuare i 20 prodotti più venduti per quantità totale ordinata.

Il report deve mostrare:

- codice prodotto;
- nome prodotto;
- linea prodotto;
- quantità totale ordinata.

### Suggerimenti

- Collegare `prodotti` e `dettagli_ordini`.
- Usare `SUM(quantita_ordinata)`.
- Raggruppare per prodotto.
- Ordinare in modo decrescente.
- Usare `LIMIT 20`.

### Evidenza richiesta

Salvare la query e indicare il prodotto più venduto.

---

## Esercizio autonomo 2.B - Clienti sopra una soglia di fatturato ordinato



### Richiesta aziendale

Estrarre i clienti con valore totale degli ordini superiore a 100000.

Il report deve mostrare:

- codice cliente;
- ragione sociale;
- paese;
- numero ordini;
- totale ordini.

### Suggerimenti

- La query parte da `clienti`.
- Collegare `ordini` e `dettagli_ordini`.
- Usare `GROUP BY`.
- Applicare il filtro sui gruppi con `HAVING`.

### Evidenza richiesta

Salvare la query e indicare quanti clienti superano la soglia.

---

## Domande di verifica della sessione 2



1. Perché il totale ordine si calcola da `dettagli_ordini` e non direttamente da `ordini`?
2. Quando serve `COUNT(DISTINCT ...)`?
3. Che differenza c'è tra `WHERE` e `HAVING`?
4. Perché la tabella `linee_prodotto` è importante nell'analisi delle vendite per linea?

---

---

# Sessione 3 - Pagamenti, credito e rischio commerciale

Durata: 2 ore  
Livello: intermedio  
Focus tecnico: `LEFT JOIN`, `COALESCE`, CTE, aggregazioni separate per evitare duplicazioni.

## Scenario aziendale



L'ufficio amministrativo vuole confrontare valore ordinato e pagamenti ricevuti. La direzione vuole individuare clienti con esposizione elevata, pagamenti mancanti o rapporto critico rispetto al limite di credito.

## Obiettivi della sessione



Al termine della sessione il partecipante deve saper:

- distinguere ordini e pagamenti;
- aggregare dati provenienti da processi diversi;
- usare `LEFT JOIN` per non perdere clienti senza pagamenti;
- usare `COALESCE` per sostituire valori nulli;
- usare CTE per costruire query più leggibili.

---

## Richiamo operativo: uso delle CTE negli esercizi autonomi

Negli esercizi autonomi della sessione 3 e della sessione 4 può essere utile usare una **CTE**.

Una CTE si scrive con `WITH` e permette di dare un nome a una query intermedia:

```sql
WITH nome_cte AS (
    SELECT ...
)
SELECT ...
FROM nome_cte;
```

Quando servono più passaggi intermedi, le CTE si separano con una virgola:

```sql
WITH prima_cte AS (
    SELECT ...
),
seconda_cte AS (
    SELECT ...
)
SELECT ...;
```

Nel laboratorio autonomo le CTE sono utili soprattutto quando bisogna calcolare prima un totale e poi usarlo in una query finale, ad esempio per confrontare totale ordini e totale pagamenti.

---

## Esercizio autonomo 3.A - Clienti con ordini ma senza pagamenti



### Richiesta aziendale

Individuare i clienti che hanno effettuato almeno un ordine ma non hanno pagamenti registrati.

Il report deve mostrare:

- codice cliente;
- ragione sociale;
- paese;
- numero ordini.

### Suggerimenti

- Usare `clienti`, `ordini` e `pagamenti`.
- Per mantenere i clienti senza pagamenti serve una `LEFT JOIN` verso `pagamenti`.
- Raggruppare per cliente.
- Filtrare i clienti per cui non esiste pagamento.

### Evidenza richiesta

Salvare la query e indicare se il fenomeno esiste nel database.

---

## Esercizio autonomo 3.B - Clienti con esposizione elevata



### Richiesta aziendale

Estrarre i clienti con esposizione superiore a 50000.

L'esposizione è definita come:

```text
totale ordini - totale pagamenti
```

### Suggerimenti

- Usare due CTE separate: una per il totale ordini, una per il totale pagamenti.
- Collegare le CTE alla tabella `clienti`.
- Usare `COALESCE` per gestire clienti senza ordini o senza pagamenti.
- Filtrare sull'esposizione calcolata.

### Evidenza richiesta

Salvare la query e riportare i primi 10 clienti per esposizione.

---

## Domande di verifica della sessione 3



1. Perché non conviene collegare direttamente `ordini`, `dettagli_ordini` e `pagamenti` nella stessa aggregazione senza attenzione?
2. A cosa serve `COALESCE` in questi report?
3. Perché `LEFT JOIN` è utile nelle analisi amministrative?
4. Perché il saldo ordini-pagamenti non equivale automaticamente a un insoluto certo?

---

---

# Sessione 4 - Query avanzate e mini dashboard aziendale

Durata: 2 ore  
Livello: intermedio/avanzato  
Focus tecnico: CTE, sottoquery, funzioni finestra, ranking, analisi per struttura commerciale.

## Scenario aziendale



La direzione vuole un set di report più vicini a una dashboard: migliori clienti, migliori prodotti per linea, andamento commerciale per dipendente e ufficio, anomalie operative.

## Obiettivi della sessione



Al termine della sessione il partecipante deve saper:

- usare CTE multiple;
- creare classifiche con `RANK` e `ROW_NUMBER`;
- usare `PARTITION BY`;
- produrre report riepilogativi per cliente, prodotto, ufficio e paese;
- individuare anomalie operative tramite query.

---

## Esercizio autonomo 4.A - Prodotti ad alta vendita e bassa scorta



### Richiesta aziendale

Individuare prodotti con quantità totale ordinata superiore a 800 e quantità in magazzino inferiore a 1000.

Il report deve mostrare:

- codice prodotto;
- nome prodotto;
- linea prodotto;
- quantità totale ordinata;
- quantità in magazzino;
- prezzo di acquisto;
- valore residuo di magazzino.

### Suggerimenti

- Aggregare prima le vendite per prodotto.
- Collegare il risultato con `prodotti`.
- Usare `HAVING` oppure una CTE con filtro finale.
- Calcolare il valore residuo con `quantita_magazzino * prezzo_acquisto`.

### Evidenza richiesta

Salvare la query e indicare quali prodotti potrebbero richiedere attenzione operativa.

---

## Esercizio autonomo 4.B - Ordini spediti in ritardo per cliente



### Richiesta aziendale

Individuare i clienti con ordini spediti dopo la data richiesta.

Il report deve mostrare:

- codice cliente;
- ragione sociale;
- paese;
- numero ordini in ritardo;
- ritardo medio;
- ritardo massimo.

### Suggerimenti

- Usare `ordini` e `clienti`.
- Considerare solo ordini con `data_spedizione IS NOT NULL`.
- Il ritardo si calcola con `DATEDIFF(data_spedizione, data_richiesta)`.
- Filtrare solo i casi con ritardo maggiore di zero.
- Raggruppare per cliente.

### Evidenza richiesta

Salvare la query e indicare i primi 10 clienti per ritardo massimo.

---

## Esercizio autonomo 4.C - Miglior prodotto per ogni linea in valore venduto



### Richiesta aziendale

Per ogni linea prodotto, individuare il prodotto che ha generato il valore venduto più alto.

Il report deve mostrare:

- linea prodotto;
- codice prodotto;
- nome prodotto;
- valore totale venduto;
- posizione nella linea.

### Suggerimenti

- Calcolare il valore venduto per prodotto.
- Usare `ROW_NUMBER()` con `PARTITION BY linea_prodotto`.
- Filtrare la posizione uguale a 1.
- Collegare anche `linee_prodotto` se si vuole riportare la descrizione testuale della linea.

### Evidenza richiesta

Salvare la query e indicare il prodotto principale per ogni linea.

---

## Domande di verifica della sessione 4



1. Che differenza c'è tra `RANK()` e `ROW_NUMBER()`?
2. A cosa serve `PARTITION BY` nelle funzioni finestra?
3. Perché le CTE migliorano la leggibilità delle query complesse?
4. Perché una dashboard per paese deve separare ordini e pagamenti prima di unirli?

---
