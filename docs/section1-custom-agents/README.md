# Sezione 1: GitHub Custom Agents - Guida Completa

## Indice
1. [Introduzione](#introduzione)
2. [Cos'è un GitHub Custom Agent](#cosè-un-github-custom-agent)
3. [Struttura e Configurazione](#struttura-e-configurazione)
4. [File di Configurazione](#file-di-configurazione)
5. [Sezione Knowledge](#sezione-knowledge)
6. [Esempi Pratici](#esempi-pratici)

## Introduzione

Questa sezione illustra come creare e configurare un GitHub Custom Agent. Un Custom Agent è uno strumento potente che permette di estendere le funzionalità di GitHub Copilot con comportamenti personalizzati e conoscenze specifiche del dominio.

## Cos'è un GitHub Custom Agent

Un GitHub Custom Agent è un agente AI personalizzato che può essere configurato per:
- Fornire assistenza specializzata su argomenti specifici
- Integrare conoscenze di dominio specifiche
- Automatizzare task ripetitivi
- Interagire con strumenti e API esterni
- Guidare gli sviluppatori con best practices aziendali

## Struttura e Configurazione

### Posizione dei File

I Custom Agents devono essere posizionati nella directory:
```
.github/agents/
```

### Struttura Directory Consigliata

```
.github/
└── agents/
    ├── agent-name.md          # File di configurazione principale
    ├── agent-config.json      # Configurazione JSON (opzionale)
    └── knowledge/             # Cartella per file di conoscenza
        ├── domain-knowledge.md
        ├── api-reference.md
        └── best-practices.md
```

## File di Configurazione

### File Markdown con YAML Frontmatter

Il file principale di configurazione di un Custom Agent è un file Markdown che inizia con un blocco YAML frontmatter.

**Esempio di struttura base:**

```markdown
---
name: "Nome dell'Agent"
description: "Descrizione breve dell'agent"
version: "1.0.0"
author: "Nome Autore"
tags:
  - tag1
  - tag2
capabilities:
  - capability1
  - capability2
---

# Istruzioni dell'Agent

Qui vanno le istruzioni dettagliate per l'agent...
```

### Campi YAML Frontmatter

| Campo | Tipo | Descrizione |
|-------|------|-------------|
| `name` | string | Nome identificativo dell'agent |
| `description` | string | Descrizione breve dello scopo dell'agent |
| `version` | string | Versione dell'agent (semantic versioning) |
| `author` | string | Autore o team responsabile |
| `tags` | array | Tag per categorizzare l'agent |
| `capabilities` | array | Lista delle capacità dell'agent |

### File JSON di Configurazione

Opzionalmente, è possibile aggiungere un file JSON per configurazioni più complesse:

```json
{
  "agent": {
    "name": "Nome Agent",
    "version": "1.0.0",
    "description": "Descrizione dettagliata"
  },
  "settings": {
    "temperature": 0.7,
    "max_tokens": 2000,
    "response_format": "markdown"
  },
  "tools": [
    {
      "name": "tool_name",
      "enabled": true,
      "config": {}
    }
  ],
  "knowledge_base": {
    "sources": [
      "knowledge/domain-knowledge.md",
      "knowledge/api-reference.md"
    ],
    "update_frequency": "daily"
  }
}
```

## Sezione Knowledge

La sezione **knowledge** è fondamentale per fornire all'agent informazioni specifiche del dominio.

### Struttura della Cartella Knowledge

```
knowledge/
├── README.md              # Indice della knowledge base
├── fundamentals/          # Concetti base
│   ├── introduction.md
│   └── terminology.md
├── guides/               # Guide pratiche
│   ├── getting-started.md
│   └── best-practices.md
├── reference/            # Documentazione di riferimento
│   ├── api-reference.md
│   └── configuration.md
└── examples/             # Esempi di codice
    ├── basic-example.md
    └── advanced-example.md
```

### Best Practices per la Knowledge Base

1. **Organizzazione**: Struttura gerarchica chiara
2. **Formato**: Usa Markdown per massima compatibilità
3. **Aggiornamenti**: Mantieni la documentazione aggiornata
4. **Esempi**: Includi esempi pratici e codice funzionante
5. **Collegamenti**: Usa link interni per navigazione facile

## Esempi Pratici

Per esempi pratici di configurazione, consulta:
- [Esempio Base](examples/basic-agent.md) - Agent semplice per iniziare
- [Esempio Avanzato](examples/advanced-agent.md) - Agent con funzionalità complete
- [Configurazioni](configuration/) - Vari esempi di configurazione

## Risorse Aggiuntive

- [Documentazione ufficiale GitHub Copilot](https://docs.github.com/en/copilot)
- [Best Practices per Custom Agents](knowledge/best-practices.md)
- [Troubleshooting](knowledge/troubleshooting.md)

---

**Prossimo Step**: Vai alla [Sezione 2](../section2-oracle-integration/README.md) per vedere come integrare un Custom Agent con Oracle Database.
