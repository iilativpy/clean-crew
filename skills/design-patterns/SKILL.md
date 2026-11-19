---
name: design-patterns
description: Judgment about object-oriented design patterns - when a pattern is warranted, which one fits what varies, the lightest form of it in the language at hand, and when it is over-engineering. Covers the 23 classic patterns plus the game and performance patterns (game loop, component, event queue, object pool, data locality, dirty flag, and others). Use when a design has a variation point, a smell that a pattern cures (type-code switches, subclass explosion, tangled collaborators), when asked "which pattern", and when reviewing code that already uses patterns.
---

# Design Patterns - working rules

A pattern is a named, proven answer to a recurring design problem. It is a trade: you buy
flexibility along **one** axis of change and pay with indirection. Use this skill to decide
**whether** to buy, **which** pattern, and **how light** a form is enough. Pair it with
`clean-architecture` (where code goes) and `clean-code` (how it is written).

This file is short enough to read in full. The catalogs are in `references/` and are opened by
need (table at the end).

## 0. Principles behind the patterns [write, review]
- **Program to an interface, not an implementation.** Callers depend on what an object can do,
  not on its concrete class. Creating the concrete object is the one place that must name it
  (factories, composition root).
- **Favor composition over inheritance.** Inheritance reuses by exposing the parent's insides
  (white-box) and is fixed at compile time; composition reuses by holding another object with a
  small interface and **delegating** to it, and can change at run time. Inheritance and
  composition work together, but designs usually get simpler by leaning on composition. Heavy
  composition has its own cost: more objects, harder to follow the flow.
- **Encapsulate what varies.** Every pattern lets one aspect change without disturbing the rest.
  Find the aspect first (section 2); the pattern follows from it.
- **Apply a pattern only when the flexibility is actually needed.** Patterns add indirection,
  classes and sometimes run-time cost. The pattern is justified by a variation that exists or is
  concretely planned, never by "it might be useful".
- **Simplicity is a scarce resource.** Complexity slows every later change. Prefer the simplest
  thing that works, and a pattern earns its place only when it removes more complexity than it
  adds.

## 1. Do I need a pattern? Ask in this order [write, review]
1. **What varies, and what is the second variant?** If you cannot name a second variant that exists
   or is committed to (a requirement, a ticket), do not abstract yet. One implementation behind
   an interface is usually speculation. Wait for the second or third case, then refactor toward
   the pattern; the duplication shows you the right seam.
2. **Would something plainer do?** A function parameter, a callback, a lookup table (dict/map),
   an enum with a `match`, a data class, a module-level function or a built-in language feature
   covers many patterns at no cost (see `references/patterns-in-modern-languages.md`). Use it.
3. **What does it cost?** Count the new classes/files, the extra hop a reader must follow, the
   places that must change when a variant is added, and any performance cost. Is the benefit
   larger, in this code base, now?
4. **Does the project already have a convention for this?** Follow its idiom and its names. Do not
   add a second style next to the first.
5. **Will the next reader recognize it?** Name participants for the domain (`RefundPolicy`,
   `RetryingClient`, `InvoiceBuilder`), include the pattern word when it helps (`...Strategy`,
   `...Adapter`), and avoid abstract names (`ConcreteStrategyA`, `Manager`, `Helper`).
If steps 1-3 do not clearly favor the pattern, **do not use it**, and say so briefly.

## 2. From the problem to the pattern [write, review]
### 2a. What do you want to be able to change without redesign?
| You want to vary... | Candidates |
|---|---|
| which concrete class gets created; families of products | Factory Method, Abstract Factory, Prototype, (DI / composition root) |
| how a complex object is assembled step by step | Builder (or named/optional arguments) |
| the sole instance of something | Singleton (usually: just create once and pass it in) |
| the interface of an existing object to fit what clients expect | Adapter |
| the implementation behind an abstraction, independently of the abstraction | Bridge |
| structure: part-whole trees treated uniformly | Composite |
| responsibilities added to an object, combinably, without subclasses | Decorator |
| the interface to a whole subsystem (simplify it) | Facade |
| storage cost of huge numbers of similar objects | Flyweight |
| how or where an object is accessed (lazy, remote, protected, cached) | Proxy |
| which object handles a request | Chain of Responsibility |
| when and how a request is carried out; undo, queue, log | Command |
| the grammar of a small language | Interpreter |
| how a collection is traversed | Iterator |
| how a group of objects talk to each other | Mediator |
| what state is saved outside an object, for undo/checkpoints | Memento |
| how many objects depend on one, and how they are told | Observer |
| behavior that depends on the object's state | State |
| an algorithm, chosen at run time | Strategy |
| the steps of an algorithm whose skeleton stays fixed | Template Method |
| operations added to a stable class hierarchy | Visitor (or pattern matching) |

### 2b. Symptom in the code -> pattern that may cure it
| You see | Consider (lightest first) |
|---|---|
| `if/elif` or `switch` on a type code, kind or status, repeated in several places | polymorphism; Strategy (algorithm choice) or State (lifecycle); build once in a Factory |
| Boolean flags piling up, methods full of `if (status == ...)` | State |
| Constructor or function with many optional parameters; telescoping overloads | named/optional arguments or a parameter object; Builder if construction has stages |
| Subclass per combination of features (`BorderedScrollableView`, `CsvGzipExporter`) | Decorator, Strategy or Bridge (composition) |
| Two independent dimensions in one hierarchy (shape x renderer, message x channel) | Bridge |
| Same algorithm skeleton copied with small differences | Template Method (same flow) or Strategy (swap the varying part) |
| Third-party or legacy API used all over the core | Adapter behind a port you own (see `clean-architecture`); Facade over a messy subsystem |
| Clients repeat the same 5-step dance on a subsystem | Facade |
| Peers hold references to many peers, changes ripple everywhere | Mediator (tight group) or Observer/events (loose notification) |
| Cross-cutting add-ons: logging, caching, retry, auth, metrics around a call | Decorator/Proxy or middleware (Chain of Responsibility) |
| Need undo/redo, a job queue, an audit of operations, macros, retry later | Command (with Memento or inverse operations for undo) |
| Object tree where leaves and groups should be handled the same | Composite |
| Need many new operations on a stable set of node types | Visitor, or pattern matching on sum types |
| Concrete classes named in many `new` calls; hard to swap in tests | Factory; constructor injection |
| An object is expensive and often not used | Proxy (lazy/virtual), or lazy initialization |
| "Manager", "Helper", "Util", or a global accessor everyone calls | remove the class; give the behavior to the owner; pass the dependency in (see Singleton) |
| Notifications to an unknown, changing set of listeners | Observer / events |

## 3. Choosing the lightest form [write]
Many patterns are language features in disguise. Reach for these first:
- **Strategy / Command / Template Method (with hooks) / Observer callbacks:** a function, lambda or
  callable passed in. Use a full class when the object needs state, several operations (undo), a
  name for logging, or serialization.
- **Iterator:** the language's iterator/generator protocol. Write one by hand almost never.
- **Singleton:** create the object once at the composition root and pass it to whoever needs it.
  If you must expose it, a module-level instance is a last resort; never a hidden global that
  production code reaches into.
- **Prototype:** `copy.deepcopy`, spread/`structuredClone`, `with`-style record copies.
- **Builder:** keyword arguments, defaults, a dataclass or record. Keep a real builder for staged or
  validated construction.
- **Visitor:** pattern matching over a closed set of types (sealed classes, enums, unions).
- **Factory:** a function returning an object, a dict of constructors, or a DI container.
- **State:** an enum plus a transition table, until behavior per state outgrows a table.
- **Decorator / Chain:** function wrappers, annotations, middleware lists.
Decide the form by what the variants **do**: only a name and a value -> data; one operation -> a
function; several related operations or state -> a class.

## 4. How to apply a pattern [write]
1. Read the pattern's **Applicability and Consequences** first (in `references/gof-catalog.md`):
   is it right for this problem, and what does it cost?
2. Write the failing test or pin the current behavior with tests. Introduce the pattern as a
   **refactoring**, in small steps, tests green throughout. Do not introduce a pattern and a
   behavior change in one step.
3. Name the participants for the domain, keeping the pattern word where it helps. Keep operation
   names consistent (a `create_` prefix for factories, `visit_` for visitors).
4. Define the interface from the **caller's** need. Keep it narrow: every operation on the shared
   interface is something every variant must implement. If some variants do not fit (they need
   different inputs or a different scope), split the interface instead of widening it. Where a
   variant has parameters, make it a small immutable value that validates itself on construction
   (see `secure-design` section 9), so an invalid variant cannot exist.
5. Put the construction (which variant) in one place: the composition root or a factory. The
   pattern's clients never name concrete variants.
6. Add a test for **each variant** and for the seam (how the client uses the interface), not only
   for the happy path.
7. Record the decision where the team will see it: a one-line comment or the commit message
   stating *what varies* and *why a pattern*. Remove the pattern again if the variation never
   arrives (deleting an unneeded abstraction is a legitimate refactoring).

## 5. Over-use and misuse to watch for [review]
- **Pattern for its own sake:** an interface with one implementation and no boundary behind it
  (IO, framework, external system); a factory that builds a single class; a builder for a
  three-field object; a Strategy that never gets a second strategy.
- **Singleton and "manager" globals:** hidden shared state makes code hard to reason about,
  couples unrelated modules (a physics class that can reach the audio player), and breaks tests and
  concurrency. Prefer passing dependencies; see `references/game-and-performance-patterns.md`.
- **Service Locator** as the default way to get dependencies: it hides what a class needs. Use
  sparingly, for genuinely ambient services (logging, clock) and keep a safe default.
- **Observer cascades:** unexpected updates, notification order, listeners that outlive their
  subject (leaks), notifying while the subject is inconsistent. Keep the event contract explicit
  and unregister on teardown.
- **Mediator or Facade that grows into a god class:** the coupling moved, it did not go away.
- **Inheritance used where composition fits:** Template Method and abstract base classes with a
  long chain of hooks; a brittle base class that accretes helpers.
- **Visitor on an unstable hierarchy:** adding a node type forces every visitor to change. Use it
  only when the types are stable and the operations keep growing.
- **Decorator chains whose order matters but is not documented;** transparent wrappers that break
  identity checks.
- **Interpreter/bytecode/scripting "just a tiny language":** it grows. Start only when the need is
  real and control its scope.
- **Premature optimization patterns** (Flyweight, Object Pool, Data Locality, Dirty Flag, Spatial
  Partition): apply only after profiling shows the problem they solve. They trade clarity for speed.
- **Pattern soup:** several patterns stacked to solve one small requirement. If the explanation of
  the design takes longer than the code, simplify.
- **Names that hide intent:** `AbstractFactoryFactory`, `ConcreteStrategyA`, `Manager`. The domain
  name comes first.

## 6. Quick checklist for a design with patterns [write, review]
1. Can I state in one sentence what varies and what the second variant is?
2. Is there a plainer option (function, table, enum, language feature) that suffices?
3. Is the interface narrow and shaped by the caller, with concrete variants chosen in one place?
4. Do dependencies still point inward (the pattern did not pull a framework into the core)?
5. Does every variant have a test, and do the tests survive swapping a variant?
6. Is it named for the domain, and recognizable to the next reader?
7. If I removed the pattern, would the code get simpler? If yes, remove it.

## Reference files (load on demand)
| File | Open when |
|---|---|
| `references/gof-catalog.md` | Choosing between candidate patterns, or reading a pattern's applicability, consequences and misuse before applying it or reviewing it |
| `references/patterns-in-modern-languages.md` | Deciding the lightest form in Python, JavaScript/TypeScript, Java, Go or C# |
| `references/game-and-performance-patterns.md` | Game loops and entities, real-time systems, event queues, pooling, caching derived data, data layout, spatial queries, or when a Singleton or State machine is tempting |
