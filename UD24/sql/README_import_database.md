# Importazione database `modelli_classici_it`

Da terminale, posizionarsi nella cartella principale della UD24bis ed eseguire:

```bash
mysql -u root -p < sql/2026-05-20_modelli_classici_it.sql
```

Poi verificare:

```sql
USE modelli_classici_it;
SHOW TABLES;
```
