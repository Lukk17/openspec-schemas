# Integration notes for `e2e-runbooks`

## Lifecycle: change directory plus `e2e/` tree

OpenSpec assumes every artifact lives under `openspec/changes/<change-id>/` during the proposal cycle and that
`archive` promotes finished content to `openspec/specs/`. This schema's artifacts have a different final home —
`e2e/testing/` for spec + tasks-template pairs, `e2e/testing/runs/` for execution records — so the schema instructs a
**dual write** rather than relying on archive.

```text
openspec/changes/<change-id>/                       e2e/                     consumer project root
├── proposal.md                                     ├── README.md
├── test-spec.md           ─── identical copy ──►   ├── testing/
├── tasks-template.md      ─── identical copy ──►   │   ├── {N}-{capability}-test.md
└── run.md                 ─── timestamped copy ─►  │   ├── {N}-{capability}-tasks.template.md
                                                    │   └── runs/
                                                    │       └── <UTC-ts>_{N}-{capability}-tasks.md  (gitignored)
                                                    └── fixtures/
```

Why two copies:

- The **change-dir copy** is the proposal's audit trail. It pins the artifact to the change that introduced (or
  modified) it, and travels with the change through review and archive.
- The **`e2e/testing/` copy** is the working asset that runs consume. It is what `e2e/testing/runs/` records point
  back to. Sweeps walk `e2e/testing/*-test.md`, not the change directories.

Drift between the two copies is the most likely defect class. When you modify a spec, do the same edit in the
change-dir copy in the same commit — never let them diverge silently.

Proposal stays in the change directory only. There is no `e2e/proposals/` mirror.

## Pairing with the agent-standards skill and subagent

The methodology lives in three places that ship together via
[Lukk17/agent-standards](https://github.com/Lukk17/agent-standards):

- `.agents/skills/e2e-runbooks/SKILL.md` is the methodology source of truth.
- `.claude/agents/e2e-runner.md` and `.opencode/agents/e2e-runner.md` are the runner subagent — the recommended
  delegate when driving a sweep. It enforces the runner contract (Start/End UTC, Duration, tokens, Verdict) and the
  dual-write into `e2e/testing/runs/`.
- This schema is the operational shape for OpenSpec users.

When you edit one, audit the others in the same change. Drift between methodology, subagent behaviour, and schema
templates is the most likely defect class.

Each piece is independently installable. Skill alone works without OpenSpec. Schema alone works without
agent-standards. Together they reinforce each other.

## API client choice

The schema does not pin a client. Pick one per project and use it consistently across all tests. Recommended defaults:

| Client | When | Notes |
| --- | --- | --- |
| [Bruno CLI](https://www.usebruno.com/) | REST APIs, multi-step flows, mature collections | Single source of API truth; request file = test fixture. Pin requests to `X-User-Id` or similar per-project header. |
| [Hurl](https://hurl.dev/) | Plain-text HTTP, assertions embedded in the request file | Lighter than Bruno; assertions live next to the request. |
| `curl` | Single-shot health checks, MCP HTTP probes | Use for prereq probes inside the spec; not for the main Run step unless the test is trivially small. |
| [httpie](https://httpie.io/) | Interactive debugging | Avoid for stored test definitions; it's an ergonomic CLI, not a fixture format. |
| VS Code REST Client | Non-CLI workflows | `.http` files; OK for human-only execution paths. |

For MCP tool tests, drive through the agent (not the MCP server directly) so you assert end-to-end discovery and
routing, not just MCP-protocol mechanics.

## CI integration sketch

```yaml
# .github/workflows/e2e-sweep.yml
name: e2e-sweep
on:
  workflow_dispatch:
  schedule:
    - cron: "0 6 * * *"   # daily sweep
jobs:
  sweep:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker compose up -d --build
      - run: npm install -g @usebruno/cli   # or hurl / etc.
      - run: |
          for spec in e2e/testing/*-test.md; do
            N="${spec##*/}"
            # Open the matching run record from the template, drive through the
            # API client, tick the boxes from CI output, commit the record
            # under runs/.
          done
```

The runner contract is the same whether human or AI is executing. CI just automates the runner role.

## Recording runs in git

Defaults assume `e2e/testing/runs/` is gitignored — runs are ephemeral, useful as audit when reviewed, noise when
committed. For an audit trail, commit under `runs/<YYYY-MM>/` subfolders so a sweep's records cluster together.
