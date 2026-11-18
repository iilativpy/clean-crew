# Coverage map - book chapter -> where it lives

Abbreviations: `CC` = skills/clean-code, `CA` = skills/clean-architecture, `ref/` = that skill's references folder.
All content is paraphrased into rules/checklists; no book text is reproduced.

## Clean Code (R. C. Martin)
| Chapter | Location |
|---|---|
| 1 Clean Code (cost of mess, Boy Scout Rule, authors) | CC SKILL §0 |
| 2 Meaningful Names | CC SKILL §1 (all sections incl. mental mapping, cute, context) |
| 3 Functions | CC SKILL §2; structured programming + drafting -> also `ref/refactoring-process.md` |
| 4 Comments | CC SKILL §3; C1–C5 in `ref/smells-catalog.md` |
| 5 Formatting | CC SKILL §4 |
| 6 Objects and Data Structures (anti-symmetry, Demeter, DTO, Active Record) | CC SKILL §5 |
| 7 Error Handling (incl. normal flow / Special Case) | CC SKILL §6 |
| 8 Boundaries (third-party, learning tests) | CC SKILL §7 |
| 9 Unit Tests (TDD laws, clean tests, dual standard, F.I.R.S.T.) | CC SKILL §8 |
| 10 Classes (organization, SRP, cohesion, change, encapsulation) | CC SKILL §9 |
| 11 Systems | CC SKILL §10 summary + `ref/systems.md` |
| 12 Emergence (4 rules of simple design) | CC SKILL §10 + `ref/systems.md` |
| 13 Concurrency | CC SKILL §10 summary + `ref/concurrency.md` |
| 14 Successive Refinement | `ref/refactoring-process.md` |
| 15 JUnit Internals | `ref/refactoring-process.md` (Boy Scout in practice) |
| 16 Refactoring SerialDate | `ref/refactoring-process.md` (make it work -> make it right) |
| 17 Smells and Heuristics (66 items) | CC SKILL §11 quick list + `ref/smells-catalog.md` (all IDs) |
| Appendix A Concurrency II | `ref/concurrency.md` (deadlock, locking strategies, library, testing) |
| Appendix B SerialDate listing | - (source code listing only) |
| Appendix C Heuristics cross-reference | - (index only; IDs kept in catalog) |

## Clean Architecture (R. C. Martin)
| Chapter | Location |
|---|---|
| 1 What Is Design and Architecture? | CA SKILL §0 |
| 2 A Tale of Two Values | CA SKILL §0 |
| 3–6 Paradigms (structured, OO, functional) | CA SKILL §1 pointer + `ref/paradigms.md` |
| 7–11 SRP, OCP, LSP, ISP, DIP | CA SKILL §2 |
| 12 Components | `ref/component-principles.md` |
| 13 Component Cohesion (REP, CCP, CRP, tension) | CA SKILL §3 + `ref/component-principles.md` |
| 14 Component Coupling (ADP, SDP, SAP, I/A/D metrics) | CA SKILL §3 + `ref/component-principles.md` |
| 15 What Is Architecture? | `ref/boundaries-and-services.md` |
| 16 Independence (use cases, duplication, decoupling modes) | CA SKILL §2/§4 + `ref/boundaries-and-services.md` |
| 17 Boundaries: Drawing Lines (plugin architecture) | CA SKILL §4 + `ref/boundaries-and-services.md` |
| 18 Boundary Anatomy | `ref/boundaries-and-services.md` |
| 19 Policy and Level | CA SKILL §1 + `ref/boundaries-and-services.md` |
| 20 Business Rules (entities, use cases, request/response) | CA SKILL §1, §5 |
| 21 Screaming Architecture | CA SKILL §4 + `ref/packaging-strategies.md` |
| 22 The Clean Architecture | CA SKILL §1 |
| 23 Presenters and Humble Objects | CA SKILL §4 |
| 24 Partial Boundaries | CA SKILL §4 + `ref/boundaries-and-services.md` |
| 25 Layers and Boundaries | CA SKILL §4 + `ref/boundaries-and-services.md` |
| 26 The Main Component | CA SKILL §4 |
| 27 Services: Great and Small | CA SKILL §4 + `ref/boundaries-and-services.md` |
| 28 The Test Boundary | CA SKILL §6 |
| 29 Clean Embedded Architecture | `ref/embedded.md` |
| 30–32 Database / Web / Frameworks Are Details | CA SKILL §4 |
| 33 Case Study: Video Sales | CA SKILL §7 (worked approach) |
| 34 The Missing Chapter | `ref/packaging-strategies.md` |
| Appendix A Architecture Archaeology | - (author's career history; lessons already reflected in the principles above) |
