# Test doubles, seams, legacy code and testability

Open when choosing between stubs, mocks and fakes, breaking a dependency, making code testable, or
adding tests to code that has none.

## Vocabulary
- **Test double:** anything that stands in for a real dependency in a test.
- **Stub:** returns canned answers so the unit under test can run (an *indirect input*). It **cannot
  fail the test**; the assert is always on the unit under test.
- **Mock:** records how it was used and the test **asserts on the mock** (an *indirect output* checked
  as the result). It can fail the test.
- **Fake:** a working but simplified implementation (in-memory repository, fake clock, local
  queue). Use fakes when the dependency has real behavior the test relies on.
- **Spy:** a stub that also records calls for later assertion.
- **System under test (SUT):** the thing being tested. **Seam:** a place where you can replace
  behavior without editing the code under test.
A quick check: if the test would still pass whether or not a double was called, it is a stub. If the
test asserts on the double, it is a mock.

## The three kinds of end results
| End result | Test style | Example |
|---|---|---|
| A **return value** | value-based: call, compare | `price = total(items)` |
| A visible **state or behavior change** | state-based: act, then query the unit through its public interface | after `add_user`, `login` succeeds |
| A **call to a third party** that returns nothing you use | interaction-based: assert the call on a mock | `logger.write("..")`, `notifier.send(...)` |
Choose in that order. **Interaction tests are the last resort**: they are the most coupled to
implementation. Use them when the outgoing call *is* the requirement.

## Rules for doubles
1. **At most one mock per test**; everything else is a stub or fake. If you cannot say which double is the
   mock, the test checks too much.
2. **Never assert on a stub.** Asserting that a stub was called is overspecification (the test fails when
   the production code gets its data a different way).
3. **Do not verify what the unit's result already proves.** If the returned value shows the repository
   was consulted, do not also verify the call.
4. **Match what matters in arguments** (the order id, the amount), not everything (not exact wording of
   a log message, not unrelated fields). Over-tight matching breaks on harmless change.
5. **Do not chain fakes** (a stub returning a stub returning a mock). It signals that the code under test
   navigates too far; pass in the object it needs.
6. **Do not mock simple values or things you own that are cheap to construct.** Use the real object.
7. **Do not mock the unit under test** (partial mocks). Split the class instead.
8. **Keep doubles readable:** a small named fake (`InMemoryOrderRepository`, `FakeClock`) beats a long
   mock configuration repeated in every test.
9. **Isolation frameworks** make doubles cheap but invite traps: unreadable tests full of expectations,
   verifying the wrong thing, several mocks, overspecification. Prefer handwritten fakes for anything
   with behavior; use the framework for one-line stubs and for the single mock.

## Breaking dependencies: making a seam
Code that reads the clock, the file system, the network, a global or a singleton is hard to test. The
ways to introduce a seam, from preferred to last resort:
1. **Dependency injection through the constructor** (or a parameter): the unit receives a collaborator
   (an interface or function type) instead of creating it. The production wiring passes the real one;
   the test passes the double.
2. **Extract an interface or function type** in front of the dependency, owned by the code that uses it
   (this is also the architectural port).
3. **Extract and override** (in languages with inheritance): move the call into a small overridable
   method and override it in a test subclass.
4. **Parameter with a default** (dynamic languages): `def f(x, now=time.time)`.
5. **Property or setter injection** for optional collaborators. Prefer constructor injection for required
   ones.
6. **Patch at the module level** (`monkeypatch`, `jest.mock`): acceptable in dynamic languages at a real
   seam, and for legacy code; patch where the name is *used*, not where it is defined; always undo it.
Avoid logic in constructors, static calls in logic, singletons that hide dependencies, and creating
concrete collaborators inside methods with logic.

## Designing for testability
- **Seams everywhere the code meets I/O, time, randomness and third parties.**
- **Separate decision logic from effects:** a pure function that decides, and a thin shell that fetches
  inputs and applies results. The pure part needs only value-based tests.
- **Small classes with one job and a few dependencies.** A test that needs ten doubles points at a class
  doing too much.
- **Prefer composition and interfaces; avoid deep inheritance.**
- **Do not make everything virtual or public just for tests.** Testing private methods is a smell; extract
  a collaborator if the logic deserves its own tests.
- **Trade-offs:** designing for testability takes some extra work and indirection, and there is a risk of
  over-engineering. Weigh it: put seams at real boundaries and where tests need them, not everywhere.
  Testable code is not automatically well-designed code; combine this with the design skills.
- **Dynamic languages** reduce the need for interfaces (duck typing, passing functions) but not for seams.

## Legacy code: adding tests to code without any
1. **Do not start with a rewrite.** The goal is to change safely.
2. **Pick where to start.**
   - **Easy first:** components with few dependencies. Quick wins and the team learns the techniques; the
     hard ones get harder to approach as time passes if never started.
   - **Hard first:** components with many dependencies and a lot of logic. Slow at the start, then the cost
     falls steeply because each one you untangle helps the others; needs a team that already knows the
     techniques.
   - **Mix**, guided by *priority* (what is about to change, what is most important) and *cost*.
3. **Write integration-level (outer) tests before refactoring** code you cannot isolate yet, to catch
   breakage during the refactoring; replace them with focused unit tests as seams appear.
4. **Characterization tests:** run the code, record what it actually does (including odd results), and
   assert that. They describe behavior, not correctness; mark surprises as follow-ups.
5. **Introduce seams with the smallest safe change** (extract and override, parameter with default,
   a thin interface) before deeper redesign.
6. **New behavior is test-first** even in a legacy file; leave the old lines alone beyond what the change
   needs.
7. **Track it:** a short list of "components without tests", with the risky ones first.

## Choosing where a test lives (levels)
- **Unit:** one unit of work through its public API, with doubles for I/O and slow things. Fast.
- **Integration:** the unit with a real database, queue or file system; fewer, slower, in a separate suite.
- **Acceptance / end to end:** a user-visible behavior through the outermost layer you can drive. Few
  and valuable. Put business rules at the cheapest level that proves them.
- **Contract tests** for adapters to external systems: one test per assumption you rely on.
Keep separate suites, separate commands, separate failure meanings.
