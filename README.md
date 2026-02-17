# EXPO-Custom-Agent-PLSQL-Oracle

**Guida Completa su GitHub Custom Agents e Integrazione con Oracle Database / PL-SQL**

[![Oracle](https://img.shields.io/badge/Oracle-Database-red?logo=oracle)](https://www.oracle.com/database/)
[![PL/SQL](https://img.shields.io/badge/PL%2FSQL-Expert-blue)](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/)
[![GitHub Copilot](https://img.shields.io/badge/GitHub-Copilot-purple?logo=github)](https://github.com/features/copilot)

Questo repository contiene una spiegazione completa e dettagliata su come creare GitHub Custom Agents e come integrarli con Oracle Database per lo sviluppo PL/SQL.

## 📚 Contenuti del Repository

### [Sezione 1: GitHub Custom Agents](docs/section1-custom-agents/)
Guida completa alla creazione e configurazione di GitHub Custom Agents.

**Argomenti Trattati:**
- ✅ Cos'è un GitHub Custom Agent
- ✅ Struttura e configurazione standard
- ✅ File `.md` con YAML frontmatter
- ✅ File di configurazione JSON
- ✅ Organizzazione della cartella `.github/agents`
- ✅ Sezione KNOWLEDGE con alberatura appropriata
- ✅ Best practices e pattern di utilizzo

**Contenuti:**
- [📖 Guida Principale](docs/section1-custom-agents/README.md)
- [🔧 Esempio Base: Code Reviewer](docs/section1-custom-agents/examples/basic-agent.md)
- [⚙️ Esempio Avanzato: DevOps Assistant](docs/section1-custom-agents/examples/advanced-agent.md)
- [📝 Configurazioni JSON](docs/section1-custom-agents/configuration/)
- [📚 Knowledge Base](docs/section1-custom-agents/knowledge/)

### [Sezione 2: Integrazione Oracle Database](docs/section2-oracle-integration/)
Guida all'integrazione di GitHub Custom Agents con Oracle Database usando Oracle SQL Developer Extension e DBCODE Extension.

**Argomenti Trattati:**
- ✅ Configurazione Oracle SQL Developer Extension
- ✅ Configurazione DBCODE Extension
- ✅ Integrazione con database Oracle
- ✅ Sviluppo query SQL ottimizzate
- ✅ Sviluppo codice PL/SQL professionale
- ✅ Best practices Oracle Database

**Contenuti:**
- [📖 Guida Integrazione Oracle](docs/section2-oracle-integration/README.md)
- [🔍 Esempio 1: Oracle Query Assistant](docs/section2-oracle-integration/examples/oracle-query-assistant.md) - Agent per query SQL
- [💻 Esempio 2: PL/SQL Code Expert](docs/section2-oracle-integration/examples/oracle-plsql-expert.md) - Agent per sviluppo PL/SQL
- [⚙️ Configurazioni Oracle](docs/section2-oracle-integration/configurations/)

## 🚀 Quick Start

### 1. Esplorare la Documentazione

Inizia dalla [Sezione 1](docs/section1-custom-agents/README.md) per comprendere i fondamenti dei GitHub Custom Agents, poi passa alla [Sezione 2](docs/section2-oracle-integration/README.md) per l'integrazione con Oracle.

### 2. Agent Configurati

Questo repository include due agent Oracle già configurati nella cartella `.github/agents/`:

1. **Oracle Query Assistant** - Assistenza per query SQL
   - File: [.github/agents/oracle-query-assistant.md](.github/agents/oracle-query-assistant.md)
   - Focus: Generazione e ottimizzazione query SQL
   
2. **Oracle PL/SQL Expert** - Sviluppo codice PL/SQL
   - File: [.github/agents/oracle-plsql-expert.md](.github/agents/oracle-plsql-expert.md)
   - Focus: Stored procedures, functions, packages, triggers

### 3. Knowledge Base

La cartella [.github/agents/knowledge](.github/agents/knowledge/) contiene:
- Query SQL comuni e pattern
- Best practices PL/SQL
- Esempi di codice riutilizzabili
- Guide di ottimizzazione performance

## 📋 Prerequisiti

### Per GitHub Custom Agents
- Accesso a GitHub Copilot
- Repository GitHub
- VS Code (consigliato)

### Per Integrazione Oracle
- Oracle Database (11g o superiore)
- VS Code Extensions:
  - Oracle SQL Developer Extension
  - DBCODE Extension
- Credenziali database Oracle

## 📖 Struttura del Repository

```
EXPO-Custom-Agent-PLSQL-Oracle/
├── .github/
│   └── agents/                          # Custom Agents configurati
│       ├── oracle-query-assistant.md    # Agent per query SQL
│       ├── oracle-plsql-expert.md       # Agent per PL/SQL
│       └── knowledge/                   # Knowledge base
│           ├── common-queries.md        # Query comuni
│           └── plsql-best-practices.md  # Best practices
├── docs/
│   ├── section1-custom-agents/          # Sezione 1: Guida Custom Agents
│   │   ├── README.md                    # Guida principale
│   │   ├── examples/                    # Esempi di agent
│   │   │   ├── basic-agent.md          # Esempio base
│   │   │   └── advanced-agent.md       # Esempio avanzato
│   │   ├── configuration/               # Configurazioni JSON
│   │   │   ├── basic-config.json
│   │   │   └── advanced-config.json
│   │   └── knowledge/                   # Knowledge base esempi
│   │       └── best-practices.md
│   └── section2-oracle-integration/     # Sezione 2: Oracle Integration
│       ├── README.md                    # Guida integrazione Oracle
│       ├── examples/                    # Esempi Oracle
│       │   ├── oracle-query-assistant.md
│       │   └── oracle-plsql-expert.md
│       └── configurations/              # Configurazioni Oracle
│           ├── oracle-query-assistant-config.json
│           └── oracle-plsql-expert-config.json
└── README.md                            # Questo file
```

## 🎯 Casi d'Uso

### Sezione 1: Custom Agents Generici
- Code review automatizzato
- Assistenza DevOps e CI/CD
- Generazione documentazione
- Analisi sicurezza codice
- Best practices enforcement

### Sezione 2: Oracle Database
- Generazione query SQL ottimizzate
- Sviluppo stored procedures e functions
- Creazione packages PL/SQL
- Implementazione triggers
- Analisi e ottimizzazione performance
- Gestione eccezioni robusta
- Unit testing codice PL/SQL

## 💡 Esempi Pratici

### Esempio 1: Query SQL con l'Agent

Chiedi all'Oracle Query Assistant:
```
"Crea una query per trovare i top 10 clienti per fatturato negli ultimi 6 mesi"
```

L'agent genererà una query SQL ottimizzata con:
- JOIN appropriati
- Aggregazioni corrette
- Indici suggeriti
- Execution plan

### Esempio 2: Stored Procedure con l'Agent

Chiedi all'Oracle PL/SQL Expert:
```
"Crea una stored procedure per processare ordini con gestione transazionale completa"
```

L'agent creerà:
- Procedure con exception handling
- Gestione transazioni (COMMIT/ROLLBACK)
- Validazione input
- Logging audit
- Commenti e documentazione

## 🔧 Configurazione Extensions

### Oracle SQL Developer Extension

```json
{
  "oracle.sqlDeveloper.connections": [
    {
      "name": "Development",
      "host": "${DB_HOST}",
      "port": 1521,
      "serviceName": "${DB_SERVICE}",
      "username": "${DB_USER}",
      "password": "${DB_PASSWORD}"
    }
  ]
}
```

### DBCODE Extension

```json
{
  "dbcode.enableIntelliSense": true,
  "dbcode.enableCodeCompletion": true,
  "dbcode.formatOnSave": true
}
```

## 📚 Risorse

### Documentazione Ufficiale
- [GitHub Copilot](https://docs.github.com/en/copilot)
- [Oracle Database Documentation](https://docs.oracle.com/en/database/)
- [Oracle PL/SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/)

### Extensions
- [Oracle SQL Developer Extension](https://marketplace.visualstudio.com/items?itemName=Oracle.sql-developer)
- [DBCODE Extension](https://marketplace.visualstudio.com/items?itemName=giacomelli.dbcode)

## 🤝 Contribuire

Questo repository è progettato come guida educativa. Per suggerimenti o miglioramenti:
1. Apri una issue per discutere le modifiche proposte
2. Fork il repository
3. Crea un branch per le tue modifiche
4. Apri una Pull Request

## 📝 Licenza

Questo progetto è distribuito a scopo educativo e dimostrativo.

## ✨ Caratteristiche Principali

- ✅ **Documentazione Completa**: Guide dettagliate passo-passo
- ✅ **Esempi Pratici**: 4 esempi completi e funzionanti
- ✅ **Best Practices**: Pattern e convenzioni standard del settore
- ✅ **Knowledge Base**: Repository di codice riutilizzabile
- ✅ **Configurazioni Pronte**: File JSON configurati
- ✅ **In Italiano**: Tutta la documentazione in lingua italiana
- ✅ **Oracle Focus**: Specializzazione su Oracle Database e PL/SQL

## 🎓 Percorso di Apprendimento Consigliato

1. **Inizia Qui**: Leggi questo README
2. **Sezione 1**: Impara i [fondamenti dei Custom Agents](docs/section1-custom-agents/README.md)
3. **Esempi Base**: Studia l'[esempio base](docs/section1-custom-agents/examples/basic-agent.md)
4. **Esempi Avanzati**: Approfondisci con l'[esempio avanzato](docs/section1-custom-agents/examples/advanced-agent.md)
5. **Sezione 2**: Passa all'[integrazione Oracle](docs/section2-oracle-integration/README.md)
6. **Oracle SQL**: Esplora l'[Oracle Query Assistant](docs/section2-oracle-integration/examples/oracle-query-assistant.md)
7. **PL/SQL**: Approfondisci il [PL/SQL Expert](docs/section2-oracle-integration/examples/oracle-plsql-expert.md)
8. **Pratica**: Usa gli agent configurati nel tuo progetto

---

**Creato con ❤️ per la community di sviluppatori Oracle e GitHub Copilot**

*Ultimo aggiornamento: Febbraio 2024*
