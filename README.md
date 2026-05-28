# openspec-schemas

Community [OpenSpec](https://github.com/Fission-AI/OpenSpec) schemas for spec-driven AI workflows.

## Schemas

| Schema | Purpose |
| --- | --- |
| [e2e-runbooks](./e2e-runbooks/) | Capability-level e2e test suite: spec + tasks-template + run records, behaviour-only assertions, per-run token + duration accounting. |

Every schema in this repository ships as a self-contained folder with a `schema.yaml`, a `README.md`, an
`INTEGRATION.md`, and a `templates/` subtree. Schemas are independent — installing one does not pull in the others.

## Prerequisites

| Tool | Minimum version | Why |
| --- | --- | --- |
| [Node.js](https://nodejs.org/) | 20.x | Runs the OpenSpec CLI; CI runs on Node 20. |
| [OpenSpec CLI](https://github.com/Fission-AI/OpenSpec) | 1.3.x | Drives proposals, applies, archives. Install with `npm install --global @fission-ai/openspec@latest`. |
| Git | any recent | Schema install is `git clone --depth 1 + cp -r`; tags pin a release. |

Some schemas (notably [e2e-runbooks](./e2e-runbooks/)) work best when paired with the matching skill and subagent in
[Lukk17/agent-standards](https://github.com/Lukk17/agent-standards). See each schema's own README for the specific
pairings.

## Install (any schema)

OpenSpec does not ship a `schema install <url>` command today, so installing is a manual copy. From your consumer
project's root:

```bash
git clone --depth 1 --branch v0.1.0 https://github.com/Lukk17/openspec-schemas /tmp/lukk17-schemas
```

```bash
cp -r /tmp/lukk17-schemas/<schema-name> openspec/schemas/
```

```bash
rm -rf /tmp/lukk17-schemas
```

```powershell
git clone --depth 1 --branch v0.1.0 https://github.com/Lukk17/openspec-schemas $env:TEMP\lukk17-schemas
```

```powershell
Copy-Item -Recurse $env:TEMP\lukk17-schemas\<schema-name> openspec\schemas\
```

```powershell
Remove-Item -Recurse -Force $env:TEMP\lukk17-schemas
```

Pin to a release tag (`--branch vX.Y.Z`) when you want reproducible installs; drop the flag to follow `master`.

Then either invoke the schema per-change:

```bash
openspec new change "add-<capability>-test" --schema <schema-name>
```

Or set it as the default in your project's `openspec/config.yaml`:

```yaml
default_schema: <schema-name>
```

## Usage

Each schema documents its own artifact DAG and per-artifact instructions in its `README.md` and `schema.yaml`. The
canonical OpenSpec lifecycle is:

1. `openspec new change "<id>" --schema <name>` — scaffolds the change directory and tags it with the schema.
2. `openspec instructions <artifact> --change <id>` — emits the agent-facing prompt for the next artifact in the DAG.
   The agent fills the file.
3. `openspec validate <id>` — checks the change is well-formed.
4. `openspec archive <id>` — finalises the change.

Schemas may instruct the agent to write artifacts to additional locations outside `openspec/changes/<id>/`. The
[e2e-runbooks](./e2e-runbooks/) schema, for example, dual-writes specs and run records into a separate `e2e/`
tree because OpenSpec's archive only auto-promotes `openspec/specs/`. Each schema's `INTEGRATION.md` spells out
its own out-of-tree write paths.

## Companion skills and subagents

Many of these schemas have paired skills (procedural guidance) and subagents (delegated specialists) in
[Lukk17/agent-standards](https://github.com/Lukk17/agent-standards). The skill and subagent are imported via the
agent-standards selective checkout into the consumer project; the schema is installed separately into
`openspec/schemas/`. Pull all three when adopting:

| Schema | Skill (in `agent-standards`) | Subagent (in `agent-standards`) |
| --- | --- | --- |
| [e2e-runbooks](./e2e-runbooks/) | [`.agents/skills/e2e-runbooks/`](https://github.com/Lukk17/agent-standards/tree/master/.agents/skills/e2e-runbooks) | [`.claude/agents/e2e-runner.md`, `.opencode/agents/e2e-runner.md`](https://github.com/Lukk17/agent-standards) |

The `e2e-runbooks` skill and `e2e-runner` subagent both ship in agent-standards. Schemas and skills can be adopted
independently — pairing them reduces the seams.

## Validation in CI

The repository validates every schema on every push and PR via [`.github/workflows/validate.yml`](.github/workflows/validate.yml):

```bash
openspec schema validate <schema-name>
```

A round-trip step then runs `openspec new change --schema <schema-name>` and asserts that no schema placeholder
(e.g. `{change-id}`, `{N}`) leaks into the resolved template paths. To run the same checks locally:

```bash
npm install --global @fission-ai/openspec@latest
```

```bash
openspec schema validate <schema-name>
```

(run from a consumer project where the schema has been copied into `openspec/schemas/`).

## Releases and consumer pinning

Releases are tagged `vX.Y.Z` and published as GitHub Releases by [`.github/workflows/release.yml`](.github/workflows/release.yml).
Consumers should pin to a tag for reproducibility:

```bash
git clone --depth 1 --branch v0.1.0 https://github.com/Lukk17/openspec-schemas /tmp/lukk17-schemas
```

A breaking change to a schema's artifact DAG (renamed artifact id, reordered `requires`, removed field) bumps the
major. Adding a new artifact or relaxing a constraint bumps the minor. Pure doc / instruction-prose changes bump the
patch.

OpenSpec itself has no community schema registry yet (tracked in
[Fission-AI/OpenSpec#693](https://github.com/Fission-AI/OpenSpec/issues/693)); pinning by tag is the only safe way to
guard against upstream drift.

## Contributing

Each schema is its own folder at the repo root with the standard OpenSpec bundle layout (`schema.yaml`, `README.md`,
optional `INTEGRATION.md`, `templates/`). Open a PR against `master`. CI will run the validate + round-trip + lint
workflows; all three must pass before merge.

Validate locally first:

```bash
openspec schema validate <schema-name>
```

## License

MIT, see [LICENSE](./LICENSE).
