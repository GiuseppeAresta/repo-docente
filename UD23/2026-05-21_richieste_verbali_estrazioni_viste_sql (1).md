# Simulazione di richieste utente e viste SQL

Questo file mostra, in forma sintetica, alcune possibili richieste verbali di estrazione dati e la vista SQL corrispondente da creare.

Sono riportati solo i codici SQL delle viste, senza query di verifica, diagrammi o dump di risultato.

---

## 1. V_LIBRI

**Richiesta verbale dell’utente**

> Mi serve un elenco dei libri con ISBN, titolo, genere, autore, editore ed edizione.

**Vista SQL**

```sql
CREATE VIEW V_LIBRI AS
SELECT
    L.CODICE_ISBN,
    L.TITOLO,
    G.GENERE,
    A.AUTORE,
    E.EDITORE,
    L.EDIZIONE
FROM LIBRI L
JOIN GENERI G ON G.ID = L.ID_GENERE
JOIN AUTORI A ON A.ID = L.ID_AUTORE
JOIN EDITORI E ON E.ID = L.ID_EDITORE;
```

---

## 2. V_LETTORI

**Richiesta verbale dell’utente**

> Vorrei vedere i dati principali dei lettori, compresi titolo di studio e account associato.

**Vista SQL**

```sql
CREATE VIEW V_LETTORI AS
SELECT
    L.CODICE_LETTORE,
    L.NOME,
    L.COGNOME,
    DATE_FORMAT(L.DATA_DI_NASCITA, '%d/%m/%Y') AS DATA_DI_NASCITA,
    IF(L.SESSO, 'MASCHIO', 'FEMMINA') AS SESSO,
    T.TITOLO_DI_STUDIO,
    L.CODICE_FISCALE,
    A.NOME_UTENTE,
    A.PASSWD
FROM LETTORI L
JOIN TITOLI_DI_STUDIO T ON T.ID = L.ID_TITOLO_DI_STUDIO
JOIN ACCOUNTS A ON A.ID = L.ID_ACCOUNT;
```

---

## 3. V_CONTATTI

**Richiesta verbale dell’utente**

> Mi serve l’elenco dei contatti dei lettori, indicando anche il tipo di contatto.

**Vista SQL**

```sql
CREATE VIEW V_CONTATTI AS
SELECT
    L.CODICE_LETTORE,
    L.NOME,
    L.COGNOME,
    C.CONTATTO,
    T.TIPO_CONTATTO
FROM CONTATTI C
JOIN LETTORI L ON L.ID = C.ID_LETTORE
JOIN TIPI_CONTATTO T ON T.ID = C.ID_TIPO_CONTATTO;
```

---

## 4. V_MAGAZZINO

**Richiesta verbale dell’utente**

> Vorrei controllare le copie presenti in magazzino con libro, data di carico e prezzi.

**Vista SQL**

```sql
CREATE VIEW V_MAGAZZINO AS
SELECT
    M.CODICE_LIBRO,
    L.CODICE_ISBN,
    L.TITOLO,
    DATE_FORMAT(M.DATA_CARICO, '%d/%m/%Y') AS DATA_CARICO,
    M.PREZZO_CARICO,
    M.PREZZO_SCARICO
FROM MAGAZZINO M
JOIN LIBRI L ON L.ID = M.ID_LIBRO;
```

---

## 5. V_LISTINO

**Richiesta verbale dell’utente**

> Mi serve un listino dei libri con il prezzo di prestito.

**Vista SQL**

```sql
CREATE VIEW V_LISTINO AS
SELECT DISTINCT
    L.CODICE_ISBN,
    L.TITOLO,
    M.PREZZO_SCARICO AS PREZZO
FROM MAGAZZINO M
JOIN LIBRI L ON L.ID = M.ID_LIBRO;
```

---

## 6. V_GIACENZE

**Richiesta verbale dell’utente**

> Vorrei sapere quante copie disponibili ci sono per ogni libro.

**Vista SQL**

```sql
CREATE VIEW V_GIACENZE AS
SELECT
    L.CODICE_ISBN,
    L.TITOLO,
    COUNT(*) AS GIACENZA
FROM MAGAZZINO M
JOIN LIBRI L ON L.ID = M.ID_LIBRO
WHERE M.PRESTATO = FALSE
GROUP BY L.CODICE_ISBN, L.TITOLO;
```

---

## 7. V_PRESTITI

**Richiesta verbale dell’utente**

> Mi serve una vista completa delle operazioni di prestito con le principali date operative.

**Vista SQL**

```sql
CREATE VIEW V_PRESTITI AS
SELECT
    P.CODICE_OPERAZIONE,
    M.CODICE_LIBRO,
    L.CODICE_LETTORE,
    DATE_FORMAT(P.DATA_OPERAZIONE, '%d/%m/%Y') AS DATA_OPERAZIONE,
    DATE_FORMAT(P.DATA_RITIRO, '%d/%m/%Y') AS DATA_RITIRO,
    IFNULL(DATE_FORMAT(P.DATA_PRESTITO, '%d/%m/%Y'), '-') AS DATA_PRESTITO,
    IFNULL(DATE_FORMAT(P.DATA_RESTITUZIONE, '%d/%m/%Y'), '-') AS DATA_RESTITUZIONE,
    IFNULL(DATE_FORMAT(P.DATA_CONSEGNA, '%d/%m/%Y'), '-') AS DATA_CONSEGNA
FROM PRESTITI P
JOIN MAGAZZINO M ON M.ID = P.ID_MAGAZZINO
JOIN LETTORI L ON L.ID = P.ID_LETTORE;
```

---

## 8. V_FATTURATO

**Richiesta verbale dell’utente**

> Vorrei vedere le operazioni che hanno generato fatturato perché il libro è stato prestato.

**Vista SQL**

```sql
CREATE VIEW V_FATTURATO AS
SELECT
    P.CODICE_OPERAZIONE,
    M.CODICE_LIBRO,
    DATE_FORMAT(P.DATA_PRESTITO, '%d/%m/%Y') AS DATA,
    M.PREZZO_SCARICO AS PREZZO
FROM PRESTITI P
JOIN MAGAZZINO M ON M.ID = P.ID_MAGAZZINO
JOIN LETTORI L ON L.ID = P.ID_LETTORE
WHERE P.DATA_PRESTITO IS NOT NULL;
```

---

## 9. V_FATTURATO_MENSILE

**Richiesta verbale dell’utente**

> Mi serve il fatturato raggruppato per mese.

**Vista SQL**

```sql
CREATE VIEW V_FATTURATO_MENSILE AS
SELECT
    MONTH(P.DATA_PRESTITO) AS MESE,
    DATE_FORMAT(P.DATA_PRESTITO, '%Y-%m') AS PERIODO,
    SUM(M.PREZZO_SCARICO) AS PREZZO
FROM PRESTITI P
JOIN MAGAZZINO M ON M.ID = P.ID_MAGAZZINO
JOIN LETTORI L ON L.ID = P.ID_LETTORE
WHERE P.DATA_PRESTITO IS NOT NULL
GROUP BY MESE, PERIODO;
```

---

## 10. V_LIBRI_ORDINATI

**Richiesta verbale dell’utente**

> Vorrei l’elenco dei libri prenotati ma non ancora effettivamente prestati.

**Vista SQL**

```sql
CREATE VIEW V_LIBRI_ORDINATI AS
SELECT
    P.CODICE_OPERAZIONE,
    M.CODICE_LIBRO,
    LB.CODICE_ISBN,
    LB.TITOLO,
    LE.CODICE_LETTORE,
    P.DATA_OPERAZIONE,
    P.DATA_RITIRO
FROM PRESTITI P
JOIN MAGAZZINO M ON M.ID = P.ID_MAGAZZINO
JOIN LIBRI LB ON LB.ID = M.ID_LIBRO
JOIN LETTORI LE ON LE.ID = P.ID_LETTORE
WHERE P.DATA_PRESTITO IS NULL;
```

---

## 11. V_LIBRI_PRESTATI

**Richiesta verbale dell’utente**

> Mi serve l’elenco dei libri prestati che non risultano ancora consegnati.

**Vista SQL**

```sql
CREATE VIEW V_LIBRI_PRESTATI AS
SELECT
    P.CODICE_OPERAZIONE,
    M.CODICE_LIBRO,
    LB.CODICE_ISBN,
    LB.TITOLO,
    LE.CODICE_LETTORE,
    P.DATA_OPERAZIONE,
    P.DATA_PRESTITO,
    P.DATA_RESTITUZIONE
FROM PRESTITI P
JOIN MAGAZZINO M ON M.ID = P.ID_MAGAZZINO
JOIN LIBRI LB ON LB.ID = M.ID_LIBRO
JOIN LETTORI LE ON LE.ID = P.ID_LETTORE
WHERE P.DATA_RESTITUZIONE IS NOT NULL
  AND P.DATA_CONSEGNA IS NULL;
```

---

## 12. V_LETTORI_NON_AFFIDABILI

**Richiesta verbale dell’utente**

> Vorrei individuare i lettori con ritardi medi elevati nella riconsegna dei libri.

**Vista SQL**

```sql
CREATE VIEW V_LETTORI_NON_AFFIDABILI AS
SELECT
    L.CODICE_LETTORE,
    L.NOME,
    L.COGNOME,
    CONCAT(CEIL(AVG(DATEDIFF(P.DATA_CONSEGNA, P.DATA_RESTITUZIONE))), ' gg') AS RITARDO_MEDIO
FROM PRESTITI P
JOIN LETTORI L ON L.ID = P.ID_LETTORE
WHERE P.DATA_CONSEGNA IS NOT NULL
  AND P.DATA_RESTITUZIONE IS NOT NULL
  AND DATEDIFF(P.DATA_CONSEGNA, P.DATA_RESTITUZIONE) >= 5
GROUP BY L.CODICE_LETTORE, L.NOME, L.COGNOME;
```

