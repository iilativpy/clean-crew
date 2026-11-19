---
name: testing
description: How to write, structure, review and drive code with automated tests - what a good unit test is, test-first (TDD) practice, test doubles and when to use interaction tests, trustworthy/maintainable/readable test rules, boundaries and unhappy paths, legacy code, testability, and behavior-driven (BDD) examples and acceptance scenarios. Use whenever writing or changing tests, doing test-first development, deriving test cases from requirements, reviewing a test suite, or making code testable.
---

# Testing - working rules

Tests are **executable specifications** of behavior, a safety net for change, and the first
user of your design. A bad test is worse than none: it breaks for the wrong reasons, hides real
failures, and teaches the team to ignore red. Use this skill with `clean-code` (section 8 is the
short form) and `clean-architecture` (the testable seam is a boundary).

This file is short enough to read in full. Details are in `references/` (table at the end).

## 0. What a unit test is [write, review]
- A **unit of work** is everything between calling a public entry point and one noticeable end
  result. It can be one function or several collaborating classes. **A unit test invokes one
  unit of work and checks one end result.**
- A noticeable end result is one of three things, observed through the public interface only:
  1. a **return value**,
  2. a visible **change of state or behavior** (without reading private fields),
  3. a **call to a third party you do not control** (a logger, a notifier, a payment gateway).
- A good test is **automated, fast, repeatable, isolated, readable, maintainable, and trusted**:
  when it passes you do not debug to be sure; when it fails you believe the code, not the test.
- Test **behavior through the public API**, not structure. Tests that mirror the production
  structure (one test class per class, one per method, private methods) freeze the design and
  break on every refactoring.
- No need to test code without logic (plain getters and setters). Code with a decision, a loop, a
  calculation or a state change needs a test.
- **Coverage is a smoke alarm, not a goal.** 100% coverage with tests that assert nothing proves
  nothing. Pair coverage with review of the tests themselves.

## 1. Test-first (TDD) [write]
The loop: **write a failing test, make it pass in the simplest way, refactor, repeat.**
1. **Write the test as the specification** of one behavior you wish were true, as if the code
   already worked. Name it for that behavior.
2. **Run it and see it fail, for the right reason** (a missing behavior, not a typo, a missing
   import or an unregistered test). Create the smallest stub first (the real signature, returning a neutral value
   or raising `NotImplementedError`) so the failures are **assertions, not import or missing-name
   errors**; a red run caused by a missing module only proves the module is missing. If a test **passes on its first run** (a stub's default satisfies it, or earlier
   code already did the work), find out why: it is either redundant or needs a mutation check
   (section 9) to prove it can fail. A test you never saw fail has no proof it can fail: it may
   not be collected by the runner, may assert the wrong thing, or may pass whatever the code does.
3. **Make it pass with the simplest code.** Do not add behavior no test asks for.
4. **Refactor** with the tests green: remove duplication, improve names, adjust design. Run the
   tests after each small step.
5. Pick the next behavior. Keep the **list of behaviors** (a TODO list) and tick it off: normal
   case first, then each rule, rejection, boundary and failure.
- **Why before, not after:** after the code exists there is little motivation to specify it; the
  tests drift toward the happy path, error cases get skipped ("test-after" often becomes
  "test-never"), and the design is already fixed. Writing a test first forces you to think about the
  requirement and the interface before the implementation.
- **Test-first is also a design tool:** a test that is hard to write signals a design that is hard
  to use (too many dependencies, hidden inputs, big setup). Listen to it.
- **Legacy code without tests** is the exception: pin existing behavior first with characterization
  tests (section 7), then change.
- **Stuck on where to start?** Start with a good name; or fill Given/When/Then with the obvious and
  work backwards; or start from the assertion and work backwards; or, if the method signature
  already exists, start by calling it. Details in `references/tdd-practice.md`.

## 2. Test structure and naming [write, review]
- **Arrange, Act, Assert** (Given, When, Then): set up the context, perform the one action, check
  the outcome. Keep the three visibly separate. One action per test.
- **The name says three things**: what is being exercised, under what scenario, and what is
  expected. Either `unit_scenario_expected` or a sentence (`should_reject_expired_card`). A reader
  must know what broke from the name alone, without reading the body. Follow the project's
  convention if it has one.
- **Names of variables, literals and helpers matter more in tests than in production code.** A
  magic number in an assertion is a mystery: give it a name (`EXPIRED`, `NO_ITEMS`) or build it
  from the inputs.
- **Assertion messages and helpers** should say what is being proven; no cryptic booleans.
- **Keep the action and the assertion apart:** do not call the action inside the assert.
- **Setup:** use fixtures and `setUp` only for things every test in the file needs. A factory
  function or builder that returns a ready object (with named overrides) is usually clearer than a
  long shared setup; tests that need something special say so locally.

## 3. Trustworthy, maintainable, readable [write, review]
The checklist behind every test review. IDs (`UT-n`) are in `references/unit-test-quality.md`.
- **Trustworthy**
  - **No logic in the test** (no `if`, `for`, `while`, `switch`, no `try/catch`, no computed
    expectations that repeat the production algorithm). A test is a straight line: calls and asserts.
    A **data table** handled by the framework is not logic: `pytest.mark.parametrize`,
    `@ParameterizedTest`, `test.each`, table-driven Go tests, with literal rows and readable ids. With a
    bare `unittest`, use `subTest` over literal rows or one small method per case; do not loop over
    computed expectations.
  - **One concern per test**, so a failure names one problem and the name can be precise. Several
    related asserts on one logical result are fine; asserts on different concerns are not.
  - **Unit tests and integration tests stay separate**; unit tests have no network, clock, random
    numbers, file system or database. Keep the fast, trustworthy suite green and separate from the
    slow one.
  - **When a test fails, decide which it is:** a production bug (fix the code, never the test),
    a test bug (fix the test and prove it can fail), a changed API (update the test), or a conflicting
    requirement (replace the test and say why). Never delete a test just to get green.
- **Maintainable**
  - **Test public behavior, not private methods or internal state.** A private method is part of
    some public use case: test through that. If it deserves direct tests, it may belong in its
    own class with a public contract.
  - **Remove duplication** with helpers, factories and parameterized tests (a test code base needs
    DRY as much as production code), but keep each test readable on its own.
  - **Isolation:** each test runs alone and in any order, creates its own state, and cleans up.
    No test calls another test. No shared mutable state or dependence on order.
  - **Avoid overspecification:** assert the outcome, not the route. Do not assert purely internal
    state, exact strings or order when they are not required, or that a stub was called. Do not use
    several mocks in one test.
- **Readable**
  - Name, structure and literals tell the story (sections 2).
  - Prefer tests that read like specifications of the rule, with the data that matters visible and
    the incidental data hidden.

## 4. Test doubles: stubs, mocks, fakes [write, review]
- A **stub** supplies indirect input (it returns what the test needs) and can **never fail** the
  test. A **mock** records an interaction and the test **asserts on it**. A **fake** is a working
  lightweight implementation (an in-memory repository). In a test, ask "which is my mock?"; everything
  else is a stub or a fake.
- **Prefer value-based and state-based tests.** Use **interaction tests** (assert that a call was
  made) only when the call *is* the end result: sending an email, logging, publishing an event,
  charging a card. Do not assert interactions you could observe as state or return value.
- **One mock per test** at most. Several mocks usually mean the test checks more than one thing and
  will be brittle. Asserting on a stub is overspecification.
- **Do not mock what you own cheaply** (value objects, simple data) and **do not mock the thing under
  test**. Fake chains (a stub returning a stub returning a mock) are a design smell: the code under
  test navigates too much (Law of Demeter).
- **Seams:** make dependencies replaceable where the code meets I/O, time, randomness or a third
  party: pass them in (constructor or parameter), put an interface or function type in front
  of them. That is also the architectural boundary (`clean-architecture`).
- Two defensible styles: classicist (real collaborators, state and value assertions) and
  mockist (mocks to specify the protocol between roles while designing outside-in). Follow the
  project's; in either, keep mocks few, at role boundaries, and prefer messages that tell an object
  what to do over getters that ask it for data.
- More in `references/doubles-and-testability.md`.

## 5. What to test: unhappy paths and boundaries [write]
For each behavior or rule, list cases and write a test for each that matters:
- the **normal** case;
- every **rejection** (invalid input, forbidden state, duplicate, missing);
- **boundaries**: empty, one, many, zero, maximum, exactly at the limit, just inside and just
  outside (`17` and `18` for "adult at 18"), ranges and their edges, ordering;
- **failure in a later step** (nothing half-done), and **errors from collaborators**;
- **hostile or odd input** where the unit is exposed (huge, negative, non-ASCII, `None`, wrong
  type); security cases are in `secure-coding` section 13;
- **time, concurrency and ordering** where they matter, with a controlled clock and deterministic
  scheduling;
- **money and measures:** exact decimal types (never floats), the rounding rule, units and currency,
  tested at the edges where rounding changes the result.
For rules that combine (a surcharge, a threshold and a method), test each in isolation and a few
combinations where one rule must not leak into another; reserve a full matrix for small ones. When a
case depends on an unanswered question, name the assumption in the test id or mark it expected-to-fail
so a changed answer touches few tests. Asserting that an exception is raised with the framework helper
is not logic in a test.
Each test introduces **one behavioral distinction** not specified before; if it passes the first time
without a code change, either it was redundant or the previous step did too much.

## 6. Writing values and expectations well [write]
- **Anonymous input:** when the behavior holds for any value of an equivalence class (any string,
  any positive integer), say so with a named helper (`any_string()`, `any_positive_int()`) instead
  of an arbitrary-looking literal that suggests a special meaning.
- **Distinct values:** two different inputs get **different** values, or a test passes when the code
  swaps them (a false positive). Generated distinct values (random/unique) prevent that.
- **Derived expectations:** build the expected value from the inputs with the simplest expression
  (`f"backup_{host}_{user}.zip"`) rather than a literal that hides the rule. Do not copy the
  production algorithm into the test.
- **Constants:** reference a constant that production code already defines instead of repeating its
  value, unless the value itself is the specification.
- **Boundary and rule tests:** write one test for each side of a boundary and name the edge values.
- **Determinism:** inject the clock and the random source; no sleeping, no real time.

## 7. Legacy code and testability [write, review]
- **Legacy code (no tests):** do not rewrite first. (1) Choose a seam. (2) Write **characterization
  tests** that pin what the code actually does today, including its quirks. (3) Refactor in small
  steps under those tests. (4) Add new tests for the new behavior test-first. Start where the change
  is, or with easy components to learn, and plan for the hard ones; write integration-level tests
  around hard-to-isolate code before refactoring it.
- **Design for testability:** inject dependencies instead of constructing them in methods with logic;
  avoid logic in constructors; avoid direct calls to statics, clocks and singletons in logic; keep
  logic separate from I/O (a functional core with a thin imperative shell). Testable design correlates
  with sound design principles, but testability alone does not make a design good.
- **Dynamic languages** need less ceremony: pass functions, monkeypatch at seams, use fakes. The
  discipline (isolation, one concern, no logic) is the same.

## 8. Behavior-driven development and acceptance tests [write, review]
BDD extends TDD upward: it starts from **business behavior described with concrete examples**, shared
by developers, testers and stakeholders, and keeps those examples as living, executable
documentation.
- **Start from examples.** Before coding a story, discuss **concrete examples** of the behavior with
  the people who know the domain. Examples expose misunderstandings and edge cases. Record each as a
  scenario.
- **Scenario = Given (context), When (one action), Then (observable outcome).** Use *And* only to extend
  the same part. Write in business language, in the present tense for the action and past tense for
  preconditions where it helps ("Given Bill has registered").
- **What, not how.** State the business intent; hide UI steps, ids and set-up details in the
  automation layer. A scenario that lists clicking every field is brittle and unreadable.
- **One behavior per scenario**, with only the preconditions needed. A scenario whose purpose cannot be
  stated in one sentence tests too much. Use tables or a scenario outline for the same rule over many
  examples.
- **Layer the automation:** business rules (readable scenarios) on top of reusable business steps on top
  of a technical layer (page objects, API clients). The first changes with the business, the last
  with the technology.
- **Test at the right level:** put each business rule at the cheapest level that proves it (unit, then
  service/API, then UI). Use UI tests for the user journey and rendering, not for business rules.
- **Two loops:** a failing acceptance scenario defines the feature; unit-level tests (TDD) drive the
  parts needed to make it pass.
- Details and the anti-patterns of scenarios are in `references/bdd-and-acceptance.md`.

## 9. Proving the tests work [write, review]
- **See it fail first** (section 1). For an existing test you doubt, **break the production code on
  purpose** (flip a condition, remove a line), run it, and confirm a test fails; then restore.
- For the important controls (ownership check, limit, parsing rule), do this **mutation check** in a
  throwaway copy: remove or invert the control and see whether any test fails. A control whose removal
  changes no test result is not specified; add the test.
- A test that never fails, never runs, asserts nothing, or passes whatever the code does is a **Blocker**
  in a review.

## 10. Quick checklists
**Before you commit a test**
1. Does it fail without the code and pass with it (did I see both)?
2. Does the name say unit, scenario and expectation?
3. Is it a straight line: no logic, one action, one concern?
4. Does it use only the public interface, with no private state?
5. Are doubles minimal (at most one mock), and stubs never asserted?
6. Is it isolated, deterministic and fast?
7. Are the unhappy paths and boundaries of this behavior covered?
8. Would I understand the failure message in six months?

**When reviewing tests**
Look for UT-1 to UT-22 in `references/unit-test-quality.md`; any test that cannot fail or asserts
nothing is a Blocker.

## Reference files (load on demand)
| File | Open when |
|---|---|
| `references/unit-test-quality.md` | Reviewing a suite or fixing brittle, slow or untrustworthy tests (catalog of test smells with IDs), or deciding what to do when a test fails |
| `references/tdd-practice.md` | Doing test-first, starting a test, choosing how to make a test pass (obvious, fake it, triangulate), specifying boundaries and ranges, value objects, tell-don't-ask design |
| `references/doubles-and-testability.md` | Choosing stubs/mocks/fakes, breaking dependencies, introducing seams, legacy code strategy, designing for testability |
| `references/bdd-and-acceptance.md` | Turning requirements into examples and scenarios, writing Given/When/Then, automating acceptance tests, living documentation |
