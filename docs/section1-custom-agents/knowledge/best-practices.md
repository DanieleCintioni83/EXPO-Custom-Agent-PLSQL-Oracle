# Best Practices per GitHub Custom Agents

## Indice
1. [Progettazione](#progettazione)
2. [Configurazione](#configurazione)
3. [Knowledge Base](#knowledge-base)
4. [Sicurezza](#sicurezza)
5. [Manutenzione](#manutenzione)

## Progettazione

### Definisci uno Scopo Chiaro

✅ **DO:**
- Crea agent con scopo specifico e ben definito
- Documenta chiaramente le capacità dell'agent
- Limita il dominio di conoscenza ad aree specifiche

❌ **DON'T:**
- Creare agent "tuttofare" troppo generici
- Mescolare responsabilità non correlate
- Lasciare ambiguità nelle istruzioni

### Esempio di Scopo Chiaro

```markdown
---
name: "Python Testing Assistant"
description: "Specializzato nella creazione e revisione di test Python con pytest"
scope: "Testing, pytest, mocking, fixtures, test organization"
---
```

## Configurazione

### YAML Frontmatter

**Best Practices:**

1. **Versioning**: Usa semantic versioning (1.0.0, 1.1.0, 2.0.0)
2. **Tags**: Usa tag descrittivi per categorizzazione
3. **Metadati Completi**: Includi tutti i campi rilevanti

```yaml
---
name: "Nome Descrittivo"
description: "Descrizione concisa ma completa (max 200 caratteri)"
version: "1.0.0"
author: "Team/Persona"
created: "2024-01-15"
updated: "2024-02-10"
tags:
  - categoria-principale
  - sotto-categoria
  - tecnologia
capabilities:
  - capability-1
  - capability-2
dependencies:
  - altro-agent-se-necessario
---
```

### Istruzioni Chiare

**Struttura Consigliata:**

```markdown
# Nome Agent

## Scopo
Breve descrizione dello scopo principale

## Come Usarmi
Spiegazione di come interagire con l'agent

## Cosa Posso Fare
- Capability 1
- Capability 2
- Capability 3

## Cosa NON Posso Fare
- Limitazione 1
- Limitazione 2

## Istruzioni Operative
1. Passo 1
2. Passo 2
3. Passo 3

## Esempi
[Esempi pratici di utilizzo]
```

## Knowledge Base

### Organizzazione

**Struttura Raccomandata:**

```
knowledge/
├── README.md                    # Indice e overview
├── fundamentals/               # Concetti base
│   ├── introduction.md
│   ├── core-concepts.md
│   └── terminology.md
├── guides/                     # Guide how-to
│   ├── getting-started.md
│   ├── common-tasks.md
│   └── advanced-usage.md
├── reference/                  # Documentazione di riferimento
│   ├── api-reference.md
│   ├── configuration.md
│   └── cli-commands.md
├── examples/                   # Esempi di codice
│   ├── basic-examples.md
│   ├── advanced-examples.md
│   └── real-world-cases.md
└── troubleshooting/           # Risoluzione problemi
    ├── common-errors.md
    └── debugging.md
```

### Formato dei Documenti

**Template per Documento Knowledge:**

```markdown
# Titolo del Documento

## Overview
Breve descrizione del contenuto (2-3 frasi)

## Prerequisiti
- Prerequisito 1
- Prerequisito 2

## Contenuto Principale
[Spiegazione dettagliata con esempi]

## Esempi Pratici

### Esempio 1: [Nome]
\`\`\`language
// Codice esempio
\`\`\`

**Spiegazione**: ...

### Esempio 2: [Nome]
...

## Best Practices
- Practice 1
- Practice 2

## Errori Comuni
- Errore 1: Come evitarlo
- Errore 2: Come risolverlo

## Vedi Anche
- [Link ad altri documenti correlati]
```

### Qualità del Contenuto

✅ **DO:**
- Usa esempi pratici e testati
- Mantieni informazioni aggiornate
- Includi spiegazioni del "perché" oltre al "come"
- Usa diagrammi quando appropriato
- Fornisci esempi di codice funzionanti

❌ **DON'T:**
- Lasciare documentazione obsoleta
- Usare esempi non testati
- Essere troppo verboso senza esempi
- Duplicare informazioni tra documenti

## Sicurezza

### Protezione Dati Sensibili

✅ **DO:**
```json
{
  "database": {
    "host": "${DB_HOST}",
    "password": "${DB_PASSWORD}"
  }
}
```

❌ **DON'T:**
```json
{
  "database": {
    "host": "prod-db.company.com",
    "password": "SecretPassword123!"
  }
}
```

### Best Practices di Sicurezza

1. **Variabili d'Ambiente**: Usa variabili per dati sensibili
2. **Validazione Input**: Sempre validare input utente
3. **Least Privilege**: Concedi solo permessi necessari
4. **Audit Logging**: Log delle azioni dell'agent
5. **Rate Limiting**: Previeni abusi

### Esempio di Validazione

```javascript
// Agent instruction
function validateInput(userInput) {
  // Controlla tipo
  if (typeof userInput !== 'string') {
    throw new Error('Input must be a string');
  }
  
  // Controlla lunghezza
  if (userInput.length > 1000) {
    throw new Error('Input too long');
  }
  
  // Sanitize
  const sanitized = userInput.replace(/[<>]/g, '');
  
  return sanitized;
}
```

## Manutenzione

### Ciclo di Vita

1. **Sviluppo**: Creazione iniziale dell'agent
2. **Testing**: Validazione con casi d'uso reali
3. **Deployment**: Rilascio alla repository
4. **Monitoring**: Raccolta feedback e metriche
5. **Iterazione**: Miglioramenti basati su feedback

### Versioning

Segui Semantic Versioning:

- **MAJOR** (1.0.0 → 2.0.0): Breaking changes
- **MINOR** (1.0.0 → 1.1.0): Nuove funzionalità backward-compatible
- **PATCH** (1.0.0 → 1.0.1): Bug fixes

### Changelog

Mantieni un CHANGELOG.md:

```markdown
# Changelog

## [2.0.0] - 2024-02-15
### Added
- Nuova capability per analisi security
- Supporto per TypeScript 5.0

### Changed
- Migliorata performance analisi codice (30% più veloce)

### Deprecated
- Sintassi vecchia per configurazione (rimossa in 3.0.0)

### Fixed
- Bug nella gestione di file grandi
- Crash con caratteri speciali in commenti

## [1.1.0] - 2024-01-20
...
```

### Testing

**Checklist di Test:**

- [ ] L'agent risponde correttamente a prompt tipici?
- [ ] Gestisce correttamente casi edge?
- [ ] Fornisce error messages utili?
- [ ] La knowledge base è accessibile?
- [ ] Le configurazioni funzionano come atteso?
- [ ] Performance accettabile con knowledge base grande?

### Metriche da Monitorare

1. **Usage Metrics**:
   - Numero di invocazioni
   - Tipologie di richieste
   - Tempo di risposta medio

2. **Quality Metrics**:
   - User satisfaction
   - Accuracy delle risposte
   - False positive rate

3. **Performance Metrics**:
   - Response time
   - Token usage
   - Cache hit rate

## Ottimizzazione

### Performance

1. **Knowledge Base**:
   - Usa indici per navigazione rapida
   - Evita duplicazione di contenuti
   - Comprimi file quando possibile

2. **Configurazione**:
   - Cache risposte frequenti
   - Ottimizza temperature per use case
   - Limita token count appropriatamente

3. **Istruzioni**:
   - Sii conciso ma completo
   - Usa bullet points per chiarezza
   - Evita ridondanza

### Esempio di Ottimizzazione

**Prima (Verboso):**
```markdown
Per analizzare il codice, prima leggi tutto il file dall'inizio alla fine, 
poi cerca problemi di sintassi, poi cerca problemi di stile, poi cerca 
problemi di sicurezza, poi cerca problemi di performance...
```

**Dopo (Ottimizzato):**
```markdown
Analizza il codice seguendo questo ordine:
1. Sintassi
2. Stile
3. Sicurezza
4. Performance
```

## Collaborazione

### Multi-Agent Systems

Quando creare agent multipli:
- Domini di conoscenza distinti
- Responsabilità separate
- Team diversi di manutenzione

**Esempio di Coordinazione:**

```yaml
---
name: "Frontend Agent"
depends_on:
  - design-system-agent
  - accessibility-agent
delegates_to:
  - backend-agent  # per API questions
  - devops-agent   # per deployment questions
---
```

## Risorse

- [Documentazione GitHub Copilot](https://docs.github.com/)
- [Best Practices Generali](https://github.com/github/copilot-best-practices)
- [Security Guidelines](https://docs.github.com/en/code-security)

---

**Ultima Revisione**: 2024-02-17
**Versione**: 1.0.0
