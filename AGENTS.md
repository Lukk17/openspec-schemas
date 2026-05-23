# AGENTS.md

This file provides shared instructions to all AI coding agents working in this repository (Claude Code, Kilo Code,
OpenCode, Codex CLI). Standards and skills are imported from
[agent-standards](https://github.com/Lukk17/agent-standards).

---

## Skills

This project includes agent skills in [.agents/skills/](.agents/skills/). Invoke relevant skills before starting
implementation work. Examples:

- `/code-reviewer` before reviewing code
- `/security-review` before auditing for vulnerabilities
- `/coding-standards` before writing new code
- `/tdd-workflow` before adding features or fixing bugs
- `/e2e-runbooks` before adding or running an end-to-end capability test against the live stack
- `/markdown-writer` before authoring or polishing human-facing markdown (READMEs, schema docs, INTEGRATION.md)

Slash commands may appear as `/name` or `/name.md` in your agent's autocomplete; use whichever your agent shows.

---

## Subagents

This project ships a catalogue of specialised subagents: narrow-scope agents the main session delegates to. Claude
Code reads [.claude/agents/](.claude/agents/); OpenCode and Kilo Code both read [.opencode/agents/](.opencode/agents/).
Codex CLI has no per-agent file mechanism; it sees [AGENTS.md](AGENTS.md) plus skills only.

These files are generated artifacts pulled from agent-standards. Do **not** hand-edit them; changes will be
overwritten on the next pull. To modify a subagent permanently, edit its canonical source in the agent-standards repo
(`subagents/<name>.md`), regenerate there, and re-import.

A few of the most-used:

- `code-reviewer`: security-aware diff review before merge
- `test-automator`: write missing tests and fix failures without weakening assertions
- `security-auditor`: threat modelling, secure-coding review, compliance gap analysis
- `backend-architect`: contract-first service and API design
- `database-expert`: schema design and query / index optimisation
- `debugger`: root-cause analysis for a single failing test or runtime error
- `devops-troubleshooter`: live incident response with postmortem

Full catalogue: see the agent-standards README's "Subagents catalog" section, or list `.claude/agents/*.md` (or
`.opencode/agents/*.md`) in this project.

### When to use them

**Use subagents proactively, not reactively.** If a task has a subagent that fits, spawn it. Do not try to do the
specialist's work yourself from the main session; a short main-session reply that delegates correctly beats a long
main-session reply that reinvented the specialist's checklist. The catalogue exists so the main session can stay
high-level.

**Spawn them in parallel when the work is independent.** Multiple subagents running concurrently is the point. Review
breadth, multi-stack coverage, and fast turnaround all depend on it. Send one message with multiple `Agent` tool calls
when nothing blocks anything else.

Concrete patterns:

- **Reviewing code**: always spawn `code-reviewer`, AND every stack or concern-relevant specialist in parallel.
  `security-auditor` for auth, payments, secrets, or PII handling; `database-expert` for schema or query changes;
  `accessibility-expert` for UI work; `performance-engineer` after a hot-path edit; `api-tester` after an API surface
  change; `legal-advisor` for anything touching ToS, privacy, or consent. Default to *more* reviewers, not fewer.
- **Adding a feature**: `backend-architect` for the contract first; the relevant stack expert (`java-pro`,
  `python-pro`, `flutter-expert`, `angular-expert`, `react-nextjs-expert`) for the implementation; `test-automator`
  for the test plan.
- **Debugging**: `debugger` for a single failing test; `error-detective` for cross-service log correlation;
  `devops-troubleshooter` for a live incident.
- **Migrations and dependency bumps**: `code-archaeologist` first when the legacy area is unfamiliar; then
  `legacy-modernizer` for the phased plan.
- **Docs and content**: `docs-architect` for long-form technical writing; `seo-content-marketer` for anything
  externally facing; `api-documenter` for API reference work.

**You may direct subagents to invoke specific skills.** When you spawn a subagent, tell it which skills to use as part
of its work, e.g. "review this diff, invoke `/security-review` and `/coding-standards` while you do". The subagent
loads those skills the same way the main session does. This is the recommended way to bias a generalist subagent
toward a specific quality bar.

**Codex CLI exception.** Codex has no per-agent file mechanism, so subagent delegation is not available there. From
Codex, do the work in-session and lean harder on skills.

---

## MCP servers

This project exposes one MCP server: **Context7** for up-to-date library and tool documentation lookups. Use it before
re-deriving an answer about a third-party library, framework, or CLI from training data. Human-side setup lives in
[docs/MCP_SETUP.md](docs/MCP_SETUP.md).

---

## End-to-end capability tests

This repository does not ship runnable code, so there is no `e2e/` directory and no live stack to exercise. The
`e2e-runbooks` schema *defined* by this repo is methodology that consumer projects install; it is not exercised here.

If end-to-end coverage is ever added (e.g., a script that validates a schema against a fixture consumer project), it
should follow the `/e2e-runbooks` skill: per-capability `e2e/testing/{N}-{capability}-test.md` spec, immutable
`tasks.template.md` checklist, and one timestamped run record per execution under `e2e/testing/runs/`. Assertions
must be observable behaviour only.

---

## Working With Agents

All supported agents read this `AGENTS.md` from the project root and auto-discover skills from
[.agents/skills/](.agents/skills/). Start your agent from the project root:

- **Claude Code**: run `claude`. Reads [.claude/CLAUDE.md](.claude/CLAUDE.md), which imports this file.
- **Kilo Code**: reads `AGENTS.md` automatically. Optional [kilo.jsonc](kilo.jsonc.example) for extra config.
- **OpenCode**: reads `AGENTS.md` automatically. Optional [opencode.json](opencode.json.example) at project root.
- **Codex CLI**: run `codex`. Reads `AGENTS.md` automatically. Global settings in `~/.codex/config.toml`.

---

## Working Principles

Apply these to every task, in order. They govern *how* you work; the `coding-standards` skill governs *what the code
should look like*.

### 1. Think Before Coding

State assumptions explicitly. When the prompt is ambiguous, surface the interpretations and ask. Do not pick one
silently and run with it. If a simpler approach exists, propose it before writing code. Stop and ask when genuinely
unsure. A clarifying question costs less than a wrong implementation.

### 2. Simplicity First

Write the minimum code that solves the problem.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for scenarios that cannot happen.
- If 200 lines could be 50, rewrite it.

Test: would a senior engineer call this overcomplicated? If yes, simplify.

### 3. Surgical Changes

Touch only what the task requires.

- Do not "improve" adjacent code, comments, or formatting.
- Do not refactor code that is not broken.
- Match existing style, even if you would write it differently.
- If you notice unrelated dead code, mention it; do not delete it.
- Remove imports, variables, and helpers that *your* changes orphan. Leave pre-existing dead code alone unless asked.

Test: every changed line should trace directly to the request.

### 4. Goal-Driven Execution

Define success before starting. Convert vague asks into verifiable goals:

| Instead of...       | Transform to...                                                       |
| ------------------- | --------------------------------------------------------------------- |
| "Add validation"    | "Write tests for invalid inputs, then make them pass"                 |
| "Fix the bug"       | "Write a failing test that reproduces it, then make it pass"          |
| "Refactor X"        | "Ensure tests pass before and after, behavior unchanged"              |

For multi-step work, state the plan first:

1. `<step>` then verify: `<check>`
2. `<step>` then verify: `<check>`
3. `<step>` then verify: `<check>`

Then loop until each check passes. Do not claim a task is done without running the verification.

---

## OpenSpec Workflow

This project does **not** initialise its own `openspec/` directory. The repo *hosts* OpenSpec schemas for other
projects to consume; it does not consume OpenSpec internally. The `e2e-runbooks` schema defined here is itself an
OpenSpec custom schema (a `schema.yaml` plus templates) that other projects install under their
`openspec/schemas/e2e-runbooks/` directory.

**Validating a schema while you work**: spin up a throwaway consumer project, install the schema, run validate +
round-trip. The same flow lives in [`.github/workflows/validate.yml`](.github/workflows/validate.yml); the snippet
below mirrors the local development loop:

```bash
tmp=$(mktemp -d)
cd "$tmp" && openspec init --tools none
mkdir -p openspec/schemas
cp -r <path-to-this-repo>/<schema-name> openspec/schemas/
openspec schema validate <schema-name> --verbose
openspec new change "probe-$(date +%s)" --schema <schema-name> --description "probe"
openspec templates --schema <schema-name>
```

If a future change to this repo genuinely warrants OpenSpec-style proposal tracking (e.g., a new schema beyond
`e2e-runbooks`), initialise OpenSpec at that point and follow the standard lifecycle. For now: trivial edits land
directly; non-trivial work is planned per the conversation and committed in logical groupings.

---

## What This Repo Is

`openspec-schemas` is a public collection of community [OpenSpec](https://github.com/Fission-AI/OpenSpec) custom
schemas published by Łukasz Sarna. OpenSpec is a spec-driven workflow for AI coding agents; a *schema* is a bundle
(`schema.yaml` plus markdown templates) that defines a custom artifact lifecycle the agent can drive via OpenSpec
slash commands (e.g. `openspec new change`, `openspec instructions`, `openspec archive`).

The first schema, `e2e-runbooks`, operationalises capability-level end-to-end testing: each capability gets an
immutable spec, an immutable tasks template, and one timestamped run record per execution. Assertions are observable
behaviour only (HTTP status, response body, persisted state — never log substrings). Each run records start/end UTC,
duration, and best-estimate LLM token consumption. The schema ships alongside a methodology skill
(`.agents/skills/e2e-runbooks/`) and a runner subagent (`.claude/agents/e2e-runner.md`,
`.opencode/agents/e2e-runner.md`) in [`agent-standards`](https://github.com/Lukk17/agent-standards). Skill, subagent,
and schema each install independently; together they reinforce each other.

Consumers install a schema by copying its folder into their own project's `openspec/schemas/<name>/` directory.
OpenSpec has no registry, marketplace, or `schema install <url>` CLI today (tracked in
[Fission-AI/OpenSpec#693](https://github.com/Fission-AI/OpenSpec/issues/693)); install is a manual
`git clone --depth 1 --branch vX.Y.Z ... && cp -r` step documented in [README.md](README.md). Pinning to a release
tag is the only safe way to guard against upstream drift.

The repo itself contains no runtime code, no application tests. It ships:

- One folder per schema at the repo root.
- A GitHub Actions pipeline ([`.github/workflows/`](.github/workflows/)) that validates every schema on PR/push,
  round-trips a `new change` for each, runs markdownlint + lychee link-checking, and releases on `v*.*.*` tag.
- The agent-standards import in `.agents/`, `.claude/`, `.opencode/`, `.codex/`.

---

## Architecture

- **Repository type**: documentation / template bundle plus CI. No application code, no application tests; the test
  surface is the schemas themselves, exercised by the validate workflow.
- **Layout**: one top-level directory per schema (e.g., `e2e-runbooks/`), each containing a `schema.yaml`,
  `README.md`, `INTEGRATION.md`, and a `templates/` subdirectory with the artifact markdown templates and a
  `scaffold/` for one-time consumer-project scaffolding files.
- **Contract surface**: each `schema.yaml` declares the artifact DAG (which file produces which, in what order) and
  per-artifact agent instructions. OpenSpec consumes these files; the schema's correctness is measured by whether
  `openspec schema validate <name>` passes and `openspec new change --schema <name>` plus
  `openspec instructions <artifact> --change <id>` resolve every artifact to a sensible path inside the change
  directory.
- **Deploy target**: GitHub public repo. Consumption is `git clone --depth 1 --branch vX.Y.Z + cp -r`. Releases are
  produced by the `release` workflow on `v*.*.*` tag push.
- **Versioning**: per-schema semver. Breaking changes to a schema's artifact DAG (renamed artifact id, reordered
  `requires`, removed field) bump the major; new artifacts or relaxed constraints bump the minor; doc and
  instruction-prose changes bump the patch.
- **OpenSpec model gotchas** (carried in [`e2e-runbooks/INTEGRATION.md`](e2e-runbooks/INTEGRATION.md); apply to any
  future schema too):
  - OpenSpec resolves `generates:` paths relative to the change directory. Use bare filenames like `proposal.md`;
    `openspec/changes/{change-id}/proposal.md` produces a doubled path and leaves `{change-id}` literal.
  - OpenSpec does not substitute schema placeholders (`{N}`, `{capability}`, `{change-id}`, etc.) in `generates:`
    paths. Placeholders only work in instruction prose, where the agent fills them.
  - `openspec archive` only auto-promotes `openspec/specs/`. Artifacts with a permanent home elsewhere (e.g.
    `e2e/testing/`) must be written by the agent in BOTH locations via dual-write instruction prose.
  - `apply.requires` must be an **array** (`[run]`), never a scalar.
  - The CI validate workflow round-trips `openspec new change` and checks `openspec templates` output for
    unsubstituted placeholders; this catches the placeholder-leak class of bug.
- **Constraints**:
  - Templates must remain agent-agnostic: nothing in a schema may assume Claude Code specifically; OpenCode, Kilo,
    and Codex must all be able to drive the lifecycle.
  - Schemas must not depend on each other. Each is independently installable.
  - Skill-side methodology (in `agent-standards`) is the source of truth for *why*. Schema templates are the source
    of truth for *what files exist and in what order*. The subagent (in `agent-standards`) is the source of truth for
    *runner behaviour at execution time*. Drift across the three is the most likely defect class — update all three
    together.
- **Agent tooling**: this repo imports `agent-standards` (see [docs/AGENT_TOOLING.md](docs/AGENT_TOOLING.md)) so the
  authoring agent gets the same skills, subagents, and conventions used in consumer projects. That keeps the schema
  author's workflow honest: if a skill doesn't work for the schema's own author, it won't work for the schema's
  consumers either.
