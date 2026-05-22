# {capability}: e2e capability test proposal

## Why

<!-- 50-1000 chars. What behaviour needs an end-to-end assertion? What gap in
the current test suite does this fill? Anchor to a real risk or a recent
incident if you have one. -->

## What this will verify

<!-- Bullet list. Observable behaviours only: HTTP status codes, response-body
content, persisted state in backing services. No log substrings. -->

-
-

## Setup cost class

<!-- Pick one. Lower numbers run first in a sweep. Determines the {N} prefix
in the spec filename. -->

- [ ] 1: No state to reset, no fixtures, hits a single endpoint or MCP tool.
- [ ] 2: Single fixture upload OR vision-capable model OR PDF parsing.
- [ ] 3: Single-service reset (e.g. Redis only).
- [ ] 4: Multi-service reset (DB + Redis + Qdrant + MinIO).
- [ ] 5: Seeded state + observation of an async background process.

## Fixtures needed

<!-- List canary files this test will read. Each must contain content unique
enough that a passing test proves retrieval, not memorisation. Leave empty
if none. -->

-

## API client invocation

<!-- Name the exact request file. Bruno YAML, hurl file, REST Client .http,
or curl invocation. -->

-

## Number assignment

<!-- Pick the lowest unused {N} that matches the setup cost class above. -->

N:

## Next steps

After this proposal is approved:

- Generate `e2e/testing/{N}-{capability}-test.md` from the `test-spec` artifact.
- Generate `e2e/testing/{N}-{capability}-tasks.template.md` from the `tasks-template` artifact.
- Each execution generates a `run` record under `e2e/testing/runs/`.
