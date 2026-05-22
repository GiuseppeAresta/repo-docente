# LAB24 - Laboratorio guidato SQL per analisi dati aziendale

## Collocazione nel percorso

Questa laboratorio di consolidamento consolida l'uso operativo di SQL dopo l'introduzione a DDL, DML e DQL.

Il laboratorio usa il database `modelli_classici_it` e richiede di costruire interrogazioni orientate all'analisi di dati aziendali.

## Durata

8 ore complessive, suddivise in 4 sessioni da 2 ore.

## Obiettivi

Al termine del laboratorio guidato il partecipante deve saper:

- leggere dati da tabelle singole;
- collegare tabelle tramite `JOIN`;
- calcolare indicatori economici con colonne calcolate;
- usare `GROUP BY` e funzioni aggregate;
- distinguere `WHERE` e `HAVING`;
- usare CTE e funzioni finestra in query di analisi;
- leggere il risultato di una query come informazione utile per una decisione aziendale.

## Prerequisiti

- Conoscenza di base di tabelle, colonne, chiavi primarie e chiavi esterne.
- Conoscenza iniziale di `SELECT`, `WHERE`, `ORDER BY`.
- Database `modelli_classici_it` già importato.

## File collegati

- `01_Schema_database_modelli_classici_it.md`
- `02_Approfondimento_ENGINE_CHARSET_COLLATE.md`
- `04_LAB24_autonomo_query_analisi_aziendale.md`
- `05_Soluzione_docente_LAB24_autonomo.md`

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

## Laboratorio guidato 1.1 - Catalogo prodotti con linea commerciale



### Richiesta aziendale

Estrarre l'elenco dei prodotti mostrando anche la relativa linea prodotto e la descrizione testuale della linea. Il report serve per controllare se tutti i prodotti sono correttamente classificati.

### Query

```sql
SELECT
    p.codice_prodotto,
    p.nome_prodotto,
    p.linea_prodotto,
    lp.descrizione_testuale,
    p.scala_prodotto,
    p.prezzo_listino
FROM prodotti p
INNER JOIN linee_prodotto lp
    ON p.linea_prodotto = lp.linea_prodotto
ORDER BY p.linea_prodotto, p.nome_prodotto;
```

### Lettura del risultato

Ogni riga rappresenta un prodotto del catalogo. La join consente di arricchire i dati tecnici del prodotto con le informazioni descrittive della linea commerciale.

---

## Laboratorio guidato 1.2 - Margine teorico dei prodotti



### Richiesta aziendale

Calcolare per ogni prodotto il margine teorico unitario, cioè la differenza tra prezzo di listino e prezzo di acquisto.

### Query

```sql
SELECT
    codice_prodotto,
    nome_prodotto,
    linea_prodotto,
    prezzo_acquisto,
    prezzo_listino,
    ROUND(prezzo_listino - prezzo_acquisto, 2) AS margine_unitario_teorico
FROM prodotti
ORDER BY margine_unitario_teorico DESC;
```

### Lettura del risultato

I prodotti in cima all'elenco hanno il margine unitario teorico più alto. Il dato è teorico perché non considera sconti, costi di gestione, spedizioni o resi.

---

## Laboratorio guidato 1.3 - Percentuale di margine



### Richiesta aziendale

Calcolare la percentuale di margine rispetto al prezzo di acquisto.

Formula:

```text
((prezzo_listino - prezzo_acquisto) / prezzo_acquisto) * 100
```

### Query

```sql
SELECT
    codice_prodotto,
    nome_prodotto,
    linea_prodotto,
    prezzo_acquisto,
    prezzo_listino,
    ROUND(((prezzo_listino - prezzo_acquisto) / prezzo_acquisto) * 100, 2) AS percentuale_margine
FROM prodotti
ORDER BY percentuale_margine DESC;
```

### Lettura del risultato

La percentuale permette di confrontare prodotti con prezzi molto diversi. Un prodotto economico può avere un margine assoluto basso ma una percentuale di margine elevata.

---

## Laboratorio guidato 1.4 - Prodotti con scorta bassa



### Richiesta aziendale

Individuare i prodotti con quantità in magazzino inferiore a 1000 unità, ordinandoli dalla scorta più bassa alla più alta.

### Query

```sql
SELECT
    codice_prodotto,
    nome_prodotto,
    linea_prodotto,
    quantita_magazzino,
    prezzo_acquisto,
    ROUND(quantita_magazzino * prezzo_acquisto, 2) AS valore_scorta_a_costo
FROM prodotti
WHERE quantita_magazzino < 1000
ORDER BY quantita_magazzino ASC;
```

### Lettura del risultato

Il report aiuta a individuare prodotti potenzialmente critici per disponibilità. La colonna `valore_scorta_a_costo` mostra il valore economico della giacenza residua.

---

## Laboratorio guidato 1.5 - Classificazione prodotti per fascia prezzo



### Richiesta aziendale

Classificare i prodotti in fasce commerciali in base al prezzo di listino:

- `Economico`: prezzo inferiore a 70;
- `Medio`: prezzo tra 70 e 150;
- `Premium`: prezzo superiore a 150.

### Query

```sql
SELECT
    codice_prodotto,
    nome_prodotto,
    linea_prodotto,
    prezzo_listino,
    CASE
        WHEN prezzo_listino < 70 THEN 'Economico'
        WHEN prezzo_listino BETWEEN 70 AND 150 THEN 'Medio'
        ELSE 'Premium'
    END AS fascia_prezzo
FROM prodotti
ORDER BY prezzo_listino DESC;
```

### Lettura del risultato

La classificazione permette di leggere il catalogo per segmenti commerciali, utile per campagne, listini e analisi del posizionamento.

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

## Laboratorio guidato 2.1 - Valore delle righe ordine



### Richiesta aziendale

Calcolare il valore economico di ogni riga ordine.

### Query

```sql
SELECT
    d.numero_ordine,
    d.codice_prodotto,
    p.nome_prodotto,
    p.linea_prodotto,
    d.quantita_ordinata,
    d.prezzo_unitario,
    ROUND(d.quantita_ordinata * d.prezzo_unitario, 2) AS valore_riga
FROM dettagli_ordini d
INNER JOIN prodotti p
    ON d.codice_prodotto = p.codice_prodotto
ORDER BY d.numero_ordine, d.numero_riga_ordine;
```

### Lettura del risultato

Ogni riga rappresenta una voce dell'ordine. Il valore della riga è la base per tutte le successive analisi di vendita.

---

## Laboratorio guidato 2.2 - Totale di ogni ordine



### Richiesta aziendale

Calcolare il valore totale di ogni ordine.

### Query

```sql
SELECT
    numero_ordine,
    ROUND(SUM(quantita_ordinata * prezzo_unitario), 2) AS totale_ordine
FROM dettagli_ordini
GROUP BY numero_ordine
ORDER BY totale_ordine DESC;
```

### Lettura del risultato

Il report consente di identificare gli ordini economicamente più importanti.

---

## Laboratorio guidato 2.3 - Totale ordine con dati cliente



### Richiesta aziendale

Mostrare numero ordine, data ordine, stato ordine, cliente, paese e totale ordine.

### Query

```sql
SELECT
    o.numero_ordine,
    o.data_ordine,
    o.stato_ordine,
    c.codice_cliente,
    c.ragione_sociale,
    c.paese,
    ROUND(SUM(d.quantita_ordinata * d.prezzo_unitario), 2) AS totale_ordine
FROM ordini o
INNER JOIN clienti c
    ON o.codice_cliente = c.codice_cliente
INNER JOIN dettagli_ordini d
    ON o.numero_ordine = d.numero_ordine
GROUP BY
    o.numero_ordine,
    o.data_ordine,
    o.stato_ordine,
    c.codice_cliente,
    c.ragione_sociale,
    c.paese
ORDER BY totale_ordine DESC;
```

### Lettura del risultato

La query unisce la dimensione commerciale del cliente con il valore economico dell'ordine.

---

## Laboratorio guidato 2.4 - Vendite per cliente



### Richiesta aziendale

Calcolare il valore totale degli ordini per ogni cliente.

### Query

```sql
SELECT
    c.codice_cliente,
    c.ragione_sociale,
    c.paese,
    COUNT(DISTINCT o.numero_ordine) AS numero_ordini,
    ROUND(SUM(d.quantita_ordinata * d.prezzo_unitario), 2) AS totale_ordini
FROM clienti c
INNER JOIN ordini o
    ON c.codice_cliente = o.codice_cliente
INNER JOIN dettagli_ordini d
    ON o.numero_ordine = d.numero_ordine
GROUP BY
    c.codice_cliente,
    c.ragione_sociale,
    c.paese
ORDER BY totale_ordini DESC;
```

### Lettura del risultato

Il report permette di distinguere clienti con molti ordini da clienti con ordini di valore elevato.

---

## Laboratorio guidato 2.5 - Vendite per linea prodotto



### Richiesta aziendale

Calcolare il valore totale venduto per ogni linea prodotto, includendo la descrizione della linea.

### Query

```sql
SELECT
    lp.linea_prodotto,
    lp.descrizione_testuale,
    ROUND(SUM(d.quantita_ordinata * d.prezzo_unitario), 2) AS totale_venduto
FROM linee_prodotto lp
INNER JOIN prodotti p
    ON lp.linea_prodotto = p.linea_prodotto
INNER JOIN dettagli_ordini d
    ON p.codice_prodotto = d.codice_prodotto
GROUP BY
    lp.linea_prodotto,
    lp.descrizione_testuale
ORDER BY totale_venduto DESC;
```

### Lettura del risultato

La query usa la tabella `linee_prodotto`, quindi non tratta la linea solo come testo presente nella tabella `prodotti`, ma come entità del modello dati.

---

## Laboratorio guidato 2.6 - Paesi con maggiore valore ordini



### Richiesta aziendale

Calcolare, per ogni paese dei clienti:

- numero clienti con almeno un ordine;
- numero ordini;
- valore totale ordini.

### Query

```sql
SELECT
    c.paese,
    COUNT(DISTINCT c.codice_cliente) AS numero_clienti_con_ordini,
    COUNT(DISTINCT o.numero_ordine) AS numero_ordini,
    ROUND(SUM(d.quantita_ordinata * d.prezzo_unitario), 2) AS valore_totale_ordini
FROM clienti c
INNER JOIN ordini o
    ON c.codice_cliente = o.codice_cliente
INNER JOIN dettagli_ordini d
    ON o.numero_ordine = d.numero_ordine
GROUP BY c.paese
ORDER BY valore_totale_ordini DESC;
```

### Lettura del risultato

Il report permette una prima analisi geografica delle vendite.

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

## Laboratorio guidato 3.1 - Totale pagamenti per cliente



### Richiesta aziendale

Calcolare il totale dei pagamenti ricevuti per ciascun cliente.

### Query

```sql
SELECT
    c.codice_cliente,
    c.ragione_sociale,
    c.paese,
    ROUND(SUM(p.importo), 2) AS totale_pagamenti
FROM clienti c
INNER JOIN pagamenti p
    ON c.codice_cliente = p.codice_cliente
GROUP BY
    c.codice_cliente,
    c.ragione_sociale,
    c.paese
ORDER BY totale_pagamenti DESC;
```

### Lettura del risultato

Il report misura gli incassi registrati, non il valore degli ordini.

---

## Prima di proseguire: che cos'è una CTE

Una **CTE** significa **Common Table Expression**.

In SQL una CTE è un risultato intermedio temporaneo, definito all'inizio di una query tramite la parola chiave `WITH`.

La forma generale è:

```sql
WITH nome_cte AS (
    SELECT
        colonne
    FROM tabella
    WHERE condizioni
)
SELECT
    colonne
FROM nome_cte;
```

La CTE non crea una nuova tabella fisica nel database. Il nome indicato dopo `WITH` vale solo per la query immediatamente successiva.

### Perché usare una CTE

Una CTE è utile quando una query diventa troppo lunga o quando il calcolo deve essere diviso in passaggi più chiari.

Nel contesto di questo laboratorio, le CTE sono particolarmente utili per:

- calcolare prima il totale degli ordini per cliente;
- calcolare separatamente il totale dei pagamenti per cliente;
- unire poi questi risultati alla tabella `clienti`;
- evitare duplicazioni di importi causate da join tra tabelle con granularità diverse.

### Esempio semplificato

```sql
WITH totali_ordini_cliente AS (
    SELECT
        o.codice_cliente,
        SUM(d.quantita_ordinata * d.prezzo_unitario) AS totale_ordini
    FROM ordini o
    INNER JOIN dettagli_ordini d
        ON o.numero_ordine = d.numero_ordine
    GROUP BY o.codice_cliente
)
SELECT
    codice_cliente,
    totale_ordini
FROM totali_ordini_cliente;
```

In questo esempio:

1. la CTE `totali_ordini_cliente` calcola il totale degli ordini per ogni cliente;
2. la query finale legge il risultato della CTE come se fosse una tabella temporanea;
3. il risultato resta disponibile solo per questa query.

### CTE multiple

È possibile definire più CTE nella stessa query separandole con una virgola:

```sql
WITH prima_cte AS (
    SELECT ...
),
seconda_cte AS (
    SELECT ...
)
SELECT
    ...
FROM prima_cte
INNER JOIN seconda_cte
    ON ...;
```

Questa tecnica viene usata nei laboratori successivi per confrontare ordini e pagamenti senza mescolare direttamente righe di dettaglio ordine e righe di pagamento.

### Errori frequenti da evitare

- Pensare che la CTE crei una tabella permanente.
- Usare il nome della CTE in una query successiva separata.
- Scrivere più blocchi `WITH` separati invece di dichiarare più CTE nello stesso blocco.
- Collegare ordini, dettagli ordine e pagamenti in un'unica aggregazione senza prima controllare le cardinalità.

---

## Laboratorio guidato 3.2 - Confronto ordini e pagamenti per cliente



### Richiesta aziendale

Per ogni cliente, confrontare totale ordini e totale pagamenti.

### Query

```sql
WITH totali_ordini_cliente AS (
    SELECT
        o.codice_cliente,
        ROUND(SUM(d.quantita_ordinata * d.prezzo_unitario), 2) AS totale_ordini
    FROM ordini o
    INNER JOIN dettagli_ordini d
        ON o.numero_ordine = d.numero_ordine
    GROUP BY o.codice_cliente
),
totali_pagamenti_cliente AS (
    SELECT
        codice_cliente,
        ROUND(SUM(importo), 2) AS totale_pagamenti
    FROM pagamenti
    GROUP BY codice_cliente
)
SELECT
    c.codice_cliente,
    c.ragione_sociale,
    c.paese,
    COALESCE(toc.totale_ordini, 0) AS totale_ordini,
    COALESCE(tpc.totale_pagamenti, 0) AS totale_pagamenti,
    ROUND(COALESCE(toc.totale_ordini, 0) - COALESCE(tpc.totale_pagamenti, 0), 2) AS saldo_ordini_pagamenti
FROM clienti c
LEFT JOIN totali_ordini_cliente toc
    ON c.codice_cliente = toc.codice_cliente
LEFT JOIN totali_pagamenti_cliente tpc
    ON c.codice_cliente = tpc.codice_cliente
ORDER BY saldo_ordini_pagamenti DESC;
```

### Lettura del risultato

Un saldo positivo indica che il valore degli ordini supera il totale dei pagamenti registrati. Il dato va interpretato con cautela perché il database non collega direttamente ogni pagamento a uno specifico ordine.

---

## Laboratorio guidato 3.3 - Esposizione rispetto al limite di credito



### Richiesta aziendale

Individuare i clienti per cui il saldo tra ordini e pagamenti supera il limite di credito.

### Query

```sql
WITH totali_ordini_cliente AS (
    SELECT
        o.codice_cliente,
        ROUND(SUM(d.quantita_ordinata * d.prezzo_unitario), 2) AS totale_ordini
    FROM ordini o
    INNER JOIN dettagli_ordini d
        ON o.numero_ordine = d.numero_ordine
    GROUP BY o.codice_cliente
),
totali_pagamenti_cliente AS (
    SELECT
        codice_cliente,
        ROUND(SUM(importo), 2) AS totale_pagamenti
    FROM pagamenti
    GROUP BY codice_cliente
),
saldi_clienti AS (
    SELECT
        c.codice_cliente,
        c.ragione_sociale,
        c.paese,
        COALESCE(c.limite_credito, 0) AS limite_credito,
        COALESCE(toc.totale_ordini, 0) AS totale_ordini,
        COALESCE(tpc.totale_pagamenti, 0) AS totale_pagamenti,
        ROUND(COALESCE(toc.totale_ordini, 0) - COALESCE(tpc.totale_pagamenti, 0), 2) AS esposizione
    FROM clienti c
    LEFT JOIN totali_ordini_cliente toc
        ON c.codice_cliente = toc.codice_cliente
    LEFT JOIN totali_pagamenti_cliente tpc
        ON c.codice_cliente = tpc.codice_cliente
)
SELECT
    codice_cliente,
    ragione_sociale,
    paese,
    limite_credito,
    totale_ordini,
    totale_pagamenti,
    esposizione,
    ROUND(esposizione - limite_credito, 2) AS superamento_limite
FROM saldi_clienti
WHERE esposizione > limite_credito
ORDER BY superamento_limite DESC;
```

### Lettura del risultato

La query individua possibili situazioni da verificare con l'amministrazione clienti.

---

## Laboratorio guidato 3.4 - Clienti senza ordini



### Richiesta aziendale

Individuare i clienti presenti in anagrafica che non hanno mai effettuato ordini.

### Query

```sql
SELECT
    c.codice_cliente,
    c.ragione_sociale,
    c.paese,
    c.referente_commerciale
FROM clienti c
LEFT JOIN ordini o
    ON c.codice_cliente = o.codice_cliente
WHERE o.numero_ordine IS NULL
ORDER BY c.paese, c.ragione_sociale;
```

### Lettura del risultato

Il report può essere usato per attività commerciali di riattivazione o pulizia anagrafica.

---

## Laboratorio guidato 3.5 - Analisi tempi di spedizione



### Richiesta aziendale

Analizzare gli ordini spediti calcolando:

- giorni tra ordine e spedizione;
- giorni tra ordine e data richiesta;
- eventuale ritardo rispetto alla data richiesta.

### Query

```sql
SELECT
    o.numero_ordine,
    o.data_ordine,
    o.data_richiesta,
    o.data_spedizione,
    o.stato_ordine,
    c.ragione_sociale,
    DATEDIFF(o.data_spedizione, o.data_ordine) AS giorni_per_spedire,
    DATEDIFF(o.data_richiesta, o.data_ordine) AS giorni_disponibili,
    DATEDIFF(o.data_spedizione, o.data_richiesta) AS giorni_ritardo
FROM ordini o
INNER JOIN clienti c
    ON o.codice_cliente = c.codice_cliente
WHERE o.data_spedizione IS NOT NULL
ORDER BY giorni_ritardo DESC, giorni_per_spedire DESC;
```

### Lettura del risultato

Un valore positivo in `giorni_ritardo` indica una spedizione successiva alla data richiesta.

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

## Laboratorio guidato 4.1 - Classifica clienti per valore ordini



### Richiesta aziendale

Creare una classifica dei clienti in base al valore totale degli ordini.

### Query

```sql
WITH vendite_clienti AS (
    SELECT
        c.codice_cliente,
        c.ragione_sociale,
        c.paese,
        ROUND(SUM(d.quantita_ordinata * d.prezzo_unitario), 2) AS totale_ordini
    FROM clienti c
    INNER JOIN ordini o
        ON c.codice_cliente = o.codice_cliente
    INNER JOIN dettagli_ordini d
        ON o.numero_ordine = d.numero_ordine
    GROUP BY
        c.codice_cliente,
        c.ragione_sociale,
        c.paese
)
SELECT
    codice_cliente,
    ragione_sociale,
    paese,
    totale_ordini,
    RANK() OVER (ORDER BY totale_ordini DESC) AS posizione
FROM vendite_clienti
ORDER BY posizione;
```

### Lettura del risultato

La funzione `RANK()` assegna una posizione in classifica in base al totale ordini.

---

## Laboratorio guidato 4.2 - Top 3 prodotti per ogni linea prodotto



### Richiesta aziendale

Per ogni linea prodotto, individuare i primi 3 prodotti per quantità totale venduta.

### Query

```sql
WITH vendite_prodotti AS (
    SELECT
        p.codice_prodotto,
        p.nome_prodotto,
        p.linea_prodotto,
        SUM(d.quantita_ordinata) AS quantita_totale
    FROM prodotti p
    INNER JOIN dettagli_ordini d
        ON p.codice_prodotto = d.codice_prodotto
    GROUP BY
        p.codice_prodotto,
        p.nome_prodotto,
        p.linea_prodotto
),
classifica AS (
    SELECT
        codice_prodotto,
        nome_prodotto,
        linea_prodotto,
        quantita_totale,
        ROW_NUMBER() OVER (
            PARTITION BY linea_prodotto
            ORDER BY quantita_totale DESC
        ) AS posizione_linea
    FROM vendite_prodotti
)
SELECT
    linea_prodotto,
    posizione_linea,
    codice_prodotto,
    nome_prodotto,
    quantita_totale
FROM classifica
WHERE posizione_linea <= 3
ORDER BY linea_prodotto, posizione_linea;
```

### Lettura del risultato

`PARTITION BY linea_prodotto` crea una classifica separata per ogni linea prodotto.

---

## Laboratorio guidato 4.3 - Primo e ultimo ordine per cliente



### Richiesta aziendale

Per ogni cliente con ordini, individuare la data del primo ordine e la data dell'ultimo ordine.

### Query

```sql
SELECT
    c.codice_cliente,
    c.ragione_sociale,
    c.paese,
    MIN(o.data_ordine) AS data_primo_ordine,
    MAX(o.data_ordine) AS data_ultimo_ordine,
    COUNT(o.numero_ordine) AS numero_ordini
FROM clienti c
INNER JOIN ordini o
    ON c.codice_cliente = o.codice_cliente
GROUP BY
    c.codice_cliente,
    c.ragione_sociale,
    c.paese
ORDER BY data_ultimo_ordine DESC;
```

### Lettura del risultato

Il report aiuta a distinguere clienti storici, clienti recenti e clienti potenzialmente inattivi.

---

## Laboratorio guidato 4.4 - Performance commerciale per dipendente



### Richiesta aziendale

Calcolare il valore totale degli ordini dei clienti assegnati a ogni referente commerciale.

### Query

```sql
SELECT
    d.matricola_dipendente,
    CONCAT(d.nome, ' ', d.cognome) AS referente_commerciale,
    d.ruolo,
    COUNT(DISTINCT c.codice_cliente) AS numero_clienti_serviti,
    COUNT(DISTINCT o.numero_ordine) AS numero_ordini,
    ROUND(SUM(det.quantita_ordinata * det.prezzo_unitario), 2) AS totale_ordini
FROM dipendenti d
INNER JOIN clienti c
    ON d.matricola_dipendente = c.referente_commerciale
INNER JOIN ordini o
    ON c.codice_cliente = o.codice_cliente
INNER JOIN dettagli_ordini det
    ON o.numero_ordine = det.numero_ordine
GROUP BY
    d.matricola_dipendente,
    d.nome,
    d.cognome,
    d.ruolo
ORDER BY totale_ordini DESC;
```

### Lettura del risultato

La query misura il valore degli ordini generati dai clienti assegnati a ciascun referente commerciale.

---

## Laboratorio guidato 4.5 - Performance per ufficio



### Richiesta aziendale

Calcolare il valore totale degli ordini gestiti dai clienti assegnati ai dipendenti di ogni ufficio.

### Query

```sql
SELECT
    u.codice_ufficio,
    u.citta,
    u.paese,
    u.territorio,
    COUNT(DISTINCT d.matricola_dipendente) AS numero_dipendenti_coinvolti,
    COUNT(DISTINCT c.codice_cliente) AS numero_clienti_serviti,
    COUNT(DISTINCT o.numero_ordine) AS numero_ordini,
    ROUND(SUM(det.quantita_ordinata * det.prezzo_unitario), 2) AS totale_ordini
FROM uffici u
INNER JOIN dipendenti d
    ON u.codice_ufficio = d.codice_ufficio
INNER JOIN clienti c
    ON d.matricola_dipendente = c.referente_commerciale
INNER JOIN ordini o
    ON c.codice_cliente = o.codice_cliente
INNER JOIN dettagli_ordini det
    ON o.numero_ordine = det.numero_ordine
GROUP BY
    u.codice_ufficio,
    u.citta,
    u.paese,
    u.territorio
ORDER BY totale_ordini DESC;
```

### Lettura del risultato

Il report aggrega le vendite per struttura territoriale interna.

---

## Laboratorio guidato 4.6 - Dashboard sintetica per paese



### Richiesta aziendale

Creare un report per paese con:

- numero clienti;
- numero ordini;
- valore totale ordini;
- totale pagamenti;
- saldo ordini-pagamenti.

### Query

```sql
WITH ordini_paese AS (
    SELECT
        c.paese,
        COUNT(DISTINCT c.codice_cliente) AS numero_clienti_con_ordini,
        COUNT(DISTINCT o.numero_ordine) AS numero_ordini,
        ROUND(SUM(d.quantita_ordinata * d.prezzo_unitario), 2) AS totale_ordini
    FROM clienti c
    INNER JOIN ordini o
        ON c.codice_cliente = o.codice_cliente
    INNER JOIN dettagli_ordini d
        ON o.numero_ordine = d.numero_ordine
    GROUP BY c.paese
),
pagamenti_paese AS (
    SELECT
        c.paese,
        ROUND(SUM(p.importo), 2) AS totale_pagamenti
    FROM clienti c
    INNER JOIN pagamenti p
        ON c.codice_cliente = p.codice_cliente
    GROUP BY c.paese
)
SELECT
    op.paese,
    op.numero_clienti_con_ordini,
    op.numero_ordini,
    op.totale_ordini,
    COALESCE(pp.totale_pagamenti, 0) AS totale_pagamenti,
    ROUND(op.totale_ordini - COALESCE(pp.totale_pagamenti, 0), 2) AS saldo
FROM ordini_paese op
LEFT JOIN pagamenti_paese pp
    ON op.paese = pp.paese
ORDER BY op.totale_ordini DESC;
```

### Lettura del risultato

Il report sintetizza vendite e incassi per area geografica cliente.

---

---

# Evidenza consigliata per il laboratorio guidato

Creare un file:

```text
docs/evidence_LAB24_guidato.md
```

Il file deve contenere:

1. query principali eseguite;
2. eventuali errori incontrati e correzioni applicate;
3. una breve nota su almeno tre risultati aziendali osservati;
4. screenshot o copia dei risultati più significativi, se richiesto dal docente.
