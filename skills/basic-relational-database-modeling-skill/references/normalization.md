# Relational Database Normalization

## Purpose

This document defines practical guidelines for applying relational database normalization principles when designing, reviewing, or refactoring relational database schemas.

The primary objective is to minimize data redundancy, prevent data anomalies, preserve data integrity, and establish clear relationships between entities.

Normalization should improve data consistency without introducing unnecessary complexity.

---

## 1. Fundamental Concepts

Before applying normalization rules, understand the following concepts.

### Functional Dependency

A functional dependency exists when one attribute (or a set of attributes) uniquely determines another attribute.

Notation:

`A → B`

This means that for each value of A, there can be only one corresponding value of B.

Example:

`employee_id → employee_name`

An employee ID uniquely determines an employee's name.

### Candidate Key

A candidate key is a minimal set of attributes that uniquely identifies each row in a relation.

A table may have multiple candidate keys, but only one is selected as the primary key.

### Prime Attribute

An attribute is prime if it belongs to at least one candidate key.

### Data Anomalies

Poorly normalized schemas can introduce three common anomalies:

- **Insertion anomaly:** Data cannot be inserted without introducing unrelated information.
- **Update anomaly:** The same information must be updated in multiple places, increasing inconsistency risks.
- **Deletion anomaly:** Removing a record unintentionally removes other valuable information.

---

## 2. First Normal Form (1NF)

### Definition

A relation satisfies First Normal Form when:

- Every column contains values from a single, well-defined domain.
- Each cell contains one logical value rather than a collection of independent values.
- There are no repeating groups of attributes.
- Each row can be uniquely identified.

### Incorrect Example

| order_id | customer_name | products |
|---|---|---|
| 1 | John | Laptop, Mouse, Keyboard |
| 2 | Sarah | Monitor, Mouse |

The `products` column stores multiple independent values.

### Correct Example

Separate order information from individual order items.

**orders**

| order_id | customer_name |
|---|---|
| 1 | John |
| 2 | Sarah |

**order_items**

| order_id | product_name |
|---|---|
| 1 | Laptop |
| 1 | Mouse |
| 1 | Keyboard |
| 2 | Monitor |
| 2 | Mouse |

### Design Guidelines

- Avoid comma-separated values in relational columns.
- Avoid columns such as `phone_1`, `phone_2`, and `phone_3` when the values represent a repeating collection.
- Use related tables to represent repeating entities.
- Do not confuse atomicity with simple data types: atomicity depends on the domain and how values are used.

---

## 3. Second Normal Form (2NF)

### Definition

A relation satisfies Second Normal Form when:

1. It satisfies 1NF.
2. Every non-prime attribute is fully functionally dependent on every candidate key.

In practical terms, non-key attributes must not depend on only part of a composite candidate key.

Note: 2NF is automatically satisfied when every candidate key consists of a single attribute.

### Incorrect Example

Consider an order details table:

| order_id | product_id | product_name | quantity |
|---|---|---|---|
| 1 | 10 | Keyboard | 2 |
| 1 | 20 | Mouse | 1 |
| 2 | 10 | Keyboard | 3 |

Composite primary key:

`(order_id, product_id)`

Functional dependencies:

- `(order_id, product_id) → quantity`
- `product_id → product_name`

The `product_name` attribute depends only on `product_id`, which is part of the composite key.

This creates a partial dependency.

### Correct Example

Separate product information from order details.

**products**

| product_id | product_name |
|---|---|
| 10 | Keyboard |
| 20 | Mouse |

**order_items**

| order_id | product_id | quantity |
|---|---|---|
| 1 | 10 | 2 |
| 1 | 20 | 1 |
| 2 | 10 | 3 |

### Design Guidelines

- Identify all candidate keys, especially composite keys.
- Inspect dependencies of non-prime attributes.
- Move attributes that depend on only part of a composite key into the appropriate relation.
- Preserve relationships using foreign keys.

---

## 4. Third Normal Form (3NF)

### Definition

A relation satisfies Third Normal Form when:

1. It satisfies 2NF.
2. For every non-trivial functional dependency `X → A`, at least one of the following is true:
   - X is a superkey.
   - A is a prime attribute.

In practical terms, non-key attributes should not depend transitively on candidate keys through other non-key attributes.

### Incorrect Example

Consider an employee table:

| employee_id | employee_name | department_id | department_name |
|---|---|---|---|
| 1 | John | 10 | Engineering |
| 2 | Sarah | 20 | Finance |
| 3 | Mark | 10 | Engineering |

Functional dependencies:

- `employee_id → employee_name, department_id`
- `department_id → department_name`

Therefore:

`employee_id → department_id → department_name`

The department name is transitively dependent on the employee ID.

### Correct Example

Separate departments from employees.

**departments**

| department_id | department_name |
|---|---|
| 10 | Engineering |
| 20 | Finance |

**employees**

| employee_id | employee_name | department_id |
|---|---|---|
| 1 | John | 10 |
| 2 | Sarah | 20 |
| 3 | Mark | 10 |

### Design Guidelines

- Identify attributes that describe different business entities.
- Move attributes dependent on another non-key determinant into their own relation.
- Preserve the relationship using foreign keys.
- Avoid duplicating descriptive information across multiple records.

---

## 5. Boyce-Codd Normal Form (BCNF)

### Definition

A relation satisfies BCNF when, for every non-trivial functional dependency:

`X → Y`

X must be a superkey.

BCNF is stricter than 3NF and addresses certain redundancy problems that 3NF permits.

### Example

Consider a table representing instructors assigned to courses and the rooms they use.

Assume:

- Each instructor teaches a course in one room.
- Each course can have multiple instructors.
- Each room is assigned to only one instructor.

**instructor_courses**

| instructor_id | course_id | room_id |
|---|---|---|
| 1 | 101 | A1 |
| 1 | 102 | A1 |
| 2 | 103 | B1 |

Candidate key:

`(instructor_id, course_id)`

Functional dependency:

`room_id → instructor_id`

The determinant `room_id` is not a superkey.

This violates BCNF.

### Possible Decomposition

**room_instructors**

| room_id | instructor_id |
|---|---|
| A1 | 1 |
| B1 | 2 |

**room_courses**

| room_id | course_id |
|---|---|
| A1 | 101 |
| A1 | 102 |
| B1 | 103 |

The decomposition removes the problematic dependency while preserving the relationship information.

### Design Guidelines

- Evaluate BCNF when multiple candidate keys or overlapping candidate keys exist.
- Identify functional dependencies whose determinants are not superkeys.
- Check that decomposition preserves the required information.
- Prefer lossless decompositions.
- Evaluate whether the decomposition preserves functional dependencies without requiring additional joins or constraints.

---

## 6. Fourth Normal Form (4NF)

### Definition

A relation satisfies Fourth Normal Form when:

1. It satisfies BCNF.
2. It contains no non-trivial multivalued dependency unless its determinant is a superkey.

A multivalued dependency occurs when two or more independent multi-valued facts are stored in the same relation.

### Example

An employee can have multiple skills and multiple languages, independently of each other.

**employee_attributes**

| employee_id | skill | language |
|---|---|---|
| 1 | Python | English |
| 1 | Python | Spanish |
| 1 | Java | English |
| 1 | Java | Spanish |

The combinations create unnecessary duplication.

### Correct Example

**employee_skills**

| employee_id | skill |
|---|---|
| 1 | Python |
| 1 | Java |

**employee_languages**

| employee_id | language |
|---|---|
| 1 | English |
| 1 | Spanish |

### Design Guidelines

- Identify independent multi-valued relationships.
- Avoid combining unrelated collections in a single relation.
- Use separate relations for independent multi-valued facts.

---

## 7. Denormalization

### Definition

Denormalization is the intentional introduction of redundancy or precomputed data to improve specific access patterns or performance characteristics.

Denormalization is not inherently incorrect, but it must be justified.

### Common Scenarios

- Frequently accessed aggregated values.
- Expensive joins in read-heavy workloads.
- Reporting and analytical workloads.
- Materialized views or cached representations.
- Read models in CQRS architectures.

### Rules

Before denormalizing:

1. Identify the actual performance bottleneck.
2. Validate the problem using query execution plans and workload measurements.
3. Evaluate whether indexing or query optimization can solve it.
4. Document the duplicated information.
5. Define how consistency will be maintained.
6. Evaluate synchronization and transaction requirements.
7. Measure the improvement after implementation.

### Important Considerations

- Denormalized data introduces consistency responsibilities.
- Duplicated values can become outdated.
- Additional storage and write operations may be required.
- The source of truth must remain clearly defined.

Never denormalize solely to avoid writing JOIN statements.

---

## 8. Normalization Review Checklist

Use this checklist when reviewing a relational database model.

### First Normal Form

- [ ] Every column represents a single logical value.
- [ ] No comma-separated collections or repeating groups exist.
- [ ] Every row has a reliable identification strategy.

### Second Normal Form

- [ ] Candidate keys have been identified.
- [ ] Composite keys have been reviewed.
- [ ] Non-prime attributes depend on entire candidate keys.
- [ ] No partial dependencies remain.

### Third Normal Form

- [ ] Non-key attributes do not have problematic transitive dependencies.
- [ ] Attributes describe the entity represented by their relation.
- [ ] Descriptive information is not unnecessarily duplicated.

### BCNF

- [ ] Functional dependencies have been evaluated.
- [ ] Every determinant of a non-trivial functional dependency is a superkey.

### Fourth Normal Form

- [ ] Independent multi-valued relationships have been identified.
- [ ] Unnecessary combinations of independent collections have been eliminated.

### Denormalization

- [ ] Every intentional redundancy has a documented justification.
- [ ] A clear consistency strategy exists.
- [ ] Performance benefits have been measured or supported by a concrete requirement.

---

## 9. Practical Guidelines for Database Agents

When designing or reviewing a schema:

1. Do not assume that splitting every attribute into a separate table produces a better model.
2. Identify business entities before identifying tables.
3. Analyze functional dependencies instead of applying normalization mechanically.
4. Consider all candidate keys, not just the primary key.
5. Distinguish between genuine relationships and duplicated descriptive information.
6. Prefer lossless decompositions.
7. Consider dependency preservation when decomposing relations.
8. Do not introduce unnecessary joins through excessive normalization.
9. Preserve intentional historical snapshots when business requirements demand them.
10. Document intentional deviations from normalization rules.

## 10. Final Design Principle

The objective of normalization is not to maximize the number of tables or achieve the highest normal form at any cost.

The objective is to establish a reliable relational model where:

- Each fact has a clearly defined place.
- Relationships accurately represent the business domain.
- Data anomalies are minimized.
- Integrity is enforced consistently.
- Query performance remains appropriate for actual workloads.
- Design decisions are understandable and maintainable.

Use the highest practical normal form justified by the domain and workload, and introduce denormalization only through explicit, documented engineering decisions.
