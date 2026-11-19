# Classic pattern catalog (the 23 object-oriented patterns)

One card per pattern: what it does, when it fits, what it costs, how to keep it light, and what
it is often confused with. Wording is condensed and original; the examples are generic. Use
`SKILL.md` section 1 first: most problems do not need a pattern at all.

Related patterns are named in brackets. "Light form" points to
`patterns-in-modern-languages.md`.

## How the catalog is organized
- **Creational** patterns abstract *how objects get created*, so code depends on interfaces
  instead of concrete classes.
- **Structural** patterns compose classes and objects into larger structures.
- **Behavioral** patterns distribute responsibility and describe how objects communicate.
Patterns of the same kind are alternatives: read the comparison notes at the end of each group.

---
## Creational

### Abstract Factory
- **Does:** provides an interface for creating a *family* of related products without naming their
  concrete classes.
- **Use when:** the system must work with one of several families (a UI toolkit per platform, a
  set of database-access classes per vendor) and products of one family must be used together.
- **Cost:** adding a new *kind* of product changes the factory interface and every concrete
  factory. Overkill when there is one family.
- **Light form:** pass a small object, module or dict of constructors chosen once at startup.
- **Often confused with:** Factory Method (one product, chosen by subclass); Prototype (a factory
  that clones).

### Builder
- **Does:** separates the step-by-step construction of a complex object from its representation, so
  the same construction process can produce different results.
- **Use when:** the object has many optional parts, construction has stages or validation, or one
  recipe must yield several representations (an HTML and a text rendering of the same document).
- **Cost:** an extra builder per product kind; a mutable builder must not be reused carelessly.
- **Light form:** keyword/optional arguments, a dataclass/record with defaults, or a fluent helper
  that validates in `build()`.
- **Often confused with:** Abstract Factory (creates families in one shot; Builder works stepwise
  under a director's control).

### Factory Method
- **Does:** defines a method for creating an object and lets a subclass (or a parameter) decide
  which class to instantiate.
- **Use when:** a class cannot anticipate the type of object it must create; subclasses specify the
  objects; you want to localize which helper class is used.
- **Cost:** a client may have to subclass just to change the product, which is a new point of
  evolution. A parameterized factory function often removes that.
- **Light form:** a function (or injected callable) that returns the object; a registry of
  constructors keyed by name.
- **Often confused with:** Template Method (a factory method is usually a step in one).

### Prototype
- **Does:** creates new objects by copying a registered prototypical instance.
- **Use when:** classes to instantiate are chosen at run time; to avoid a factory hierarchy parallel
  to the product hierarchy; when instances differ by a few combinations of state (clone a
  preconfigured one).
- **Cost:** every product must implement a correct clone; deep vs shallow copy and cyclic
  references are the usual bugs.
- **Light form:** `copy`/`deepcopy`, spread, `structuredClone`, immutable copy-with.

### Singleton
- **Does:** ensures a class has one instance and provides global access to it.
- **Use when:** there truly must be exactly one (a hardware port, a process-wide registry) **and**
  you accept global access. Treat the two halves separately: "one instance" is sometimes needed,
  "global access" almost never is.
- **Cost:** it is a global variable. It hides dependencies, couples modules that should not know each
  other, makes tests share state, and is unfriendly to concurrency. Lazy initialization and
  subclassing add more traps.
- **Instead:** see if the class is needed at all (a "manager" often only babysits other objects);
  create one instance at the composition root and pass it in; enforce single use with an assertion
  if one instance is a hard rule; reach it through an object that is already available.
  See `game-and-performance-patterns.md`.

**Creational comparison.** Factory Method needs a subclass per variation; Abstract Factory and
Prototype configure a client with an object instead. Prototype avoids a factory hierarchy. Builder
controls assembly. Often a design starts with Factory Method (simple) and moves to the others when
more flexibility is proven necessary.

---
## Structural

### Adapter
- **Does:** converts the interface of an existing class into the one clients expect.
- **Use when:** you must use a class whose interface does not match; you want one reusable class to
  cooperate with unrelated classes; you want to keep a third-party API out of your core.
- **Cost:** the amount of adapting ranges from renaming operations to supporting different
  operations. A class adapter commits to one concrete adaptee; an object adapter can wrap the
  adaptee and its subclasses but makes overriding harder.
- **Keep it clean:** do not leak the adaptee's types through the adapter. In a layered design it is
  the implementation of a port owned by the inner side (`clean-architecture`).
- **Often confused with:** Bridge (designed up front for independent variation), Decorator (same
  interface), Proxy (same interface), Facade (new simpler interface for many classes).

### Bridge
- **Does:** decouples an abstraction from its implementation so the two vary independently.
- **Use when:** you want to avoid a permanent binding (choose or switch the implementation at run
  time); both the abstractions and the implementations should be extensible; a hierarchy is
  multiplying along two dimensions.
- **Cost:** up-front design with a clean split; useless if only one dimension actually varies.
- **Light form:** hold a reference to an implementor interface and delegate to it.

### Composite
- **Does:** composes objects into trees representing part-whole hierarchies and lets clients treat
  single objects and groups uniformly.
- **Use when:** the domain is naturally a tree (files and folders, UI containers, menu groups,
  expression trees) and clients should ignore the difference between leaf and group.
- **Cost:** the design becomes overly general: the type system cannot restrict which components a
  composite accepts, so you need run-time checks. Decide where child-management methods live
  (uniform interface vs only on composites).
- **Often combined with:** Iterator, Visitor, Decorator, Flyweight (shared leaves).

### Decorator
- **Does:** attaches extra responsibilities to an object dynamically by wrapping it with an object
  of the same interface.
- **Use when:** extension by subclassing would explode into many classes; behaviors should be
  addable, removable and combinable at run time (logging, caching, retry, compression, encryption,
  rate limiting around a service).
- **Cost:** many small objects; a decorated object is not identical to the original; order can
  matter; keep the component interface small or decorators become heavy.
- **Light form:** function wrappers, language decorators/annotations, middleware lists.
- **Often confused with:** Proxy (controls access, usually one fixed wrapper), Adapter (changes the
  interface), Composite (aggregates; a decorator adds duties).

### Facade
- **Does:** gives a simple, unified interface to a set of interfaces in a subsystem.
- **Use when:** you want an easy default entry to a complex subsystem; you want to decouple clients
  from subsystem internals; you want an entry point per layer.
- **Cost:** it can become a god class; it does not forbid direct subsystem access, which is a
  feature but also a leak.
- **Often confused with:** Adapter (adapts one interface to another; a facade simplifies many),
  Mediator (a facade is one-way; a mediator is two-way coordination).

### Flyweight
- **Does:** shares fine-grained objects to support very large numbers of them efficiently, by
  splitting intrinsic (shared, context-independent) state from extrinsic (passed in) state.
- **Use when all are true:** a very large number of objects; storage cost is high because of
  quantity; most state can be made extrinsic; object identity does not matter.
- **Cost:** run-time cost of passing and computing extrinsic state; complexity. **An optimization:
  apply only after measuring.**

### Proxy
- **Does:** provides a surrogate for another object to control access to it.
- **Variants:** remote proxy (local stand-in for a remote object), virtual proxy (create an
  expensive object on demand), protection proxy (check access rights), smart reference (extra
  bookkeeping on access: counting, locking, caching, copy-on-write).
- **Cost:** another level of indirection, and it can hide latency or failure modes (a remote call
  that looks like a local one).
- **Often confused with:** Decorator (adds behavior to a component for clients to compose), Adapter.

**Structural comparison.** Adapter, Decorator and Proxy all wrap an object; they differ in intent
(change interface / add duties / control access). Composite and Decorator share recursive
structure. Facade simplifies a subsystem; Mediator coordinates peers.

---
## Behavioral

### Chain of Responsibility
- **Does:** passes a request along a chain of handlers until one handles it, decoupling sender from
  receiver.
- **Use when:** more than one object may handle a request and which one is not known in advance;
  the set of handlers should be configurable; request pipelines and middleware, validation chains,
  event bubbling.
- **Cost:** no guarantee a request is handled (it can fall off the end); behavior depends on chain
  order and configuration; harder to debug. Make "unhandled" an explicit outcome.

### Command
- **Does:** encapsulates a request as an object, so you can parameterize, queue, log, schedule and
  undo requests.
- **Use when:** you need undo/redo (the command stores state to reverse itself, kept in a history
  list); queued or deferred execution; an audit or replay log; macros (composite commands);
  decoupling the invoker from the receiver (menus, buttons, key bindings, job runners).
- **Cost:** a class per request; trivial callbacks do not need it. Undo requires careful state
  capture.
- **Light form:** a closure or callable. Use an object when you need undo, serialization, a name,
  or several operations.

### Interpreter
- **Does:** defines a representation of a simple language's grammar plus an interpreter that
  evaluates sentences represented as syntax trees.
- **Use when:** the grammar is simple and efficiency is not critical (rule expressions, filters,
  small query languages).
- **Cost:** one class per grammar rule; large grammars become unmanageable. Use a parser
  generator or an existing library for anything non-trivial, and watch the scope.
- **Often combined with:** Composite (the tree), Visitor (new operations on the tree), Iterator.

### Iterator
- **Does:** provides sequential access to the elements of an aggregate without exposing its
  representation; supports several simultaneous traversals.
- **Use when:** you need uniform traversal of different structures or several independent
  traversals.
- **Light form:** the language's iterator or generator protocol. Rarely hand-written.

### Mediator
- **Does:** defines an object that encapsulates how a set of objects interact, so they do not refer
  to each other explicitly.
- **Use when:** objects communicate in well-defined but complex ways and the interdependencies are
  unstructured; reuse is hard because an object refers to many others (dialog widgets, form
  validation, chat rooms, workflow coordinators).
- **Benefits:** limits subclassing, decouples colleagues, turns many-to-many into one-to-many,
  clarifies the protocol.
- **Cost:** it centralizes control, so the mediator can become a monolith more complex than any
  colleague. Keep it focused.
- **Often confused with:** Observer (broadcast, no central logic), Facade (one-directional).

### Memento
- **Does:** captures and externalizes an object's internal state, without violating its
  encapsulation, so it can be restored later (undo, checkpoints, transactions).
- **Participants:** the originator creates and restores from the memento; the caretaker stores it
  and never looks inside.
- **Cost:** copying large state is expensive, and storing many mementos uses memory (store
  increments or use immutable snapshots). Languages without friend access make an opaque memento
  awkward.
- **Often combined with:** Command (stores a memento to undo).

### Observer
- **Does:** defines a one-to-many dependency so that when a subject changes state, its observers are
  notified and update themselves.
- **Use when:** a change in one object requires changing others and you do not know how many; the
  subject should not be coupled to its observers; subjects and observers should vary independently.
- **Cost / pitfalls:**
  - Unexpected updates: a harmless change can cascade through observers that cannot see each other.
  - The subject must be in a consistent state **before** it notifies.
  - Observers can outlive their subject (or the reverse): unregister on teardown; a subject that
    is destroyed should tell observers.
  - Who triggers notification: the subject on every change (many updates) or the client after a
    batch (easy to forget).
  - Push model (details in the notification) couples subject to observers' needs; pull model makes
    observers work to find what changed. Registering interest in specific events reduces noise.
  - For complex many-to-many dependencies consider a change manager or a Mediator.
- **Light form:** callbacks, events, signals, reactive streams, pub/sub.

### State
- **Does:** lets an object change behavior when its internal state changes; each state is an object
  (or a table row) that handles requests, and transitions are explicit.
- **Use when:** behavior depends on state and the code is full of conditionals on a status/mode; a
  lifecycle (order, connection, document, tool) with defined transitions.
- **Cost:** more classes; transition logic can be spread across states; share stateless state
  objects where possible. A finite state machine is deliberately limited: for shared behavior use
  a hierarchy of states (superstate handles what substates do not), a stack of states for "return
  to previous", or run several machines concurrently. For richer decision making consider
  behavior trees or planning.
- **Light form:** an enum plus a transition table, until per-state behavior outgrows it.
- **Often confused with:** Strategy (same shape: State changes itself over time, Strategy is
  chosen by the client).

### Strategy
- **Does:** defines a family of interchangeable algorithms behind one interface and lets the client
  choose among them.
- **Use when:** many related classes differ only in behavior; you need variants of an algorithm;
  conditionals select among algorithms.
- **Cost:** clients must know the strategies to choose; extra objects; the context and the strategy
  must agree on how data flows (pass the data in, or pass the context in).
- **Light form:** pass a function. Use a class when the strategy has state or several operations.
- **Often confused with:** State, Template Method (Strategy varies the whole algorithm by
  composition, Template Method varies steps by inheritance).

### Template Method
- **Does:** defines the skeleton of an algorithm in a base class and defers some steps to
  subclasses, which redefine steps without changing the structure.
- **Use when:** the invariant part of an algorithm should be written once and the varying steps
  are known extension points; frameworks ("don't call us, we'll call you").
- **Cost:** it relies on inheritance, so it is rigid and subject to the brittle-base-class problem.
  Keep the number of overridable primitive steps small, mark which are required and which are
  optional hooks, and name them consistently. If the variation is chosen per instance or at run
  time, use Strategy.

### Visitor
- **Does:** represents an operation on the elements of an object structure as a separate object, so
  you can add new operations without changing the element classes (double dispatch).
- **Use when:** the element classes are **stable** and you keep adding unrelated operations
  (compilers: type-check, pretty-print, generate code over a syntax tree).
- **Cost:** adding a new element class requires changing every visitor; elements must expose enough
  of their state, which can weaken encapsulation; traversal order must be settled.
- **Light form:** pattern matching over a closed type (sealed classes, enums, unions).

**Behavioral comparison.** Command/Strategy/State/Template Method/Visitor all separate "what is
done" from "who does it" in different ways: a request as an object, an interchangeable algorithm, a
state-dependent behavior, a skeleton with hooks, a new operation over a structure. Observer,
Mediator and Chain of Responsibility are three ways to decouple senders from receivers: broadcast,
central coordination, and sequential offering.

---
## Using the catalog
- **Select:** state what varies (SKILL.md 2a), list candidates, read each card's "Use when" and
  "Cost", pick the one whose cost you can afford. Compare patterns of like purpose.
- **Combine sparingly:** Composite + Iterator + Visitor; Command + Memento; Observer + Mediator;
  Decorator + Strategy. Each combination adds explanatory cost.
- **Avoid the common misapplications:** a pattern with one variant; Singleton as a convenience;
  Visitor over a changing hierarchy; Template Method chains where Strategy fits; Mediator as a
  dumping ground; Decorator stacks whose order nobody documents.
