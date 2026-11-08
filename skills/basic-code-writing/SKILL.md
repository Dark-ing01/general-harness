---
name: basic-coding-writing-skill
description: Apply fundamental software engineering principles before and during implementation to produce clean, reusable, maintainable, and scalable code.
---

# Code Principles

You must think before coding. Prioritize good engineering decisions over writing code quickly.

## Core Principles

1. **Think Before Coding:** Understand the problem, requirements, existing architecture, and constraints before implementing anything.

2. **DRY (Don't Repeat Yourself):** Avoid duplicated logic. Identify opportunities to reuse existing functions, components, services, and abstractions.

3. **SOLID Principles:** Apply SOLID principles where appropriate to improve maintainability, extensibility, and separation of responsibilities.

4. **KISS (Keep It Simple, Stupid):** Prefer straightforward solutions over unnecessary abstractions, patterns, or complexity.

5. **YAGNI (You Aren't Gonna Need It):** Do not implement features, abstractions, or flexibility that are not currently required.

6. **Separation of Concerns:** Keep responsibilities clearly defined. Avoid mixing business logic, infrastructure, and presentation concerns.

7. **Reuse Before Rebuilding:** Always inspect the existing codebase before creating new implementations. Extend or reuse existing solutions whenever reasonable.

8. **Consistency:** Follow established project conventions, naming patterns, architectural decisions, and coding standards.

9. **Maintainability:** Write code that other developers can easily understand, test, modify, and extend.

10. **Testability:** Design implementations that can be tested independently. Cover relevant behavior and edge cases.

## Implementation Workflow

Before writing code:

1. Understand the requirements and expected behavior.
2. Explore the existing codebase for reusable solutions.
3. Identify affected components and dependencies.
4. Evaluate the simplest solution that satisfies the requirements.
5. Consider potential side effects and architectural implications.

During implementation:

- Avoid unnecessary changes outside the scope.
- Reuse existing abstractions whenever possible.
- Keep functions and classes focused on a single responsibility.
- Avoid premature optimization and overengineering.
- Follow the project's existing conventions.

## Golden Rule

**Do not write code just because you can. Write code because it is necessary, and make sure it is the simplest, most reusable, and maintainable solution available.**
