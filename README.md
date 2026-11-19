# clean-crew

**Clean Code, Clean Architecture, design patterns and secure coding for AI coding agents.**
Works with **Claude Code** and **GitHub Copilot CLI**.

[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-d97757)
![GitHub Copilot CLI](https://img.shields.io/badge/Copilot%20CLI-plugin-181717)

AI agents write code that works. `clean-crew` makes them write code that is also
**easy to read, easy to change, put in the right place, and hard to abuse**. It gives your
agent five skills and four ready-made agents that apply them every time it writes, reviews,
designs or audits code.

## What you get

| Piece | What it decides |
|---|---|
| `clean-architecture` skill | *Where* code goes and *which way dependencies point*: SOLID, the Dependency Rule, component cohesion/coupling, boundaries, use cases vs. details |
| `clean-code` skill | *How* code is written: naming, small functions, comments, formatting, error handling, classes, unit tests, and a smells checklist |
| `secure-design` skill | *What can go wrong* before the code exists: a lightweight threat model (data-flow sketch, trust boundaries, STRIDE, abuse cases, ranked threats with a response and a test) and secure design principles |
| `secure-coding` skill | *How to write it safely*: input handling, injection, output encoding, authentication, sessions and tokens, access control, secrets, logging, business-logic abuse, uploads. Includes a numbered finding catalog mapped to CWE, and web, API and attack-pattern references |
| `design-patterns` skill | *Whether and which* pattern: when a pattern is warranted (a named variation with a real second variant), what varies versus which pattern, the lightest form in the language at hand, how to introduce one safely, and when it is over-engineering. Includes a condensed catalog of the 23 classic patterns and the game and performance patterns (game loop, component, event queue, object pool, data locality, dirty flag, and more) |
| `clean-coder` agent | A general-purpose coding agent that loads the skills before any add/change/refactor (the security skills when the change touches input, auth, files, secrets or money), then self-reviews its diff against them |
| `clean-reviewer` agent | A read-only reviewer for diffs and PRs: fresh context, prioritized findings, smell IDs and concrete fixes. It never edits files |
| `pattern-advisor` agent | A read-only design advisor: for a requirement or code it decides "no pattern needed", "not yet" or the lightest fitting pattern, and flags patterns that are over-engineered or misapplied. Default answer is the simpler code |
| `security-auditor` agent | A read-only security auditor for a diff, a feature, a whole repository or a design: maps the attack surface, traces untrusted data to dangerous operations, probes in a throwaway copy, and reports ranked findings with CWE IDs, evidence, fixes and the missing tests |

Each skill has a short `SKILL.md` (always loaded) plus `references/` files that the agent
opens only when the task needs them (concurrency, packaging, service boundaries, the full
smells catalog, the finding catalog, and so on). That keeps your context small.

## What the agent does on every code change

1. Loads the two skills and only the reference files the task needs.
2. Reads the surrounding code. **Your project's conventions win** over generic rules.
3. Runs a design check: use case, policy vs. detail, dependency direction.
4. Writes the test first (or alongside), then implements.
5. Reviews its own diff against the smells and design checklists, citing smell IDs (G28, N7, ...).
6. Builds and runs tests, then reports what changed and why, plus any pre-existing smells it left alone.

Guardrails: no unrequested refactors, no over-engineering, no fighting the existing codebase.

## What the pattern advisor does

1. Names what varies and checks that a second variant really exists; if not, it says "not yet" and
   what would trigger the refactoring.
2. Looks for something plainer first: a function, a table, an enum, a language feature.
3. If a pattern is warranted, picks the lightest form and the pattern whose cost is affordable, says
   where it goes, who chooses the variant, and which tests prove it.
4. In review mode, reports missing, misapplied and over-engineered patterns with evidence.

## What the security auditor does

1. Maps entry points, trust boundaries, assets and where authorization is enforced.
2. Checks authorization first (missing object-level and function-level checks), then
   authentication, injection, output and browser controls, files and outgoing requests, secrets,
   errors, business logic, dependencies and tests.
3. Separates **confirmed** findings (reproduced or fully traced) from **possible** ones, so a grep
   hit is never reported as a vulnerability.
4. Never edits the project, never touches live systems, redacts any secret it finds, and says
   what it did not check.

## Install

### Claude Code (plugin)

```text
/plugin marketplace add iilativpy/clean-crew
/plugin install clean-crew@clean-crew
```

Then run the agent with `claude --agent clean-crew:clean-coder`, or just let Claude pick up
the skills automatically when you ask it to write or refactor code. To review a change, ask
Claude to "use the clean-reviewer agent on my diff", "use the pattern-advisor agent: do we need
a pattern for the payment providers?", or "use the security-auditor agent to audit the login flow".

### GitHub Copilot CLI (plugin)

```text
copilot plugin marketplace add iilativpy/clean-crew
copilot plugin install clean-crew@clean-crew
```

Select an agent with `/agent` and choose `clean-coder`, `clean-reviewer`, `pattern-advisor` or `security-auditor`.

> Copilot CLI support is based on the documented plugin format (it reads the same
> `.claude-plugin/` manifests). If something doesn't load for you, please
> [open an issue](https://github.com/iilativpy/clean-crew/issues).

### Manual install (no plugin system)

<details>
<summary>Claude Code, per repo (or <code>~/.claude</code> for all projects)</summary>

```bash
mkdir -p .claude/skills .claude/agents
cp -r skills/* .claude/skills/
cp agents/*.md .claude/agents/
```
</details>

<details>
<summary>GitHub Copilot CLI, per repo</summary>

```bash
mkdir -p .github/skills .github/agents
cp -r skills/* .github/skills/
for a in clean-coder clean-reviewer pattern-advisor security-auditor; do cp agents/$a.md .github/agents/$a.agent.md; done
```
</details>

### Lightest option: no agent, just a rule

Add this to `CLAUDE.md` / `.github/copilot-instructions.md`:

> Whenever you add or change code, first load the `clean-architecture` and `clean-code`
> skills and follow their checklists; self-review your diff against them before finishing.
> If the change touches input, authentication, permissions, files, secrets or money, also
> follow the `secure-coding` skill and write tests that send hostile input.

## Repository layout

```text
.claude-plugin/plugin.json          # plugin manifest (Claude Code + Copilot CLI)
.claude-plugin/marketplace.json     # makes this repo installable as a marketplace
agents/clean-coder.md               # writes code following the skills
agents/clean-reviewer.md            # read-only code-quality reviewer
agents/pattern-advisor.md           # read-only design-pattern advisor
agents/security-auditor.md          # read-only security auditor
skills/clean-architecture/          # design-level rules + references/
skills/clean-code/                  # code-level rules + references/
skills/design-patterns/             # pattern judgment + catalog + game/performance patterns
skills/secure-design/               # threat modeling + design principles + template
skills/secure-coding/               # secure-coding rules + finding catalog + references/
```

## Credits

The rules are paraphrased into short checklists and are not a substitute for the source
material.

- Clean Code and Clean Architecture: *Clean Code* and *Clean Architecture* by Robert C. Martin.
- Design patterns: *Design Patterns: Elements of Reusable Object-Oriented Software* (Gamma,
  Helm, Johnson, Vlissides) and *Game Programming Patterns* (Robert Nystrom). The catalog cards
  are condensed and reworded, with modern-language notes added.
- Security: the attack and defense material draws on *The Web Application Hacker's Handbook*
  (Dafydd Stuttard, Marcus Pinto) and on the access-control and API chapter of *API Security in
  Action* (Neil Madden). Current recommendations (password hashing parameters, cookies, CSRF,
  JWT, OAuth2, SSRF, CSP, uploads) follow the OWASP Cheat Sheet Series, and the threat-modeling
  method follows the four-question framework of Adam Shostack and the Threat Modeling Manifesto.
  Finding IDs map to CWE numbers.

The security skills are a review and design aid, not a guarantee. They do not replace a
penetration test, a dependency scanner, or a security professional for high-risk systems.

## Contributing

Issues and PRs are welcome: missing rules, language-specific examples, wrong advice,
or Copilot/Claude compatibility problems. Keep rules short, actionable, and checkable.

## License

[MIT](LICENSE)
