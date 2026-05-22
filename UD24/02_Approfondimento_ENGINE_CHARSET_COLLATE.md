# LAB24 - Approfondimento MySQL: `ENGINE`, `CHARSET` e `COLLATE`

## Obiettivo

In MySQL, quando si crea una tabella, è possibile specificare alcune opzioni aggiuntive alla fine del comando `CREATE TABLE`.

Esempio:

```sql
CREATE TABLE linee_prodotto (
  linea_prodotto varchar(50) NOT NULL,
  descrizione_testuale varchar(4000) DEFAULT NULL,
  descrizione_html mediumtext,
  immagine mediumblob,
  PRIMARY KEY (linea_prodotto)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

La parte finale:

```sql
ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
```

non è obbligatoria dal punto di vista sintattico, ma è spesso consigliata per rendere esplicito il comportamento della tabella.

---

## La tabella può essere creata anche senza queste opzioni?

Sì. La tabella può essere creata anche così:

```sql
CREATE TABLE linee_prodotto (
  linea_prodotto varchar(50) NOT NULL,
  descrizione_testuale varchar(4000) DEFAULT NULL,
  descrizione_html mediumtext,
  immagine mediumblob,
  PRIMARY KEY (linea_prodotto)
);
```

In questo caso MySQL userà le impostazioni predefinite del database o del server.

Questo significa che il risultato può dipendere dalla configurazione dell'ambiente in cui lo script viene eseguito.

---

## `ENGINE=InnoDB`

La clausola:

```sql
ENGINE=InnoDB
```

indica che la tabella deve usare il motore di archiviazione **InnoDB**.

InnoDB è il motore normalmente consigliato per database relazionali perché supporta:

- chiavi esterne;
- transazioni;
- rollback;
- integrità referenziale;
- gestione più sicura dei dati collegati tra loro.

Nel database di esempio sono presenti relazioni tra tabelle, ad esempio:

```text
prodotti.linea_prodotto -> linee_prodotto.linea_prodotto
```

Per questo motivo, in un database con vincoli tra tabelle, è opportuno usare InnoDB.

---

## `DEFAULT CHARSET=utf8mb4`

La clausola:

```sql
DEFAULT CHARSET=utf8mb4
```

indica il set di caratteri predefinito della tabella.

`utf8mb4` è consigliato perché supporta correttamente i caratteri Unicode, inclusi:

- lettere accentate;
- simboli;
- caratteri internazionali;
- caratteri speciali più estesi.

Per un database con testi in italiano, usare `utf8mb4` aiuta a evitare problemi con caratteri come:

```text
à è é ì ò ù
```

---

## `COLLATE=utf8mb4_unicode_ci`

La clausola:

```sql
COLLATE=utf8mb4_unicode_ci
```

definisce il criterio usato da MySQL per confrontare e ordinare le stringhe.

La parte `ci` significa **case insensitive**, cioè non distingue tra maiuscole e minuscole nei confronti testuali.

Ad esempio, nei confronti testuali possono essere considerate equivalenti stringhe come:

```text
Auto
auto
AUTO
```

La collation influenza operazioni come:

- confronti con `WHERE`;
- ordinamenti con `ORDER BY`;
- ricerche testuali;
- confronti tra valori di colonne testuali.

---

## È indispensabile inserire queste opzioni in ogni `CREATE TABLE`?

No, non è indispensabile.

Tuttavia, è consigliabile farlo quando si vuole essere certi che lo script produca lo stesso risultato anche su ambienti MySQL configurati in modo diverso.

Esempio con opzioni esplicite:

```sql
CREATE TABLE linee_prodotto (
  linea_prodotto varchar(50) NOT NULL,
  descrizione_testuale varchar(4000) DEFAULT NULL,
  descrizione_html mediumtext,
  immagine mediumblob,
  PRIMARY KEY (linea_prodotto)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## Alternativa: impostare charset e collation sul database

Invece di ripetere `DEFAULT CHARSET` e `COLLATE` su ogni tabella, è possibile impostarli direttamente quando si crea il database.

Esempio:

```sql
DROP DATABASE IF EXISTS modelli_classici_it;

CREATE DATABASE modelli_classici_it
  DEFAULT CHARACTER SET utf8mb4
  DEFAULT COLLATE utf8mb4_unicode_ci;

USE modelli_classici_it;
```

A questo punto, le tabelle create all'interno del database erediteranno queste impostazioni, salvo diversa indicazione.

La tabella può quindi essere scritta in modo più compatto:

```sql
CREATE TABLE linee_prodotto (
  linea_prodotto varchar(50) NOT NULL,
  descrizione_testuale varchar(4000) DEFAULT NULL,
  descrizione_html mediumtext,
  immagine mediumblob,
  PRIMARY KEY (linea_prodotto)
) ENGINE=InnoDB;
```

---

## Confronto tra le due soluzioni

### Soluzione 1: opzioni ripetute su ogni tabella

```sql
CREATE TABLE linee_prodotto (
  linea_prodotto varchar(50) NOT NULL,
  descrizione_testuale varchar(4000) DEFAULT NULL,
  descrizione_html mediumtext,
  immagine mediumblob,
  PRIMARY KEY (linea_prodotto)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

Vantaggio:

- ogni tabella dichiara esplicitamente motore, charset e collation.

Svantaggio:

- lo script è più ripetitivo.

---

### Soluzione 2: impostazioni definite a livello di database

```sql
CREATE DATABASE modelli_classici_it
  DEFAULT CHARACTER SET utf8mb4
  DEFAULT COLLATE utf8mb4_unicode_ci;

USE modelli_classici_it;

CREATE TABLE linee_prodotto (
  linea_prodotto varchar(50) NOT NULL,
  descrizione_testuale varchar(4000) DEFAULT NULL,
  descrizione_html mediumtext,
  immagine mediumblob,
  PRIMARY KEY (linea_prodotto)
) ENGINE=InnoDB;
```

Vantaggio:

- lo script è meno ripetitivo;
- charset e collation sono centralizzati a livello di database.

Svantaggio:

- bisogna ricordare che le tabelle ereditano le impostazioni del database.

---

## Tabella riepilogativa

| Opzione | Indispensabile? | Consigliata? | Motivo |
|---|---:|---:|---|
| `ENGINE=InnoDB` | No | Sì | Supporta chiavi esterne, transazioni e integrità referenziale |
| `DEFAULT CHARSET=utf8mb4` | No | Sì | Gestisce correttamente caratteri accentati e Unicode |
| `COLLATE=utf8mb4_unicode_ci` | No | Sì | Definisce confronti e ordinamenti coerenti tra stringhe |

---

## Forma consigliata per un laboratorio didattico

Per un laboratorio didattico, una soluzione chiara è impostare charset e collation a livello di database e specificare `ENGINE=InnoDB` nelle tabelle.

Esempio:

```sql
DROP DATABASE IF EXISTS modelli_classici_it;

CREATE DATABASE modelli_classici_it
  DEFAULT CHARACTER SET utf8mb4
  DEFAULT COLLATE utf8mb4_unicode_ci;

USE modelli_classici_it;

CREATE TABLE linee_prodotto (
  linea_prodotto varchar(50) NOT NULL,
  descrizione_testuale varchar(4000) DEFAULT NULL,
  descrizione_html mediumtext,
  immagine mediumblob,
  PRIMARY KEY (linea_prodotto)
) ENGINE=InnoDB;
```

In questo modo:

- il database gestisce correttamente i caratteri testuali;
- le tabelle usano un motore adatto alle relazioni;
- lo script resta leggibile;
- le impostazioni principali sono esplicite e controllabili.

---

## Conclusione

Le opzioni `ENGINE`, `CHARSET` e `COLLATE` non sono obbligatorie per creare una tabella, ma sono importanti per controllare il comportamento del database.

In particolare:

- `ENGINE=InnoDB` è consigliato per database con relazioni e chiavi esterne;
- `utf8mb4` è consigliato per gestire correttamente testi e caratteri accentati;
- `utf8mb4_unicode_ci` rende più coerenti confronti e ordinamenti testuali.

Per esercitazioni SQL e database relazionali, è buona pratica rendere queste impostazioni esplicite nello script.
