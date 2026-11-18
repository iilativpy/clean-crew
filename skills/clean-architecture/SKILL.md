---
name: clean-architecture
description: Design-level rules - SOLID, component cohesion/coupling principles, the Dependency Rule, boundaries, use cases vs details. Use whenever deciding where new code belongs, adding a module/class/endpoint/integration, changing dependencies between packages or layers, or reviewing a design.
---

# Clean Architecture - working rules

Use this before writing code that adds a new type, module, endpoint, integration or
dependency. Answer "**where does this belong and which way do dependencies point?**"
first, then write the code (with the `clean-code` skill).

## 0. Goal of architecture
- Minimize the human effort to build **and maintain** the system.
- Behavior matters, but **structure (changeability) matters more** - a working system that
  can't change becomes worthless; a changeable one can be made to work.
- Keep options open: defer decisions about details (DB, framework, web, UI, vendors).
- Architecture is *important but rarely urgent* (Eisenhower matrix); features are urgent but
  often less important. Developers are responsible for defending structure against the
  pressure to just ship features - a "we'll clean it later" mess makes every release slower.
- There's no real line between design and architecture: low-level decisions and high-level
  structure are one continuum.

## 1. The Dependency Rule (non-negotiable)
Source-code dependencies point **only inward**, toward higher-level policy.

```
   Frameworks & Drivers (web, DB, UI, devices, external APIs)
      Interface Adapters (controllers, presenters, gateways, repositories impl)
         Application Business Rules (use cases / interactors)
            Enterprise Business Rules (entities)
```
- Inner circles know **nothing** of outer ones: no framework annotations, ORM entities,
  HTTP types, SQL, or vendor SDK types in entities or use cases.
- Data crossing a boundary is plain, simple structures shaped for the inner circle's
  convenience - not DB rows or framework request objects.
- When control flow must go outward (use case -> DB), **invert** it: the use case owns an
  output port interface; the adapter implements it (DIP).
- **Level = distance from inputs/outputs**: the farther from I/O, the higher the level.
  Dependencies follow level, not data flow.
- Polymorphism (OO) is the mechanism that makes any dependency invertible; immutability
  (FP) limits where state can change -> `references/paradigms.md`.

## 2. SOLID (module/class level)
- **SRP** - a module is responsible to one actor. Code that changes for different actors
  (e.g. accounting vs ops) must be separated, even if it looks similar. Symptoms of
  violation: accidental duplication (shared helper changed for one actor breaks another)
  and merge conflicts between teams. Fixes: separate data from functions, or a Facade.
- **True vs. accidental duplication**: unify only code that must always change together.
  Similar-looking code serving different actors/use cases (or a DB row vs. a view model)
  is accidental - keep it separate.
- **OCP** - extend behavior by adding code, not modifying it. Arrange components so the
  higher-level ones are protected from changes in lower-level ones.
- **LSP** - subtypes must be substitutable; no `if (x instanceof Special)` patches. Applies
  to interfaces and REST contracts too.
- **ISP** - don't depend on things you don't use; split fat interfaces per client.
- **DIP** - depend on abstractions. Volatile concretions (your own frequently-changing
  classes) are referenced through interfaces; stable ones (String, JDK) are fine directly.
  Create volatile objects via factories/DI so callers don't depend on concrete classes.

## 3. Component principles (package / module / jar / service level)
**Cohesion - what goes together**
- **REP** (Reuse/Release Equivalence): the unit of reuse is the unit of release - components
  are cohesive and versioned together.
- **CCP** (Common Closure): group classes that change for the same reason at the same time
  (SRP for components).
- **CRP** (Common Reuse): don't force users to depend on things they don't use (ISP for
  components).
- Balance these; early in a project favor CCP (develop-ability), later more REP.
- Component structure **evolves**; it can't be designed top-down before the code exists.

**Coupling - how they relate**
- **ADP** (Acyclic Dependencies): no cycles in the component graph. Break cycles with DIP
  or by extracting a new component.
- **SDP** (Stable Dependencies): depend in the direction of stability.
  Instability `I = Fan-out / (Fan-in + Fan-out)`; dependencies should point toward lower I.
- **SAP** (Stable Abstractions): stable components should be abstract; unstable ones concrete.
  Avoid the *Zone of Pain* (stable + concrete, e.g. a rigid shared util everyone uses) and the
  *Zone of Uselessness* (abstract + nobody depends on it).
- Metrics `A = Na/Nc`, distance `D = |A + I − 1|`, cycle-breaking techniques ->
  `references/component-principles.md`.

## 4. Boundaries and structure
- Draw boundaries between things that change at different rates/for different reasons:
  business rules vs. GUI, DB, frameworks, external services.
- **The database is a detail. The web is a detail. Frameworks are details.** Use them;
  don't marry them. Keep framework code at the edge (main, config, adapters).
- **Screaming architecture**: top-level structure should reveal the domain/use cases
  (`ordering/`, `billing/`), not the framework (`controllers/`, `services/`, `models/`).
  Prefer package-by-feature/component over package-by-layer when practical.
- **Humble Object**: split hard-to-test behavior at boundaries into a humble part (thin,
  untested glue: views, DB driver calls) and a testable part (presenter, interactor, gateway logic).
- **Main component** is the dirtiest low-level detail: it creates factories, wires
  dependencies, reads config, then hands control to the policy. Everything framework-y lives here.
- Partial boundaries (Strategy, Facade, one-sided interfaces) are acceptable when a full
  boundary is too costly - but make the choice consciously.
- **Services are not an architecture by themselves**: microservices can still be coupled
  by shared data/contracts. Apply the same rules inside each service.
- **Decoupling modes**: source-level (monolith) -> deployment-level (jars/DLLs) ->
  service-level. Decouple so a service *could* be extracted, but stay in one process as long
  as possible. Threads are not boundaries. -> `references/boundaries-and-services.md`.
- Boundaries are everywhere; full ones are costly, missing ones are costly to add later.
  Watch for friction and add a boundary when implementing it becomes cheaper than ignoring it.
- **Package structure** (by layer / by feature / ports & adapters / by component) and
  **enforcing it with the compiler** (minimal `public`, modules, separate build modules) ->
  `references/packaging-strategies.md`.

## 5. Use cases
- Each use case = one application-specific business operation: input data -> interactor ->
  output data. Name it by intent (`PlaceOrder`, `ApproveInvoice`).
- Use cases orchestrate entities; they don't contain UI, transport or persistence logic.
- Request/response models are simple, independent of frameworks and of entities' internals.

## 6. Tests as part of the architecture
- Tests are the outermost circle; they depend inward, nothing depends on them.
- Avoid fragile tests coupled to GUI/structure - test through a **testing API** at the use case
  boundary. The testing API can bypass security, skip expensive resources and force states.
- Avoid **structural coupling** (one test class per production class, one test per method):
  it freezes the production structure. Tests should depend on behavior, so production code can
  be refactored freely.
- Keep the testing API's "superpowers" out of production deployments (separate component).
- Enforce the dependency rule automatically where possible (e.g. ArchUnit for Java/Kotlin,
  dependency-cruiser for JS/TS, import-linter for Python) instead of trusting review alone.

## 7. Worked approach (example)
1. Identify the **actors** (SRP: each actor is a source of change).
2. List the **use cases** per actor; factor out abstract/shared use cases carefully.
3. Draw the component architecture: views -> presenters -> interactors (use cases) ->
   controllers/gateways/DB, separated by the actors and by the Dependency Rule.
4. Check every dependency crossing a boundary points toward the higher-level policy.
5. Choose the deployment granularity (decoupling mode) separately - it can change later.

## 8. Design checklist (run before writing new code)
1. Which **actor/use case** drives this change? Which component owns it?
2. Is this **policy** (entity/use case) or **detail** (adapter/framework/IO)? Put it in the
   matching layer.
3. Do all new imports point **inward**? Any framework/vendor type leaking into the core?
4. If inner code needs outer behavior, did I add a **port interface** owned by the inner side?
5. Does the change create a **dependency cycle** between packages/modules?
6. Do classes in the touched component **change together** (CCP)? Is anything unrelated
   being dragged in (CRP)?
7. Is the new thing testable without DB/web/framework (Humble Object)?
8. Does the folder/package name **scream the domain**?
9. Is the boundary I'm adding proportional to the risk (full vs. partial boundary)?

## Reference files (load on demand)
| File | Open when |
|---|---|
| `references/component-principles.md` | Creating/splitting modules, packages, libraries; dependency cycles; stability metrics |
| `references/boundaries-and-services.md` | Adding a boundary, extracting a service, choosing monolith vs. modules vs. microservices, sync vs. async |
| `references/packaging-strategies.md` | New packages/modules, reorganizing a code base, enforcing architecture rules |
| `references/paradigms.md` | Polymorphism/plugin design, immutability, event sourcing, mutable shared state |
| `references/embedded.md` | Firmware, drivers, hardware/OS-near code only |

## Pragmatism
- Respect the existing architecture of the project. If it violates these rules, don't
  re-architect inside a feature change: implement cleanly within the current structure,
  avoid making it worse, and report the violation with a suggested incremental fix.
- Over-engineering is also a failure: don't add layers/ports for code that has one
  implementation and no foreseeable variation unless it crosses a real boundary (IO, framework,
  external system).
