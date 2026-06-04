# Spring Boot, IoC e Dependency Injection

## Obiettivo del file

Questo file introduce i concetti minimi necessari per comprendere come Spring crea e collega i componenti dell'applicazione.

Nelle unità precedenti abbiamo creato manualmente gli oggetti nel `main`.

Esempio semplificato:

```java
CorsoRepository corsoRepository = new CorsoRepository();
CorsoMapper corsoMapper = new CorsoMapper();
CatalogoService catalogoService = new CatalogoService(corsoRepository, corsoMapper);
CorsiHandler corsiHandler = new CorsiHandler(catalogoService);
```

In Spring Boot questa composizione viene gestita dal container Spring.

---

## Che cos'è Spring Boot

Spring Boot è un modo semplificato per creare applicazioni basate su Spring.

Si occupa di:

- avviare l'applicazione;
- configurare automaticamente molte parti comuni;
- integrare server web embedded;
- configurare Jackson per JSON;
- integrare JPA/Hibernate;
- creare e collegare i componenti applicativi.

Spring Boot non sostituisce Java, JPA o il database. Li coordina dentro una applicazione più strutturata.

---

## Inversion of Control

**IoC** significa **Inversion of Control**.

In un'applicazione Java manuale siamo noi a creare gli oggetti:

```java
IscrizioneRepository repository = new IscrizioneRepository();
IscrizioneService service = new IscrizioneService(repository);
```

Con Spring, invece, molte classi vengono create dal container Spring.

Il controllo della creazione degli oggetti passa dall'applicazione al framework.

```text
Prima:
il main crea gli oggetti

Con Spring:
il container crea gli oggetti e li collega
```

---

## Bean Spring

Un **bean** è un oggetto gestito dal container Spring.

Se una classe è riconosciuta da Spring come componente, Spring può:

- crearne un'istanza;
- conservarla nel container;
- passarla ad altre classi che ne hanno bisogno;
- gestirne il ciclo di vita.

Esempio:

```java
@Service
public class IscrizioneService {
}
```

`IscrizioneService` diventa un componente gestito da Spring.

---

## Annotazioni principali dei componenti

| Annotazione | Uso principale |
|---|---|
| `@SpringBootApplication` | classe principale dell'applicazione |
| `@RestController` | classe che gestisce endpoint REST |
| `@Service` | classe che contiene logica applicativa |
| `@Repository` | classe o interfaccia collegata all'accesso ai dati |
| `@Component` | componente generico gestito da Spring |

`@Service`, `@Repository` e `@RestController` sono specializzazioni concettuali di componenti Spring. Servono anche a rendere più chiara la responsabilità della classe.

---

## Dependency Injection

La **Dependency Injection** è il meccanismo con cui Spring fornisce a una classe gli oggetti di cui ha bisogno.

Esempio:

```java
@Service
public class IscrizioneService {

    private final EdizioneRepository edizioneRepository;
    private final IscrizioneRepository iscrizioneRepository;

    public IscrizioneService(
            EdizioneRepository edizioneRepository,
            IscrizioneRepository iscrizioneRepository
    ) {
        this.edizioneRepository = edizioneRepository;
        this.iscrizioneRepository = iscrizioneRepository;
    }
}
```

In questo esempio `IscrizioneService` ha bisogno di due repository.

Non li crea con `new`.

Li riceve nel costruttore.

Spring vede il costruttore, cerca bean compatibili e li passa automaticamente.

---

## Constructor injection

Nel laboratorio useremo soprattutto la **constructor injection**.

Questo stile rende esplicite le dipendenze della classe.

```java
@Service
public class CatalogoService {

    private final CorsoRepository corsoRepository;
    private final CorsoMapper corsoMapper;

    public CatalogoService(CorsoRepository corsoRepository, CorsoMapper corsoMapper) {
        this.corsoRepository = corsoRepository;
        this.corsoMapper = corsoMapper;
    }
}
```

Vantaggi:

- le dipendenze sono visibili nel costruttore;
- i campi possono essere `final`;
- la classe è più semplice da testare;
- è chiaro di cosa ha bisogno per funzionare.

---

## Introduzione a `@Autowired`

`@Autowired` è l'annotazione storicamente usata per indicare a Spring dove iniettare una dipendenza.

Può essere usata:

- su un costruttore;
- su un setter;
- su un campo.

### Constructor injection con `@Autowired`

```java
@Service
public class CatalogoService {

    private final CorsoRepository corsoRepository;

    @Autowired
    public CatalogoService(CorsoRepository corsoRepository) {
        this.corsoRepository = corsoRepository;
    }
}
```

In questo caso `@Autowired` dice a Spring di usare quel costruttore per fornire la dipendenza.

### Constructor injection senza `@Autowired`

Se una classe ha un solo costruttore, Spring può usarlo anche senza `@Autowired`.

```java
@Service
public class CatalogoService {

    private final CorsoRepository corsoRepository;

    public CatalogoService(CorsoRepository corsoRepository) {
        this.corsoRepository = corsoRepository;
    }
}
```

Nel laboratorio useremo questa forma.

---

## Field injection

È possibile trovare codice scritto così:

```java
@Service
public class CatalogoService {

    @Autowired
    private CorsoRepository corsoRepository;
}
```

Questa forma funziona, ma nel laboratorio non la useremo come stile principale.

Motivi:

- la dipendenza è meno esplicita;
- il campo non può essere dichiarato `final`;
- la classe risulta meno chiara da testare;
- il costruttore non documenta ciò che serve alla classe.

È comunque importante conoscerla, perché può comparire in progetti esistenti.

---

## Confronto tra gli stili

| Stile | Esempio | Uso nel laboratorio |
|---|---|---|
| Field injection | `@Autowired private Repository repo;` | da conoscere, ma non usare come stile principale |
| Constructor injection con `@Autowired` | `@Autowired public Service(Repo repo)` | utile per capire il meccanismo |
| Constructor injection senza `@Autowired` | `public Service(Repo repo)` | stile principale del laboratorio |

---

## Collegamento con le unità precedenti

In UD29 e UD30 la composizione era manuale.

```java
CorsoRepository corsoRepository = new CorsoRepository();
CorsoMapper corsoMapper = new CorsoMapper();
CatalogoService catalogoService = new CatalogoService(corsoRepository, corsoMapper);
```

In Spring Boot:

- `CorsoRepository` è gestito da Spring Data;
- `CorsoMapper` può essere un componente Spring;
- `CatalogoService` viene creato da Spring;
- il controller riceve il service tramite costruttore.

Il risultato è lo stesso dal punto di vista delle collaborazioni, ma la creazione e il collegamento degli oggetti sono gestiti dal container Spring.

---

## Sintesi

| Concetto | Significato |
|---|---|
| IoC | il framework gestisce la creazione dei componenti |
| Bean | oggetto gestito dal container Spring |
| Dependency Injection | Spring fornisce a una classe le dipendenze necessarie |
| `@Autowired` | annotazione che può indicare dove iniettare una dipendenza |
| Constructor injection | dipendenze ricevute nel costruttore |

Nel laboratorio useremo constructor injection senza `@Autowired` quando la classe ha un solo costruttore. Introdurremo comunque `@Autowired` per saperlo riconoscere e interpretare nei progetti esistenti.
