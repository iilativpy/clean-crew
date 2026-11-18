# Boundaries, decoupling modes and services (Clean Architecture ch. 15–19, 24–27)

Load this when deciding whether to add a boundary, split something into a module/service,
choose sync vs. async communication, or design a microservice.

## What architecture is for (ch. 15)
- Support the system's lifecycle: **development** (team structure), **deployment** (ideally a
  single action), **operation** (usually solvable with hardware, so cheapest concern),
  **maintenance** (most costly: "spelunking" through code and risk of breakage).
- **Keep options open**: separate policy from details, and defer decisions about details
  (DB, web server, frameworks, REST vs. messaging, DI framework) as long as possible.
  Device independence is the classic example: code that didn't know about the specific
  device survived device changes.

## Independence (ch. 16)
- The architecture must make the **use cases** visible and support operation, development
  (Conway's law: system structure mirrors team communication) and deployment ("immediate
  deployment" after build, no manual fiddling).
- **Decouple layers** (UI, application-specific rules, business rules, DB) - they change for
  different reasons.
- **Decouple use cases** - vertical slices through the layers (add-order vs. delete-order
  change at different rates).
- **True vs. accidental duplication**: if two pieces change for different reasons at different
  rates, they're not real duplicates - don't unify them (e.g. similar screens of different
  use cases; a DB row and a view model that look alike -> keep a separate view model).
- **Decoupling modes**:
  - *Source level* - modules in one executable; changes don't force recompilation of others (monolith).
  - *Deployment level* - separate deployable units (jars, DLLs) that don't force rebuild/redeploy of others.
  - *Service level* - only data structures cross the boundary over the network; fully independent.
- The best mode changes over time. Default: **decouple to the point where a service *could* be
  formed, but stay in one address space as long as possible**. Service-by-default is expensive
  and tends to be coarse-grained. Be ready to move up (or back down) as needs change.

## Drawing lines (ch. 17)
- Premature coupling to decisions that don't matter for business rules (frameworks, DB,
  web server, DI, distribution) is what kills productivity.
- Draw lines between **things that matter and things that don't**: GUI <-> business rules,
  DB <-> business rules, etc. The arrow always points to the business rules.
- I/O is irrelevant to the business rules - the system's behavior can be driven by a console,
  a GUI or tests alike.
- **Plugin architecture**: core business rules; everything else (UI, DB, external services)
  is a plugin. Plugins depend on the core, never the reverse.

## Boundary anatomy (ch. 18)
A boundary crossing is a call from one side to the other plus data; what matters is managing
**source-code dependencies** across it.
| Boundary | Communication | Notes |
|---|---|---|
| Monolith (source-level) | function calls, very fast | most common; boundaries enforced by polymorphism/DIP; can be chatty |
| Deployment components | function calls | jars/DLLs; same rules as monolith |
| Threads | - | **not** an architectural boundary - a way to organize scheduling |
| Local processes | IPC, sockets, shared memory; moderately expensive | low-level process must depend on high-level one (via plugins/abstractions) |
| Services | network; very slow | assume network; avoid chattiness; same dependency rule applies |
Most systems combine several kinds.

## Policy and level (ch. 19)
- Programs are statements of **policy**. Group policies that change for the same reasons/at the
  same times; separate those that don't.
- **Level = distance from inputs and outputs.** The farther from I/O, the higher the level.
  Source-code dependencies must point toward **higher** levels - decoupled from data flow,
  coupled to level (e.g. an `Encrypt` policy depends on `CharReader`/`CharWriter` interfaces,
  console implementations depend on them).
- Higher-level policies change less often and for more important reasons.

## Partial boundaries (ch. 24)
Full boundaries are expensive. When a boundary might be needed later, consider:
- **Skip the last step**: build separate components but deploy them together.
- **One-dimensional boundary**: a Strategy-style interface on one side only (cheap, but
  easy to bypass).
- **Facade**: a single class routes calls; even the dependency inversion is dropped.

## Layers and boundaries (ch. 25)
- Boundaries hide everywhere - even a tiny text game has several axes of change (UI language,
  delivery mechanism, data storage, network) and the data streams can split and merge.
- Full boundaries are expensive; missing ones are expensive to add later. YAGNI vs. foresight:
  **watch** the system, note where boundaries may be needed, and implement them at the point
  where the cost of implementing becomes lower than the cost of ignoring. Revisit often.

## Services (ch. 27)
- Services are not an architecture by themselves. The architecture is defined by boundaries
  separating policy from details following the Dependency Rule; services that merely split
  behavior are expensive function calls.
- **Decoupling fallacy**: services are decoupled at the variable level but still coupled by
  shared data schemas/contracts - a change to a shared record ripples through many services.
- **Independent development/deployment fallacy**: shared data and contracts force coordinated
  changes; large systems can be built as monoliths or component systems too.
- **Kitty problem**: a new cross-cutting feature (e.g. "deliver kittens" in a taxi system)
  touches every service when services are split by function.
- Remedy: design services **internally** with component architecture (SOLID, polymorphic
  extension points, Dependency Rule) so new features are added as new components (e.g. new
  jars extending abstract classes) without modifying existing ones. Architectural boundaries
  run **through** services, not only between them.
