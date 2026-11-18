# Refactoring process (Clean Code ch. 3, 14, 15, 16)

Load this before any refactoring, or when a change grows messy while implementing.

## Write dirty, then clean
- Nobody writes clean code in one pass. Like prose: draft first, then restructure and
  polish. First drafts of functions are long, nested, badly named, duplicated — **that's
  fine only if they are covered by tests**, and only until you clean them.
- "It works" is not "done". Moving on as soon as code works is how messes are born; cleaning
  it later rarely happens.

## Notice when to stop adding features
- When each new feature makes the structure noticeably worse (e.g., every new type needs
  edits in three places), **stop adding features and refactor** before the mess gets too big.
- Look for the missing concept (repeated "do X for each kind" → a class hierarchy /
  polymorphic object).

## Incrementalism
- Massive "improvement" rewrites often ruin programs — it's hard to get back to the old
  behavior.
- Keep the system **working at all times** (TDD discipline): many tiny steps, run the tests
  after each, never a change that breaks the build for long.
- If a step breaks tests, fix it immediately before any other change.
- Introduce new structure alongside the old (e.g., add a skeleton class), migrate callers
  piece by piece, then delete the old code.

## First make it work, then make it right
Reviewing/refactoring existing code (even good code):
1. **Check the tests**: run them, measure coverage (T2); add independent tests for missing
   behavior and edge cases until coverage is high (T1, T5). Treat obvious-but-missing
   behavior as tests to make pass (G2).
2. **Make it work**: fix bugs the new tests reveal; look at coverage patterns for dead or
   wrong code (T8).
3. **Make it right**: walk the code top to bottom, applying the heuristics (name the smell —
   N1, G5, C2…), running all tests after every change.
4. Expect trial and error: you will sometimes undo a refactoring. Refactoring is iterative
   and converges.

## Professional review
- Critiquing code (yours or others') is a normal professional activity, not an attack.
  Welcome reviews of your own code.

## Boy Scout Rule in practice
- Even well-kept modules can be improved a little: remove scope prefixes, encapsulate a
  conditional, invert a negative, unify inconsistent names, fix off-by-one boundary logic.
  Leave each module slightly cleaner than you found it — within the scope of your change.

## Structured programming note (ch. 3)
- Dijkstra's single-entry/single-exit rule matters for large functions. If functions are
  small, an early `return`, `break` or `continue` can be clearer than contortions to keep a
  single exit. Never `goto`.
