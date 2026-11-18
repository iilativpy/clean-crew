# Packaging strategies and enforcement (Clean Architecture ch. 21, 34 "The Missing Chapter")

Load this when creating new packages/modules, organizing a code base, or deciding how to
enforce architecture rules.

## Four ways to organize the same use case (e.g. "view orders")
| Style | Structure | Pros | Cons |
|---|---|---|---|
| **Package by layer** | `web/`, `service/`, `data/` | simple, familiar | doesn't scream the domain; nothing prevents a controller from skipping the service and calling the repository directly |
| **Package by feature** | `orders/` holding controller, service, repository | screams the domain; one place per use case | everything in one package — weak separation of web/business/persistence |
| **Ports & adapters** (hexagonal) | `domain/` (inside: `OrdersService`, `Orders`) + `web/`, `database/` (outside) | domain independent of frameworks; dependencies point inward; domain names use the ubiquitous language (`Orders`, not `OrdersRepository`) | more types/packages |
| **Package by component** | `orders/` exposing one public `OrdersComponent` interface; business logic + persistence hidden inside; web outside | one entry point per coarse-grained component; compiler enforces encapsulation; stepping stone to services | business and persistence are grouped (separated only internally) |

## The devil is in the implementation details
- **Overusing `public` collapses all four styles into the same thing**: packages become mere
  folders, and nothing stops code from bypassing the intended structure.
- Make types **as non-public as possible** (package-private/`internal`); expose only interfaces
  with inbound dependencies; inject implementations at runtime.
- **Prefer the compiler to enforce architecture** over discipline or post-compile checks:
  1. Access modifiers + package placement.
  2. Module systems (Java modules/JPMS, OSGi): distinguish *public* from *published* types.
  3. Separate source trees / build modules (Maven/Gradle/MSBuild): e.g. `domain`,
     `web`, `persistence` (or at least `domain` vs. `infrastructure`) so forbidden imports
     don't compile.
  4. Static analysis rules (ArchUnit, dependency-cruiser, import-linter) as a fallback —
     useful, but feedback is later and fallible.
- Watch for coupling in other places, e.g. shared data models across components.
- Be pragmatic: weigh team size, skill, complexity, time and budget.

## Screaming architecture (ch. 21)
- Top-level structure should tell you what the system **does** (health care, ordering), not
  which framework it uses. Frameworks are tools, not the architecture.
- The architecture should let you test all use cases without the web server, database or
  framework in place.
