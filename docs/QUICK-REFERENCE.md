# Quick Reference Guide - GitHub Custom Agents & Oracle Integration

## Navigazione Rapida

### 📚 Documentazione Principale
- [README Principale](../README.md)
- [Sezione 1: Custom Agents](section1-custom-agents/README.md)
- [Sezione 2: Oracle Integration](section2-oracle-integration/README.md)

### 🎯 Esempi Pratici

#### Sezione 1: Custom Agents Generici
1. **Code Reviewer** - [Esempio Base](section1-custom-agents/examples/basic-agent.md)
   - Revisione codice
   - Best practices checking
   - Security review

2. **DevOps Assistant** - [Esempio Avanzato](section1-custom-agents/examples/advanced-agent.md)
   - Pipeline CI/CD
   - Infrastructure as Code
   - Kubernetes deployments

#### Sezione 2: Oracle Database
1. **Oracle Query Assistant** - [Esempio 1](section2-oracle-integration/examples/oracle-query-assistant.md)
   - Generazione query SQL
   - Ottimizzazione query
   - Analisi execution plans
   - Suggerimenti indici

2. **PL/SQL Code Expert** - [Esempio 2](section2-oracle-integration/examples/oracle-plsql-expert.md)
   - Stored procedures
   - Functions e packages
   - Triggers
   - Bulk operations
   - Exception handling
   - Unit testing

### ⚙️ Configurazioni

#### JSON Configuration Files
- [Basic Config](section1-custom-agents/configuration/basic-config.json)
- [Advanced Config](section1-custom-agents/configuration/advanced-config.json)
- [Oracle Query Config](section2-oracle-integration/configurations/oracle-query-assistant-config.json)
- [PL/SQL Expert Config](section2-oracle-integration/configurations/oracle-plsql-expert-config.json)

### 📖 Knowledge Base

#### Generale
- [Best Practices](section1-custom-agents/knowledge/best-practices.md)

#### Oracle Specific
- [Common Queries](../.github/agents/knowledge/common-queries.md)
- [PL/SQL Best Practices](../.github/agents/knowledge/plsql-best-practices.md)

### 🤖 Agent Configurati

I seguenti agent sono già configurati e pronti all'uso in `.github/agents/`:

1. **oracle-query-assistant.md** - SQL Query helper
2. **oracle-plsql-expert.md** - PL/SQL development expert

## Comandi Rapidi

### Installazione Extensions VS Code

```bash
# Oracle SQL Developer Extension
code --install-extension oracle.sql-developer

# DBCODE Extension
code --install-extension giacomelli.dbcode
```

### Struttura Directory Raccomandata

```
your-project/
├── .github/
│   └── agents/
│       ├── your-agent.md              # Agent configuration
│       ├── your-agent-config.json     # Optional JSON config
│       └── knowledge/                 # Knowledge base
│           ├── domain-knowledge.md
│           └── examples.md
```

## Template Rapidi

### Agent File Template

```markdown
---
name: "Your Agent Name"
description: "Brief description"
version: "1.0.0"
author: "Your Name"
tags:
  - tag1
  - tag2
capabilities:
  - capability1
  - capability2
---

# Your Agent Name

## Scopo
What this agent does...

## Come Usarmi
How to use this agent...

## Esempi
Examples...
```

### JSON Config Template

```json
{
  "agent": {
    "name": "Agent Name",
    "version": "1.0.0",
    "description": "Description"
  },
  "settings": {
    "temperature": 0.7,
    "max_tokens": 2000
  },
  "knowledge_base": {
    "sources": [
      "knowledge/file1.md"
    ]
  }
}
```

## Casi d'Uso Comuni

### Per Query SQL
```
"Crea una query per trovare i clienti con ordini negli ultimi 30 giorni"
"Ottimizza questa query" + [incolla query]
"Suggerisci indici per migliorare performance"
```

### Per PL/SQL
```
"Crea una stored procedure per processare ordini"
"Implementa un package per gestione clienti"
"Scrivi un trigger per audit automatico"
"Converti questo loop in bulk operation"
```

### Per Code Review
```
"Rivedi questo codice per sicurezza"
"Suggerisci miglioramenti per questa funzione"
"Identifica problemi di performance"
```

## Troubleshooting

### Agent Non Funziona
1. Verifica YAML frontmatter corretto
2. Controlla posizione file in `.github/agents/`
3. Riavvia VS Code
4. Controlla log GitHub Copilot

### Oracle Extension Problemi
1. Verifica credenziali database
2. Controlla network/firewall
3. Verifica service name Oracle
4. Riavvia extension

### Performance Issues
1. Riduci dimensione knowledge base
2. Usa pagination per grandi risultati
3. Implementa caching se possibile
4. Ottimizza query database

## Best Practices Checklist

- [ ] YAML frontmatter completo e corretto
- [ ] Description chiara e concisa
- [ ] Tags appropriati per categorizzazione
- [ ] Knowledge base organizzata
- [ ] Esempi pratici inclusi
- [ ] Error handling implementato
- [ ] Sicurezza considerata
- [ ] Performance ottimizzata
- [ ] Documentazione completa
- [ ] Testing effettuato

## Link Utili

### Documentazione Ufficiale
- [GitHub Copilot Docs](https://docs.github.com/en/copilot)
- [Oracle Database Docs](https://docs.oracle.com/en/database/)
- [PL/SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/)

### Extensions
- [Oracle SQL Developer](https://marketplace.visualstudio.com/items?itemName=Oracle.sql-developer)
- [DBCODE](https://marketplace.visualstudio.com/items?itemName=giacomelli.dbcode)

---

**💡 Suggerimento**: Inizia con gli esempi base, poi passa agli avanzati quando ti senti a tuo agio con i concetti fondamentali.
