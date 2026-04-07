# 11. SQL DATABASES (PostgreSQL, MySQL)

## 11.1 Core SQL Concepts

### Data Types (PostgreSQL)
```sql
-- Numbers
INTEGER, BIGINT, DECIMAL(10,2), FLOAT, SERIAL (auto-increment)

-- Strings
VARCHAR(255), TEXT, CHAR(10)

-- Date/Time
DATE, TIMESTAMP, TIMESTAMPTZ, INTERVAL

-- Boolean
BOOLEAN

-- JSON (PostgreSQL special)
JSON, JSONB (binary, indexable, faster queries)

-- Arrays (PostgreSQL special)
INTEGER[], TEXT[]

-- UUID
UUID
```

### CRUD Operations
```sql
-- Create
INSERT INTO users (name, email, age) VALUES ('Saad', 'saad@email.com', 28);

-- Read
SELECT * FROM users WHERE age > 25 ORDER BY name ASC LIMIT 10 OFFSET 0;

-- Update
UPDATE users SET name = 'Saad Sohail' WHERE id = 1;

-- Delete
DELETE FROM users WHERE id = 1;
```

### JOINs (Critical Interview Topic)

```sql
-- INNER JOIN — only matching rows from both tables
SELECT u.name, p.title
FROM users u
INNER JOIN posts p ON u.id = p.user_id;

-- LEFT JOIN — all rows from left table + matching from right
SELECT u.name, p.title
FROM users u
LEFT JOIN posts p ON u.id = p.user_id;
-- Users without posts will show with NULL for title

-- RIGHT JOIN — all rows from right table + matching from left
SELECT u.name, p.title
FROM users u
RIGHT JOIN posts p ON u.id = p.user_id;

-- FULL OUTER JOIN — all rows from both tables
SELECT u.name, p.title
FROM users u
FULL OUTER JOIN posts p ON u.id = p.user_id;
```

```
INNER JOIN:     LEFT JOIN:      RIGHT JOIN:     FULL OUTER JOIN:
  ┌───┐           ┌───┐           ┌───┐           ┌───┐
  │ A │∩│ B │     █ A █∩│ B │     │ A │∩█ B █     █ A █∩█ B █
  └───┘           └───┘           └───┘           └───┘
Only matching   All A +          All B +         All from
rows            matching B       matching A      both tables
```

### Indexes
```sql
-- B-Tree index (default, most common)
CREATE INDEX idx_users_email ON users(email);

-- Unique index
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Composite index
CREATE INDEX idx_users_name_age ON users(name, age);
-- Follows "leftmost prefix" rule: this index works for
-- queries on (name), (name, age), but NOT (age) alone

-- Partial index (PostgreSQL)
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- GIN index (for JSONB, arrays, full-text search)
CREATE INDEX idx_user_data ON users USING GIN(metadata);
```

**Interview Q: When NOT to add indexes?**
- Small tables (< 1000 rows)
- Columns with low cardinality (boolean, status with 2-3 values)
- Tables with heavy write operations (indexes slow down INSERT/UPDATE)
- Columns rarely used in WHERE/JOIN/ORDER BY

### Transactions & ACID
```sql
BEGIN;

UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;

-- If anything fails between BEGIN and COMMIT, nothing is saved
COMMIT;  -- or ROLLBACK; to undo
```

**ACID Properties:**
- **Atomicity** — All or nothing. Either all operations succeed or none do.
- **Consistency** — Database moves from one valid state to another.
- **Isolation** — Concurrent transactions don't interfere with each other.
- **Durability** — Once committed, data survives crashes.

### Aggregate Functions
```sql
SELECT
  department,
  COUNT(*) as total,
  AVG(salary) as avg_salary,
  MAX(salary) as max_salary,
  MIN(salary) as min_salary,
  SUM(salary) as total_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000
ORDER BY avg_salary DESC;
```

### Subqueries vs JOINs
```sql
-- Subquery (slower for large datasets)
SELECT * FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 1000);

-- JOIN equivalent (usually faster)
SELECT DISTINCT u.*
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE o.total > 1000;
```

### Window Functions (Advanced)
```sql
-- ROW_NUMBER, RANK, DENSE_RANK
SELECT
  name,
  department,
  salary,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rank
FROM employees;

-- Running total
SELECT
  date,
  amount,
  SUM(amount) OVER (ORDER BY date) as running_total
FROM transactions;
```

### Normalization

| Form | Rule | Example |
|------|------|---------|
| **1NF** | No repeating groups, atomic values | Split "skills: JS, Python" into separate rows |
| **2NF** | 1NF + no partial dependencies | Remove columns that depend on part of composite key |
| **3NF** | 2NF + no transitive dependencies | If A→B and B→C, don't store C in same table |

