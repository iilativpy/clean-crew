# Unit-test quality: catalog of test smells

Use these IDs when reviewing tests, as the code smells catalog is used for production code. Severity
in a review: **Blocker** (the test cannot be trusted: it cannot fail, asserts nothing, encodes a bug),
**Major** (will cause false failures or hide real ones), **Minor** (readability/maintenance), **Nit**.

## Trustworthiness
| ID | Smell | Why it hurts | Fix | Default |
|---|---|---|---|---|
| UT-1 | **Logic in the test**: `if`/`else`, loops, `switch`, `try/catch`, random data, threads, computed expectations that repeat the production algorithm | A test with logic can have bugs of its own, is hard to read and to reproduce; usually tests several things | Straight line: calls and asserts. Split into several tests or use a parameterized test with literal rows | Major |
| UT-2 | **Several concerns in one test** (asserts on unrelated results; return value plus state change plus a call) | The first failing assert hides the rest; the name is vague; the failure does not point at the problem | One concern per test; several asserts only when they describe one logical result | Major |
| UT-3 | **Test never seen failing** / cannot fail (not collected by the runner, no assert, asserts a constant, passes against an empty implementation) | False confidence | Break the code on purpose and watch it fail; make sure the runner discovers it | Blocker |
| UT-4 | **No real assertion** (only "does not throw", only coverage) | Coverage theater | Assert the observable result | Blocker |
| UT-5 | **Test asserts wrong behavior** (encodes a bug or a stale rule) | Defends the defect | Correct the test and say so; never leave it | Blocker |
| UT-6 | **Same value for different inputs** (the code can swap two arguments and the test still passes) | False positive | Give distinct values per input (generated or clearly different) | Major |
| UT-7 | **Non-determinism**: real clock, random, network, file system, ordering of a set/dict, threads | Flaky tests are ignored, then real failures are ignored | Inject the clock/random; use fakes; sort before comparing; no sleeps | Major |
| UT-8 | **Unit and integration tests mixed** in one suite or class | Slow or flaky unit suite; people stop running it | Separate by speed/type; keep the unit suite fast and fully green | Major |
| UT-9 | **Test depends on a mock-heavy fake chain** (stub returns stub returns mock) | Brittle, hides a Demeter violation | Pass the needed value directly; fix the design | Minor |

## Maintainability
| ID | Smell | Why it hurts | Fix | Default |
|---|---|---|---|---|
| UT-10 | **Overspecification**: asserts internal state, exact strings/order that are not required, that a stub was called, or several mocks | Test breaks on harmless refactoring | Assert the end result only; at most one mock; never assert stubs | Major |
| UT-11 | **Tests a private/protected method or private state** | Binds the test to the structure | Test through the public use case; extract a class if it deserves its own contract | Major |
| UT-12 | **Order dependence / hidden test call / shared state** (a test calls another test, expects state from a previous one, mutates a shared object or a global/database without reset) | Fails when run alone or in a different order; "sometimes passes" | Each test builds its own state and cleans up; reset or avoid shared mutable state | Major |
| UT-13 | **Duplicated set-up and assertions** | Every API change breaks many tests | Factory/builder/helper functions, parameterized tests | Minor |
| UT-14 | **Setup abuse**: one big shared `setUp` creating objects only some tests need, long hard-to-read setup, mocks configured in setup | Tests are unreadable and coupled | Keep `setUp` for what *all* tests need; use local helpers/factories; configure doubles in the test | Minor |
| UT-15 | **Structural coupling**: one test class per production class, one test per method | Freezes the production structure | Organize tests by behavior and unit of work | Minor |
| UT-16 | **Slow test** (sleeps, real I/O, heavy setup) | Skipped by developers | Fakes, in-memory stores, no sleeps, small data | Major |
| UT-17 | **Broad "assert equals everything"** on a large object when one field matters | Breaks on unrelated change | Assert the fields that matter, or compare via a purpose-made comparer | Minor |

## Readability
| ID | Smell | Why it hurts | Fix | Default |
|---|---|---|---|---|
| UT-18 | **Unclear name** (`test1`, `works`, `BadlyNamedTest`) | Reader must read the body to learn the purpose | Name unit, scenario, expectation (`withdraw_amount_above_balance_is_rejected`) | Minor |
| UT-19 | **Magic numbers/strings** in arrange and assert (`-100`, `"MY_HOST_NAME"`) | Reader cannot tell what matters or why | Named constants, anonymous-value helpers, derived expectations | Minor |
| UT-20 | **Action hidden inside the assert** or three phases tangled | Hard to see what is tested | Separate Arrange, Act, Assert | Minor |
| UT-21 | **Vague failure message** (`assertTrue(x)`) | Hard to diagnose | Assert the value (`assertEqual`) or add a message that states the expectation | Nit |

## Coverage of behavior
| ID | Smell | Why it hurts | Fix | Default |
|---|---|---|---|---|
| UT-22 | **Only the happy path**: no rejection, boundary, empty/zero/max, ordering or later-step-failure tests | Defects live at the edges | List the cases for each rule (see `SKILL.md` section 5) | Major |
| UT-23 | **Coverage chasing** (tests written to raise a number) | Proves nothing | Review the tests; measure with a mutation check | Minor |
| UT-24 | **New behavior with no test** | Regressions | Add the test first | Blocker (in a review) |

## When a test fails: what to do
1. **Production bug** (you changed code and an existing test broke for a real reason): that is the
   test doing its job. Fix the code. Do not touch the test.
2. **Bug in the test:** fix it; then make the production code show the bug on purpose and confirm the
   test fails; restore and confirm it passes. Do not trust a repaired test you have not seen fail.
3. **API/semantics changed, behavior is the same:** update the test to the new call; keep the intent.
4. **Requirement changed and the test conflicts:** replace or remove it, and say why in the commit.
5. **Duplicate or renamed:** remove duplicates and rename for clarity; never remove a test only to
   get a green run.
Hunting for a "production bug" in code that is correct, while the real bug sits in the test, wastes
hours: when a test fails unexpectedly, check the test first for logic, shared state and stale data.

## Review procedure for a test suite
1. Run the suite alone, then in random or reverse order (isolation, UT-12).
2. Skim names (UT-18): can you list the behaviors from the names alone?
3. Read ten tests for UT-1, UT-2, UT-10, UT-19.
4. Check what is **not** tested (UT-22): rejections, boundaries, failures.
5. Mutation-check the critical rules (UT-3): remove or invert the control, change a boundary, drop a
   side effect, and see a test fail. A short script that applies each mutation to a copy and runs the
   suite makes ten mutations as cheap as three. A rule found broken this way is a production bug: write
   the failing test first.
6. Check speed and determinism (UT-7, UT-16).
