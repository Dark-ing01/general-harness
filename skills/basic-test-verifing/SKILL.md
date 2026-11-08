---
name: test-verifying
description: Use when unit tests were written or changed, in any language or framework, and need to be verified before they are accepted — checking that they pass, are isolated and deterministic, cover the code, catch real bugs, and meet quality gates.
---

# Test Verifying

Prove that tests are trustworthy, not just green. Report findings; don't fix tests unless asked. Cite `TST-xxx` rule IDs from the `test-writing` skill when it is available.

## Scope
1. Target: the test files written or changed (diff, or the files given) and the production units they cover.
2. Detect the stack from the repo: test runner, mocking library, coverage tool, mutation tool, and the repo's own coverage gate.
3. Note `git status` before starting. At the end, tracked files must be unchanged and any file you created (coverage data, helper scripts in the repo) removed.

## Checks
### 1. Structure and conventions
Location and naming follow the repo; Arrange–Act–Assert with exactly one Act; one behavior per test; fixtures reused with relevant values visible; no narrating comments; no framework/app context in unit tests.

### 2. Green without dependencies
Run the target tests once, in declaration order (disable any random-order plugin; order effects belong to Isolation), with no infrastructure running and infra env vars (DB URLs, broker hosts, API keys) unset. Any connection attempt, timeout or skipped test is a failure. Also scan the tests for real clients, container/app boots, real URLs, file system or network access.

### 3. Isolation
Run each target test alone, then the whole file/suite in random order with at least 5 seeds, and report any failing seed (JUnit `junit.jupiter.testmethod.order.default=org.junit.jupiter.api.MethodOrderer$Random`, `pytest -p randomly`, `jest --randomize`, `go test -shuffle=on`). A result that changes with order, or shared mutable/global state in the code, fails the gate.

### 4. Determinism
Run the suite 3 times, plus once with a different time zone and locale (`TZ=America/Mexico_City`, `LANG=es_MX.UTF-8`). Then run once with the clock moved forward (a year and to a month end, using a fake-time tool, or by patching the clock/date source) to expose hard-coded dates that will expire. Any difference is a flaky test. Flag sleeps, the uncontrolled system clock, unseeded randomness and reliance on unordered-collection iteration.

### 5. Coverage
Measure line and branch coverage of the covered units with the repo's tool (JaCoCo, coverage.py, Istanbul/`--coverage`, `go test -cover`). List uncovered branches with `file:line`.

### 6. Mutation testing
Use the repo's mutation tool if configured (PIT, mutmut, Stryker, go-mutesting), limited to the covered units. Otherwise mutate by hand, one change at a time. Business-rule lines are conditions, calculations, persistence/publish calls and returns; mutate each operand of a compound condition separately, because branch coverage does not see them:
- flip a condition or negate it
- shift a boundary (`<` ↔ `<=`, `+ 1` removed)
- remove a call with side effects (save, publish)
- replace a return value (null/empty/constant)

Run the tests in declaration order after each mutant; a mutant that no test kills **survives**. A mutant that cannot change behavior (equivalent) is excluded and listed with the reason. Revert after each one (`git checkout -- <file>`).

### 7. Assertions
Every test asserts something observable. Flag:
- no assertion, or only "does not throw" / "is not null"
- asserting the value a mock was stubbed to return (tautology)
- a generic error type when the contract defines a specific one or code
- `verify` on interactions that are not the contract, or a missing `never()` on failure paths
- several unrelated behaviors in one test

### 8. Existing tests
List the tests that exist on `origin/main` and are modified, deleted or skipped in the diff (`git diff origin/main...HEAD` on test files). For each one, compare the old and new assertion with the spec or `Existing tests changed` reason you were given. Flag any change that has no reason, or that weakens a check without the spec changing that behavior. Examples: a looser matcher, a removed assertion or case, a changed expected value, an added skip or ignore.

## Quality gates
The repo's thresholds win when stricter.

| Gate | Pass when |
|---|---|
| Green | 100% of target tests pass, none skipped |
| No dependencies | 0 infrastructure or network access |
| Isolation | 0 order-dependent results |
| Determinism | 0 differences across runs, time zone and locale |
| Coverage | ≥ 80% lines and ≥ 70% branches on covered units |
| Mutation | ≥ 70% of mutants killed, and 0 survivors on business-rule lines |
| Assertions | 0 tests without a meaningful assertion |
| Existing tests | 0 changes to pre-existing tests without a spec reason |

## Output
```
| Gate | Result | Threshold | Status |
...
Verdict: PASS | FAIL
```
Then one block per finding, most severe first:
```
[GATE] path/to/file:line — TST-xxx
Problem: one sentence (for survivors and missing tests: the mutant and the production line).
Fix: the test or assertion to add.
```
Report only what was actually run and observed; if a check could not run (missing tool, no build), say so and mark the gate `NOT RUN`, never PASS.
