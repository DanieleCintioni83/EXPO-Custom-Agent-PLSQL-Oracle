# PL/SQL Best Practices

## Code Organization

### Package Structure
```sql
-- Separate specification from body
-- Specification: Public interface
CREATE OR REPLACE PACKAGE pkg_name AS
    -- Constants
    c_constant_name CONSTANT VARCHAR2(50) := 'VALUE';
    
    -- Types
    TYPE type_name IS RECORD (...);
    
    -- Exceptions
    e_exception_name EXCEPTION;
    
    -- Functions and Procedures
    FUNCTION func_name RETURN type;
    PROCEDURE proc_name (params);
END pkg_name;
/

-- Body: Implementation
CREATE OR REPLACE PACKAGE BODY pkg_name AS
    -- Private variables
    g_private_var VARCHAR2(100);
    
    -- Private procedures
    PROCEDURE private_proc IS
    BEGIN
        NULL;
    END;
    
    -- Public implementations
    FUNCTION func_name RETURN type IS
    BEGIN
        RETURN value;
    END;
    
END pkg_name;
/
```

## Naming Conventions

```
p_   - Parameters (IN, OUT, IN OUT)
l_   - Local variables
g_   - Global variables (package-level)
c_   - Constants
e_   - Exceptions
t_   - Types
v_   - Views
idx_ - Indexes
seq_ - Sequences
trg_ - Triggers
pkg_ - Packages
```

## Exception Handling

### Comprehensive Exception Handling
```sql
DECLARE
    e_custom_exception EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_custom_exception, -20001);
BEGIN
    -- Your code here
    
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        -- Handle specific exception
        log_error('No data found', 'PROCEDURE_NAME');
        RAISE;
        
    WHEN TOO_MANY_ROWS THEN
        -- Handle specific exception
        log_error('Multiple rows returned', 'PROCEDURE_NAME');
        RAISE;
        
    WHEN e_custom_exception THEN
        -- Handle custom exception
        ROLLBACK;
        log_error(SQLERRM, 'PROCEDURE_NAME');
        
    WHEN OTHERS THEN
        -- Catch-all for unexpected errors
        ROLLBACK;
        log_error('Unexpected error: ' || SQLERRM, 'PROCEDURE_NAME');
        RAISE;
END;
```

## Performance Optimization

### Use Bulk Operations
```sql
-- BAD: Row-by-row processing
FOR rec IN (SELECT * FROM large_table) LOOP
    UPDATE another_table SET col = rec.val WHERE id = rec.id;
END LOOP;

-- GOOD: Bulk operations
DECLARE
    TYPE id_table IS TABLE OF large_table.id%TYPE;
    TYPE val_table IS TABLE OF large_table.val%TYPE;
    
    l_ids   id_table;
    l_vals  val_table;
BEGIN
    SELECT id, val
    BULK COLLECT INTO l_ids, l_vals
    FROM large_table;
    
    FORALL i IN 1..l_ids.COUNT
        UPDATE another_table 
        SET col = l_vals(i) 
        WHERE id = l_ids(i);
END;
```

### Limit Bulk Collect
```sql
DECLARE
    CURSOR c_data IS SELECT * FROM large_table;
    TYPE data_table IS TABLE OF c_data%ROWTYPE;
    l_data data_table;
    c_limit CONSTANT PLS_INTEGER := 1000;
BEGIN
    OPEN c_data;
    LOOP
        FETCH c_data BULK COLLECT INTO l_data LIMIT c_limit;
        EXIT WHEN l_data.COUNT = 0;
        
        -- Process batch
        FORALL i IN 1..l_data.COUNT
            -- Your bulk operation
            NULL;
            
    END LOOP;
    CLOSE c_data;
END;
```

## Avoid Common Pitfalls

### 1. Don't Use SELECT * in Production
```sql
-- BAD
SELECT * INTO l_rec FROM employees WHERE id = 100;

-- GOOD
SELECT 
    employee_id, 
    first_name, 
    last_name, 
    email
INTO 
    l_id, 
    l_first_name, 
    l_last_name, 
    l_email
FROM 
    employees 
WHERE 
    employee_id = 100;
```

### 2. Use %TYPE and %ROWTYPE
```sql
-- BAD: Hard-coded types
DECLARE
    l_name VARCHAR2(100);
    l_salary NUMBER(10,2);
BEGIN
    NULL;
END;

-- GOOD: Use %TYPE
DECLARE
    l_name employees.first_name%TYPE;
    l_salary employees.salary%TYPE;
    l_emp_rec employees%ROWTYPE;
BEGIN
    NULL;
END;
```

### 3. Avoid Implicit Cursors for Large Data
```sql
-- BAD: Implicit cursor for large dataset
BEGIN
    FOR rec IN (SELECT * FROM large_table) LOOP
        -- Process
    END LOOP;
END;

-- GOOD: Explicit cursor with bulk collect
DECLARE
    CURSOR c_data IS SELECT * FROM large_table;
    TYPE data_table IS TABLE OF c_data%ROWTYPE;
    l_data data_table;
BEGIN
    OPEN c_data;
    FETCH c_data BULK COLLECT INTO l_data LIMIT 1000;
    CLOSE c_data;
    
    FOR i IN 1..l_data.COUNT LOOP
        -- Process
        NULL;
    END LOOP;
END;
```

## Security Best Practices

### 1. Use Bind Variables
```sql
-- BAD: Dynamic SQL with concatenation (SQL Injection risk!)
DECLARE
    l_sql VARCHAR2(1000);
    l_name VARCHAR2(100) := p_user_input;
BEGIN
    l_sql := 'SELECT * FROM users WHERE name = ''' || l_name || '''';
    EXECUTE IMMEDIATE l_sql;
END;

-- GOOD: Bind variables
DECLARE
    l_sql VARCHAR2(1000);
    l_cursor SYS_REFCURSOR;
BEGIN
    l_sql := 'SELECT * FROM users WHERE name = :name';
    OPEN l_cursor FOR l_sql USING p_user_input;
END;
```

### 2. Validate Input
```sql
CREATE OR REPLACE PROCEDURE create_user (
    p_username IN VARCHAR2,
    p_email    IN VARCHAR2
) IS
BEGIN
    -- Validate username
    IF p_username IS NULL OR LENGTH(p_username) < 3 THEN
        RAISE_APPLICATION_ERROR(-20001, 'Username must be at least 3 characters');
    END IF;
    
    -- Validate email format
    IF NOT REGEXP_LIKE(p_email, '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$') THEN
        RAISE_APPLICATION_ERROR(-20002, 'Invalid email format');
    END IF;
    
    -- Sanitize input
    INSERT INTO users (username, email)
    VALUES (TRIM(p_username), LOWER(TRIM(p_email)));
END;
```

## Transaction Management

### Proper Commit/Rollback
```sql
CREATE OR REPLACE PROCEDURE process_order (
    p_order_id IN NUMBER
) IS
    l_success BOOLEAN := FALSE;
BEGIN
    -- Start transaction
    SAVEPOINT start_order;
    
    -- Step 1
    UPDATE inventory SET quantity = quantity - 1;
    
    -- Step 2
    INSERT INTO orders VALUES (...);
    
    -- Step 3
    UPDATE customer_balance SET balance = balance - total;
    
    -- All steps successful
    COMMIT;
    l_success := TRUE;
    
EXCEPTION
    WHEN OTHERS THEN
        -- Rollback to savepoint
        ROLLBACK TO start_order;
        
        -- Log error
        INSERT INTO error_log (error_msg, error_date)
        VALUES (SQLERRM, SYSDATE);
        COMMIT;  -- Commit error log only
        
        -- Re-raise
        RAISE;
END;
```

## Autonomous Transactions

### Use for Logging
```sql
CREATE OR REPLACE PROCEDURE log_error (
    p_error_msg IN VARCHAR2,
    p_location  IN VARCHAR2
) IS
    PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN
    INSERT INTO error_log (
        error_message,
        location,
        error_date,
        username
    ) VALUES (
        p_error_msg,
        p_location,
        SYSDATE,
        USER
    );
    
    COMMIT;  -- Independent of main transaction
END;
```

## Documentation

### Comment Your Code
```sql
/*
 * Package: customer_mgmt_pkg
 * Purpose: Manage customer CRUD operations
 * Author: Developer Name
 * Created: 2024-01-15
 * Modified: 2024-02-17
 *
 * Change History:
 *   2024-02-17 - Added email validation
 *   2024-01-20 - Initial version
 */
CREATE OR REPLACE PACKAGE customer_mgmt_pkg AS
    
    /**
     * Creates a new customer
     * @param p_name Customer name
     * @param p_email Customer email
     * @return Customer ID
     * @throws e_invalid_email if email format is invalid
     */
    FUNCTION create_customer (
        p_name  IN VARCHAR2,
        p_email IN VARCHAR2
    ) RETURN NUMBER;
    
END customer_mgmt_pkg;
```

## Testing

### Unit Test Structure
```sql
CREATE OR REPLACE PACKAGE test_package AS
    PROCEDURE run_all_tests;
    PROCEDURE test_case_1;
    PROCEDURE test_case_2;
END test_package;
/

CREATE OR REPLACE PACKAGE BODY test_package AS
    
    g_pass_count NUMBER := 0;
    g_fail_count NUMBER := 0;
    
    PROCEDURE assert_equals (
        p_expected IN VARCHAR2,
        p_actual   IN VARCHAR2,
        p_message  IN VARCHAR2
    ) IS
    BEGIN
        IF p_expected = p_actual THEN
            g_pass_count := g_pass_count + 1;
            DBMS_OUTPUT.PUT_LINE('PASS: ' || p_message);
        ELSE
            g_fail_count := g_fail_count + 1;
            DBMS_OUTPUT.PUT_LINE('FAIL: ' || p_message);
            DBMS_OUTPUT.PUT_LINE('  Expected: ' || p_expected);
            DBMS_OUTPUT.PUT_LINE('  Got: ' || p_actual);
        END IF;
    END;
    
    PROCEDURE test_case_1 IS
    BEGIN
        assert_equals('expected', 'actual', 'Test description');
    END;
    
    PROCEDURE run_all_tests IS
    BEGIN
        g_pass_count := 0;
        g_fail_count := 0;
        
        test_case_1;
        test_case_2;
        
        DBMS_OUTPUT.PUT_LINE('Tests Passed: ' || g_pass_count);
        DBMS_OUTPUT.PUT_LINE('Tests Failed: ' || g_fail_count);
    END;
    
END test_package;
```
