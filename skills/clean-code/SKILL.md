---
name: clean-code
description: Code-level craftsmanship rules distilled from "Clean Code" (R. C. Martin) - naming, functions, comments, formatting, error handling, classes, unit tests and code smells. Use whenever writing, changing, refactoring or reviewing code at the line, function or class level, in any language.
---

# Clean Code - working rules

Apply these whenever you add or change code. They are defaults, not dogma: when the
project has an established convention that conflicts, follow the project and mention it.

## 0. Prime directives
- Code is read far more than written (the book estimates over 10:1). Optimize for the next reader - you are an author.
- Messes slow everyone down until productivity approaches zero, and a grand rewrite rarely
  saves you. Speed comes from keeping the code clean, and cleanup deferred "for later" never happens.
- **Boy Scout Rule**: leave the code you touched a little cleaner - but only inside the
  scope of the change (rename a bad local, extract one muddled block). No drive-by rewrites.
- Make it work -> make it right -> (only if measured) make it fast.
- Every change keeps the tests green. No tests around the area? Add them first.

## 1. Names
- Reveal intent: `elapsedDays`, not `d`. If a name needs a comment, rename it.
- No disinformation: don't call it `accountList` unless it is a List.
- Make meaningful distinctions - no `data`/`info`/`obj`/`a1`, `a2`, no noise words.
- Pronounceable and searchable; single-letter names only for tiny loop scopes.
- No encodings: no Hungarian notation, no `m_` prefixes, no `I` prefix just for interfaces
  (unless it's the project convention).
- Classes/objects -> nouns (`Customer`, `AddressParser`). Avoid `Manager`, `Processor`,
  `Data`, `Info` as vague catch-alls.
- Methods -> verbs (`save`, `deletePage`); accessors/predicates `get`/`set`/`is`/`has`.
- One word per concept (pick `fetch` OR `get` OR `retrieve`, stay consistent). Don't pun
  (don't reuse `add` for both "sum" and "append").
- Use solution-domain terms (patterns, algorithms) for technical concepts, problem-domain
  terms for business concepts.
- Name length ∝ scope size.
- No mental mapping: readers shouldn't translate `r` into "the URL without host" in their head.
- Don't be cute: `deleteItems`, not `holyHandGrenade`; no jokes or slang.
- Add meaningful context: group related names in a well-named class/function
  (`Address.state`, not a loose `state`); prefix only as a last resort.
- Don't add gratuitous context: no app-wide prefixes (`GSDAccountAddress`); shorter is
  better as long as it's clear.

## 2. Functions
- **Small.** Then smaller. A function body rarely needs more than ~20 lines; blocks inside
  `if`/`else`/`while` ideally one line (a well-named call).
- **Do one thing.** Test: can you extract another function with a name that isn't a restatement?
  Then it does more than one thing.
- **One level of abstraction per function.** Don't mix `getHtml()` with `.append("\n")`.
- **Stepdown rule**: read top-to-bottom like a narrative; callers above callees.
- **Switch/when statements**: bury them once, in a factory, to create polymorphic objects.
  Don't repeat the same switch across the code.
- **Arguments**: 0 ideal, 1–2 fine, 3 needs justification, more -> introduce a parameter
  object. **No boolean flag arguments** - split into two functions.
- No output arguments; if a function must change state, change the state of its owner.
- **No hidden side effects.** `checkPassword()` must not also initialize a session.
- **Command–Query Separation**: a function either does something or answers something.
- **Prefer exceptions to error codes**; extract `try` bodies into their own function -
  error handling is "one thing".
- **DRY**: duplication is the root of much evil; remove it via extraction/abstraction.
- Single entry/exit (structured programming) matters for big functions; in small ones an
  early `return`/`break` is fine. Never `goto`.
- Nobody writes functions like this on the first pass: draft, then refine under tests
  (see `references/refactoring-process.md`).

## 3. Comments
- A comment is a failure to express yourself in code. First try to rename/extract.
- Good comments: legal headers, explanation of *intent* or *why*, warnings of consequences,
  clarification of an unchangeable API, TODOs (tracked), public API docs.
- Bad comments: redundant restatements, mandated javadoc on every getter, journal/changelog
  comments, position markers/banners, closing-brace comments, attributions, **commented-out
  code (delete it - VCS remembers)**, misleading or outdated comments.
- Never leave a comment that lies after your change - update or delete it.

## 4. Formatting
- Newspaper metaphor: high-level at top, details below.
- Vertical openness between concepts; vertical density for tightly related lines.
- Declare variables close to use; instance variables at the top of the class.
- Dependent functions close together, caller above callee.
- Keep lines short; follow the team formatter. Formatting is not negotiable per-file.

## 5. Objects and data structures
- Objects hide data and expose behavior; data structures expose data and have no behavior.
  Don't build hybrids (public getters/setters for everything **plus** business logic).
- **Anti-symmetry trade-off**: procedural code (data structures + functions) makes adding
  new *functions* easy and new *types* hard; OO makes new *types* easy and new *functions*
  hard. Choose per area based on which will change more - "everything is an object" is a myth.
- Don't add getters/setters reflexively; expose abstract operations (`percentFuelRemaining()`),
  not raw fields.
- Active Record / ORM entities are data structures - keep business rules in separate objects.
- **Law of Demeter**: a method talks to its own fields, params, objects it creates - not to
  strangers. Avoid train wrecks `a.getB().getC().doIt()`; tell, don't ask.
- DTOs/records are fine at boundaries - keep business logic out of them.

## 6. Error handling
- Use exceptions, not return codes. Prefer unchecked exceptions in application code.
- Provide context in exceptions: what operation failed and why.
- Define exception classes by how the **caller** needs to handle them; wrap third-party APIs
  so their exceptions don't leak.
- **Define the normal flow**: don't let exception handling clutter business logic; use the
  Special Case pattern so the caller never needs to handle the exceptional case.
- **Don't return null. Don't pass null.** Return empty collections, `Optional`, or a
  Special Case / Null Object.
- Write the `try-catch-finally` first when code can throw - it defines the transaction scope.

## 7. Boundaries (third-party code)
- Wrap third-party APIs behind your own interface; don't pass e.g. a raw `Map` with library
  types around the system.
- Write **learning tests** for third-party libraries - they also detect breaking upgrades.
- For not-yet-existing APIs, define the interface you wish you had, and adapt later.

## 8. Unit tests
- TDD laws: no production code without a failing test; just enough test to fail; just enough
  code to pass.
- Test code is held to the same quality as production code.
- One concept per test; minimal asserts per test; Given–When–Then / Build–Operate–Check.
- Dirty tests are worse than none: they slow every change and end up abandoned, and then
  production code rots because nobody dares to change it. Tests keep code flexible.
- Build a domain-specific testing language (helpers) for readability.
- Dual standard: test code must be just as readable, but needn't be as efficient as
  production code (string concatenation, etc. are fine in tests).
- One assert per test is a good guideline; keep asserts to a minimum.
- **F.I.R.S.T.**: Fast, Independent, Repeatable, Self-validating, Timely.

## 9. Classes
- Small - measured in **responsibilities**, not lines. If you can't describe the class in
  ~25 words without "and/or/but", it has too many.
- **Single Responsibility**: one reason to change.
- **High cohesion**: most methods use most fields. Losing cohesion -> split the class.
- Organize for change: open for extension, closed for modification; depend on abstractions
  to isolate from change.
- Order: constants, static vars, instance vars, public methods, private helpers right
  after the public method that uses them.
- Keep things private; loosen to package/protected for a test only as a last resort.
- Splitting big functions often reveals new classes: when a group of extracted functions
  shares a subset of variables, that's a class trying to get out.
- Isolate from change: depend on interfaces for volatile details (e.g. an external price
  API) so tests can substitute them.

## 10. Systems, emergence, concurrency (summaries - open the reference for details)
- Separate construction (wiring, `main`, DI container) from use; keep domain logic in
  plain objects; handle cross-cutting concerns non-invasively -> `references/systems.md`.
- Kent Beck's simple design, in priority order: **runs all tests -> no duplication ->
  expresses intent -> minimal classes and methods.**
- Concurrency: keep concurrency code separate, limit shared data, prefer copies and
  independent threads, keep synchronized sections small, design shutdown early, break
  deadlock by global lock ordering -> **always open `references/concurrency.md` when the
  change touches threads, async, locks or shared mutable state.**

## 11. Smell checklist (run before finishing a change)
Quick list below; the full 66-item catalog with IDs (C1–C5, E1–E2, F1–F4, G1–G36, J1–J3,
N1–N7, T1–T9) is in `references/smells-catalog.md` - use it for reviews and cite IDs.
**Comments** - obsolete, redundant, commented-out code, poorly written.
**Environment** - build or tests need more than one step.
**Functions** - too many args, output/flag args, dead functions.
**General**
- Duplication (copy-paste, repeated switch/if chains, similar algorithms)
- Code at the wrong level of abstraction; base class depending on derivatives
- Too much information exposed; dead code; vertical separation too large
- Inconsistency; clutter; artificial coupling; feature envy (method uses another class's data more than its own)
- Selector/flag arguments; obscured intent; misplaced responsibility
- Inappropriate static (should be polymorphic); missing explanatory variables
- Function names that don't say what they do; not understanding the algorithm
- Logical dependency instead of physical dependency
- if/else or switch chains instead of polymorphism; unencapsulated conditionals
  (`if (shouldBeDeleted(timer))` beats `if (timer.hasExpired() && !timer.isRecurrent())`)
- Negative conditionals; functions doing more than one thing; hidden temporal couplings
- Magic numbers -> named constants; imprecision (floats for money, ignoring concurrency)
- Transitive navigation (Law of Demeter); boundary conditions not encapsulated
**Names** - non-descriptive, wrong abstraction level, ambiguous, encoded, hiding side effects.
**Tests** - insufficient, untested boundaries, ignored tests, slow tests, no coverage tool,
skipping trivial tests, not testing near known bugs.

## Reference files (load on demand)
| File | Open when |
|---|---|
| `references/smells-catalog.md` | Reviewing a diff/PR, or self-review of a non-trivial change |
| `references/concurrency.md` | Threads, executors, async, locks, pools, shared mutable state |
| `references/systems.md` | DI/wiring, startup, config, frameworks, cross-cutting concerns |
| `references/refactoring-process.md` | Any refactoring, or when a change is getting messy |

## How to apply in a change
1. Read the surrounding code and existing conventions first.
2. Write/adjust the test.
3. Implement in small, well-named functions.
4. Run the smell checklist on **your diff** and fix what you introduced.
5. Report any pre-existing smells you saw but deliberately left out of scope.
