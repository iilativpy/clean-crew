# Programming paradigms as architectural tools (Clean Architecture Part II, ch. 3–6)

Each paradigm **takes something away** from the programmer — a discipline, not a feature.
Together they map to the three concerns of architecture:

| Paradigm | Discipline imposed on | Architectural use |
|---|---|---|
| Structured | direct transfer of control (no `goto`) | algorithmic foundation of modules; functional decomposition into testable units |
| Object-oriented | indirect transfer of control (no raw function pointers) | **polymorphism to cross boundaries** → plugin architecture |
| Functional | assignment (immutability) | discipline on where and how data is mutated |

## Structured programming (ch. 4)
- All programs can be built from sequence, selection and iteration; using only these makes
  modules recursively decomposable into small provable units.
- Formal proofs never caught on; **software is like science**: tests can show the presence of
  bugs, never their absence. We deem code correct when we fail to prove it incorrect.
- Therefore design for **falsifiability**: decompose functions, modules, components and services
  so each is small and testable. Functional decomposition remains a best practice at every level.

## Object-oriented programming (ch. 5)
- Encapsulation and inheritance existed before OO (and some OO languages weaken encapsulation).
  What OO really made **safe and convenient is polymorphism**.
- Polymorphism lets you **invert any source-code dependency** regardless of the flow of
  control (dependency inversion). The architect gains full control over dependency direction.
- Use it so that UI and database depend on the business rules, not the reverse → they become
  **plugins**, compiled and deployed as separate components → **independent deployability and
  independent developability**.
- Practical rule: whenever a high-level module would import a low-level one, put an interface
  owned by the high-level side in between.

## Functional programming (ch. 6)
- In functional code, variables don't vary. **All race conditions, deadlocks and
  concurrent-update problems require mutable state** — no mutation, no such problems.
- Full immutability needs infinite storage/CPU, so compromise:
  - **Segregate mutability**: push as much processing as possible into immutable components;
    keep mutable state in a few components protected by transactional/atomic mechanisms
    (compare-and-swap, transactional memory, DB transactions).
  - **Event sourcing**: store the transactions (events), not the current state; compute state
    by replaying (with snapshots as an optimization). Storage becomes create/read only — no
    update/delete — so there are no concurrent-update problems.
- Practical rules: default to immutable value objects/records; make mutable state explicit,
  small and guarded; consider append-only event logs where history and concurrency matter.
