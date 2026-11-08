---
name: test-writing
description: Use when writing or extending unit tests in any language or framework — new code, changed behavior, bug fixes, or coverage gaps.
---

# Test Writing

Unit tests prove the behavior of our code, in isolation and deterministically. Rules carry IDs (`TST-xxx`) so reviews can cite them.

## Before writing
0. Load **basic-code-writing**
1. Detect the stack from the repo (build file, existing tests) and use its test framework, mocking library and conventions. Never introduce a new framework when one exists.
2. Read the unit under test, its collaborators and the existing tests next to it. Reuse their fixtures, helpers and naming style.
3. If required generate common data/mock/models providers to reuse.
3. List scenarios from the unit's contract: happy path, each business-rule branch, invalid input, boundaries (empty, zero, max, dates on the limit), dependency failures. Test null/None/undefined input only when the code defines behavior for it.

## When an existing test fails
A test you didn't write that fails after a change is a signal that the change may have broken something. Treat it as a bug report. It is not an obstacle to make green. Before touching it, answer:
1. **Why does it fail?** Run it alone and read the assertion. Name the behavior it protects.
2. **Who uses that behavior?** Find the callers of the changed code in this repo and in the other services (HTTP clients, events, shared libraries, the codebase-memory graph or grep).
3. **Did we break it?** Compare the old expectation with the spec.
   - If the spec doesn't explicitly change that behavior, the **code is wrong**: fix the code and keep the test.
   - If the spec does change it, update the test to the new behavior, and check every caller found in step 2.

Never weaken, delete or skip an existing test to get green. If you can't tell which side is right, stop and report it as an open question. List every existing test you changed in your Output, with the reason.

## What to test
- **Test:** our business logic, validators, services/use cases, our own mapping or adapter decisions, request handlers' delegation and validation.
- **Don't test library or framework code:** generated code (mappers, accessors, ORM repositories), framework wiring/DI, third-party SDKs, language built-ins. Test our code that uses them.

## Rules
| ID | Rule |
|---|---|
| TST-001 | No real infrastructure. Databases, caches, queues/brokers (Kafka, Pub/Sub), HTTP/gRPC APIs, file/object storage and the network are mocked or faked at our boundary (port, client, repository). Integration tests live separately. |
| TST-002 | No framework/application context in unit tests. Instantiate the unit directly with its dependencies. Reserve container/app boots (Spring context, app factories, DI containers) for integration tests. |
| TST-003 | Real objects for entities, value objects, DTOs and pure functions. Mock only external boundaries or collaborators whose interaction is the contract. |
| TST-004 | Independent. No required order, no mutable shared/global state, no test using another's result; fresh fixtures per test. |
| TST-005 | Meaningful assertions: the observable result, the exact error type (and code/message if the contract defines one), or a relevant external interaction. Executing code without asserting is not a test. |
| TST-006 | Stub only what the scenario uses; no unused or catch-all stubs. Verify an interaction only when it is the contract (a save, a published event, "never called" on a failure path). |
| TST-007 | DRY without hiding the scenario. Shared factories/builders/fixtures return valid defaults; each test sets only the values that matter to it, visibly. |
| TST-008 | Positive, negative and boundary cases for every public behavior. |
| TST-009 | No sleeps, uncontrolled system clock or real timers. Inject a clock or fixed time, use fake timers, run async work synchronously or await it deterministically. |
| TST-010 | Deterministic. Same code and data, same result: no network, unseeded randomness, unordered-collection iteration order, locale or time-zone dependency. |

## Shape
- **Arrange–Act–Assert:** three blocks separated by blank lines, exactly one Act. Add `given/when/then` markers only if the repo's tests already use them.
- **Atomic:** one behavior per test. Several asserts are fine when they describe the same outcome.
- **Exceptions:** the raising block (`assertThrows`, `pytest.raises`, `expect(() => …).toThrow`) wraps only the call under test.
- **Captured arguments** (captors, `call_args`, `toHaveBeenCalledWith`) are checked in Assert.
- Expected value first where the framework distinguishes it; use the specific matcher (`isNull`, `is None`, `toBeNull`) over generic equality.
- The name states the scenario and expectation in the repo's style; no comments narrating the test.

## Example (pytest; same shape in any framework)
```python
TODAY = date(2026, 3, 10)

@pytest.fixture
def balance_port():
    return Mock(spec=VacationBalancePort)

@pytest.fixture
def request_port():
    return Mock(spec=VacationRequestPort)

@pytest.fixture
def service(balance_port, request_port):
    return VacationService(balance_port, request_port, clock=FixedClock(TODAY))

def test_request_when_days_exceed_balance_raises_and_does_not_save(service, balance_port, request_port):
    request = vacation_request(start=TODAY + timedelta(days=1), days=5)
    balance_port.available_days.return_value = 4

    with pytest.raises(InsufficientBalanceError):
        service.request(request)

    request_port.save.assert_not_called()
```

## Done when
1. The new tests pass with the repo's test command, and the repo's coverage gate passes.
2. Each test fails when the behavior it covers is broken: temporarily break the production line it targets, confirm a failure, then revert.

## Output
```
Status: DONE | PARTIAL | BLOCKED
Repo: <absolute path>   Branch: <branch>
Target tests: <test files added or changed>
Units covered: <production files the tests exercise>
Tests:
- <test name> — <scenario and expected behavior>
Run: <exact command> — <passed>/<total>, coverage <lines%>/<branches%> or NOT RUN
Existing tests changed: <test — old expectation → new, spec reference, callers checked> | none
Not covered: <behaviors left untested and why>
Assumptions: <anything assumed about the code or contract>
```
`Target tests` and `Units covered` are the inputs of the `test-verifying` skill. Report only what was actually run; if the tests could not be run, use `NOT RUN` and `PARTIAL`.
