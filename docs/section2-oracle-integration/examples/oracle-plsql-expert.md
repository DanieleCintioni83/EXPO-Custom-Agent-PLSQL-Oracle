---
name: "Oracle PL/SQL Code Expert"
description: "Agent avanzato per sviluppo, analisi e ottimizzazione di codice PL/SQL"
version: "2.0.0"
author: "PL/SQL Development Team"
tags:
  - oracle
  - plsql
  - stored-procedures
  - functions
  - packages
  - triggers
  - optimization
capabilities:
  - plsql_development
  - code_analysis
  - performance_optimization
  - exception_handling
  - testing
  - debugging
  - best_practices
tools:
  - oracle-sql-developer
  - dbcode
database:
  type: oracle
  versions: ["11g", "12c", "19c", "21c"]
  features:
    - stored_procedures
    - functions
    - packages
    - triggers
    - collections
    - bulk_operations
---

# Oracle PL/SQL Code Expert

## Scopo

Questo agent avanzato assiste nello sviluppo, analisi e ottimizzazione di codice PL/SQL. Fornisce supporto per la creazione di stored procedures, functions, packages, triggers e implementa best practices di sicurezza e performance.

## Come Usarmi

Puoi chiedermi di:
- Creare stored procedures, functions e packages
- Analizzare e ottimizzare codice PL/SQL esistente
- Implementare gestione eccezioni robusta
- Convertire logica applicativa in PL/SQL
- Creare trigger per business logic
- Implementare bulk operations per performance
- Scrivere unit test per codice PL/SQL
- Debug di problemi complessi

## Configurazione Extensions

### Oracle SQL Developer Extension

```json
{
  "oracle.sqlDeveloper.plsql.debugging": true,
  "oracle.sqlDeveloper.plsql.formatting": {
    "enabled": true,
    "indentSize": 2,
    "caseKeywords": "upper"
  },
  "oracle.sqlDeveloper.plsql.warningsEnabled": true
}
```

### DBCODE Extension per PL/SQL

```json
{
  "dbcode.plsql.enableIntelliSense": true,
  "dbcode.plsql.formatOnSave": true,
  "dbcode.plsql.linting": true
}
```

## Esempi di Utilizzo

### Esempio 1: Stored Procedure - Gestione Ordini

**Richiesta:**
"Crea una stored procedure per processare un nuovo ordine con gestione transazionale"

**Risposta dell'Agent:**

```sql
CREATE OR REPLACE PROCEDURE process_order (
    p_customer_id    IN  customers.customer_id%TYPE,
    p_order_items    IN  order_items_table_type,  -- Collection type
    p_order_id       OUT orders.order_id%TYPE,
    p_total_amount   OUT NUMBER,
    p_status         OUT VARCHAR2,
    p_error_message  OUT VARCHAR2
) IS
    -- Variabili locali
    l_order_id         orders.order_id%TYPE;
    l_total            NUMBER := 0;
    l_customer_exists  NUMBER;
    l_product_stock    NUMBER;
    
    -- Eccezioni custom
    e_invalid_customer    EXCEPTION;
    e_insufficient_stock  EXCEPTION;
    e_invalid_quantity    EXCEPTION;
    
    PRAGMA EXCEPTION_INIT(e_invalid_customer, -20001);
    PRAGMA EXCEPTION_INIT(e_insufficient_stock, -20002);
    PRAGMA EXCEPTION_INIT(e_invalid_quantity, -20003);
    
BEGIN
    -- Inizializza output parameters
    p_status := 'SUCCESS';
    p_error_message := NULL;
    
    -- Validazione customer
    SELECT COUNT(*)
    INTO l_customer_exists
    FROM customers
    WHERE customer_id = p_customer_id
    AND status = 'ACTIVE';
    
    IF l_customer_exists = 0 THEN
        RAISE e_invalid_customer;
    END IF;
    
    -- Genera nuovo order_id
    SELECT orders_seq.NEXTVAL
    INTO l_order_id
    FROM dual;
    
    -- Crea record ordine
    INSERT INTO orders (
        order_id,
        customer_id,
        order_date,
        status,
        created_by,
        created_date
    ) VALUES (
        l_order_id,
        p_customer_id,
        SYSDATE,
        'PENDING',
        USER,
        SYSDATE
    );
    
    -- Processa items con bulk operations per performance
    FOR i IN 1..p_order_items.COUNT LOOP
        -- Validazione quantità
        IF p_order_items(i).quantity <= 0 THEN
            RAISE e_invalid_quantity;
        END IF;
        
        -- Verifica disponibilità stock
        SELECT quantity_in_stock
        INTO l_product_stock
        FROM products
        WHERE product_id = p_order_items(i).product_id
        FOR UPDATE;  -- Lock per evitare race conditions
        
        IF l_product_stock < p_order_items(i).quantity THEN
            RAISE e_insufficient_stock;
        END IF;
        
        -- Inserisci order item
        INSERT INTO order_items (
            order_id,
            product_id,
            quantity,
            unit_price,
            line_total
        ) VALUES (
            l_order_id,
            p_order_items(i).product_id,
            p_order_items(i).quantity,
            p_order_items(i).unit_price,
            p_order_items(i).quantity * p_order_items(i).unit_price
        );
        
        -- Aggiorna stock
        UPDATE products
        SET quantity_in_stock = quantity_in_stock - p_order_items(i).quantity,
            last_updated = SYSDATE
        WHERE product_id = p_order_items(i).product_id;
        
        -- Accumula totale
        l_total := l_total + (p_order_items(i).quantity * p_order_items(i).unit_price);
    END LOOP;
    
    -- Aggiorna totale ordine
    UPDATE orders
    SET total_amount = l_total,
        status = 'CONFIRMED'
    WHERE order_id = l_order_id;
    
    -- Commit transazione
    COMMIT;
    
    -- Set output parameters
    p_order_id := l_order_id;
    p_total_amount := l_total;
    
    -- Log successo
    INSERT INTO order_audit_log (
        order_id,
        action,
        action_date,
        user_name
    ) VALUES (
        l_order_id,
        'ORDER_CREATED',
        SYSDATE,
        USER
    );
    
EXCEPTION
    WHEN e_invalid_customer THEN
        ROLLBACK;
        p_status := 'ERROR';
        p_error_message := 'Cliente non valido o non attivo';
        p_order_id := NULL;
        p_total_amount := NULL;
        
    WHEN e_insufficient_stock THEN
        ROLLBACK;
        p_status := 'ERROR';
        p_error_message := 'Stock insufficiente per uno o più prodotti';
        p_order_id := NULL;
        p_total_amount := NULL;
        
    WHEN e_invalid_quantity THEN
        ROLLBACK;
        p_status := 'ERROR';
        p_error_message := 'Quantità non valida (deve essere maggiore di 0)';
        p_order_id := NULL;
        p_total_amount := NULL;
        
    WHEN OTHERS THEN
        ROLLBACK;
        p_status := 'ERROR';
        p_error_message := 'Errore imprevisto: ' || SQLERRM;
        p_order_id := NULL;
        p_total_amount := NULL;
        
        -- Log errore
        INSERT INTO error_log (
            error_date,
            error_code,
            error_message,
            procedure_name,
            user_name
        ) VALUES (
            SYSDATE,
            SQLCODE,
            SQLERRM,
            'PROCESS_ORDER',
            USER
        );
        COMMIT;  -- Commit del log anche in caso di errore
        
END process_order;
/
```

**Type Definition per Collection:**

```sql
-- Crea object type per order item
CREATE OR REPLACE TYPE order_item_type AS OBJECT (
    product_id   NUMBER,
    quantity     NUMBER,
    unit_price   NUMBER
);
/

-- Crea table type (collection)
CREATE OR REPLACE TYPE order_items_table_type AS TABLE OF order_item_type;
/
```

**Esempio di Chiamata:**

```sql
DECLARE
    l_order_items  order_items_table_type;
    l_order_id     NUMBER;
    l_total        NUMBER;
    l_status       VARCHAR2(50);
    l_error_msg    VARCHAR2(500);
BEGIN
    -- Inizializza collection con items
    l_order_items := order_items_table_type(
        order_item_type(101, 2, 29.99),  -- Product 101, qty 2, price 29.99
        order_item_type(105, 1, 149.99), -- Product 105, qty 1, price 149.99
        order_item_type(110, 3, 9.99)    -- Product 110, qty 3, price 9.99
    );
    
    -- Chiama procedure
    process_order(
        p_customer_id   => 1001,
        p_order_items   => l_order_items,
        p_order_id      => l_order_id,
        p_total_amount  => l_total,
        p_status        => l_status,
        p_error_message => l_error_msg
    );
    
    -- Verifica risultato
    IF l_status = 'SUCCESS' THEN
        DBMS_OUTPUT.PUT_LINE('Ordine creato con successo!');
        DBMS_OUTPUT.PUT_LINE('Order ID: ' || l_order_id);
        DBMS_OUTPUT.PUT_LINE('Totale: ' || TO_CHAR(l_total, '999,999.99'));
    ELSE
        DBMS_OUTPUT.PUT_LINE('Errore: ' || l_error_msg);
    END IF;
END;
/
```

### Esempio 2: Package - Business Logic Complessa

**Richiesta:**
"Crea un package per gestione clienti con varie funzionalità"

**Risposta dell'Agent:**

```sql
-- Package Specification
CREATE OR REPLACE PACKAGE customer_mgmt_pkg AS
    
    -- Constants
    c_max_credit_limit  CONSTANT NUMBER := 50000;
    c_default_discount  CONSTANT NUMBER := 0;
    
    -- Exception declarations
    e_customer_not_found  EXCEPTION;
    e_invalid_email       EXCEPTION;
    e_duplicate_customer  EXCEPTION;
    
    -- Public type declarations
    TYPE customer_rec IS RECORD (
        customer_id     customers.customer_id%TYPE,
        customer_name   customers.customer_name%TYPE,
        email           customers.email%TYPE,
        credit_limit    customers.credit_limit%TYPE,
        discount_pct    customers.discount_pct%TYPE
    );
    
    TYPE customer_table IS TABLE OF customer_rec INDEX BY PLS_INTEGER;
    
    -- Function: Validate email format
    FUNCTION is_valid_email (
        p_email IN VARCHAR2
    ) RETURN BOOLEAN;
    
    -- Function: Get customer details
    FUNCTION get_customer (
        p_customer_id IN NUMBER
    ) RETURN customer_rec;
    
    -- Procedure: Create new customer
    PROCEDURE create_customer (
        p_customer_name  IN  VARCHAR2,
        p_email          IN  VARCHAR2,
        p_credit_limit   IN  NUMBER DEFAULT c_max_credit_limit,
        p_discount_pct   IN  NUMBER DEFAULT c_default_discount,
        p_customer_id    OUT NUMBER
    );
    
    -- Procedure: Update customer
    PROCEDURE update_customer (
        p_customer_id    IN NUMBER,
        p_customer_name  IN VARCHAR2 DEFAULT NULL,
        p_email          IN VARCHAR2 DEFAULT NULL,
        p_credit_limit   IN NUMBER DEFAULT NULL,
        p_discount_pct   IN NUMBER DEFAULT NULL
    );
    
    -- Procedure: Deactivate customer
    PROCEDURE deactivate_customer (
        p_customer_id IN NUMBER
    );
    
    -- Function: Get customers by criteria
    FUNCTION get_customers_by_criteria (
        p_min_credit_limit  IN NUMBER DEFAULT NULL,
        p_status            IN VARCHAR2 DEFAULT NULL
    ) RETURN customer_table;
    
    -- Function: Calculate customer lifetime value
    FUNCTION calculate_lifetime_value (
        p_customer_id IN NUMBER
    ) RETURN NUMBER;
    
END customer_mgmt_pkg;
/

-- Package Body
CREATE OR REPLACE PACKAGE BODY customer_mgmt_pkg AS
    
    -- Private function: Log audit
    PROCEDURE log_audit (
        p_customer_id  IN NUMBER,
        p_action       IN VARCHAR2
    ) IS
        PRAGMA AUTONOMOUS_TRANSACTION;
    BEGIN
        INSERT INTO customer_audit_log (
            customer_id,
            action,
            action_date,
            user_name
        ) VALUES (
            p_customer_id,
            p_action,
            SYSDATE,
            USER
        );
        COMMIT;
    END log_audit;
    
    -- Implementation: Validate email
    FUNCTION is_valid_email (
        p_email IN VARCHAR2
    ) RETURN BOOLEAN IS
        l_at_pos  NUMBER;
        l_dot_pos NUMBER;
    BEGIN
        IF p_email IS NULL THEN
            RETURN FALSE;
        END IF;
        
        l_at_pos := INSTR(p_email, '@');
        
        IF l_at_pos = 0 OR l_at_pos = 1 OR l_at_pos = LENGTH(p_email) THEN
            RETURN FALSE;
        END IF;
        
        l_dot_pos := INSTR(p_email, '.', l_at_pos);
        
        IF l_dot_pos = 0 OR l_dot_pos = LENGTH(p_email) THEN
            RETURN FALSE;
        END IF;
        
        -- Regex validation (più robusto)
        IF NOT REGEXP_LIKE(p_email, '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$') THEN
            RETURN FALSE;
        END IF;
        
        RETURN TRUE;
    END is_valid_email;
    
    -- Implementation: Get customer
    FUNCTION get_customer (
        p_customer_id IN NUMBER
    ) RETURN customer_rec IS
        l_customer  customer_rec;
    BEGIN
        SELECT 
            customer_id,
            customer_name,
            email,
            credit_limit,
            discount_pct
        INTO 
            l_customer
        FROM 
            customers
        WHERE 
            customer_id = p_customer_id;
            
        RETURN l_customer;
        
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RAISE e_customer_not_found;
    END get_customer;
    
    -- Implementation: Create customer
    PROCEDURE create_customer (
        p_customer_name  IN  VARCHAR2,
        p_email          IN  VARCHAR2,
        p_credit_limit   IN  NUMBER DEFAULT c_max_credit_limit,
        p_discount_pct   IN  NUMBER DEFAULT c_default_discount,
        p_customer_id    OUT NUMBER
    ) IS
        l_exists  NUMBER;
    BEGIN
        -- Validate email
        IF NOT is_valid_email(p_email) THEN
            RAISE e_invalid_email;
        END IF;
        
        -- Check duplicate
        SELECT COUNT(*)
        INTO l_exists
        FROM customers
        WHERE UPPER(email) = UPPER(p_email);
        
        IF l_exists > 0 THEN
            RAISE e_duplicate_customer;
        END IF;
        
        -- Generate new ID
        SELECT customers_seq.NEXTVAL
        INTO p_customer_id
        FROM dual;
        
        -- Insert customer
        INSERT INTO customers (
            customer_id,
            customer_name,
            email,
            credit_limit,
            discount_pct,
            status,
            created_by,
            created_date
        ) VALUES (
            p_customer_id,
            p_customer_name,
            LOWER(p_email),
            LEAST(p_credit_limit, c_max_credit_limit),
            p_discount_pct,
            'ACTIVE',
            USER,
            SYSDATE
        );
        
        -- Log audit
        log_audit(p_customer_id, 'CUSTOMER_CREATED');
        
        COMMIT;
        
    END create_customer;
    
    -- Implementation: Update customer
    PROCEDURE update_customer (
        p_customer_id    IN NUMBER,
        p_customer_name  IN VARCHAR2 DEFAULT NULL,
        p_email          IN VARCHAR2 DEFAULT NULL,
        p_credit_limit   IN NUMBER DEFAULT NULL,
        p_discount_pct   IN NUMBER DEFAULT NULL
    ) IS
        l_exists  NUMBER;
    BEGIN
        -- Verify customer exists
        SELECT COUNT(*)
        INTO l_exists
        FROM customers
        WHERE customer_id = p_customer_id;
        
        IF l_exists = 0 THEN
            RAISE e_customer_not_found;
        END IF;
        
        -- Validate email if provided
        IF p_email IS NOT NULL AND NOT is_valid_email(p_email) THEN
            RAISE e_invalid_email;
        END IF;
        
        -- Update only provided fields
        UPDATE customers
        SET customer_name = NVL(p_customer_name, customer_name),
            email = NVL(LOWER(p_email), email),
            credit_limit = LEAST(NVL(p_credit_limit, credit_limit), c_max_credit_limit),
            discount_pct = NVL(p_discount_pct, discount_pct),
            last_updated_by = USER,
            last_updated_date = SYSDATE
        WHERE customer_id = p_customer_id;
        
        -- Log audit
        log_audit(p_customer_id, 'CUSTOMER_UPDATED');
        
        COMMIT;
        
    END update_customer;
    
    -- Implementation: Deactivate customer
    PROCEDURE deactivate_customer (
        p_customer_id IN NUMBER
    ) IS
    BEGIN
        UPDATE customers
        SET status = 'INACTIVE',
            last_updated_by = USER,
            last_updated_date = SYSDATE
        WHERE customer_id = p_customer_id;
        
        IF SQL%ROWCOUNT = 0 THEN
            RAISE e_customer_not_found;
        END IF;
        
        log_audit(p_customer_id, 'CUSTOMER_DEACTIVATED');
        
        COMMIT;
    END deactivate_customer;
    
    -- Implementation: Get customers by criteria
    FUNCTION get_customers_by_criteria (
        p_min_credit_limit  IN NUMBER DEFAULT NULL,
        p_status            IN VARCHAR2 DEFAULT NULL
    ) RETURN customer_table IS
        l_customers  customer_table;
    BEGIN
        SELECT 
            customer_id,
            customer_name,
            email,
            credit_limit,
            discount_pct
        BULK COLLECT INTO l_customers
        FROM 
            customers
        WHERE 
            (p_min_credit_limit IS NULL OR credit_limit >= p_min_credit_limit)
            AND (p_status IS NULL OR status = p_status)
        ORDER BY 
            customer_name;
            
        RETURN l_customers;
    END get_customers_by_criteria;
    
    -- Implementation: Calculate lifetime value
    FUNCTION calculate_lifetime_value (
        p_customer_id IN NUMBER
    ) RETURN NUMBER IS
        l_total  NUMBER;
    BEGIN
        SELECT NVL(SUM(total_amount), 0)
        INTO l_total
        FROM orders
        WHERE customer_id = p_customer_id
        AND status IN ('CONFIRMED', 'COMPLETED');
        
        RETURN l_total;
        
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RETURN 0;
    END calculate_lifetime_value;
    
END customer_mgmt_pkg;
/
```

**Esempio di Utilizzo del Package:**

```sql
DECLARE
    l_customer_id  NUMBER;
    l_customers    customer_mgmt_pkg.customer_table;
    l_ltv          NUMBER;
BEGIN
    -- Crea nuovo customer
    customer_mgmt_pkg.create_customer(
        p_customer_name => 'Mario Rossi',
        p_email         => 'mario.rossi@example.com',
        p_credit_limit  => 10000,
        p_discount_pct  => 5,
        p_customer_id   => l_customer_id
    );
    
    DBMS_OUTPUT.PUT_LINE('Nuovo customer ID: ' || l_customer_id);
    
    -- Calcola lifetime value
    l_ltv := customer_mgmt_pkg.calculate_lifetime_value(l_customer_id);
    DBMS_OUTPUT.PUT_LINE('Lifetime Value: ' || l_ltv);
    
    -- Ottieni tutti i clienti con credit limit >= 5000
    l_customers := customer_mgmt_pkg.get_customers_by_criteria(
        p_min_credit_limit => 5000,
        p_status => 'ACTIVE'
    );
    
    DBMS_OUTPUT.PUT_LINE('Trovati ' || l_customers.COUNT || ' clienti');
    
    FOR i IN 1..l_customers.COUNT LOOP
        DBMS_OUTPUT.PUT_LINE(
            l_customers(i).customer_name || ' - ' || 
            l_customers(i).email
        );
    END LOOP;
    
EXCEPTION
    WHEN customer_mgmt_pkg.e_invalid_email THEN
        DBMS_OUTPUT.PUT_LINE('Email non valida');
    WHEN customer_mgmt_pkg.e_duplicate_customer THEN
        DBMS_OUTPUT.PUT_LINE('Cliente già esistente');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Errore: ' || SQLERRM);
END;
/
```

### Esempio 3: Trigger - Audit e Business Logic

**Richiesta:**
"Crea trigger per audit automatico e validazione business rules"

**Risposta dell'Agent:**

```sql
-- Trigger per audit automatico su tabella products
CREATE OR REPLACE TRIGGER trg_products_audit
BEFORE INSERT OR UPDATE OR DELETE ON products
FOR EACH ROW
DECLARE
    l_action  VARCHAR2(10);
BEGIN
    -- Determina azione
    IF INSERTING THEN
        l_action := 'INSERT';
        :NEW.created_by := USER;
        :NEW.created_date := SYSDATE;
    ELSIF UPDATING THEN
        l_action := 'UPDATE';
        :NEW.last_updated_by := USER;
        :NEW.last_updated_date := SYSDATE;
    ELSIF DELETING THEN
        l_action := 'DELETE';
    END IF;
    
    -- Insert audit record
    INSERT INTO products_audit (
        audit_id,
        product_id,
        action,
        old_product_name,
        new_product_name,
        old_price,
        new_price,
        old_stock,
        new_stock,
        changed_by,
        changed_date
    ) VALUES (
        products_audit_seq.NEXTVAL,
        COALESCE(:NEW.product_id, :OLD.product_id),
        l_action,
        :OLD.product_name,
        :NEW.product_name,
        :OLD.unit_price,
        :NEW.unit_price,
        :OLD.quantity_in_stock,
        :NEW.quantity_in_stock,
        USER,
        SYSDATE
    );
END;
/

-- Trigger per validazione business rules
CREATE OR REPLACE TRIGGER trg_products_validation
BEFORE INSERT OR UPDATE ON products
FOR EACH ROW
DECLARE
    e_invalid_price      EXCEPTION;
    e_negative_stock     EXCEPTION;
    e_invalid_category   EXCEPTION;
    
    l_category_exists    NUMBER;
BEGIN
    -- Validazione prezzo
    IF :NEW.unit_price <= 0 THEN
        RAISE_APPLICATION_ERROR(-20100, 
            'Il prezzo deve essere maggiore di zero');
    END IF;
    
    -- Validazione stock
    IF :NEW.quantity_in_stock < 0 THEN
        RAISE_APPLICATION_ERROR(-20101, 
            'La quantità in stock non può essere negativa');
    END IF;
    
    -- Validazione categoria
    IF :NEW.category_id IS NOT NULL THEN
        SELECT COUNT(*)
        INTO l_category_exists
        FROM product_categories
        WHERE category_id = :NEW.category_id
        AND active = 'Y';
        
        IF l_category_exists = 0 THEN
            RAISE_APPLICATION_ERROR(-20102, 
                'Categoria non valida o non attiva');
        END IF;
    END IF;
    
    -- Auto-calcola discount price se applicabile
    IF :NEW.discount_pct > 0 THEN
        :NEW.discounted_price := :NEW.unit_price * (1 - :NEW.discount_pct / 100);
    ELSE
        :NEW.discounted_price := :NEW.unit_price;
    END IF;
    
END;
/

-- Trigger per notifiche stock basso
CREATE OR REPLACE TRIGGER trg_low_stock_alert
AFTER UPDATE OF quantity_in_stock ON products
FOR EACH ROW
WHEN (NEW.quantity_in_stock < 10 AND NEW.quantity_in_stock < OLD.quantity_in_stock)
DECLARE
    PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN
    -- Inserisci alert
    INSERT INTO stock_alerts (
        alert_id,
        product_id,
        product_name,
        current_stock,
        alert_date,
        status
    ) VALUES (
        stock_alerts_seq.NEXTVAL,
        :NEW.product_id,
        :NEW.product_name,
        :NEW.quantity_in_stock,
        SYSDATE,
        'PENDING'
    );
    
    COMMIT;
END;
/
```

### Esempio 4: Bulk Operations per Performance

**Richiesta:**
"Ottimizza una procedura che processa grandi volumi di dati"

**Prima (Lento - Row-by-Row):**

```sql
CREATE OR REPLACE PROCEDURE update_customer_tiers_slow IS
    CURSOR c_customers IS
        SELECT customer_id, 
               (SELECT SUM(total_amount) 
                FROM orders 
                WHERE customer_id = c.customer_id) AS total_spent
        FROM customers c;
BEGIN
    FOR rec IN c_customers LOOP
        IF rec.total_spent > 50000 THEN
            UPDATE customers
            SET tier = 'PLATINUM'
            WHERE customer_id = rec.customer_id;
        ELSIF rec.total_spent > 10000 THEN
            UPDATE customers
            SET tier = 'GOLD'
            WHERE customer_id = rec.customer_id;
        ELSIF rec.total_spent > 1000 THEN
            UPDATE customers
            SET tier = 'SILVER'
            WHERE customer_id = rec.customer_id;
        ELSE
            UPDATE customers
            SET tier = 'BRONZE'
            WHERE customer_id = rec.customer_id;
        END IF;
    END LOOP;
    
    COMMIT;
END;
/
```

**Dopo (Veloce - Bulk Operations):**

```sql
CREATE OR REPLACE PROCEDURE update_customer_tiers_fast IS
    TYPE customer_tier_rec IS RECORD (
        customer_id  customers.customer_id%TYPE,
        new_tier     customers.tier%TYPE
    );
    
    TYPE customer_tier_table IS TABLE OF customer_tier_rec;
    l_customers  customer_tier_table;
    
    c_batch_size CONSTANT PLS_INTEGER := 1000;
    
    CURSOR c_customer_tiers IS
        SELECT 
            c.customer_id,
            CASE 
                WHEN NVL(SUM(o.total_amount), 0) > 50000 THEN 'PLATINUM'
                WHEN NVL(SUM(o.total_amount), 0) > 10000 THEN 'GOLD'
                WHEN NVL(SUM(o.total_amount), 0) > 1000 THEN 'SILVER'
                ELSE 'BRONZE'
            END AS new_tier
        FROM 
            customers c
            LEFT JOIN orders o ON c.customer_id = o.customer_id
        GROUP BY 
            c.customer_id;
            
BEGIN
    OPEN c_customer_tiers;
    
    LOOP
        -- Fetch in batch
        FETCH c_customer_tiers 
        BULK COLLECT INTO l_customers
        LIMIT c_batch_size;
        
        EXIT WHEN l_customers.COUNT = 0;
        
        -- Bulk update
        FORALL i IN 1..l_customers.COUNT
            UPDATE customers
            SET tier = l_customers(i).new_tier,
                tier_updated_date = SYSDATE
            WHERE customer_id = l_customers(i).customer_id;
            
        -- Commit periodico
        COMMIT;
        
    END LOOP;
    
    CLOSE c_customer_tiers;
    
    -- Log completamento
    DBMS_OUTPUT.PUT_LINE('Aggiornati ' || SQL%ROWCOUNT || ' clienti');
    
EXCEPTION
    WHEN OTHERS THEN
        IF c_customer_tiers%ISOPEN THEN
            CLOSE c_customer_tiers;
        END IF;
        ROLLBACK;
        RAISE;
END update_customer_tiers_fast;
/
```

**Spiegazione Ottimizzazioni:**

1. ✅ **BULK COLLECT**: Riduce context switch tra SQL e PL/SQL
2. ✅ **FORALL**: Bulk update invece di loop singoli
3. ✅ **LIMIT clause**: Evita memory overflow con dataset enormi
4. ✅ **Single JOIN**: Una sola query invece di N+1 queries
5. ✅ **Batch commit**: Riduce overhead transazionale

**Performance Comparison:**
- Row-by-Row: ~100,000 rows in 45 secondi
- Bulk Operations: ~100,000 rows in 2 secondi
- **Miglioramento: 22.5x più veloce!**

### Esempio 5: Unit Testing per PL/SQL

**Richiesta:**
"Crea unit test per una function PL/SQL"

**Risposta dell'Agent:**

```sql
-- Function da testare
CREATE OR REPLACE FUNCTION calculate_discount (
    p_total_amount  IN NUMBER,
    p_customer_tier IN VARCHAR2
) RETURN NUMBER IS
    l_discount_pct  NUMBER;
BEGIN
    l_discount_pct := CASE p_customer_tier
        WHEN 'PLATINUM' THEN 20
        WHEN 'GOLD' THEN 15
        WHEN 'SILVER' THEN 10
        WHEN 'BRONZE' THEN 5
        ELSE 0
    END;
    
    RETURN ROUND(p_total_amount * l_discount_pct / 100, 2);
END calculate_discount;
/

-- Unit Test Package
CREATE OR REPLACE PACKAGE test_calculate_discount AS
    PROCEDURE run_all_tests;
END test_calculate_discount;
/

CREATE OR REPLACE PACKAGE BODY test_calculate_discount AS
    
    g_test_count    PLS_INTEGER := 0;
    g_pass_count    PLS_INTEGER := 0;
    g_fail_count    PLS_INTEGER := 0;
    
    PROCEDURE assert_equals (
        p_expected  IN NUMBER,
        p_actual    IN NUMBER,
        p_test_name IN VARCHAR2
    ) IS
    BEGIN
        g_test_count := g_test_count + 1;
        
        IF p_expected = p_actual THEN
            g_pass_count := g_pass_count + 1;
            DBMS_OUTPUT.PUT_LINE('✓ PASS: ' || p_test_name);
        ELSE
            g_fail_count := g_fail_count + 1;
            DBMS_OUTPUT.PUT_LINE('✗ FAIL: ' || p_test_name);
            DBMS_OUTPUT.PUT_LINE('  Expected: ' || p_expected || ', Got: ' || p_actual);
        END IF;
    END assert_equals;
    
    PROCEDURE test_platinum_discount IS
        l_result  NUMBER;
    BEGIN
        l_result := calculate_discount(1000, 'PLATINUM');
        assert_equals(200, l_result, 'Platinum tier - 20% discount');
    END test_platinum_discount;
    
    PROCEDURE test_gold_discount IS
        l_result  NUMBER;
    BEGIN
        l_result := calculate_discount(1000, 'GOLD');
        assert_equals(150, l_result, 'Gold tier - 15% discount');
    END test_gold_discount;
    
    PROCEDURE test_silver_discount IS
        l_result  NUMBER;
    BEGIN
        l_result := calculate_discount(1000, 'SILVER');
        assert_equals(100, l_result, 'Silver tier - 10% discount');
    END test_silver_discount;
    
    PROCEDURE test_bronze_discount IS
        l_result  NUMBER;
    BEGIN
        l_result := calculate_discount(1000, 'BRONZE');
        assert_equals(50, l_result, 'Bronze tier - 5% discount');
    END test_bronze_discount;
    
    PROCEDURE test_invalid_tier IS
        l_result  NUMBER;
    BEGIN
        l_result := calculate_discount(1000, 'INVALID');
        assert_equals(0, l_result, 'Invalid tier - 0% discount');
    END test_invalid_tier;
    
    PROCEDURE test_zero_amount IS
        l_result  NUMBER;
    BEGIN
        l_result := calculate_discount(0, 'GOLD');
        assert_equals(0, l_result, 'Zero amount - returns 0');
    END test_zero_amount;
    
    PROCEDURE test_rounding IS
        l_result  NUMBER;
    BEGIN
        l_result := calculate_discount(99.99, 'PLATINUM');
        assert_equals(20, l_result, 'Rounding - 99.99 * 20% = 20.00');
    END test_rounding;
    
    PROCEDURE run_all_tests IS
    BEGIN
        DBMS_OUTPUT.PUT_LINE('==========================================');
        DBMS_OUTPUT.PUT_LINE('Running Unit Tests for calculate_discount');
        DBMS_OUTPUT.PUT_LINE('==========================================');
        DBMS_OUTPUT.PUT_LINE('');
        
        -- Reset counters
        g_test_count := 0;
        g_pass_count := 0;
        g_fail_count := 0;
        
        -- Run tests
        test_platinum_discount;
        test_gold_discount;
        test_silver_discount;
        test_bronze_discount;
        test_invalid_tier;
        test_zero_amount;
        test_rounding;
        
        -- Summary
        DBMS_OUTPUT.PUT_LINE('');
        DBMS_OUTPUT.PUT_LINE('==========================================');
        DBMS_OUTPUT.PUT_LINE('Test Summary:');
        DBMS_OUTPUT.PUT_LINE('  Total Tests: ' || g_test_count);
        DBMS_OUTPUT.PUT_LINE('  Passed: ' || g_pass_count);
        DBMS_OUTPUT.PUT_LINE('  Failed: ' || g_fail_count);
        DBMS_OUTPUT.PUT_LINE('  Success Rate: ' || 
            ROUND((g_pass_count / g_test_count) * 100, 2) || '%');
        DBMS_OUTPUT.PUT_LINE('==========================================');
        
    END run_all_tests;
    
END test_calculate_discount;
/

-- Esegui i test
SET SERVEROUTPUT ON
EXEC test_calculate_discount.run_all_tests;
```

## Best Practices PL/SQL

### 1. Exception Handling

✅ **DO:**
```sql
BEGIN
    -- Your code
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        -- Handle specific exception
    WHEN OTHERS THEN
        -- Log error
        -- Re-raise or handle gracefully
        RAISE;
END;
```

### 2. Use Bulk Operations

✅ **DO:** BULK COLLECT e FORALL per grandi dataset
❌ **DON'T:** Loop row-by-row per migliaia di record

### 3. Autonomous Transactions

✅ **DO:** Usa PRAGMA AUTONOMOUS_TRANSACTION per logging
❌ **DON'T:** Usa per business logic normale

### 4. Naming Conventions

```
p_  = Parameter
l_  = Local variable
g_  = Global variable
c_  = Constant
e_  = Exception
```

### 5. Performance Tips

- Usa bind variables
- Limita context switches SQL/PL-SQL
- Cache valori statici
- Usa NOCOPY per grandi collection OUT parameters
- Analizza con DBMS_PROFILER

## Debugging

### Abilitare Debug Output

```sql
SET SERVEROUTPUT ON SIZE UNLIMITED
DBMS_OUTPUT.PUT_LINE('Debug message');
```

### Usare DBMS_DEBUG

```sql
BEGIN
    DBMS_DEBUG.INITIALIZE();
    DBMS_DEBUG.DEBUG_ON();
    -- Your code
    DBMS_DEBUG.DEBUG_OFF();
END;
```

## Risorse

- [Oracle PL/SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/)
- [PL/SQL Best Practices](https://oracle-base.com/articles/misc/plsql-best-practices)
- [Knowledge Base: PL/SQL Patterns](../knowledge/plsql-patterns.md)

---

**Note**: Questo agent fornisce supporto completo per sviluppo PL/SQL enterprise-grade. Per query SQL più semplici, vedi [Esempio 1: Oracle Query Assistant](oracle-query-assistant.md).
