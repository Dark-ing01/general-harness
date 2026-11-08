# Database Design & Review

## Purpose

Design, review, and improve relational database models following established database engineering principles. Prioritize data integrity, consistency, maintainability, and query performance over unnecessary complexity.

## Core Principles

### 1. Understand Before Designing

- Understand business requirements and domain entities before creating tables.
- Identify entities, attributes, relationships, cardinalities, and business constraints.
- Never introduce tables, columns, or relationships without a clear purpose.
- Identify expected query patterns and data access requirements.

### 2. Relational Modeling

- Use appropriate primary keys for every table.
- Define foreign keys to enforce referential integrity.
- Choose appropriate data types and avoid unnecessarily large types.
- Define `NOT NULL`, `UNIQUE`, `CHECK`, and other constraints where applicable.
- Model one-to-one, one-to-many, and many-to-many relationships correctly.
- Use junction tables for many-to-many relationships.
- Avoid storing multiple values in a single column.
- Avoid duplicated or derived data unless there is a justified performance or business requirement.

### 3. Database Normalization

- Apply First Normal Form (1NF): atomic values and no repeating groups.
- Apply Second Normal Form (2NF): eliminate partial dependencies on composite keys.
- Apply Third Normal Form (3NF): eliminate transitive dependencies.
- Consider BCNF when functional dependencies require stricter normalization.
- Denormalize only when there is a measurable or justified requirement.
- Document intentional denormalization and its consistency implications.

### 4. Indexing Strategy

- Evaluate indexes based on actual query patterns.
- Ensure primary keys and unique constraints have appropriate indexes.
- Consider indexes on frequently queried foreign keys.
- Index columns frequently used in filtering, joining, and sorting.
- Evaluate composite indexes based on column order and query predicates.
- Avoid redundant, overlapping, or unnecessary indexes.
- Consider the write-performance and storage costs of every index.
- Do not index every column by default.

### 5. Data Integrity & Consistency

- Prefer database constraints over relying exclusively on application-level validation.
- Define appropriate `ON DELETE` and `ON UPDATE` behaviors.
- Use transactions for operations that must remain atomic.
- Define appropriate default values.
- Handle nullable attributes intentionally.
- Avoid ambiguous states and invalid combinations of values.

### 6. Naming Conventions

- Use consistent, descriptive, and predictable naming.
- Prefer singular or plural table names consistently across the schema.
- Use `snake_case` unless the project defines another convention.
- Name primary and foreign keys consistently.
- Avoid reserved SQL keywords and ambiguous abbreviations.

### 7. Performance & Scalability

- Consider expected data volume and growth.
- Avoid premature optimization.
- Review expensive joins and potentially unbounded queries.
- Consider pagination for large result sets.
- Evaluate partitioning only when data volume and access patterns justify it.
- Consider database-specific capabilities when they provide measurable benefits.

## Review Workflow

When reviewing an existing database model:

1. Understand the domain and business requirements.
2. Inspect tables, columns, relationships, and constraints.
3. Identify normalization violations and unnecessary duplication.
4. Review primary keys, foreign keys, and referential integrity.
5. Evaluate indexes against expected query patterns.
6. Identify potential performance and scalability issues.
7. Present findings classified by severity.
8. Propose concrete improvements with their rationale.

## Development Workflow

When creating or modifying a database model:

1. Define entities and their responsibilities.
2. Identify relationships and cardinalities.
3. Define primary keys and foreign keys.
4. Apply normalization principles.
5. Define constraints and data types.
6. Design indexes based on expected queries.
7. Generate migrations or DDL following project conventions.
8. Validate the resulting model against business requirements.

## Rules for the Agent

- Never generate a database schema without understanding the domain.
- Never introduce indexes without explaining their purpose.
- Never remove constraints or integrity rules without justification.
- Never denormalize without explaining the trade-offs.
- Never assume database-specific behavior is portable across engines.
- Prefer simple, explicit relational models over unnecessary abstractions.
- Preserve backward compatibility when modifying existing schemas.
- Always consider migration safety and existing production data.
- Explain significant design decisions and trade-offs.

## Review Output

When performing a review, organize findings into:

- **Critical:** Data integrity violations or risks of data loss.
- **High:** Significant modeling or performance problems.
- **Medium:** Maintainability, consistency, or optimization concerns.
- **Low:** Naming, conventions, and minor improvements.

For each finding, include:

- Location (table, column, relationship, or index).
- Problem description.
- Potential impact.
- Recommended solution.

Prioritize actionable recommendations over theoretical observations.
