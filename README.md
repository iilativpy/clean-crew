# clean-crew

**Clean Code + Clean Architecture for AI coding agents.**
Works with **Claude Code** and **GitHub Copilot CLI**.

[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-d97757)
![GitHub Copilot CLI](https://img.shields.io/badge/Copilot%20CLI-plugin-181717)

AI agents write code that works. `clean-crew` makes them write code that is also
**easy to read, easy to change, and put in the right place**. It gives your agent two
skills and a ready-made agent that applies them every time it touches code.

## What you get

| Piece | What it decides |
|---|---|
| `clean-architecture` skill | *Where* code goes and *which way dependencies point*: SOLID, the Dependency Rule, component cohesion/coupling, boundaries, use cases vs. details |
| `clean-code` skill | *How* code is written: naming, small functions, comments, formatting, error handling, classes, unit tests, and a smells checklist |
| `clean-coder` agent | A general-purpose coding agent that loads both skills before any add/change/refactor, then self-reviews its diff against them |

Each skill has a short `SKILL.md` (always loaded) plus `references/` files that the agent
opens only when the task needs them (concurrency, packaging, service boundaries, the full
smells catalog, and so on). That keeps your context small.

## What the agent does on every code change

1. Loads the two skills and only the reference files the task needs.
2. Reads the surrounding code. **Your project's conventions win** over generic rules.
3. Runs a design check: use case, policy vs. detail, dependency direction.
4. Writes the test first (or alongside), then implements.
5. Reviews its own diff against the smells and design checklists, citing smell IDs (G28, N7, ...).
6. Builds and runs tests, then reports what changed and why, plus any pre-existing smells it left alone.

Guardrails: no unrequested refactors, no over-engineering, no fighting the existing codebase.

## Install

### Claude Code (plugin)

```text
/plugin marketplace add iilativpy/clean-crew
/plugin install clean-crew@clean-crew
```

Then run the agent with `claude --agent clean-crew:clean-coder`, or just let Claude pick up
the skills automatically when you ask it to write or refactor code.

### GitHub Copilot CLI (plugin)

```text
copilot plugin marketplace add iilativpy/clean-crew
copilot plugin install clean-crew@clean-crew
```

Select the agent with `/agent` and choose `clean-coder`.

> Copilot CLI support is based on the documented plugin format (it reads the same
> `.claude-plugin/` manifests). If something doesn't load for you, please
> [open an issue](https://github.com/iilativpy/clean-crew/issues).

### Manual install (no plugin system)

<details>
<summary>Claude Code, per repo (or <code>~/.claude</code> for all projects)</summary>

```bash
mkdir -p .claude/skills .claude/agents
cp -r skills/clean-code skills/clean-architecture .claude/skills/
cp agents/clean-coder.md .claude/agents/clean-coder.md
```
</details>

<details>
<summary>GitHub Copilot CLI, per repo</summary>

```bash
mkdir -p .github/skills .github/agents
cp -r skills/clean-code skills/clean-architecture .github/skills/
cp agents/clean-coder.md .github/agents/clean-coder.agent.md
```
</details>

### Lightest option: no agent, just a rule

Add this to `CLAUDE.md` / `.github/copilot-instructions.md`:

> Whenever you add or change code, first load the `clean-architecture` and `clean-code`
> skills and follow their checklists; self-review your diff against them before finishing.

## Repository layout

```text
.claude-plugin/plugin.json          # plugin manifest (Claude Code + Copilot CLI)
.claude-plugin/marketplace.json     # makes this repo installable as a marketplace
agents/clean-coder.md               # the agent
skills/clean-architecture/          # design-level rules + references/
skills/clean-code/                  # code-level rules + references/
COVERAGE.md                         # book chapter -> where it lives in the skills
```

## Coverage

Every chapter of *Clean Code* and *Clean Architecture* is mapped to a place in the skills.
See [COVERAGE.md](COVERAGE.md). The content is paraphrased into rules and checklists;
no book text is reproduced. If you want the full reasoning behind the rules, read the books
by Robert C. Martin.

## Contributing

Issues and PRs are welcome: missing rules, language-specific examples, wrong advice,
or Copilot/Claude compatibility problems. Keep rules short, actionable, and checkable.

## License

[MIT](LICENSE)
