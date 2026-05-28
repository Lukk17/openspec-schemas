# openspec-schemas

Community [OpenSpec](https://github.com/Fission-AI/OpenSpec) schemas for spec-driven AI workflows.

## Schemas

| Schema | Purpose |
| --- | --- |
| [e2e-runbooks](./openspec/schemas/e2e-runbooks/) | Capability-level e2e test suite: spec + tasks-template + run records, behaviour-only assertions, per-run token + duration accounting. |

Every schema in this repository ships as a self-contained folder with a `schema.yaml`, a `README.md`, an
`INTEGRATION.md`, and a `templates/` subtree. Schemas are independent — installing one does not pull in the others.

## Prerequisites

| Tool | Minimum version | Why |
| --- | --- | --- |
| [Node.js](https://nodejs.org/) | 20.x | Runs the OpenSpec CLI; CI runs on Node 20. |
| [OpenSpec CLI](https://github.com/Fission-AI/OpenSpec) | 1.3.x | Drives proposals, applies, archives. Install with `npm install --global @fission-ai/openspec@latest`. |
| Git | any recent | Schema install is `git clone --depth 1 + cp -r`; tags pin a release. |

Some schemas (notably [e2e-runbooks](./openspec/schemas/e2e-runbooks/)) work best when paired with the matching skill and subagent in
[Lukk17/agent-standards](https://github.com/Lukk17/agent-standards). See each schema's own README for the specific
pairings.

## Install

Every schema in this repo lives at `openspec/schemas/<name>/` — the exact path it needs in your consumer project.
Install is a `git fetch` + `git checkout` from inside your project that writes only the schema's directory; nothing
else in your tree is touched. Same pattern as agent-standards' selective checkout.

Your consumer project must be a git repo and must have OpenSpec initialised (`openspec init`). Run from the project
root, replacing `e2e-runbooks` with whichever schema you want and `v0.1.0` with the version to pin:

```bash
git fetch --depth 1 https://github.com/Lukk17/openspec-schemas v0.1.0
```

```bash
git checkout FETCH_HEAD -- openspec/schemas/e2e-runbooks
```

The same two commands work in PowerShell — `git` is `git` on every platform.

To install every schema, replace `openspec/schemas/e2e-runbooks` with `openspec/schemas`. To track `master` instead of
a tag, replace `v0.1.0` with `master`. The files land in your working tree AND staged — review with `git diff --staged`
and commit when ready.

If you'll be updating across releases, add the repo as a named remote once so future installs just re-run the two
commands with the new tag:

```bash
git remote add openspec-schemas https://github.com/Lukk17/openspec-schemas
```

```bash
git remote set-url --push openspec-schemas no_push
```

Then later upgrades:

```bash
git fetch --depth 1 openspec-schemas v0.2.0
```

```bash
git checkout FETCH_HEAD -- openspec/schemas/e2e-runbooks
```

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
[e2e-runbooks](./openspec/schemas/e2e-runbooks/) schema, for example, dual-writes specs and run records into a separate `e2e/`
tree because OpenSpec's archive only auto-promotes `openspec/specs/`. Each schema's `INTEGRATION.md` spells out
its own out-of-tree write paths.

## Companion skills and subagents

Many of these schemas have paired skills (procedural guidance) and subagents (delegated specialists) in
[Lukk17/agent-standards](https://github.com/Lukk17/agent-standards). The skill and subagent are imported via the
agent-standards selective checkout into the consumer project; the schema is installed separately into
`openspec/schemas/`. Pull all three when adopting:

| Schema | Skill (in `agent-standards`) | Subagent (in `agent-standards`) |
| --- | --- | --- |
| [e2e-runbooks](./openspec/schemas/e2e-runbooks/) | [`.agents/skills/e2e-runbooks/`](https://github.com/Lukk17/agent-standards/tree/master/.agents/skills/e2e-runbooks) | [`.claude/agents/e2e-runner.md`, `.opencode/agents/e2e-runner.md`](https://github.com/Lukk17/agent-standards) |

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
Consumers pin to a tag (see [Install](#install) above) so an upstream change can't silently break their installed
schema. Re-run the two-command install with a new tag to upgrade.

A breaking change to a schema's artifact DAG (renamed artifact id, reordered `requires`, removed field) bumps the
major. Adding a new artifact or relaxing a constraint bumps the minor. Pure doc / instruction-prose changes bump the
patch.

OpenSpec itself has no community schema registry yet (tracked in
[Fission-AI/OpenSpec#693](https://github.com/Fission-AI/OpenSpec/issues/693)); pinning by tag is the only safe way to
guard against upstream drift.

## Contributing

Each schema lives at `openspec/schemas/<name>/` with the standard OpenSpec bundle layout (`schema.yaml`, `README.md`,
optional `INTEGRATION.md`, `templates/`). Open a PR against `master`. CI will run the validate + round-trip + lint
workflows; all three must pass before merge.

Validate locally first:

```bash
openspec schema validate <schema-name>
```

## License

MIT, see [LICENSE](./LICENSE).
