# Relational Database Modeling

## Purpose

Define practical guidelines for designing and reviewing relational database schemas. Prioritize domain correctness, data integrity, simplicity, and maintainability.

## 1. Domain Modeling

- Identify business entities, attributes, relationships, and business rules before creating tables.
- Model business concepts, not application DTOs or API payloads.
- Assign attributes to the entity they logically describe.
- Avoid unnecessary tables, duplicated attributes, and ambiguous relationships.
- Document assumptions when requirements are incomplete.

## 2. Primary Keys

- Every table must have a reliable primary key.
- Prefer stable identifiers that do not change throughout an entity's lifecycle.
- Use surrogate keys when they simplify identification and relationships.
- Use natural keys when their uniqueness and stability justify them.
- Consider composite keys for naturally identified associative records.
- Enforce business-level uniqueness using UNIQUE constraints.

## 3. Foreign Keys

- Use foreign keys to enforce referential integrity.
- Reference primary keys or appropriate unique candidate keys.
- Ensure compatible data types between related columns.
- Define nullability according to business requirements.
- Follow consistent naming conventions, preferably `<referenced_entity>_id`.
- Evaluate indexes on referencing columns separately.

## 4. Relationship Cardinalities

### One-to-One (1:1)

Use a foreign key with a UNIQUE constraint on the dependent table.

- Ensure each child belongs to at most one parent.
- Use when entities have separate responsibilities, lifecycles, or access requirements.
- Avoid splitting entities unnecessarily.

### One-to-Many (1:N)

Place the foreign key on the many side.

Examples: Department → Employees, Customer → Orders.

Define whether the relationship is mandatory (`NOT NULL`) or optional (`NULL`).

### Many-to-Many (N:M)

Use an associative table containing foreign keys to both entities.

- Prevent duplicate associations using a composite primary key or UNIQUE constraint.
- Add attributes when they describe the relationship itself.
- Promote the associative table to a business entity when it has an independent lifecycle or business rules.

## 5. Constraints and Data Integrity

Use database constraints to enforce business invariants:

- `PRIMARY KEY`: Unique row identification.
- `FOREIGN KEY`: Referential integrity.
- `NOT NULL`: Mandatory attributes.
- `UNIQUE`: Business-level uniqueness.
- `CHECK`: Validates row-level conditions.
- `DEFAULT`: Provides meaningful default values.

Prefer database-level enforcement over relying exclusively on application validation.

Do not assume CHECK constraints can enforce rules involving other rows or tables.

## 6. Referential Actions

Choose deletion and update behavior intentionally.

- `RESTRICT` / `NO ACTION`: Prevent deletion when dependent records exist.
- `CASCADE`: Delete dependent records when the parent is deleted.
- `SET NULL`: Preserve dependent records without their previous relationship.
- `SET DEFAULT`: Use only when the default reference is valid and meaningful.

Rules:

- Avoid cascading deletes on important business or historical records.
- Prefer stable primary keys to avoid unnecessary key updates.
- Consider soft deletion when historical traceability is required.
- Remember that soft deletion does not replace foreign key constraints.

## 7. Data Types

Choose types according to semantic meaning, precision, and expected range.

- Use appropriate integer types for identifiers and counters.
- Use `DECIMAL` / `NUMERIC` for financial values requiring exact precision.
- Use floating-point types for approximate numerical calculations.
- Use appropriate date and timestamp types.
- Define timezone semantics explicitly.
- Use native BOOLEAN types when available.
- Avoid unnecessarily large or ambiguous data types.

## 8. Historical and Temporal Data

When the domain requires historical traceability:

- Preserve previous values instead of blindly overwriting them.
- Model effective dates and record timestamps explicitly.
- Distinguish business-valid time from record creation time.
- Define whether historical periods can overlap.
- Enforce temporal consistency where supported.

Examples: salary history, employment contracts, product prices, and account status changes.

## 9. Hierarchical Relationships

Use self-referencing foreign keys for simple parent-child structures.

Consider closure tables, materialized paths, or nested sets only when query patterns justify the additional complexity.

Prevent invalid cycles when the business domain requires a strict hierarchy.

## 10. Naming Conventions

Use consistent `snake_case` naming unless the project defines another convention.

Recommended conventions:

- Tables: consistent singular or plural naming.
- Primary keys: `id`.
- Foreign keys: `<entity>_id`.
- Indexes: `idx_<table>_<columns>`.
- Unique constraints: `uq_<table>_<columns>`.
- Foreign keys: `fk_<table>_<referenced_table>`.
- Check constraints: `chk_<table>_<rule>`.

Avoid reserved SQL keywords and ambiguous abbreviations.

## 11. Review Checklist

- [ ] Entities and attributes correctly represent the business domain.
- [ ] Every table has an appropriate primary key.
- [ ] Foreign keys enforce valid relationships.
- [ ] Cardinalities are correctly represented.
- [ ] Many-to-many relationships use associative tables.
- [ ] Mandatory and optional relationships are intentional.
- [ ] Business rules are enforced through appropriate constraints.
- [ ] Referential actions are explicitly considered.
- [ ] Data types match their semantic meaning.
- [ ] Historical data requirements are addressed.
- [ ] Naming conventions are consistent.
- [ ] The model avoids unnecessary complexity.

## 12. Agent Rules

- Understand the domain before generating a schema.
- Never invent business rules silently.
- Never omit foreign keys without explicit justification.
- Never introduce cascading deletes without evaluating their impact.
- Do not create tables solely to mirror application classes.
- Preserve historical and financial records when required.
- Prefer simple relational structures over unnecessary abstractions.
- Consider database-specific capabilities before generating DDL.
- Document significant assumptions and design decisions.

## Final Principle

Design relational schemas around business entities and relationships. Prioritize correctness, integrity, and maintainability over unnecessary abstraction or premature optimization.# Relational Database Modeling

## Purpose

Define practical guidelines for designing and reviewing relational database schemas. Prioritize domain correctness, data integrity, simplicity, and maintainability.

## 1. Domain Modeling

- Identify business entities, attributes, relationships, and business rules before creating tables.
- Model business concepts, not application DTOs or API payloads.
- Assign attributes to the entity they logically describe.
- Avoid unnecessary tables, duplicated attributes, and ambiguous relationships.
- Document assumptions when requirements are incomplete.

## 2. Primary Keys

- Every table must have a reliable primary key.
- Prefer stable identifiers that do not change throughout an entity's lifecycle.
- Use surrogate keys when they simplify identification and relationships.
- Use natural keys when their uniqueness and stability justify them.
- Consider composite keys for naturally identified associative records.
- Enforce business-level uniqueness using UNIQUE constraints.

## 3. Foreign Keys

- Use foreign keys to enforce referential integrity.
- Reference primary keys or appropriate unique candidate keys.
- Ensure compatible data types between related columns.
- Define nullability according to business requirements.
- Follow consistent naming conventions, preferably `<referenced_entity>_id`.
- Evaluate indexes on referencing columns separately.

## 4. Relationship Cardinalities

### One-to-One (1:1)

Use a foreign key with a UNIQUE constraint on the dependent table.

- Ensure each child belongs to at most one parent.
- Use when entities have separate responsibilities, lifecycles, or access requirements.
- Avoid splitting entities unnecessarily.

### One-to-Many (1:N)

Place the foreign key on the many side.

Examples: Department → Employees, Customer → Orders.

Define whether the relationship is mandatory (`NOT NULL`) or optional (`NULL`).

### Many-to-Many (N:M)

Use an associative table containing foreign keys to both entities.

- Prevent duplicate associations using a composite primary key or UNIQUE constraint.
- Add attributes when they describe the relationship itself.
- Promote the associative table to a business entity when it has an independent lifecycle or business rules.

## 5. Constraints and Data Integrity

Use database constraints to enforce business invariants:

- `PRIMARY KEY`: Unique row identification.
- `FOREIGN KEY`: Referential integrity.
- `NOT NULL`: Mandatory attributes.
- `UNIQUE`: Business-level uniqueness.
- `CHECK`: Validates row-level conditions.
- `DEFAULT`: Provides meaningful default values.

Prefer database-level enforcement over relying exclusively on application validation.

Do not assume CHECK constraints can enforce rules involving other rows or tables.

## 6. Referential Actions

Choose deletion and update behavior intentionally.

- `RESTRICT` / `NO ACTION`: Prevent deletion when dependent records exist.
- `CASCADE`: Delete dependent records when the parent is deleted.
- `SET NULL`: Preserve dependent records without their previous relationship.
- `SET DEFAULT`: Use only when the default reference is valid and meaningful.

Rules:

- Avoid cascading deletes on important business or historical records.
- Prefer stable primary keys to avoid unnecessary key updates.
- Consider soft deletion when historical traceability is required.
- Remember that soft deletion does not replace foreign key constraints.

## 7. Data Types

Choose types according to semantic meaning, precision, and expected range.

- Use appropriate integer types for identifiers and counters.
- Use `DECIMAL` / `NUMERIC` for financial values requiring exact precision.
- Use floating-point types for approximate numerical calculations.
- Use appropriate date and timestamp types.
- Define timezone semantics explicitly.
- Use native BOOLEAN types when available.
- Avoid unnecessarily large or ambiguous data types.

## 8. Historical and Temporal Data

When the domain requires historical traceability:

- Preserve previous values instead of blindly overwriting them.
- Model effective dates and record timestamps explicitly.
- Distinguish business-valid time from record creation time.
- Define whether historical periods can overlap.
- Enforce temporal consistency where supported.

Examples: salary history, employment contracts, product prices, and account status changes.

## 9. Hierarchical Relationships

Use self-referencing foreign keys for simple parent-child structures.

Consider closure tables, materialized paths, or nested sets only when query patterns justify the additional complexity.

Prevent invalid cycles when the business domain requires a strict hierarchy.

## 10. Naming Conventions

Use consistent `snake_case` naming unless the project defines another convention.

Recommended conventions:

- Tables: consistent singular or plural naming.
- Primary keys: `id`.
- Foreign keys: `<entity>_id`.
- Indexes: `idx_<table>_<columns>`.
- Unique constraints: `uq_<table>_<columns>`.
- Foreign keys: `fk_<table>_<referenced_table>`.
- Check constraints: `chk_<table>_<rule>`.

Avoid reserved SQL keywords and ambiguous abbreviations.

## 11. Review Checklist

- [ ] Entities and attributes correctly represent the business domain.
- [ ] Every table has an appropriate primary key.
- [ ] Foreign keys enforce valid relationships.
- [ ] Cardinalities are correctly represented.
- [ ] Many-to-many relationships use associative tables.
- [ ] Mandatory and optional relationships are intentional.
- [ ] Business rules are enforced through appropriate constraints.
- [ ] Referential actions are explicitly considered.
- [ ] Data types match their semantic meaning.
- [ ] Historical data requirements are addressed.
- [ ] Naming conventions are consistent.
- [ ] The model avoids unnecessary complexity.

## 12. Agent Rules

- Understand the domain before generating a schema.
- Never invent business rules silently.
- Never omit foreign keys without explicit justification.
- Never introduce cascading deletes without evaluating their impact.
- Do not create tables solely to mirror application classes.
- Preserve historical and financial records when required.
- Prefer simple relational structures over unnecessary abstractions.
- Consider database-specific capabilities before generating DDL.
- Document significant assumptions and design decisions.

## Final Principle

Design relational schemas around business entities and relationships. Prioritize correctness, integrity, and maintainability over unnecessary abstraction or premature optimization.
