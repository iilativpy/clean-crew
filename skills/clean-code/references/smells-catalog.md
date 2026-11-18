# Smells and heuristics catalog

All 66 heuristics, paraphrased as checks. IDs are stable so review findings can cite
them (e.g. "G28: unencapsulated conditional").

## Comments (C)
- **C1 Inappropriate information** - no change history, authors, dates, ticket IDs in comments; that belongs in VCS / issue tracker.
- **C2 Obsolete comment** - a comment that no longer matches the code: fix or delete it now.
- **C3 Redundant comment** - says what the code already says (`i++ // increment`), or javadoc that only repeats the signature.
- **C4 Poorly written comment** - if a comment is worth writing, make it brief, precise, grammatical.
- **C5 Commented-out code** - delete it; version control keeps history.

## Environment (E)
- **E1 Build needs more than one step** - checkout + one command must build everything.
- **E2 Tests need more than one step** - one command (or one click) runs all tests.

## Functions (F)
- **F1 Too many arguments** - 0 best, then 1, 2, 3; more than 3 needs strong justification.
- **F2 Output arguments** - callers expect inputs; mutate the receiver instead.
- **F3 Flag arguments** - a boolean parameter means the function does two things; split it.
- **F4 Dead function** - never called -> delete.

## General (G)
- **G1 Multiple languages in one file** - minimize embedded HTML/SQL/JS/YAML etc. in a source file.
- **G2 Obvious behavior unimplemented** - principle of least surprise: implement what a reader would reasonably expect (e.g., case-insensitive day names).
- **G3 Incorrect behavior at the boundaries** - don't trust intuition; find every corner case and test it.
- **G4 Overridden safeties** - don't silence warnings, disable failing tests, or hand-tune safety mechanisms to "get it green".
- **G5 Duplication** - the most important rule. Copy-paste -> extract method; repeated switch/if chains -> polymorphism; similar algorithms -> Template Method / Strategy.
- **G6 Code at the wrong level of abstraction** - high-level concepts in base classes/modules, details in derivatives; separation must be complete (don't put `percentFull()` on a general `Stack`).
- **G7 Base class depends on derivatives** - base should know nothing of subclasses (exception: a fixed, co-deployed set such as a state machine).
- **G8 Too much information** - small, tight interfaces; hide data, utilities, constants; few methods, few instance vars, few protected members.
- **G9 Dead code** - impossible branches, never-thrown catches, unused utilities -> delete.
- **G10 Vertical separation** - locals declared right above first use; private functions just below first use.
- **G11 Inconsistency** - do similar things the same way (same names for the same kinds of things).
- **G12 Clutter** - empty constructors, unused variables, never-called functions, noise comments -> remove.
- **G13 Artificial coupling** - don't place general enums/constants/statics inside specific classes for convenience.
- **G14 Feature envy** - a method that manipulates another object's data probably belongs there (unless moving it would couple that object to an unrelated concern, e.g. report formatting).
- **G15 Selector arguments** - any argument (bool, enum, int) that selects behavior -> prefer several functions.
- **G16 Obscured intent** - dense expressions, Hungarian notation, magic numbers hide meaning; make intent visible.
- **G17 Misplaced responsibility** - put code where a reader would expect to find it (least surprise), not where it's convenient.
- **G18 Inappropriate static** - make it static only if it will never need polymorphic behavior; when in doubt, make it an instance method.
- **G19 Use explanatory variables** - break calculations into well-named intermediate values.
- **G20 Function names should say what they do** - if you must read the body to know what `add(5)` adds, rename it.
- **G21 Understand the algorithm** - don't tweak until tests pass; know *why* it works, then refactor until it's obvious.
- **G22 Make logical dependencies physical** - if module A assumes something about B (e.g., page size), ask B explicitly (`formatter.maxPageSize()`).
- **G23 Prefer polymorphism to if/else or switch** - "one switch" rule: at most one switch per type of selection, and it creates the polymorphic objects.
- **G24 Follow standard conventions** - a team coding standard based on industry norms, visible in the code itself.
- **G25 Replace magic numbers with named constants** - applies to any non-self-describing token, including magic strings in tests.
- **G26 Be precise** - no floats for money, don't assume the first query result is the only one, handle possible nulls and concurrent updates, declare the least constraining type (`List` not `ArrayList`).
- **G27 Structure over convention** - enforce decisions with structure (abstract methods, types) rather than naming conventions.
- **G28 Encapsulate conditionals** - extract boolean logic into an intention-revealing function.
- **G29 Avoid negative conditionals** - `if (buffer.shouldCompact())` over `if (!buffer.shouldNotCompact())`.
- **G30 Functions should do one thing** - split loop / check / act into separate functions.
- **G31 Hidden temporal couplings** - make required call order explicit, e.g. each step returns what the next one needs ("bucket brigade").
- **G32 Don't be arbitrary** - have a reason for the structure and make the structure communicate it (e.g., public classes at top level of their package, not nested for convenience).
- **G33 Encapsulate boundary conditions** - put `+1`/`-1` logic in one named variable (`nextLevel`).
- **G34 Functions should descend only one level of abstraction** - statements are all one level below the function's name.
- **G35 Keep configurable data at high levels** - defaults/config live at the top (e.g. argument parsing) and are passed down, not buried in low-level functions.
- **G36 Avoid transitive navigation** - Law of Demeter; no `a.getB().getC().do()`; collaborators offer the services you need.

## Java (J) - apply to Java/Kotlin, adapt for other languages
- **J1 Avoid long import lists** - wildcard imports are the traditional advice for 2+ classes from a package. *Note: many modern teams/linters forbid wildcards; follow the project's formatter/linter (G24) - the intent is "imports shouldn't drown the file".*
- **J2 Don't inherit constants** - don't implement an interface just to see its constants; use static imports or reference the owner.
- **J3 Constants versus enums** - use enums (with fields and methods) instead of `public static final int` codes.

## Names (N)
- **N1 Choose descriptive names** - and revisit them as meaning drifts; names are most of readability.
- **N2 Names at the appropriate level of abstraction** - don't leak implementation (`connect(locator)` rather than `dial(phoneNumber)` for a generic modem).
- **N3 Use standard nomenclature** - pattern names (`...Decorator`), language idioms (`toString`), the project's ubiquitous language.
- **N4 Unambiguous names** - the name should distinguish the function from its neighbors (`renamePageAndOptionallyAllReferences`).
- **N5 Long names for long scopes** - `i` fine for 5 lines; wide scopes need descriptive names.
- **N6 Avoid encodings** - no type/scope prefixes (`m_`, `f`, `I`-for-interface, Hungarian).
- **N7 Names should describe side effects** - `getOrCreateStream()`, not `getStream()` if it creates.

## Tests (T)
- **T1 Insufficient tests** - test everything that could break, not "what seems enough".
- **T2 Use a coverage tool** - it shows the gaps.
- **T3 Don't skip trivial tests** - cheap to write, valuable as documentation.
- **T4 An ignored test is a question about an ambiguity** - a disabled test marks an unresolved requirement; resolve it.
- **T5 Test boundary conditions** - the middle is usually right; edges are where bugs live.
- **T6 Exhaustively test near bugs** - bugs cluster; when you find one, test the neighborhood thoroughly.
- **T7 Patterns of failure are revealing** - the set of failing cases often points to the cause.
- **T8 Test coverage patterns are revealing** - unexecuted code can reveal why tests fail (or that an algorithm is wrong).
- **T9 Tests should be fast** - slow tests don't get run.
