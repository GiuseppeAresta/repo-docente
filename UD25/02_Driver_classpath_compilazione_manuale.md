# Driver JDBC, classpath e compilazione manuale

## 1. Struttura del progetto senza Maven

In questa UD useremo una struttura semplice:

```text
UD25_catalogo_corsi_jdbc_senza_maven/
  lib/
    mariadb-java-client-3.5.3.jar
  src/
    corso/
      ud25/
        corsi/
          Main.java
  sql/
  docs/
```

La cartella `lib/` contiene il driver JDBC.

La cartella `out/` conterrà i file `.class` generati dalla compilazione.

## 2. Preparazione del driver JDBC

### Opzione A - Driver fornito dal docente

Il docente può fornire il file:

```text
mariadb-java-client-3.5.3.jar
```

Copiare il file in:

```text
lib/
```

La struttura attesa è:

```text
lib/mariadb-java-client-3.5.3.jar
```

### Opzione B - Download manuale su Windows PowerShell

Dalla root del progetto:

```powershell
New-Item -ItemType Directory -Force lib
Invoke-WebRequest `
  -Uri "https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/3.5.3/mariadb-java-client-3.5.3.jar" `
  -OutFile ".\lib\mariadb-java-client-3.5.3.jar"
```

Verificare la presenza del file:

```powershell
Get-ChildItem .\lib\mariadb-java-client-3.5.3.jar
```

### Opzione C - Download manuale su Linux/macOS

Dalla root del progetto:

```bash
mkdir -p lib
curl -L \
  "https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/3.5.3/mariadb-java-client-3.5.3.jar" \
  -o "lib/mariadb-java-client-3.5.3.jar"
```

Verificare la presenza del file:

```bash
ls -l lib/mariadb-java-client-3.5.3.jar
```

Se `curl` non è disponibile, usare `wget`:

```bash
mkdir -p lib
wget \
  "https://repo1.maven.org/maven2/org/mariadb/jdbc/mariadb-java-client/3.5.3/mariadb-java-client-3.5.3.jar" \
  -O "lib/mariadb-java-client-3.5.3.jar"
```

## 3. Compilazione Linux/macOS

Dalla cartella principale del progetto:

```bash
mkdir -p out
javac -encoding UTF-8 -cp "lib/mariadb-java-client-3.5.3.jar" -d out $(find src -name "*.java")
```

## 4. Esecuzione Linux/macOS

```bash
java -cp "out:lib/mariadb-java-client-3.5.3.jar" corso.ud25.corsi.Main
```

Il separatore del classpath su Linux/macOS è:

```text
:
```

## 5. Compilazione Windows PowerShell

Dalla cartella principale del progetto:

```powershell
New-Item -ItemType Directory -Force out
$sources = Get-ChildItem -Recurse src -Filter *.java | ForEach-Object { $_.FullName }
javac -encoding UTF-8 -cp "lib\mariadb-java-client-3.5.3.jar" -d out $sources
```

## 6. Esecuzione Windows PowerShell

```powershell
java -cp "out;lib\mariadb-java-client-3.5.3.jar" corso.ud25.corsi.Main
```

Il separatore del classpath su Windows è:

```text
;
```

## 7. Errori frequenti

| Errore | Possibile causa | Correzione |
|---|---|---|
| `No suitable driver found` | Driver non presente nel classpath di esecuzione | Controllare il comando `java -cp` |
| `ClassNotFoundException` | Nome jar errato o percorso errato | Verificare la cartella `lib/` |
| `Connection refused` | DBMS non avviato o porta errata | Verificare MariaDB/MySQL |
| `Access denied for user` | Utente/password errati | Controllare credenziali |
| `Unknown database` | Database non creato | Eseguire lo script SQL |
| `Table doesn't exist` | Script di schema non eseguito | Eseguire lo script di creazione tabella |

## 8. Perché il classpath è importante

Il classpath indica alla JVM dove cercare:

- classi compilate del progetto;
- librerie esterne;
- driver JDBC.

In questa UD il classpath viene scritto manualmente. Nella UD26 sarà Maven a gestire struttura, dipendenze e build.
