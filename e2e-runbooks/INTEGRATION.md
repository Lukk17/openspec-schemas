# Integration notes for `e2e-runbooks`

## Pairing with the agent-standards skill

The skill at `.agents/skills/e2e-runbooks/SKILL.md` in
[Lukk17/agent-standards](https://github.com/Lukk17/agent-standards) is the methodology source of truth. The schema
templates are the operational shape. Keep them consistent on edit: any change to one should be reflected in the other.

Both are independently installable. Skill alone works in projects without OpenSpec. Schema alone works in projects
without agent-standards. Together they reinforce each other.

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
