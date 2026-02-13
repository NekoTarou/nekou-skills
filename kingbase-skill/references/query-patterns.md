# KingBase Query Patterns

This reference provides common SQL patterns and examples for working with KingBase databases through MCP.

## Table of Contents

1. [Basic Queries](#basic-queries)
2. [Filtering and Pagination](#filtering-and-pagination)
3. [Joins](#joins)
4. [Aggregations](#aggregations)
5. [Subqueries](#subqueries)
6. [Window Functions](#window-functions)
7. [Date/Time Operations](#datetime-operations)
8. [Schema Operations](#schema-operations)
9. [Performance Queries](#performance-queries)

## Basic Queries

### Select All Columns
```sql
SELECT * FROM users LIMIT 100;
```

### Select Specific Columns
```sql
SELECT id, name, email, created_at FROM users;
```

### Select with Aliases
```sql
SELECT 
    u.id AS user_id,
    u.name AS user_name,
    u.email AS user_email
FROM users u;
```

### Distinct Values
```sql
SELECT DISTINCT status FROM orders;
```

### Ordering Results
```sql
-- Single column
SELECT * FROM users ORDER BY created_at DESC;

-- Multiple columns
SELECT * FROM orders ORDER BY status ASC, created_at DESC;
```

## Filtering and Pagination

### Basic WHERE Clause
```sql
-- Exact match
SELECT * FROM users WHERE status = 'active';

-- Multiple conditions (AND)
SELECT * FROM users WHERE status = 'active' AND created_at > '2024-01-01';

-- Multiple conditions (OR)
SELECT * FROM users WHERE status = 'active' OR status = 'pending';

-- NOT condition
SELECT * FROM users WHERE status != 'inactive';
```

### Pattern Matching
```sql
-- Starts with
SELECT * FROM users WHERE name LIKE 'John%';

-- Ends with
SELECT * FROM users WHERE email LIKE '%@example.com';

-- Contains
SELECT * FROM users WHERE name LIKE '%Smith%';

-- Case-insensitive
SELECT * FROM users WHERE name ILIKE '%john%';
```

### Range Queries
```sql
-- Date range
SELECT * FROM orders WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31';

-- Numeric range
SELECT * FROM products WHERE price BETWEEN 10 AND 100;

-- IN operator
SELECT * FROM users WHERE status IN ('active', 'pending', 'verified');
```

### NULL Handling
```sql
-- IS NULL
SELECT * FROM users WHERE deleted_at IS NULL;

-- IS NOT NULL
SELECT * FROM users WHERE phone IS NOT NULL;

-- COALESCE (handle NULL with default)
SELECT COALESCE(phone, 'N/A') AS phone FROM users;
```

### Pagination
```sql
-- Limit results
SELECT * FROM users LIMIT 50;

-- Offset for pagination
SELECT * FROM users LIMIT 50 OFFSET 100;

-- Alternative syntax
SELECT * FROM users LIMIT 50 OFFSET 100;
```

## Joins

### INNER JOIN
```sql
SELECT 
    u.name,
    o.order_id,
    o.total_amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE u.status = 'active';
```

### LEFT JOIN
```sql
-- All users with their orders (including users without orders)
SELECT 
    u.name,
    o.order_id,
    o.total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

### Multiple Joins
```sql
SELECT 
    u.name AS customer_name,
    o.order_id,
    p.name AS product_name,
    oi.quantity
FROM users u
INNER JOIN orders o ON u.id = o.user_id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id;
```

### Self Join
```sql
-- Find employees and their managers
SELECT 
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

## Aggregations

### COUNT
```sql
-- Total count
SELECT COUNT(*) FROM users;

-- Count with condition
SELECT COUNT(*) FROM users WHERE status = 'active';

-- Count distinct
SELECT COUNT(DISTINCT status) FROM users;
```

### SUM, AVG, MIN, MAX
```sql
SELECT 
    COUNT(*) AS total_orders,
    SUM(total_amount) AS total_revenue,
    AVG(total_amount) AS avg_order_value,
    MIN(total_amount) AS min_order,
    MAX(total_amount) AS max_order
FROM orders
WHERE created_at >= '2024-01-01';
```

### GROUP BY
```sql
-- Group by single column
SELECT 
    status,
    COUNT(*) AS count,
    AVG(total_amount) AS avg_amount
FROM orders
GROUP BY status;

-- Group by multiple columns
SELECT 
    status,
    DATE(created_at) AS order_date,
    COUNT(*) AS count,
    SUM(total_amount) AS daily_revenue
FROM orders
GROUP BY status, DATE(created_at)
ORDER BY order_date DESC;
```

### HAVING Clause
```sql
-- Filter groups
SELECT 
    user_id,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_spent
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

## Subqueries

### Subquery in WHERE
```sql
-- Find users who have placed orders
SELECT * FROM users 
WHERE id IN (SELECT DISTINCT user_id FROM orders);

-- Find users who haven't placed orders
SELECT * FROM users 
WHERE id NOT IN (SELECT DISTINCT user_id FROM orders);
```

### Correlated Subquery
```sql
-- Find users with above-average order counts
SELECT u.name,
    (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count
FROM users u
WHERE (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) > 
    (SELECT AVG(order_count) FROM (SELECT COUNT(*) AS order_count FROM orders GROUP BY user_id) sub);
```

### Subquery in FROM
```sql
-- Query from subquery result
SELECT 
    status,
    COUNT(*) AS count
FROM (
    SELECT * FROM orders WHERE created_at >= '2024-01-01'
) recent_orders
GROUP BY status;
```

## Window Functions

### ROW_NUMBER
```sql
-- Rank orders by amount within each status
SELECT 
    order_id,
    status,
    total_amount,
    ROW_NUMBER() OVER (PARTITION BY status ORDER BY total_amount DESC) AS rank
FROM orders;
```

### RANK and DENSE_RANK
```sql
-- Rank products by price
SELECT 
    name,
    price,
    RANK() OVER (ORDER BY price DESC) AS price_rank,
    DENSE_RANK() OVER (ORDER BY price DESC) AS price_dense_rank
FROM products;
```

### Running Totals
```sql
-- Cumulative sum of orders
SELECT 
    order_id,
    created_at,
    total_amount,
    SUM(total_amount) OVER (ORDER BY created_at) AS running_total
FROM orders;
```

### Moving Averages
```sql
-- 7-day moving average
SELECT 
    DATE(created_at) AS date,
    COUNT(*) AS daily_orders,
    AVG(COUNT(*)) OVER (ORDER BY DATE(created_at) ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg
FROM orders
GROUP BY DATE(created_at);
```

### LAG and LEAD
```sql
-- Compare with previous day
SELECT 
    DATE(created_at) AS date,
    COUNT(*) AS daily_orders,
    LAG(COUNT(*)) OVER (ORDER BY DATE(created_at)) AS prev_day_orders,
    COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY DATE(created_at)) AS day_over_day_change
FROM orders
GROUP BY DATE(created_at);
```

## Date/Time Operations

### Date Functions
```sql
-- Current date/time
SELECT CURRENT_DATE;
SELECT CURRENT_TIMESTAMP;
SELECT NOW();

-- Date parts
SELECT 
    created_at,
    EXTRACT(YEAR FROM created_at) AS year,
    EXTRACT(MONTH FROM created_at) AS month,
    EXTRACT(DAY FROM created_at) AS day,
    EXTRACT(DOW FROM created_at) AS day_of_week
FROM orders;
```

### Date Arithmetic
```sql
-- Add/subtract intervals
SELECT 
    created_at,
    created_at + INTERVAL '7 days' AS due_date,
    created_at - INTERVAL '1 month' AS one_month_ago
FROM orders;

-- Age calculation
SELECT 
    name,
    birth_date,
    AGE(CURRENT_DATE, birth_date) AS age
FROM users;

-- Date difference
SELECT 
    order_id,
    created_at,
    shipped_at,
    shipped_at - created_at AS processing_time
FROM orders
WHERE shipped_at IS NOT NULL;
```

### Date Formatting
```sql
-- Format dates (PostgreSQL/KingBase specific)
SELECT 
    created_at,
    TO_CHAR(created_at, 'YYYY-MM-DD') AS date_formatted,
    TO_CHAR(created_at, 'YYYY-MM-DD HH24:MI:SS') AS datetime_formatted
FROM orders;
```

### Date Truncation
```sql
-- Group by month
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS orders,
    SUM(total_amount) AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month;
```

## Schema Operations

### Create Table
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Create Index
```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Partial index
CREATE INDEX idx_orders_pending ON orders(created_at) WHERE status = 'pending';

-- Unique index
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
```

### Add Column
```sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users ADD COLUMN is_verified BOOLEAN DEFAULT FALSE;
```

### Modify Column
```sql
-- Change type
ALTER TABLE users ALTER COLUMN phone TYPE VARCHAR(30);

-- Add NOT NULL constraint
ALTER TABLE users ALTER COLUMN email SET NOT NULL;

-- Set default
ALTER TABLE users ALTER COLUMN status SET DEFAULT 'pending';
```

### Constraints
```sql
-- Add foreign key
ALTER TABLE orders ADD CONSTRAINT fk_orders_user 
    FOREIGN KEY (user_id) REFERENCES users(id);

-- Add check constraint
ALTER TABLE users ADD CONSTRAINT chk_status 
    CHECK (status IN ('active', 'inactive', 'pending'));

-- Drop constraint
ALTER TABLE users DROP CONSTRAINT chk_status;
```

## Performance Queries

### Find Missing Indexes
```sql
-- Check for sequential scans on large tables
SELECT 
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    idx_scan,
    n_tup_ins,
    n_tup_upd,
    n_tup_del
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_tup_read DESC;
```

### Table Statistics
```sql
-- Table sizes
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### Index Usage
```sql
-- Index usage statistics
SELECT 
    schemaname,
    tablename,
    indexrelname AS index_name,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

### Long-Running Queries
```sql
-- Check for active queries (requires appropriate permissions)
SELECT 
    pid,
    usename,
    application_name,
    state,
    query_start,
    NOW() - query_start AS duration,
    query
FROM pg_stat_activity
WHERE state != 'idle'
AND query_start IS NOT NULL
ORDER BY query_start;
```

### Lock Monitoring
```sql
-- Check for locks
SELECT 
    l.locktype,
    l.relation::regclass,
    l.mode,
    l.granted,
    a.usename,
    a.query,
    a.pid
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE NOT l.granted;
```
