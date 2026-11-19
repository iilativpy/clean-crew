# Game, real-time and performance patterns

Open when building or reviewing an interactive or real-time system (games, simulations, UIs with
animation, servers with many live entities), when a performance problem is **measured**, or when a
Singleton, global "manager" or hand-rolled state machine is tempting. The patterns were collected
from game code, but each applies to any program with the same forces.

Two rules from that world apply everywhere:
- **Complexity is a cost; simplicity is a resource.** Each pattern here pays for something
  (decoupling, speed, memory) with something else (indirection, flexibility, clarity). Do not buy
  until you need it.
- **Optimization patterns are for measured problems only.** Profile first. Applying Data Locality,
  Object Pool, Dirty Flag or Spatial Partition to code that is not the bottleneck only makes it
  harder to change.

---
## Lessons about classic patterns in real-time code

### Singleton (and what to do instead)
It looks harmless and costs later: a global variable (hard to reason about; you must search the
whole code base for who touches it); it **encourages coupling** (anyone can reach it, so a physics
class ends up calling the audio player); it is not concurrency-friendly (shared mutable state).
It also solves two problems at once (one instance, global access) and you usually need only the
first. Alternatives, in order:
1. **See if you need the class at all.** Many singletons are "managers" that babysit other
   objects (`Monster` + `MonsterManager`). Move the behavior into the class it serves.
2. **Limit to a single instance** without a global: assert it at construction, or create it once
   at startup and hold it where needed.
3. **Pass the instance in** to the code that needs it (constructor or parameter). It makes the
   coupling visible, which is the point.
4. **Get it from something already available** (a base class provides the operation; an object you
   already have exposes it).
5. **Service locator**, sparingly (below), for genuinely ambient services.

### Observer
Use it to notify without coupling; know its limits. Pitfalls: observers outliving subjects or
subjects destroyed while observed (unregister, or let the subject announce its destruction),
unexpected cascades, notifying a subject that is mid-update, and code that is hard to follow because
the flow is implicit. When you only need decoupling in *who*, Observer or Command is enough; add a
queue only when you also need to decouple *when* (see Event Queue).

### Command
Beyond menus: a command per input lets the same logic drive the player, an AI or a replay. **Undo
and redo** fall out naturally: each command stores what it needs to reverse itself and a history
list moves back and forth. A level editor without undo is the quickest way to annoy its users.
Commands are also how you queue, log and replay actions.

### State (finite state machines)
State objects beat a pile of `if (state == ...)` flags and booleans. Useful refinements:
- **Entry and exit actions** run when entering or leaving a state, so transition code is written
  once even when many transitions lead to the same state.
- **Hierarchical state machines:** a state with a superstate; unhandled input rolls up to the
  superstate (shared "on ground" behavior). Inheritance is a strong coupling, so use it
  deliberately.
- **Pushdown automata:** a stack of states so you can "return to what I was doing" (a firing state
  that pops back to the previous one).
- **Concurrent state machines:** independent machines for independent concerns (what the hands do
  vs what the legs do) instead of one machine with a combinatorial state space.
- **Limits:** a finite state machine has a fixed set of states and hard-coded transitions. For
  richer decision making consider behavior trees, planning or scripting.
A state-per-class design is heavier than an enum and transition table; start with the table.

### Flyweight and Prototype
Flyweight shares the intrinsic state of many similar objects (a tree model drawn thousands of
times with per-instance position): only when counts are large and measured. Prototype lets data
define new kinds by cloning configured instances (a spawner that clones a monster template).

---
## Sequencing patterns

### Game Loop
- **Does:** processes input without blocking, updates state, renders, and tracks elapsed time to
  control the rate of play, decoupling progress of game time from user input and processor speed.
- **Use:** you will use it. An engine owns the loop and calls your code; a library is called by your
  code. Even turn-based games keep animation and audio running while waiting for input.
- **Keep in mind:** this is some of the hottest code in the program. Coordinate with the platform
  event loop (in a browser, `requestAnimationFrame` is your loop). Use a fixed time step for
  deterministic updates and render as fast as you can; handle slow frames deliberately.

### Update Method
- **Does:** each object in a collection implements `update()` that advances one frame of its
  behavior; the game loop updates every object.
- **Use when:** many objects run simultaneously, mostly independent, simulated over time. Poor fit
  for abstract games where pieces do not act on their own.
- **Keep in mind:** splitting behavior into per-frame slices is more complex than straight-line
  code (each object must keep its own state between calls). Coroutines, generators or a bytecode
  interpreter can restore straight-line logic. Be careful when objects are added or removed during
  an update pass, and decide whether to pass elapsed time.

### Double Buffer
- **Does:** keeps a *current* and a *next* buffer; writes go to next, reads come from current, then
  swap, so readers never see partial work (no tearing; simultaneous updates see a consistent past).
- **Use when:** state is modified incrementally, may be read mid-modification, readers must not see
  work in progress, and you do not want readers to wait.
- **Keep in mind:** the swap must be cheap (swap pointers, not copy); there is extra memory; the
  reader sees one step behind.

---
## Behavioral patterns

### Subclass Sandbox
- **Does:** a base class defines an abstract "sandbox" method and a set of protected operations;
  each subclass implements its behavior using only those operations.
- **Use when:** many derived classes overlap in behavior and you want to share code and keep them
  decoupled from the rest of the program.
- **Keep in mind:** the base class accretes code and becomes a brittle, over-coupled "giant bowl";
  move provided operations into separate classes it delegates to (Component) when it grows.

### Type Object
- **Does:** moves the definition of a "kind" from subclasses into objects: a *typed object* holds a
  reference to a *type object* that carries shared data and behavior.
- **Use when:** you cannot know all the kinds up front, or you want to add and change kinds without
  recompiling (data-driven monsters, products, item types).
- **Keep in mind:** you must manage type objects yourself (create, keep alive, validate the
  reference) and you lose compiler help; behavior beyond data needs extra care (a type holding
  functions or a script).

### Bytecode
- **Does:** an instruction set encoded as bytes, executed by a small virtual machine with a stack,
  so behavior can be defined as data.
- **Use when:** you have lots of behavior to define and the host language is too low-level, too slow
  to iterate on, or too trusting (you need sandboxing).
- **Keep in mind:** the most complex pattern; it is slower than native code and it needs an
  authoring front end (nobody writes bytecode by hand). Small languages grow without bound:
  deliberately limit the scope or choose an existing scripting language.

---
## Decoupling patterns

### Component
- **Does:** one entity spans several domains (input, physics, graphics, AI); each domain's code
  lives in its own component class; the entity is a container of components.
- **Use when:** a class touches multiple domains you want decoupled, it is getting huge, or
  inheritance cannot pick exactly the parts to reuse.
- **Keep in mind:** more complexity than one class: more objects to create and wire, harder
  communication between components (through the container, shared state, or messages), indirection
  in hot paths. It often improves cache behavior (Data Locality). Do not over-engineer a problem you
  do not have.

### Event Queue
- **Does:** a first-in, first-out queue of notifications or requests; the sender enqueues and
  returns, a processor handles items later, decoupling sender and receiver in **time**.
- **Use when:** you need to decouple *when*, not only *who* (Observer and Command are simpler for
  who). A push producer and a pull consumer need a buffer between them.
- **Keep in mind:** the receiver gains control and the sender loses it (a poor fit when the sender
  needs a reply); a central queue is effectively a global; the world can change between enqueue and
  processing, so events must carry enough state; feedback loops (handling an event enqueues another)
  must be bounded. Decide on a ring buffer or growth policy and what happens when it is full.

### Service Locator
- **Does:** a service defines an abstract interface; a provider implements it; a locator finds the
  provider and hides its concrete type and how it was found.
- **Use sparingly.** First pass the dependency in. Use it for ambient services nobody should have to
  plumb through ten layers (logging, audio output, clock). Anything global invites trouble.
- **Keep in mind:** it hides what a class depends on, so reading code no longer shows the coupling;
  the service must be locatable (provide a null/default service so lookups never fail); a service
  must work in any context because anyone can call it.

---
## Optimization patterns (measure first)

### Data Locality
- **Does:** keeps data in contiguous memory in the order it is processed, so CPU caches hit.
- **Use when:** profiling shows **cache misses** are the problem (a profiler that reports them; not
  just a stopwatch). Hot loops over many objects (update every entity, every particle).
- **Keep in mind:** it works against abstraction: pointers, virtual calls and interfaces scatter
  data and cost cache misses, so you give up some of inheritance and indirection. Contiguous arrays
  of structs or components, hot/cold splitting and packing are the tools. There is no silver bullet,
  only trade-offs.

### Dirty Flag
- **Does:** primary data changes; derived data is expensive to compute; a *dirty* flag marks the
  derived data stale when the primary data changes, and the work is redone only when the derived
  data is needed.
- **Use when:** the primary data changes more often than the derived value is used, and it is hard to
  update incrementally. Two uses: expensive calculation (a transform in a scene graph) and
  synchronization (saving to disk or sending over a network).
- **Keep in mind:** you must set the flag on every path that changes the primary data (a missed path
  is a stale-cache bug); deferring can cause a visible stall when the result is finally needed; if
  the process dies before synchronization, work is lost (auto-save compensates).

### Object Pool
- **Does:** a fixed collection of reusable objects allocated up front; "in use" tracking; creating
  and destroying objects means flipping state, not allocating.
- **Use when:** frequent creation and destruction of similar-size objects, slow or fragmenting
  allocation, or objects wrapping expensive resources (connections, threads).
- **Keep in mind:** you now own memory management: the pool may be too big (waste) or too small (what
  happens when it is empty: fail, grow, drop the oldest, or tune to never overflow); reused objects
  must be fully reset (stale state is the classic bug); a pooled object may be referenced after it
  was released.

### Spatial Partition
- **Does:** organizes objects by position in a spatial structure (grid, quadtree, k-d tree, BSP) so
  queries by location are fast; objects are re-filed when they move.
- **Use when:** many objects each with a position, and location queries (collision, visibility,
  nearest neighbor) are a measured bottleneck; typically turns O(n) or O(n^2) into something
  manageable.
- **Keep in mind:** small n does not justify it; moving objects must be updated in the structure
  (like a hash table whose keys change); extra memory for bookkeeping.

---
## How to measure before applying an optimization pattern
| Suspected problem | What to measure | Applies if |
|---|---|---|
| Allocation / garbage-collection pauses (Object Pool) | Record a performance trace of a worst-case scenario for 20-30 s; look at frame times and GC events (the browser profiler's memory track, or the runtime's GC log); count allocations per frame in the hot path | GC events line up with frames over budget, or frame time worsens as object counts grow on the target hardware |
| Cache misses (Data Locality) | A profiler that reports cache-miss counts (hardware counters, or a cache simulator); compare a hot loop over contiguous vs scattered data | Misses dominate the hot loop; a stopwatch alone cannot tell you this |
| Recomputed derived values (Dirty Flag) | Count how often the derived value is computed vs how often it is used after a change | It is recomputed much more often than it is needed, and updating incrementally is hard |
| Slow spatial queries (Spatial Partition) | Time the nearest/collision query as the object count n grows | Time grows like n or n squared and is a measurable share of the frame |
| Too many similar objects (Flyweight) | Memory per object times count; share of memory that is identical state | Memory is a measured limit and most state can be shared |
Record the baseline and the target (frame time, memory) before changing code, and re-measure after.
If there is no problem on the target hardware, do not apply the pattern.

## Choosing among them (short)
| Problem | Reach for |
|---|---|
| Keep the program running in real time and responsive | Game Loop; Update Method for many actors |
| Readers see half-finished work | Double Buffer |
| Behavior depends on a mode; flag soup | State (enum + table first) |
| Same logic driven by player, AI, replay; undo | Command |
| Tell unknown listeners about a change | Observer; add an Event Queue only to decouple in time |
| One entity, many concerns; giant class | Component |
| Kinds of things defined by data | Type Object (bytecode/scripting only if truly needed) |
| Shared base behavior, decoupled subclasses | Subclass Sandbox (watch the base class) |
| Many allocations of similar objects | Object Pool (measure first) |
| Slow due to cache misses | Data Locality (measure first) |
| Expensive derived values recomputed too often | Dirty Flag (measure first) |
| Slow spatial queries | Spatial Partition (measure first) |
| Need a global-ish service | pass it in; Service Locator only as a last resort |
