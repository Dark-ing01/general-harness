# Relational Database Indexing

## Purpose

This document defines practical guidelines for designing, reviewing, and optimizing indexes in relational databases.

The objective is to improve query performance while balancing storage consumption, write overhead, and database maintenance costs.

Indexes must be introduced based on actual query patterns and measurable requirements, not speculation.

---

## 1. Fundamental Concepts

### What Is an Index?

An index is a data structure maintained by the database engine to accelerate data retrieval.

Instead of scanning every row in a table, the database may use an index to locate relevant records more efficiently.

Indexes introduce additional costs:

- Storage consumption.
- Additional work during INSERT operations.
- Additional work during UPDATE operations affecting indexed columns.
- Additional work during DELETE operations.
- Index maintenance and fragmentation, depending on the database engine.

### Selectivity

Selectivity describes how effectively an index distinguishes between rows.

A highly selective column has many distinct values relative to the total number of rows.

Examples:

| Column | Typical selectivity |
|---|---|
| Email | High |
| Employee ID | High |
| Country code | Low |
| Boolean status | Low |

High selectivity often makes a column a good indexing candidate, but selectivity alone does not determine whether an index is useful.

The database optimizer also considers query patterns, table size, statistics, and estimated execution costs.

### Cardinality

Cardinality refers to the number of distinct values in a column or combination of columns.

Higher cardinality can improve the usefulness of an index for selective queries.

Always distinguish between cardinality and selectivity when evaluating index candidates.

---

## 2. Primary Key Indexes

Every relational table should have a reliable primary key.

Primary key constraints generally create an associated unique index, although the exact implementation depends on the database engine.

Guidelines:

- Define a primary key for every table.
- Prefer stable identifiers.
- Avoid mutable business attributes as primary keys.
- Evaluate composite primary keys when the domain naturally requires them.
- Avoid creating an additional index that duplicates the primary key index.

Example:

```sql
CREATE TABLE employees (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) NOT NULL
);
```

The primary key already provides an index in common relational database engines.

---

## 3. Unique Indexes

Unique indexes enforce uniqueness and can accelerate lookups.

Use them when a business rule requires that a value or combination of values must be unique.

Example:

```sql
CREATE UNIQUE INDEX uq_employees_email
ON employees (email);
```

Composite uniqueness:

```sql
CREATE UNIQUE INDEX uq_employee_company
ON employees (company_id, employee_number);
```

Guidelines:

- Use unique constraints or unique indexes to enforce actual business rules.
- Do not rely exclusively on application-level validation for uniqueness.
- Consider nullable-column behavior because it differs between database engines.
- Evaluate case sensitivity and collation requirements for textual identifiers.
- Do not create redundant unique indexes when an equivalent constraint already exists.

---

## 4. Foreign Key Indexes

Foreign keys maintain referential integrity but do not universally guarantee that an index is automatically created on the referencing columns.

Example:

```sql
CREATE TABLE departments (
    id BIGINT PRIMARY KEY
);

CREATE TABLE employees (
    id BIGINT PRIMARY KEY,
    department_id BIGINT NOT NULL,
    CONSTRAINT fk_employees_department
        FOREIGN KEY (department_id)
        REFERENCES departments (id)
);
```

Potential index:

```sql
CREATE INDEX idx_employees_department_id
ON employees (department_id);
```

Benefits:

- Faster joins between related tables.
- More efficient filtering by foreign key.
- Potentially more efficient parent-row updates and deletes.
- Reduced lookup overhead during referential integrity checks in engines that benefit from referencing-side indexes.

Guidelines:

- Review foreign keys as potential indexing candidates.
- Prioritize frequently joined or filtered relationships.
- Consider table size and relationship cardinality.
- Avoid blindly indexing every foreign key.
- Understand how the database engine handles foreign key enforcement.

---

## 5. Single-Column Indexes

Single-column indexes are appropriate when queries frequently filter, join, or sort using one column.

Example:

```sql
CREATE INDEX idx_employees_status
ON employees (status);
```

Typical query:

```sql
SELECT id, email
FROM employees
WHERE status = 'ACTIVE';
```

Guidelines:

- Identify frequently queried columns.
- Evaluate selectivity and table size.
- Consider whether the query returns a significant percentage of the table.
- Avoid indexes on columns that are rarely queried.
- Low-cardinality indexes are not automatically useless; their usefulness depends on data distribution and query patterns.

---

## 6. Composite Indexes

A composite index contains multiple columns.

Example:

```sql
CREATE INDEX idx_employees_department_status
ON employees (department_id, status);
```

This index can support queries such as:

```sql
SELECT id, email
FROM employees
WHERE department_id = 10;
```

And:

```sql
SELECT id, email
FROM employees
WHERE department_id = 10
  AND status = 'ACTIVE';
```

Depending on the database engine and index type, it may also support queries involving the leading column.

### Column Order

Column order is critical.

For a conventional B-tree composite index:

`(department_id, status, created_at)`

The index is generally most useful for queries that constrain the leading column or a contiguous prefix of indexed columns.

Examples:

- `department_id`
- `department_id, status`
- `department_id, status, created_at`

A query filtering only by `status` generally cannot efficiently use the same index as a leading-column lookup.

### Ordering Guidelines

When designing composite indexes:

1. Analyze the actual WHERE predicates.
2. Identify equality conditions.
3. Evaluate range conditions.
4. Consider ORDER BY and GROUP BY requirements.
5. Consider selectivity and data distribution.
6. Consider whether an existing index already covers the query.

Do not apply a universal rule such as always placing the most selective column first.

The optimal order depends on the query workload and database optimizer.

---

## 7. Covering Indexes

A covering index contains all the columns required by a query, allowing the database to retrieve the required information directly from the index in supported execution plans.

Example:

```sql
CREATE INDEX idx_employees_department
ON employees (department_id);
```

Some database engines support included columns:

```sql
CREATE INDEX idx_employees_department_covering
ON employees (department_id)
INCLUDE (email, status);
```

Query:

```sql
SELECT email, status
FROM employees
WHERE department_id = 10;
```

Guidelines:

- Use covering indexes for frequent and performance-sensitive queries.
- Avoid excessively wide indexes.
- Evaluate the additional storage and write costs.
- Verify actual index-only or covering behavior with execution plans.
- Remember that support and syntax vary by database engine.

---

## 8. Partial and Filtered Indexes

Partial or filtered indexes contain only rows matching a specific condition.

They can be useful when queries repeatedly access a small subset of a table.

Example (PostgreSQL):

```sql
CREATE INDEX idx_active_employees
ON employees (department_id)
WHERE status = 'ACTIVE';
```

Guidelines:

- Consider partial indexes when a stable predicate selects a relatively small subset of rows.
- Ensure queries use predicates compatible with the index condition.
- Evaluate whether the optimizer can prove that the query condition implies the index predicate.
- Consider write overhead for rows entering or leaving the indexed subset.
- Verify engine-specific syntax and limitations.

---

## 9. Indexes for Sorting and Range Queries

Indexes can improve queries involving ORDER BY and range predicates.

Example:

```sql
CREATE INDEX idx_employees_created_at
ON employees (created_at);
```

Query:

```sql
SELECT id, email
FROM employees
WHERE created_at >= '2026-01-01'
ORDER BY created_at;
```

Guidelines:

- Evaluate indexes for frequently executed sorting operations.
- Consider column order and sort direction in composite indexes.
- Consider whether the index can satisfy both filtering and ordering.
- Evaluate pagination patterns.
- Remember that range conditions can affect the usefulness of subsequent columns in a composite index.

---

## 10. Index Types

Different index types serve different workloads.

### B-tree

The general-purpose choice for relational workloads.

Common use cases:

- Equality comparisons.
- Range queries.
- Sorting.
- Ordered lookups.

### Hash

Designed primarily for equality lookups where supported.

Limitations:

- Generally unsuitable for range queries.
- Does not provide ordered traversal.

### Full-Text

Designed for searching textual content using language-aware tokenization and search capabilities.

Useful for:

- Document searches.
- Textual descriptions.
- Searchable content fields.

### Spatial

Designed for geographical and geometric data.

Useful for:

- Geographic coordinates.
- Spatial intersections.
- Distance-based queries.

### Other Specialized Indexes

Some engines provide specialized structures such as GIN, GiST, BRIN, and bitmap indexes.

Use specialized indexes only when the data type, query operators, and workload justify them.

Never assume index types or their capabilities are portable across database engines.

---

## 11. Indexing Anti-Patterns

Avoid the following mistakes:

### Indexing Every Column

Each index introduces storage and write overhead.

Create indexes only when justified by query patterns or integrity requirements.

### Redundant Indexes

Avoid creating indexes that duplicate existing primary key, unique, or composite indexes without a measurable benefit.

Example:

```sql
CREATE INDEX idx_employees_department
ON employees (department_id);

CREATE INDEX idx_employees_department_status
ON employees (department_id, status);
```

The first index may be redundant because the second index can generally support lookups using its leading column.

Verify actual workload requirements before removing either index.

### Excessive Composite Indexes

Avoid creating many overlapping composite indexes without evidence that each serves a distinct workload.

### Indexing Frequently Updated Columns Without Justification

Frequently modified indexed columns can increase write amplification and maintenance costs.

### Assuming Low-Cardinality Indexes Are Always Useless

A low-cardinality index can still be valuable when:

- A predicate selects a small subset of rows.
- Data distribution is highly skewed.
- A partial index is appropriate.
- The index supports other filtering or ordering requirements.

### Ignoring Existing Indexes

Always inspect the existing schema before creating additional indexes.

### Using Functions on Indexed Columns Without Evaluation

Expressions such as:

```sql
WHERE LOWER(email) = 'john@example.com'
```

May prevent the use of a conventional index on `email`, depending on the database engine.

Consider expression indexes or normalized searchable values when justified.

---

## 12. Query Analysis and Execution Plans

Never assume an index improves a query simply because it exists.

Use the database engine's execution plan tools.

Examples:

PostgreSQL:

```sql
EXPLAIN ANALYZE
SELECT *
FROM employees
WHERE department_id = 10;
```

MySQL:

```sql
EXPLAIN
SELECT *
FROM employees
WHERE department_id = 10;
```

Evaluate:

- Sequential or full table scans.
- Index scans and index-only scans.
- Estimated versus actual row counts.
- Join strategies.
- Sorting operations.
- Filter selectivity.
- Execution time.
- Buffer and I/O statistics when available.

Important:

A sequential scan is not inherently a problem. For small tables or queries returning a large percentage of rows, it can be more efficient than using an index.

Use execution plans to understand the optimizer's decisions rather than forcing index usage without evidence.

---

## 13. Indexing Review Checklist

### Existing Indexes

- [ ] Primary key indexes are present.
- [ ] Unique constraints are enforced appropriately.
- [ ] Existing indexes have been inspected.
- [ ] Redundant and overlapping indexes have been identified.

### Query Patterns

- [ ] Frequently executed queries have been identified.
- [ ] Filtering columns have been evaluated.
- [ ] Join conditions have been evaluated.
- [ ] Sorting and pagination requirements have been considered.
- [ ] Composite indexes reflect actual query patterns.

### Performance

- [ ] Index selectivity and data distribution have been considered.
- [ ] Execution plans have been reviewed when available.
- [ ] Index usage has been evaluated.
- [ ] Write overhead and storage costs have been considered.

### Maintenance

- [ ] Index naming follows project conventions.
- [ ] Index creation and removal are handled through migrations.
- [ ] Production index operations consider locking and availability.
- [ ] Unused indexes are evaluated before removal.

---

## 14. Practical Guidelines for Database Agents

When creating or reviewing indexes:

1. Understand the workload before proposing indexes.
2. Inspect existing indexes before creating new ones.
3. Identify the queries that each proposed index is intended to optimize.
4. Explain the expected benefit and trade-offs.
5. Consider equality, range, join, sorting, and pagination predicates.
6. Evaluate composite index column order carefully.
7. Avoid redundant indexes.
8. Consider the impact on INSERT, UPDATE, and DELETE operations.
9. Use execution plans when query performance is under investigation.
10. Verify syntax and behavior against the target database engine's official documentation.
11. Do not claim performance improvements without measurements or sufficient evidence.
12. Prefer a small, purposeful index set over an unnecessarily large one.

---

## 15. Final Design Principle

Indexes are performance optimization structures, not substitutes for good relational modeling or efficient queries.

A good indexing strategy:

- Supports actual application workloads.
- Minimizes unnecessary storage.
- Balances read and write performance.
- Avoids redundant structures.
- Is validated using execution plans and measurements.
- Evolves as application access patterns change.

Every index should have a clear purpose and a defensible engineering justification.
