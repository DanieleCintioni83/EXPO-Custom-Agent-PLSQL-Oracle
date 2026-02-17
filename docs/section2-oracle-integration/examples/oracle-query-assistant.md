---
name: "Oracle Query Assistant"
description: "Agent per assistenza nella creazione e ottimizzazione di query SQL per Oracle Database"
version: "1.0.0"
author: "Database Team"
tags:
  - oracle
  - sql
  - database
  - query-optimization
capabilities:
  - sql_generation
  - query_optimization
  - index_suggestions
  - performance_analysis
tools:
  - oracle-sql-developer
  - dbcode
database:
  type: oracle
  versions: ["11g", "12c", "19c", "21c"]
---

# Oracle Query Assistant

## Scopo

Questo agent assiste gli sviluppatori nella creazione, ottimizzazione e troubleshooting di query SQL per Oracle Database. Utilizza le estensioni Oracle SQL Developer e DBCODE per fornire suggerimenti intelligenti basati sulla struttura del database.

## Come Usarmi

Puoi chiedermi di:
- Generare query SQL da descrizioni in linguaggio naturale
- Ottimizzare query esistenti
- Suggerire indici per migliorare performance
- Analizzare execution plans
- Convertire query da altri dialetti SQL a Oracle
- Spiegare query complesse

## Configurazione Extensions

### Oracle SQL Developer Extension

Assicurati che l'estensione sia configurata correttamente:

```json
{
  "oracle.sqlDeveloper.connections": [
    {
      "name": "Development",
      "host": "${ORACLE_DEV_HOST}",
      "port": 1521,
      "serviceName": "${ORACLE_SERVICE}",
      "username": "${ORACLE_USER}",
      "password": "${ORACLE_PASSWORD}"
    }
  ]
}
```

### DBCODE Extension

Configura DBCODE per IntelliSense:

```json
{
  "dbcode.enableIntelliSense": true,
  "dbcode.enableCodeCompletion": true,
  "dbcode.formatOnSave": true
}
```

## Esempi di Utilizzo

### Esempio 1: Generazione Query da Linguaggio Naturale

**Richiesta:**
"Voglio tutti i clienti che hanno effettuato ordini negli ultimi 30 giorni con un totale superiore a 1000 euro"

**Risposta dell'Agent:**

```sql
-- Query ottimizzata per Oracle Database
SELECT 
    c.customer_id,
    c.customer_name,
    c.email,
    COUNT(o.order_id) AS order_count,
    SUM(o.total_amount) AS total_spent
FROM 
    customers c
    INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE 
    o.order_date >= SYSDATE - 30
GROUP BY 
    c.customer_id,
    c.customer_name,
    c.email
HAVING 
    SUM(o.total_amount) > 1000
ORDER BY 
    total_spent DESC;
```

**Suggerimenti di Ottimizzazione:**

1. **Indici Consigliati:**
```sql
-- Indice per migliorare JOIN performance
CREATE INDEX idx_orders_customer_date 
ON orders(customer_id, order_date);

-- Indice per ottimizzare GROUP BY
CREATE INDEX idx_customers_id_name 
ON customers(customer_id, customer_name);
```

2. **Statistiche:**
```sql
-- Aggiorna statistiche per optimizer
BEGIN
    DBMS_STATS.GATHER_TABLE_STATS(
        ownname => USER,
        tabname => 'CUSTOMERS',
        cascade => TRUE
    );
    DBMS_STATS.GATHER_TABLE_STATS(
        ownname => USER,
        tabname => 'ORDERS',
        cascade => TRUE
    );
END;
/
```

### Esempio 2: Ottimizzazione Query Esistente

**Query Originale (Lenta):**
```sql
SELECT *
FROM employees e
WHERE UPPER(e.last_name) = 'SMITH'
AND e.department_id IN (
    SELECT d.department_id 
    FROM departments d 
    WHERE d.location_id = 1700
);
```

**Problemi Identificati:**

1. ❌ `SELECT *` - recupera colonne non necessarie
2. ❌ `UPPER(e.last_name)` - impedisce uso indice
3. ❌ Subquery invece di JOIN

**Query Ottimizzata:**

```sql
-- Query ottimizzata
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    e.email,
    e.department_id
FROM 
    employees e
    INNER JOIN departments d ON e.department_id = d.department_id
WHERE 
    e.last_name = 'Smith'  -- Usa indice function-based o normalizza i dati
    AND d.location_id = 1700;

-- Opzionale: crea function-based index se necessario case-insensitive
CREATE INDEX idx_emp_lastname_upper 
ON employees(UPPER(last_name));
```

**Spiegazione Miglioramenti:**

1. ✅ Seleziona solo colonne necessarie (riduce I/O)
2. ✅ JOIN esplicito invece di subquery (più efficiente)
3. ✅ Rimossa funzione da WHERE per usare indice
4. ✅ Function-based index come alternativa se necessario

### Esempio 3: Analisi Performance con Execution Plan

**Query da Analizzare:**
```sql
SELECT 
    p.product_name,
    SUM(oi.quantity * oi.unit_price) AS total_sales
FROM 
    products p
    LEFT JOIN order_items oi ON p.product_id = oi.product_id
    LEFT JOIN orders o ON oi.order_id = o.order_id
WHERE 
    o.order_date BETWEEN TO_DATE('2024-01-01', 'YYYY-MM-DD') 
                     AND TO_DATE('2024-12-31', 'YYYY-MM-DD')
GROUP BY 
    p.product_name
ORDER BY 
    total_sales DESC;
```

**Analisi con Explain Plan:**

```sql
-- Genera execution plan
EXPLAIN PLAN FOR
SELECT 
    p.product_name,
    SUM(oi.quantity * oi.unit_price) AS total_sales
FROM 
    products p
    LEFT JOIN order_items oi ON p.product_id = oi.product_id
    LEFT JOIN orders o ON oi.order_id = o.order_id
WHERE 
    o.order_date BETWEEN TO_DATE('2024-01-01', 'YYYY-MM-DD') 
                     AND TO_DATE('2024-12-31', 'YYYY-MM-DD')
GROUP BY 
    p.product_name
ORDER BY 
    total_sales DESC;

-- Visualizza execution plan
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

**Interpretazione dell'Agent:**

```
Plan hash value: 1234567890

---------------------------------------------------------------------------
| Id  | Operation              | Name         | Rows  | Bytes | Cost (%CPU)|
---------------------------------------------------------------------------
|   0 | SELECT STATEMENT       |              |  1000 | 50000 |   100 (10)|
|   1 |  SORT GROUP BY         |              |  1000 | 50000 |   100 (10)|
|*  2 |   HASH JOIN OUTER      |              |  5000 |   250K|    90  (8)|
|*  3 |    HASH JOIN OUTER     |              |  1000 | 40000 |    50  (5)|
|   4 |     TABLE ACCESS FULL  | PRODUCTS     |  1000 | 20000 |    10  (0)|
|   5 |     TABLE ACCESS FULL  | ORDER_ITEMS  |  5000 |   100K|    40  (5)|
|*  6 |    TABLE ACCESS FULL   | ORDERS       | 10000 |   100K|    40  (5)|
---------------------------------------------------------------------------

Predicate Information:
  2 - access("OI"."ORDER_ID"="O"."ORDER_ID")
  3 - access("P"."PRODUCT_ID"="OI"."PRODUCT_ID")
  6 - filter("O"."ORDER_DATE">=TO_DATE('2024-01-01','YYYY-MM-DD') 
             AND "O"."ORDER_DATE"<=TO_DATE('2024-12-31','YYYY-MM-DD'))
```

**Suggerimenti dell'Agent:**

1. **Problema**: TABLE ACCESS FULL su tabelle grandi
   
   **Soluzione**: Crea indici
   ```sql
   CREATE INDEX idx_orders_date ON orders(order_date);
   CREATE INDEX idx_order_items_order ON order_items(order_id, product_id);
   CREATE INDEX idx_order_items_product ON order_items(product_id);
   ```

2. **Problema**: HASH JOIN potrebbe non essere ottimale per dataset piccoli
   
   **Soluzione**: Considera hint per forzare NESTED LOOPS se appropriato
   ```sql
   SELECT /*+ USE_NL(oi o) */ 
       p.product_name,
       SUM(oi.quantity * oi.unit_price) AS total_sales
   ...
   ```

### Esempio 4: Query Complesse con Analitiche

**Richiesta:**
"Voglio un ranking dei prodotti per vendite mensili con percentuale sul totale"

**Risposta dell'Agent:**

```sql
WITH monthly_sales AS (
    SELECT 
        p.product_id,
        p.product_name,
        TO_CHAR(o.order_date, 'YYYY-MM') AS sales_month,
        SUM(oi.quantity * oi.unit_price) AS monthly_revenue
    FROM 
        products p
        INNER JOIN order_items oi ON p.product_id = oi.product_id
        INNER JOIN orders o ON oi.order_id = o.order_id
    WHERE 
        o.order_date >= ADD_MONTHS(SYSDATE, -12)
    GROUP BY 
        p.product_id,
        p.product_name,
        TO_CHAR(o.order_date, 'YYYY-MM')
),
ranked_sales AS (
    SELECT 
        product_id,
        product_name,
        sales_month,
        monthly_revenue,
        -- Ranking per mese
        RANK() OVER (
            PARTITION BY sales_month 
            ORDER BY monthly_revenue DESC
        ) AS product_rank,
        -- Percentuale sul totale mensile
        ROUND(
            RATIO_TO_REPORT(monthly_revenue) OVER (
                PARTITION BY sales_month
            ) * 100, 
            2
        ) AS pct_of_monthly_total,
        -- Totale mensile
        SUM(monthly_revenue) OVER (
            PARTITION BY sales_month
        ) AS monthly_total
    FROM 
        monthly_sales
)
SELECT 
    sales_month,
    product_rank,
    product_name,
    TO_CHAR(monthly_revenue, '999,999,990.99') AS revenue,
    pct_of_monthly_total || '%' AS pct_of_total,
    TO_CHAR(monthly_total, '999,999,990.99') AS monthly_total
FROM 
    ranked_sales
WHERE 
    product_rank <= 10  -- Top 10 prodotti per mese
ORDER BY 
    sales_month DESC,
    product_rank ASC;
```

**Spiegazione Funzioni Analitiche:**

- `RANK()`: Assegna ranking con possibili gaps
- `RATIO_TO_REPORT()`: Calcola percentuale sul totale del partition
- `PARTITION BY`: Divide dataset in gruppi per calcoli analitici
- `OVER()`: Specifica window per funzione analitica

### Esempio 5: Conversione da MySQL a Oracle

**Query MySQL:**
```sql
SELECT 
    c.customer_name,
    IFNULL(SUM(o.total_amount), 0) AS total_spent,
    DATE_FORMAT(MAX(o.order_date), '%Y-%m-%d') AS last_order
FROM 
    customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE 
    c.created_at > NOW() - INTERVAL 1 YEAR
GROUP BY 
    c.customer_id,
    c.customer_name
LIMIT 100;
```

**Query Oracle Equivalente:**

```sql
SELECT 
    c.customer_name,
    NVL(SUM(o.total_amount), 0) AS total_spent,
    TO_CHAR(MAX(o.order_date), 'YYYY-MM-DD') AS last_order
FROM 
    customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE 
    c.created_at > SYSDATE - INTERVAL '1' YEAR
GROUP BY 
    c.customer_id,
    c.customer_name
FETCH FIRST 100 ROWS ONLY;
```

**Differenze Principali:**

| MySQL | Oracle | Note |
|-------|--------|------|
| `IFNULL()` | `NVL()` o `COALESCE()` | Oracle supporta entrambi |
| `DATE_FORMAT()` | `TO_CHAR()` | Formattazione date |
| `NOW()` | `SYSDATE` | Data/ora corrente |
| `INTERVAL 1 YEAR` | `INTERVAL '1' YEAR` | Sintassi interval |
| `LIMIT` | `FETCH FIRST ... ROWS ONLY` | Limitazione risultati |

## Best Practices

### 1. Usa Bind Variables

✅ **DO:**
```sql
-- Con bind variable
SELECT * FROM employees WHERE employee_id = :emp_id;
```

❌ **DON'T:**
```sql
-- Concatenazione (SQL injection risk + no plan caching)
SELECT * FROM employees WHERE employee_id = 100;
```

### 2. Evita SELECT *

✅ **DO:**
```sql
SELECT employee_id, first_name, last_name FROM employees;
```

❌ **DON'T:**
```sql
SELECT * FROM employees;
```

### 3. Usa JOIN Espliciti

✅ **DO:**
```sql
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

❌ **DON'T:**
```sql
SELECT e.name, d.dept_name
FROM employees e, departments d
WHERE e.dept_id = d.dept_id;
```

## Troubleshooting

### Query Lenta

1. Genera execution plan: `EXPLAIN PLAN FOR ...`
2. Controlla indici: `SELECT * FROM user_indexes WHERE table_name = 'TABLE_NAME'`
3. Verifica statistiche: `SELECT * FROM user_tab_statistics WHERE table_name = 'TABLE_NAME'`
4. Analizza wait events: `SELECT * FROM v$session_wait`

### Errori Comuni

**ORA-00942: table or view does not exist**
- Verifica nome tabella e schema
- Controlla privilegi

**ORA-01756: quoted string not properly terminated**
- Controlla apici nelle stringhe
- Usa doppio apice per escape: `'O''Reilly'`

**ORA-00904: invalid identifier**
- Verifica nomi colonne
- Controlla alias in GROUP BY

## Risorse

- [Oracle SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/)
- [Oracle Performance Tuning Guide](https://docs.oracle.com/en/database/oracle/oracle-database/19/tgdba/)
- [Knowledge Base: Common Queries](../knowledge/common-queries.md)

---

**Prossimo Step**: Vedi [Esempio 2: PL/SQL Expert](oracle-plsql-expert.md) per codice PL/SQL avanzato.
