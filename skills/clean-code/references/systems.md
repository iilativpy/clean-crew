# Systems and emergent design (Clean Code ch. 11–12)

Load this when wiring the application (DI, configuration, startup), adding cross-cutting
concerns (transactions, security, logging, caching, persistence), or choosing frameworks.

## Separate construction from use
- Startup (creating objects and wiring dependencies) is a different concern from runtime
  logic. Don't scatter `if (x == null) x = new ConcreteImpl(...)` lazy-init idioms around —
  each one hard-codes a dependency, complicates tests, and breaks SRP.
- Options, all keeping dependencies pointing **away from main**:
  - **Separation of main**: main (or modules it calls) builds everything; the application
    assumes it receives ready objects.
  - **Abstract Factory**: when the application must decide *when* to create something,
    give it a factory interface; the implementation lives on the main side.
  - **Dependency Injection / IoC**: objects don't resolve their own dependencies; they
    expose constructor (preferred) or setter parameters and a container or main wires them.
    Service-locator lookups are only "partial" DI.
- Lazy initialization is an optimization — possibly premature.

## Scaling up
- You can't get a system "right the first time". Implement today's stories, then refactor
  and expand. Software architecture **can grow incrementally — if concerns are separated**.
- Invasive frameworks (the book's example: EJB2) couple business logic to the container,
  make unit testing hard, block reuse and even inheritance. Keep domain logic in
  **plain objects** (POJOs) that know nothing of the framework.

## Cross-cutting concerns
- Persistence, transactions, security, caching, etc. cut across object boundaries. Handle
  them **non-invasively** with aspect-like mechanisms: proxies/decorators, AOP frameworks,
  framework annotations/configuration at the edge — not by sprinkling the same code
  through domain classes.
- Prefer the lightest mechanism that works (a decorator or a framework proxy before
  heavyweight bytecode weaving).

## Test-drive the system architecture
- With domain logic in plain objects and concerns integrated non-invasively, the
  architecture itself can be test-driven and evolved from "naively simple but well
  decoupled" to sophisticated as needed.
- Avoid **Big Design Up Front** (designing everything before implementing anything) — it
  resists change. Up-front *thinking* about scope, goals and general structure is still fine.

## Optimize decision making
- Give decisions to the most qualified people and **postpone them to the last responsible
  moment**, when you have the most information. Modular systems make late decisions cheap.

## Use standards wisely
- Adopt a standard/framework when it adds **demonstrable value** (reuse, hiring, proven ideas),
  not because it's hyped. A good API mostly disappears from view.

## Domain-specific languages
- Well-designed DSLs or fluent internal APIs let code read like the domain expert's language,
  narrowing the gap between concept and implementation.

## Emergence — Kent Beck's four rules of simple design (in priority order)
1. **Runs all the tests** — a testable system pushes you toward small, single-purpose,
   decoupled classes (DIP, DI, interfaces).
2. **Contains no duplication** — including duplicated implementation and similar
   algorithms (Template Method, Strategy).
3. **Expresses the intent of the programmer** — good names, small functions/classes,
   standard pattern names, well-written tests as documentation. Care enough to make it readable.
4. **Minimizes the number of classes and methods** — don't create classes/interfaces
   dogmatically (e.g. an interface for every class, data/behavior split everywhere).
   Lowest priority of the four.

Rules 2–4 are achieved by **refactoring** with tests green.

## Rule of thumb
At every level — module or system — use the simplest thing that can possibly work.
