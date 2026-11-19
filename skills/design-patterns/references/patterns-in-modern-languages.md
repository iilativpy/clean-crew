# The lightest form of each pattern

Most classic patterns were written for languages without first-class functions, closures,
generics, iterators, modules, records or pattern matching. Use the language before building
structure. Rule: if the variants differ only by **data**, use data; by **one operation**, use a
function; by **several operations or state**, use a class.

## Cheat sheet by language feature
| Pattern | Python | JavaScript / TypeScript | Java / C# | Go |
|---|---|---|---|---|
| Strategy | pass a function / callable; `functools.partial` | pass a function | functional interface / `Func<>` / lambda; class if it has state | function type or small interface |
| Command | callable; `functools.partial`; class with `execute`/`undo` when you need undo or serialization | closure; object with `execute`/`undo` | `Runnable`/`Action`/lambda; class for undo | func value or small interface |
| Observer | callback list, `signals`, event-emitter library | `EventEmitter`, `addEventListener`, RxJS observables | listeners, `PropertyChangeSupport`, events/`IObservable` | channels or callbacks |
| Iterator | `__iter__`, generators (`yield`) | `Symbol.iterator`, generators | `Iterable`/`IEnumerable`, `yield return` | `range`, channels, iterator funcs |
| Template Method | pass hook functions; or a base class with hooks as a last resort | callbacks / options object | abstract class if inheritance is already there | embed + func fields |
| Factory / Abstract Factory | function or dict of constructors; `type`/`classmethod` | factory function; object of constructors | static factory methods; DI container | constructor function `NewX` |
| Builder | keyword args, `dataclass(frozen=True)`, `replace` | options object, spread | records, fluent builder when staged | functional options |
| Prototype | `copy.copy/deepcopy`, `dataclasses.replace` | spread, `structuredClone` | copy constructors, `record with` | struct copy |
| Singleton | module-level object created once and **passed in**; avoid hidden globals | module singleton passed in / DI | DI container with singleton scope | construct in `main`, pass in |
| Decorator | `@decorator`, wrapper functions, `functools.wraps` | higher-order functions, middleware arrays | annotations/AOP, wrapper classes | middleware funcs `func(next) func` |
| Chain of Responsibility | list of handler callables, loop with early return | middleware stack (`next()`) | filter chain | middleware chain |
| Visitor | `match`/`singledispatch` on a closed set of types | discriminated unions + `switch` | sealed types + pattern `switch` | type switch |
| State | `Enum` + dict of transitions; state objects when behavior is large | union type + reducer | enum with methods; sealed states | `type State func(*M) State` |
| Adapter | thin wrapper class or function; `Protocol` as the port | wrapper object | wrapper implementing your interface | small adapter type |
| Facade | a module exposing a few functions | a module/index file | a service class | a package with a small API |
| Proxy | wrapper, `functools.cache`, lazy properties | `Proxy` objects, lazy getters | dynamic proxies | wrapper type |
| Memento | immutable snapshot (frozen dataclass, `copy`) | immutable object / serialized state | immutable record / serialization | struct copy |
| Composite | recursive dataclass / class with `children` | recursive object | interface with leaf and composite | interface + slice of children |
| Mediator | an object handling events between components, or an event bus | event bus, state store | mediator object / message bus | central coordinator goroutine |
| Flyweight | interning (`sys.intern`), caches, `__slots__`, numpy arrays | shared objects, typed arrays | `Integer` cache, `String` pool | shared immutable values |
| Bridge | constructor takes the implementor | constructor takes the implementor | constructor takes the interface | field of interface type |
| Interpreter | small AST + `match`, or a parser library | AST + `switch`, or a library | AST + visitor; ANTLR | AST + type switch |

## Game-style patterns in a typical language
| Pattern | Light form |
|---|---|
| Component | an entity holding a few small component objects (or plain functions) it delegates to; a data-oriented design with arrays per component type when you have thousands |
| Object Pool | a preallocated array plus a free list or an `active` flag; `acquire`/`release`/`reset`; decide the overflow policy; add only after profiling shows allocation or GC trouble |
| Type Object | a record/dataclass per kind loaded from data (`EnemyType`), and instances that hold a reference to it |
| Event Queue | an array or ring buffer processed once per frame; events carry the data they need |
| Game Loop | `requestAnimationFrame` in a browser, a fixed-step accumulator elsewhere |
| Dirty Flag | a boolean on the owner, set in every setter of the primary data, cleared on recompute |

## When a variant is data plus one behavior
A row that is mostly data but has one small behavior (a carrier with prices and an ETA rule) is a
frozen dataclass/record with a callable field, kept in a dict keyed by name. Move to a class
hierarchy only when variants need several operations or their own state.

## Principles when choosing the form
- **Dynamic languages and ports:** a port can be a function or a duck-typed object (or a `Protocol`);
  you do not need an abstract base class unless you want to enforce the contract.
- **Immutability makes several patterns cheaper:** value objects, copy-on-change (Prototype,
  Memento), safe sharing (Flyweight), no defensive copying.
- **Dependency injection frameworks** replace hand-written Factory and Singleton in many stacks;
  constructor injection plus a composition root is the same idea with no framework.
- **Keep the pattern name in the code only if it helps readers:** `RetryingClient` (a Decorator)
  reads better than `ClientDecorator`.
- **Check the project's existing idiom** before choosing: if it uses middleware, add middleware; do
  not introduce a Decorator hierarchy beside it.
- **Test the seam:** whatever the form, a test passes in a fake variant and checks the client
  behaves; that test is what proves the abstraction is real.

## Examples of "enough" versus "too much"
- Sorting with different comparison rules: pass a key function (not a Strategy class hierarchy).
- A report exported as CSV or JSON: a dict `{"csv": write_csv, "json": write_json}` (not an abstract
  `ExporterFactory`), until the exporters need configuration and lifecycle.
- A retry/cache/log around an API call: a wrapper function (not three classes), until you need to
  swap and reorder them at run time.
- Parsing a handful of commands: a dict of command name to function (not a Command class per
  command), until commands need undo, history or queuing.
- An order with `status` in {new, paid, shipped, cancelled} and three rules: enum plus a transitions
  table, until each state carries real behavior and data.
- A small, closed set of node types: `match` on the type (not Visitor).
- Variants that need **several operations, a few parameters and persistence** (discount kinds:
  price, explain, validate, stored as `kind` + params): one small frozen dataclass (or record) per
  variant implementing a narrow interface (`Protocol`/interface), the parameters as fields,
  validation in the constructor so an invalid variant cannot exist, and a dict registry from the
  stored `kind` to the class. It is Strategy by composition without factories or managers; a new
  variant is one class, one registry line and its tests. Add a contract test that loops over the
  registry so a forgotten operation fails for every future variant. Keep stored kind names and
  parameter names stable: they are now a data contract.
- When the interface does not fit every variant (a discount that spans several lines, or is not a
  line discount at all), do not stretch one interface into a god-context. Shape the interface from
  the caller's need, and split it in two small interfaces if the variants fall into two groups.
