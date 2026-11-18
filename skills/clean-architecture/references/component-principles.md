# Components, cohesion, coupling and metrics (Clean Architecture Part IV, ch. 12–14)

Load this when creating/splitting/merging modules, packages, jars, Maven/Gradle modules,
libraries or services, or when analyzing dependency structure.

## Components (ch. 12)
- A component is the **unit of deployment** (jar, DLL, gem, shared library) - the smallest
  thing that can be deployed. Dynamically linked components that plug together at runtime
  make a plugin architecture the normal default.

## Cohesion - which classes belong together (ch. 13)
- **REP – Reuse/Release Equivalence**: the granule of reuse is the granule of release.
  Classes in a component should form a cohesive group that makes sense to release and
  version together (release numbers, release notes).
- **CCP – Common Closure**: gather classes that change for the same reasons at the same times;
  separate those that change for different reasons. (SRP for components.) Minimizes the
  number of components touched/redeployed per change.
- **CRP – Common Reuse**: don't force users to depend on things they don't need; classes that
  aren't used together shouldn't share a component. (ISP for components.)
- **Tension triangle**: REP and CCP make components bigger (inclusive); CRP makes them smaller
  (exclusive). Ignoring REP -> hard to reuse; ignoring CCP -> too many components change at
  once; ignoring CRP -> too many unnecessary releases. Early in a project favor CCP
  (develop-ability); as the project matures and others reuse it, shift toward REP.

## Coupling - how components relate (ch. 14)
### ADP – Acyclic Dependencies
- No cycles in the component dependency graph (cycles cause "the morning-after syndrome":
  your code breaks because someone changed something you depend on, and cycles make
  everything in the cycle effectively one big component to build, test and release).
- Partition into releasable components; teams release versions others adopt at their own pace.
- **Breaking a cycle**: (1) apply DIP - put an interface the dependent side needs into its own
  component and implement it on the other side; or (2) create a new component that both
  depend on and move the shared classes there.
- Watch for cycles continuously (CI check) - the structure "jitters" as the system grows.

### Top-down design doesn't work for components
- The component structure is a map of **buildability and maintainability**, not of function.
  It can't be designed first; it evolves with the logical design as classes appear and
  change. Don't freeze a module layout before there is code to group.

### SDP – Stable Dependencies
- Depend in the direction of stability. Stability = how hard it is to change (many
  dependents), not how rarely it changes.
- **Instability** `I = Fan-out / (Fan-in + Fan-out)`, range 0 (max stable) – 1 (max unstable).
  Fan-in = classes outside depending on classes inside; Fan-out = classes inside depending
  on classes outside.
- Rule: **I of a component should be larger than the I of the components it depends on.**
- Not every component should be stable - volatile ones are needed for change. If a stable
  component must use a volatile one, invert the dependency with an interface in a stable,
  abstract component.

### SAP – Stable Abstractions
- A component's **abstractness should match its stability**: stable components contain
  interfaces/abstract classes so they can be extended without modification (OCP);
  unstable components contain concrete code that's easy to change.
- **Abstractness** `A = Na / Nc` (abstract classes + interfaces / total classes), range 0–1.
- **Main Sequence**: the line from (I=0, A=1) to (I=1, A=0). Best positions are its endpoints.
- **Zone of Pain** (I≈0, A≈0): stable and concrete - rigid (e.g. a DB schema, a concrete
  shared util). Only a problem if it's volatile; stable non-volatile things (like `String`) are fine.
- **Zone of Uselessness** (I≈1, A≈1): abstract but nobody depends on it - dead abstractions.
- **Distance** `D = |A + I − 1|`, range 0–1. Investigate components with D far from 0; track D
  per component over releases with a control threshold (e.g. 0.1). Metrics are guides, not gods.

## Tooling suggestions
- Java/Kotlin: ArchUnit (cycles, layer rules), JDepend-style metrics, jQAssistant.
- JS/TS: dependency-cruiser, madge. Python: import-linter. .NET: NetArchTest, NDepend.
