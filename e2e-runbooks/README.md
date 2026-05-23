# e2e-runbooks

OpenSpec schema for spec-driven, AI-or-human-executable, behaviour-only e2e capability tests.

## What it does

Each capability test you add gets an **immutable spec** describing what it verifies, a paired **immutable
tasks-template** with checkboxes mirroring the spec, and one **run record** per execution. Run records carry start /
end UTC, computed duration, and best-estimate LLM token consumption.

Assertions are observable behaviour only: HTTP status codes, response body content, persisted state in MinIO / Qdrant
/ Postgres / etc. Never log substrings; they drift across versions and aren't visible from every runner's shell.

## Artifact DAG

```text
proposal ─► test-spec ─► tasks-template ─► run (per execution)
```

- **proposal** — required only when adding a new capability test. Names the behaviour, setup cost class, fixtures, API
  client invocation, chosen N prefix.
- **test-spec** — immutable spec describing one capability in six fixed sections.
- **tasks-template** — immutable checklist mirroring the spec.
- **run** — execution record. Filled per execution.

## Where files land

OpenSpec drafts every artifact inside the change directory (`openspec/changes/<change-id>/`). For this schema, three
of the four artifacts also have a permanent home in the consumer project's `e2e/` tree. The agent writes to BOTH
locations because OpenSpec's `archive` command only auto-promotes `openspec/specs/` content; everything else needs an
explicit dual-write:

| Artifact         | Change-dir path (OpenSpec audit trail)                | Final home (working asset)                                              |
| ---------------- | ------------------------------------------------------ | ----------------------------------------------------------------------- |
| `proposal`       | `openspec/changes/<id>/proposal.md`                    | _none — proposal stays in the change dir_                               |
| `test-spec`      | `openspec/changes/<id>/test-spec.md`                   | `e2e/testing/{N}-{capability}-test.md`                                  |
| `tasks-template` | `openspec/changes/<id>/tasks-template.md`              | `e2e/testing/{N}-{capability}-tasks.template.md`                        |
| `run`            | `openspec/changes/<id>/run.md`                         | `e2e/testing/runs/{utc-timestamp}_{N}-{capability}-tasks.md` (gitignored)|

Each artifact's `instruction:` in [`schema.yaml`](./schema.yaml) tells the agent to do the dual-write. `{N}`,
`{capability}`, and `{utc-timestamp}` are filled by the agent from the proposal's content — OpenSpec itself does not
substitute schema placeholders in `generates:` paths.

## Install

Your consumer project must already have OpenSpec initialised (`openspec init`); the schema installs under its
`openspec/schemas/` directory. OpenSpec has no schema-install CLI yet, so the install is a manual copy. From the
consumer project's root:

```bash
git clone --depth 1 --branch v0.1.0 https://github.com/Lukk17/openspec-schemas /tmp/lukk17-schemas
```

```bash
cp -r /tmp/lukk17-schemas/e2e-runbooks openspec/schemas/
```

```bash
rm -rf /tmp/lukk17-schemas
```

```powershell
git clone --depth 1 --branch v0.1.0 https://github.com/Lukk17/openspec-schemas $env:TEMP\lukk17-schemas
```

```powershell
Copy-Item -Recurse $env:TEMP\lukk17-schemas\e2e-runbooks openspec\schemas\
```

```powershell
Remove-Item -Recurse -Force $env:TEMP\lukk17-schemas
```

Then either invoke per-change (the `change` subcommand is required; `--schema` is an option on it):

```bash
openspec new change "add-weather-mcp-test" --schema e2e-runbooks
```

Or set it as the default in your project's `openspec/config.yaml` and drive it with `/opsx:propose`:

```yaml
default_schema: e2e-runbooks
```

## Companion skill and subagent

This schema is one corner of a three-piece kit shipped together by
[Lukk17/agent-standards](https://github.com/Lukk17/agent-standards):

- **Skill** at `.agents/skills/e2e-runbooks/SKILL.md` — methodology source of truth. Behaviour-only assertions, canary
  fixtures, API client alternatives (Bruno, hurl, REST Client, curl, httpie), runs-directory contract, generic worked
  examples.
- **Subagent** at `.claude/agents/e2e-runner.md` and `.opencode/agents/e2e-runner.md` — _planned, not yet released._
  Intended as the recommended delegate when driving a sweep end-to-end: it will own the runner contract (Start/End
  UTC, Duration, tokens, Verdict) and the change-dir / `e2e/testing/runs/` dual-write. Until it ships in
  agent-standards, drive runs from the main session or a general runner; the schema and skill work without it.
- **Schema** (this folder) — operationalises the methodology for OpenSpec users via the artifact DAG above.

Each piece is independently installable: the skill works in projects without OpenSpec, the schema works in projects
without agent-standards. Together they reinforce each other.

## OpenSpec limitations to know

This schema is designed around two known OpenSpec limitations:

- [Fission-AI/OpenSpec#777](https://github.com/Fission-AI/OpenSpec/issues/777) — schema `instruction` fields can name
  external skills, but `openspec-continue-change` doesn't reliably delegate. We embed the runner contract directly in
  the `run` artifact's instruction prose, no mid-DAG skill delegation.
- [Fission-AI/OpenSpec#666](https://github.com/Fission-AI/OpenSpec/issues/666) — OpenSpec's spec format is partly
  hard-coded in package code. We name the spec artifact `test-spec` (not `spec`) to sidestep the hard-coded format.

## License

MIT.
