# Test-first practice (TDD)

Open when doing test-first development, starting a hard test, or driving an implementation from
examples. Tests are treated as **statements of a specification**: each one states a behavior the code
must have, and the suite is the executable specification.

## The mindset
- A test is a **statement of a behavior you wish were true**. Write it first, run it, see it **fail**,
  then make it true. The transition red to green is the proof of progress.
- Test-first means **seeing a failure**. Three ways a statement can pass without proving anything,
  all avoided by watching it fail first:
  1. it was **never run** (not marked as a test, not collected, wrong name pattern);
  2. it **asserts the wrong thing** (a typo, an always-true condition, an assertion on a copy);
  3. it **passes whatever the code does** (it asserts a default that the code never changes).
- **Test-after often ends as test-never.** After the code works there is little desire to specify it,
  error paths are skipped, and the design is set. Writing the specification first keeps it complete.
- **The suite is a specification:** names read as requirements, and a failing test says which
  requirement is violated.
- **Mind the scope:** TDD is not only unit tests; the same loop applies at acceptance level
  (`bdd-and-acceptance.md`).

## How to start a test (when the blank page stalls you)
1. **Start with a good name.** Ask: what is the scope (which unit)? what behavior am I specifying?
   Write the name as a sentence (`should_display_all_entered_digits_that_are_not_leading_zeroes`).
   A good name states the behavior, not the method.
2. **Fill Given/When/Then with the obvious.** Write the three lines in plain language, then translate
   them literally into code, ignoring missing classes and functions; then add what is missing so it
   compiles and runs.
3. **Start from the end.** Write the assertion first (what must be true), then the action that
   produces it, then the set-up that makes it possible.
4. **Start by invoking a method** when the signature is already fixed (a new class implementing an
   existing interface): call it, and work out the rest.
Pick the one that fits; they combine.

## Good names (statements) vs bad ones
- **Good:** describes one behavior and its condition: `should_reject_withdrawal_larger_than_balance`,
  `charges_half_price_for_the_second_bag`.
- **Bad:** names that repeat the method (`test_withdraw`), numbered (`test2`), technical (`calls_repo`),
  or that hide the rule (`works_correctly`). A bad name is a symptom of treating the suite as
  leftovers; it rots into an unreadable pile.

## Scope of a unit-level statement
- A statement specifies **one behavior** of a unit through its **public API**: "one example of how
  this class is valuable to its users". It introduces a **behavioral distinction not specified
  before**.
- It may involve several real objects (a string, an exception, the value object): the unit is a unit
  of behavior, not necessarily one class.
- **How many assertions?** One **logical** assertion: one or more physical asserts that together state
  one condition (a list has unique items; an exception of a given type with a given property). Several
  unrelated asserts are several tests.
- Failing these rules (testing several behaviors, asserting internals) yields brittle tests.

## Developing a style: anonymous values and constrained non-determinism
Tests should convey the **rule**, not just pass values in and compare literals.
- **Anonymous input:** if any value of a kind will do (any string, any positive int), say so:
  `host = any_string()`. A literal like `"MY_HOST_NAME"` raises questions: must it be upper case?
  does it need an underscore? An anonymous value documents the equivalence class.
- **Derived values:** build the expected result from the inputs with the simplest expression
  (`f"backup_{host}_{user}.zip"`), so the rule is visible and the test does not hide it in a literal.
  (This is a simple concatenation, not a copy of the algorithm.)
- **Distinct generated values:** when two inputs are anonymous, make them differ (generate a unique
  value each call). Otherwise the code can swap the arguments and the test still passes.
- **Constant specification:** when production defines a constant that is part of the rule (a version
  number, a prefix), reference it in the test instead of repeating its value, so the test does not break
  when the constant is changed legitimately. Use the literal when the exact value is the specification.
- **Constrained non-determinism:** generated values are constrained to the equivalence class (a
  positive integer, a non-empty alphanumeric string). If a failure shows up only for some generated
  values, you have found a boundary: write a statement for it.

## Functional boundaries and conditions
An anonymous value is not enough when behavior **changes at a boundary** (a license expires, a shop
opens at 10, an age of 18, a value below zero, a rate limit).
- A boundary has **two edges and a length**: the last value of one behavior and the first value of the
  next (17 and 18). Specify **both edges** (and neighbors for ranges), not just one side.
- For a **range** ("from 10 to 18 inclusive"), specify each edge value and a value inside and outside.
- For **exceptions** to a general rule, specify the rule with anonymous values and each exception
  explicitly (`0`, negatives, empty, maximum).
- For **conditions** on enums or combinations, enumerate the combinations that matter, using a
  parameterized test with literal rows for readability.
- **Time** is a boundary source: inject the clock and test just before, at, and just after the edge.

## Driving the implementation from the statement
Three techniques to go from a false statement to a true one:
1. **Type the obvious implementation** when the correct code is clear and small (`return a + b`).
   Match the number of statements to your confidence.
2. **Fake it ('til you make it):** make the test pass with the most obvious cheat (return the
   constant the test expects), then **remove the duplication** between the test and the fake by
   replacing constants with variables and parameters, step by step, until the real logic emerges.
3. **Triangulate:** the most conservative: write a second example with different values, and only
   then generalize the code, because **two examples reveal what varies**. Repeat with more examples
   until the implementation is general.
Use the obvious implementation for trivial logic, fake it when you are unsure of the shape,
triangulate when the rule is subtle. Always refactor once green.

## Test-driven design in an object world
- **Compose objects, tell them what to do.** Prefer objects that receive what they need and **tell**
  collaborators what to do rather than **asking** for their data and deciding: this is easier to
  compose and to test (the outgoing call is the result). Most getters can be removed; fewer return
  values means fewer asserts on state.
- **Roles and small interfaces:** specify the **protocol** between collaborating roles (who sends
  which message, in what order, with what meaning) with mocks, at role boundaries, in a small number
  of tests. Keep interfaces small and shaped by the **sender's intention** (`login_with(user, pw)`,
  not three setters). Protocols have rules (what -1 means, call order); a mock-based test states
  them.
- **Single responsibility:** a class that is hard to test usually does several jobs: split it.
- **Value objects:** small immutable objects defined by their value (money, names, ids, ranges) with
  equality, a clear string form, and validation at construction. They are cheap to test, remove
  primitive obsession and make illegal values unrepresentable. Test their equality, hash,
  immutability and invalid-construction cases.
- **Composition root:** objects are wired in one place at the edge; tests wire them differently
  (with fakes). Tests also exercise the wiring at a coarser level (an acceptance or integration test).
- **Mocks as a tool, not a religion:** use them to specify protocols and replace slow or external
  collaborators; do not mock value objects or everything. If a test is full of mocks, the design
  probably couples too many collaborators.

## Anti-patterns in the TDD process
- Writing many tests, then all the code (lose the feedback and the design pressure).
- Writing the test after the code and calling it TDD.
- Never refactoring (the green step is half the loop).
- Testing implementation (private state, call order) instead of behavior, then being unable to refactor.
- Skipping the failing run.
- Large steps: if you cannot make the test pass in a few minutes, the step is too big; write a smaller
  test.
- Over-specifying with mocks before the design is clear.
