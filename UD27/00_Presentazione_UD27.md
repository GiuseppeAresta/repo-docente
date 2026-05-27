# UD27 - DAO, Service e accesso ai dati con JDBC

## Scenario

Finora abbiamo collegato Java a MariaDB/MySQL usando JDBC e, nella UD26, abbiamo iniziato a organizzare il progetto con Maven. In questa unità facciamo un passo ulteriore: il codice JDBC non deve rimanere sparso nel `main` o mescolato alla logica applicativa.

L'obiettivo è costruire una piccola applicazione Java organizzata in livelli, dove ogni parte ha una responsabilità chiara:

- il `model` rappresenta i dati del dominio;
- il `dao` definisce le operazioni di accesso ai dati;
- il `dao.jdbc` contiene l'implementazione basata su JDBC;
- il `service` contiene le regole applicative;
- l'`app` avvia il programma e coordina la demo.

Questa struttura prepara direttamente il passaggio alle API, a JPA e a Spring Boot.

## Problema da risolvere

Un'applicazione può funzionare anche se contiene query SQL direttamente nel `main`, ma diventa rapidamente difficile da modificare. Il problema non è solo tecnico: quando SQL, validazione, mapping e stampa sono nello stesso punto, diventa difficile capire dove intervenire.

Esempio da evitare:

```java
public class GestioneCorsiApp {
    public static void main(String[] args) throws SQLException {
        Connection conn = DriverManager.getConnection(...);
        PreparedStatement ps = conn.prepareStatement("SELECT * FROM corsi");
        ResultSet rs = ps.executeQuery();

        while (rs.next()) {
            System.out.println(rs.getString("titolo"));
        }
    }
}
```

In questo esempio il `main` sta facendo troppe cose: apre la connessione, conosce SQL, legge il `ResultSet`, decide cosa stampare e gestisce il flusso applicativo.

## Obiettivo generale

In questa UD riorganizziamo il codice usando il pattern DAO e un service applicativo.

Il service non deve conoscere i dettagli JDBC. Deve lavorare con un'interfaccia:

```java
public class EdizioneService {
    private final EdizioneCorsoDao edizioneDao;

    public EdizioneService(EdizioneCorsoDao edizioneDao) {
        this.edizioneDao = edizioneDao;
    }
}
```

In questo modo il service dipende da un contratto, non da una classe JDBC specifica. È un'applicazione concreta di interfacce, polimorfismo e separazione delle responsabilità.

## Risultati attesi

Al termine della UD27 saremo in grado di:

1. spiegare il ruolo del pattern DAO in una piccola applicazione Java;
2. distinguere tra interfaccia DAO e implementazione JDBC;
3. progettare metodi DAO coerenti con le operazioni applicative;
4. implementare query `SELECT`, `INSERT`, `UPDATE` e `DELETE` in classi dedicate;
5. trasformare righe di un `ResultSet` in oggetti Java tramite mapping manuale;
6. usare `PreparedStatement` per query parametrizzate;
7. gestire gli errori JDBC con una eccezione applicativa dedicata;
8. usare un service che dipende da interfacce DAO;
9. riconoscere il collegamento tra DAO JDBC, repository, JPA e Spring Data.

## Agenda indicativa

| Fase | Durata | Attività |
|---|---:|---|
| Apertura | 20 min | Ripresa da Maven/JDBC e problema del codice SQL disperso |
| Teoria guidata | 70 min | DAO, interfacce, implementazioni JDBC, service |
| Approfondimento pratico | 40 min | Mapping manuale, eccezioni e verifiche |
| Laboratorio guidato | 110 min | Refactoring di un accesso JDBC verso DAO |
| Laboratorio autonomo | 180 min | Gestione edizioni corso con DAO JDBC |
| Chiusura | 60 min | Review, evidence e collegamento a JPA/Spring |

## Nota sulle transazioni

In questa UD non sviluppiamo ancora un laboratorio dedicato alle transazioni. Manteniamo il focus su DAO, service, responsabilità dei livelli e accesso ai dati. Il tema sarà ripreso più avanti in modo mirato nel contesto Spring, dove `@Transactional` avrà un significato applicativo più chiaro.

## Collegamento con le prossime UD

La UD27 prepara direttamente:

- API REST e DTO;
- JPA/Hibernate;
- Spring Boot;
- Dependency Injection;
- Spring Data JPA.

Quando useremo un repository Spring, ritroveremo lo stesso problema affrontato qui: separare la logica applicativa dall'accesso ai dati.
