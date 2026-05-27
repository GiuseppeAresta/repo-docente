# 02 - Mapping manuale, eccezioni e verifiche

## Il mapping manuale

Con JDBC il database restituisce righe e colonne tramite `ResultSet`. L'applicazione Java, invece, lavora con oggetti.

Il mapping manuale è il passaggio da una riga del database a un oggetto Java.

```java
private Docente mapRow(ResultSet rs) throws SQLException {
    return new Docente(
        rs.getInt("id"),
        rs.getString("nome"),
        rs.getString("email")
    );
}
```

Questo metodo ha una responsabilità precisa: trasformare una riga SQL in un oggetto del dominio.

## Perché isolare il mapping

Se il mapping viene scritto più volte, ogni modifica alla struttura dei dati diventa più rischiosa. Conviene avere un metodo privato dedicato, usato da tutti i metodi DAO che leggono la stessa tabella.

Esempio:

```java
private Docente mapDocente(ResultSet rs) throws SQLException {
    return new Docente(
        rs.getInt("id"),
        rs.getString("nome"),
        rs.getString("email")
    );
}
```

Il DAO rimane più leggibile e il codice duplicato diminuisce.

## Eccezione DAO

Il codice JDBC produce spesso `SQLException`. Se propaghiamo `SQLException` in tutta l'applicazione, anche il service e il main diventano dipendenti da JDBC.

Una soluzione didattica è introdurre una eccezione applicativa:

```java
public class DaoException extends RuntimeException {
    public DaoException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

Il DAO cattura `SQLException` e rilancia `DaoException`.

```java
try {
    // codice JDBC
} catch (SQLException e) {
    throw new DaoException("Errore durante la lettura dei docenti", e);
}
```

Il service non deve conoscere `SQLException`; deve lavorare con regole applicative e con oggetti del dominio.

## Verifiche minime del DAO

Per ogni metodo DAO conviene verificare almeno:

- caso positivo;
- ricerca con ID esistente;
- ricerca con ID inesistente;
- inserimento di un dato valido;
- aggiornamento che modifica realmente il database;
- eliminazione controllata;
- gestione degli errori JDBC tramite `DaoException`.

## DAO e testabilità

Se un service dipende da un'interfaccia:

```java
private final EdizioneCorsoDao edizioneDao;
```

in futuro sarà possibile sostituire il DAO JDBC con una versione finta o in memoria per testare la logica applicativa senza collegarsi al database.

Esempio concettuale:

```java
EdizioneCorsoDao dao = new InMemoryEdizioneCorsoDao();
EdizioneService service = new EdizioneService(corsoDao, docenteDao, dao);
```

Questa è la stessa idea che ritroveremo con la Dependency Injection di Spring: il service riceve ciò che gli serve, invece di crearlo direttamente.

## Dove vengono applicate le regole

Il DAO non decide se un'operazione è valida dal punto di vista applicativo. Questa responsabilità appartiene al service.

Esempio:

```java
public boolean prenotaPosto(int edizioneId) {
    EdizioneCorso edizione = edizioneDao.findById(edizioneId)
            .orElseThrow(() -> new IllegalArgumentException("Edizione inesistente"));

    if (!edizione.isAttiva()) {
        throw new IllegalStateException("L'edizione non è attiva");
    }

    if (edizione.getPostiDisponibili() <= 0) {
        throw new IllegalStateException("Non ci sono posti disponibili");
    }

    return edizioneDao.updatePostiDisponibili(edizioneId, edizione.getPostiDisponibili() - 1);
}
```

In questo esempio il service decide se la prenotazione può essere fatta. Il DAO esegue soltanto l'aggiornamento sul database.

## Collegamento con le transazioni

In questa UD ci limitiamo a riconoscere che alcune operazioni possono richiedere coerenza tra più modifiche. Non implementiamo ancora un caso transazionale completo. Il laboratorio dedicato sarà collocato più avanti, quando useremo Spring e potremo vedere `@Transactional` nel livello service.
