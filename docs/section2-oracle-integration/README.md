# Sezione 2: GitHub Custom Agent - Integrazione con Oracle Database

## Indice
1. [Introduzione](#introduzione)
2. [Prerequisiti](#prerequisiti)
3. [Estensioni Necessarie](#estensioni-necessarie)
4. [Configurazione Base](#configurazione-base)
5. [Esempi Pratici](#esempi-pratici)
6. [Best Practices](#best-practices)

## Introduzione

Questa sezione illustra come creare un GitHub Custom Agent che si interfaccia con Oracle Database, utilizzando le estensioni Oracle SQL Developer e DBCODE per fornire assistenza intelligente nella gestione di database Oracle e nello sviluppo di codice PL/SQL.

## Prerequisiti

### Software Richiesto

1. **Oracle Database** (11g o superiore)
2. **Oracle SQL Developer** (Versione 20.x o superiore)
3. **VS Code** con estensioni:
   - Oracle SQL Developer Extension
   - DBCODE Extension
   - GitHub Copilot

### Conoscenze Richieste

- Familiarità con SQL e PL/SQL
- Comprensione base di Oracle Database
- Conoscenza di GitHub Custom Agents (vedi [Sezione 1](../section1-custom-agents/README.md))

## Estensioni Necessarie

### 1. Oracle SQL Developer Extension

**Installazione:**
```bash
code --install-extension oracle.sql-developer
```

**Configurazione:**
```json
{
  "oracle.sqlDeveloper.connections": [
    {
      "name": "Production DB",
      "host": "${DB_HOST}",
      "port": 1521,
      "serviceName": "${DB_SERVICE}",
      "username": "${DB_USER}",
      "password": "${DB_PASSWORD}"
    }
  ],
  "oracle.sqlDeveloper.defaultConnection": "Production DB"
}
```

**Funzionalità:**
- Connessione diretta a Oracle Database
- Esecuzione di query SQL e PL/SQL
- Visualizzazione struttura database
- Debugging PL/SQL
- Code formatting e syntax highlighting

### 2. DBCODE Extension

**Installazione:**
```bash
code --install-extension giacomelli.dbcode
```

**Configurazione:**
```json
{
  "dbcode.connections": [
    {
      "name": "oracle-dev",
      "driver": "oracle",
      "server": "${DB_HOST}",
      "port": 1521,
      "database": "${DB_SERVICE}",
      "username": "${DB_USER}",
      "password": "${DB_PASSWORD}"
    }
  ],
  "dbcode.defaultConnection": "oracle-dev",
  "dbcode.enableIntelliSense": true,
  "dbcode.enableCodeCompletion": true
}
```

**Funzionalità:**
- IntelliSense per tabelle e colonne
- Auto-completion per SQL/PL-SQL
- Query history
- Results visualization
- Export data capabilities

## Configurazione Base

### Struttura Directory per Agent Oracle

```
.github/
└── agents/
    ├── oracle-query-assistant.md
    ├── oracle-plsql-expert.md
    ├── oracle-config.json
    └── knowledge/
        ├── oracle-basics/
        │   ├── sql-fundamentals.md
        │   ├── plsql-fundamentals.md
        │   └── database-objects.md
        ├── oracle-advanced/
        │   ├── performance-tuning.md
        │   ├── advanced-plsql.md
        │   └── security-best-practices.md
        └── examples/
            ├── common-queries.md
            ├── stored-procedures.md
            └── triggers-examples.md
```

### File di Configurazione Base

**oracle-config.json:**
```json
{
  "agent": {
    "name": "Oracle Database Assistant",
    "version": "1.0.0",
    "description": "Agent specializzato in Oracle Database e PL/SQL",
    "type": "database"
  },
  "settings": {
    "temperature": 0.3,
    "max_tokens": 4000,
    "response_format": "markdown"
  },
  "database": {
    "type": "oracle",
    "version": "19c",
    "features": [
      "SQL",
      "PL/SQL",
      "stored_procedures",
      "triggers",
      "packages",
      "functions"
    ]
  },
  "tools": [
    {
      "name": "oracle-sql-developer",
      "enabled": true,
      "config": {
        "auto_format": true,
        "syntax_check": true
      }
    },
    {
      "name": "dbcode",
      "enabled": true,
      "config": {
        "intellisense": true,
        "auto_complete": true
      }
    }
  ],
  "knowledge_base": {
    "sources": [
      "knowledge/oracle-basics/",
      "knowledge/oracle-advanced/",
      "knowledge/examples/"
    ],
    "update_frequency": "weekly"
  }
}
```

## Esempi Pratici

Questa sezione contiene due esempi completi di GitHub Custom Agents per Oracle Database:

### Esempio 1: Oracle Query Assistant
Un agent base per assistenza nelle query SQL e gestione database.

📄 [Vedi Esempio Completo](examples/oracle-query-assistant.md)

**Caratteristiche:**
- Generazione query SQL ottimizzate
- Suggerimenti per indici
- Analisi performance query
- Conversione da linguaggio naturale a SQL

### Esempio 2: PL/SQL Code Expert
Un agent avanzato per sviluppo e analisi di codice PL/SQL.

📄 [Vedi Esempio Completo](examples/oracle-plsql-expert.md)

**Caratteristiche:**
- Creazione stored procedures e functions
- Analisi e ottimizzazione codice PL/SQL
- Gestione eccezioni
- Best practices e security
- Testing e debugging

## Best Practices

### 1. Sicurezza

✅ **DO:**
- Usa variabili d'ambiente per credenziali
- Implementa bind variables in query
- Valida sempre input utente
- Usa principio di least privilege

❌ **DON'T:**
- Hardcode password in configurazioni
- Usare concatenazione stringhe in SQL (SQL injection)
- Esporre informazioni sensibili in logs

### 2. Performance

✅ **DO:**
- Usa explain plan per analisi query
- Implementa indici appropriati
- Usa bulk operations per grandi dataset
- Cache risultati frequenti

❌ **DON'T:**
- Fare SELECT * quando non necessario
- Usare funzioni in WHERE clause su colonne indicizzate
- Ignorare execution plans

### 3. Manutenibilità

✅ **DO:**
- Documenta codice PL/SQL
- Usa naming conventions consistenti
- Implementa error handling robusto
- Versiona stored procedures

❌ **DON'T:**
- Creare stored procedures monolitiche
- Ignorare gestione eccezioni
- Usare magic numbers

## Integrazione con GitHub Workflow

### GitHub Actions con Oracle Database

Esempio di workflow per testare codice PL/SQL:

```yaml
name: Oracle PL/SQL Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      oracle:
        image: gvenzl/oracle-xe:21-slim
        env:
          ORACLE_PASSWORD: ${{ secrets.ORACLE_PASSWORD }}
        ports:
          - 1521:1521
        options: >-
          --health-cmd healthcheck.sh
          --health-interval 10s
          --health-timeout 5s
          --health-retries 10
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Oracle Client
        run: |
          sudo apt-get update
          sudo apt-get install -y alien libaio1
          wget https://download.oracle.com/otn_software/linux/instantclient/oracle-instantclient-basic-linuxx64.rpm
          sudo alien -i oracle-instantclient*.rpm
      
      - name: Run PL/SQL Tests
        env:
          ORACLE_HOST: localhost
          ORACLE_PORT: 1521
          ORACLE_SERVICE: XEPDB1
          ORACLE_USER: system
          ORACLE_PASSWORD: ${{ secrets.ORACLE_PASSWORD }}
        run: |
          sqlplus -S ${ORACLE_USER}/${ORACLE_PASSWORD}@//${ORACLE_HOST}:${ORACLE_PORT}/${ORACLE_SERVICE} @tests/run_all_tests.sql
```

## Troubleshooting

### Problemi Comuni

1. **Connessione Database Fallita**
   - Verifica credenziali
   - Controlla firewall/network
   - Verifica service name

2. **Extension Non Funziona**
   - Riavvia VS Code
   - Verifica versione compatibile
   - Controlla logs estensione

3. **Performance Scadente**
   - Analizza execution plan
   - Controlla statistiche database
   - Valuta indici mancanti

## Risorse Aggiuntive

- [Oracle SQL Developer Documentation](https://docs.oracle.com/en/database/oracle/sql-developer/)
- [Oracle PL/SQL Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/)
- [DBCODE Extension Guide](https://marketplace.visualstudio.com/items?itemName=giacomelli.dbcode)

---

**Prossimi Step:**
1. Esplora [Esempio 1: Oracle Query Assistant](examples/oracle-query-assistant.md)
2. Esplora [Esempio 2: PL/SQL Code Expert](examples/oracle-plsql-expert.md)
3. Consulta le [Configurazioni di Esempio](configurations/)
