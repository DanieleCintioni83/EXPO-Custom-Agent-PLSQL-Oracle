# Common Oracle SQL Queries - Knowledge Base

## SELECT Queries

### Basic Select with Joins
```sql
SELECT 
    e.employee_id,
    e.first_name || ' ' || e.last_name AS full_name,
    d.department_name,
    j.job_title,
    e.salary
FROM 
    employees e
    INNER JOIN departments d ON e.department_id = d.department_id
    INNER JOIN jobs j ON e.job_id = j.job_id
WHERE 
    e.hire_date >= ADD_MONTHS(SYSDATE, -12)
ORDER BY 
    e.hire_date DESC;
```

### Aggregation with GROUP BY
```sql
SELECT 
    d.department_name,
    COUNT(e.employee_id) AS employee_count,
    ROUND(AVG(e.salary), 2) AS avg_salary,
    MIN(e.salary) AS min_salary,
    MAX(e.salary) AS max_salary
FROM 
    departments d
    LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY 
    d.department_name
HAVING 
    COUNT(e.employee_id) > 5
ORDER BY 
    avg_salary DESC;
```

### Hierarchical Queries
```sql
SELECT 
    LEVEL,
    LPAD(' ', (LEVEL-1)*2, ' ') || employee_name AS org_chart,
    employee_id,
    manager_id,
    job_title
FROM 
    employees
START WITH 
    manager_id IS NULL
CONNECT BY PRIOR 
    employee_id = manager_id
ORDER SIBLINGS BY 
    employee_name;
```

## INSERT Operations

### Single Row Insert
```sql
INSERT INTO customers (
    customer_id,
    customer_name,
    email,
    phone,
    created_date
) VALUES (
    customers_seq.NEXTVAL,
    'John Doe',
    'john.doe@example.com',
    '+1-555-1234',
    SYSDATE
);
```

### Multi-Table Insert
```sql
INSERT ALL
    INTO sales_history (sale_id, product_id, amount, sale_date)
    VALUES (sale_id, product_id, amount, sale_date)
    INTO sales_summary (year, month, total_amount)
    VALUES (EXTRACT(YEAR FROM sale_date), EXTRACT(MONTH FROM sale_date), amount)
SELECT 
    sale_id,
    product_id,
    amount,
    sale_date
FROM 
    sales
WHERE 
    sale_date >= TRUNC(SYSDATE, 'YYYY');
```

## UPDATE Operations

### Conditional Update
```sql
UPDATE employees
SET 
    salary = salary * 1.1,
    last_updated = SYSDATE
WHERE 
    department_id = 50
    AND performance_rating >= 4;
```

### Update with Subquery
```sql
UPDATE employees e
SET 
    e.salary = (
        SELECT AVG(salary)
        FROM employees
        WHERE department_id = e.department_id
    )
WHERE 
    e.employee_id IN (
        SELECT employee_id
        FROM temp_salary_adjustments
    );
```

## DELETE Operations

### Conditional Delete
```sql
DELETE FROM order_items
WHERE order_id IN (
    SELECT order_id
    FROM orders
    WHERE order_date < ADD_MONTHS(SYSDATE, -24)
    AND status = 'CANCELLED'
);
```

## Date Functions

### Common Date Operations
```sql
SELECT 
    SYSDATE AS current_date,
    TRUNC(SYSDATE) AS date_only,
    TRUNC(SYSDATE, 'MONTH') AS first_day_of_month,
    LAST_DAY(SYSDATE) AS last_day_of_month,
    ADD_MONTHS(SYSDATE, 3) AS three_months_later,
    SYSDATE - 7 AS one_week_ago,
    MONTHS_BETWEEN(SYSDATE, hire_date) AS months_employed,
    TO_CHAR(SYSDATE, 'YYYY-MM-DD HH24:MI:SS') AS formatted_datetime
FROM 
    employees;
```

## String Functions

### Common String Operations
```sql
SELECT 
    customer_name,
    UPPER(customer_name) AS uppercase,
    LOWER(customer_name) AS lowercase,
    INITCAP(customer_name) AS titlecase,
    LENGTH(customer_name) AS name_length,
    SUBSTR(customer_name, 1, 3) AS first_three,
    INSTR(customer_name, 'Smith') AS position_of_smith,
    REPLACE(phone, '-', '') AS phone_no_dashes,
    TRIM(customer_name) AS trimmed,
    CONCAT(first_name, ' ' || last_name) AS full_name
FROM 
    customers;
```

## Analytical Functions

### Window Functions
```sql
SELECT 
    employee_id,
    department_id,
    salary,
    -- Ranking functions
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS row_num,
    RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dense_rank,
    -- Aggregate functions
    SUM(salary) OVER (PARTITION BY department_id) AS dept_total_salary,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg_salary,
    COUNT(*) OVER (PARTITION BY department_id) AS dept_employee_count,
    -- Lead/Lag
    LAG(salary, 1) OVER (ORDER BY hire_date) AS previous_hire_salary,
    LEAD(salary, 1) OVER (ORDER BY hire_date) AS next_hire_salary
FROM 
    employees;
```

## WITH Clause (Common Table Expressions)

### Recursive CTE
```sql
WITH RECURSIVE category_tree AS (
    -- Anchor member
    SELECT 
        category_id,
        category_name,
        parent_category_id,
        1 AS level,
        category_name AS path
    FROM 
        categories
    WHERE 
        parent_category_id IS NULL
    
    UNION ALL
    
    -- Recursive member
    SELECT 
        c.category_id,
        c.category_name,
        c.parent_category_id,
        ct.level + 1,
        ct.path || ' > ' || c.category_name
    FROM 
        categories c
        INNER JOIN category_tree ct ON c.parent_category_id = ct.category_id
)
SELECT * FROM category_tree
ORDER BY path;
```

## Performance Optimization Patterns

### Using INDEX hints
```sql
SELECT /*+ INDEX(e emp_dept_idx) */
    e.employee_id,
    e.employee_name
FROM 
    employees e
WHERE 
    e.department_id = 50;
```

### Using PARALLEL hint
```sql
SELECT /*+ PARALLEL(sales, 4) */
    product_id,
    SUM(quantity) AS total_quantity
FROM 
    sales
GROUP BY 
    product_id;
```

## Pagination

### Oracle 12c+ Syntax
```sql
SELECT 
    customer_id,
    customer_name,
    total_orders
FROM 
    customers
ORDER BY 
    total_orders DESC
OFFSET 20 ROWS 
FETCH NEXT 10 ROWS ONLY;
```

### Pre-12c Syntax
```sql
SELECT * FROM (
    SELECT 
        a.*,
        ROWNUM rnum
    FROM (
        SELECT 
            customer_id,
            customer_name,
            total_orders
        FROM 
            customers
        ORDER BY 
            total_orders DESC
    ) a
    WHERE ROWNUM <= 30
)
WHERE rnum > 20;
```

## MERGE Statement

### Upsert Operation
```sql
MERGE INTO customers_summary cs
USING (
    SELECT 
        customer_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent
    FROM 
        orders
    WHERE 
        order_date >= TRUNC(SYSDATE, 'MONTH')
    GROUP BY 
        customer_id
) o
ON (cs.customer_id = o.customer_id)
WHEN MATCHED THEN
    UPDATE SET
        cs.monthly_orders = o.order_count,
        cs.monthly_spent = o.total_spent,
        cs.last_updated = SYSDATE
WHEN NOT MATCHED THEN
    INSERT (
        customer_id,
        monthly_orders,
        monthly_spent,
        last_updated
    ) VALUES (
        o.customer_id,
        o.order_count,
        o.total_spent,
        SYSDATE
    );
```
