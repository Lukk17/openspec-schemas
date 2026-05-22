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
- **test-spec** — immutable spec at `e2e/testing/{N}-{capability}-test.md`. Six fixed sections.
- **tasks-template** — immutable checklist at `e2e/testing/{N}-{capability}-tasks.template.md`. Mirrors the spec.
- **run** — execution record at `e2e/testing/runs/{utc-timestamp}_{N}-{capability}-tasks.md`. Filled per execution.
  Default-gitignored.

## Install

OpenSpec has no schema-install CLI yet. From your consumer project's root:

```bash
git clone --depth 1 https://github.com/Lukk17/openspec-schemas /tmp/lukk17-schemas
```

```bash
cp -r /tmp/lukk17-schemas/e2e-runbooks openspec/schemas/
```

```bash
rm -rf /tmp/lukk17-schemas
```

```powershell
git clone --depth 1 https://github.com/Lukk17/openspec-schemas $env:TEMP\lukk17-schemas
```

```powershell
Copy-Item -Recurse $env:TEMP\lukk17-schemas\e2e-runbooks openspec\schemas\
```

```powershell
Remove-Item -Recurse -Force $env:TEMP\lukk17-schemas
```

Then either invoke per-change:

```bash
openspec new --schema e2e-runbooks "add weather-mcp capability test"
```

Or set as the default in your project's `openspec/config.yaml`:

```yaml
default_schema: e2e-runbooks
```

## Companion skill

The methodology is documented in full in the
[`e2e-runbooks` skill in Lukk17/agent-standards](https://github.com/Lukk17/agent-standards/tree/master/.agents/skills/e2e-runbooks).
The skill covers behaviour-only assertions, canary fixtures, API client alternatives (Bruno, hurl, REST Client, curl,
httpie), runs-directory contract, and generic worked examples. The schema and skill encode the same methodology; the
schema operationalises it for OpenSpec users.

## OpenSpec limitations to know

This schema is designed around two known OpenSpec limitations:

- [Fission-AI/OpenSpec#777](https://github.com/Fission-AI/OpenSpec/issues/777) — schema `instruction` fields can name
  external skills, but `openspec-continue-change` doesn't reliably delegate. We embed the runner contract directly in
  the `run` artifact's instruction prose, no mid-DAG skill delegation.
- [Fission-AI/OpenSpec#666](https://github.com/Fission-AI/OpenSpec/issues/666) — OpenSpec's spec format is partly
  hard-coded in package code. We name the spec artifact `test-spec` (not `spec`) to sidestep the hard-coded format.

## License

MIT.
