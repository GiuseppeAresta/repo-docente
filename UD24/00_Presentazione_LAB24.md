# LAB24 - SQL per analisi dati aziendale

## Finalità del LAB24

Il LAB24 serve a trasformare le conoscenze SQL introdotte nelle attività precedenti in capacità operative di interrogazione e analisi.

Il partecipante non lavora su query isolate, ma su richieste realistiche di estrazione dati aziendali, come:

- analisi del catalogo prodotti;
- controllo delle giacenze;
- calcolo dei margini;
- analisi del valore degli ordini;
- aggregazione delle vendite per cliente, paese e linea prodotto;
- confronto tra ordini e pagamenti;
- individuazione di clienti con esposizione elevata;
- costruzione di classifiche e report sintetici.

## Collocazione consigliata

Questo laboratorio può essere somministrato dopo l'introduzione a DDL, DML e DQL, prima dell'avvio delle attività che richiedono interrogazioni più strutturate da applicazione Java o JDBC.

## Obiettivi didattici

Al termine del LAB24 il partecipante deve saper:

- individuare le tabelle necessarie per una richiesta di analisi;
- costruire join corrette tra tabelle collegate;
- calcolare indicatori con colonne derivate;
- usare `GROUP BY`, funzioni aggregate e `HAVING`;
- usare `LEFT JOIN` per individuare dati mancanti;
- usare CTE per separare passaggi logici complessi;
- usare funzioni finestra per classifiche e top N per gruppo;
- interpretare il risultato della query in chiave aziendale.

## Struttura del LAB24

| Parte | Durata indicativa | File |
|---|---:|---|
| Presentazione e schema | 30 min | `00_Presentazione_LAB24.md`, `01_Schema_database_modelli_classici_it.md` |
| Laboratorio guidato | 5 ore | `03_LAB24_guidato_query_analisi_aziendale.md` |
| Laboratorio autonomo | 2 ore | `04_LAB24_autonomo_query_analisi_aziendale.md` |
| Correzione e verifica | 30 min | `05_Soluzione_docente_LAB24_autonomo.md` |

## Progressione tecnica

| Sessione | Tecniche principali | Risultato atteso |
|---|---|---|
| 1 | `SELECT`, `WHERE`, `ORDER BY`, calcoli, `CASE`, join semplice | Report sul catalogo prodotti |
| 2 | `JOIN`, `GROUP BY`, aggregazioni, `HAVING` | Report su ordini, clienti e vendite |
| 3 | `LEFT JOIN`, `COALESCE`, CTE | Report su pagamenti ed esposizione clienti |
| 4 | CTE, `ROW_NUMBER`, `RANK`, `PARTITION BY` | Report avanzati e mini-dashboard |

## Nota metodologica

Il laboratorio è progettato per far emergere il legame tra modello dati e informazione aziendale. Ogni query deve essere valutata non solo per correttezza sintattica, ma anche per coerenza del risultato prodotto.
