# Changelog

All notable changes to this repo and the schemas it ships.

Entries under `## [Unreleased]` describe changes since the last tagged release. Before triggering the release
workflow, rename `## [Unreleased]` to `## [vX.Y.Z] - YYYY-MM-DD` and open a fresh `## [Unreleased]` block at the top.

## [Unreleased]

## [v0.1.0] - 2026-05-28

### Added

- `e2e-runbooks` schema: artifact pipeline of **proposal → test-spec → tasks-template → run**. Each capability test
  gets an immutable spec, an immutable tasks template, and one timestamped run record per execution. Assertions are
  behaviour-only (HTTP status, response body content, persisted state in backing services) — never log substrings.
- One-time scaffold step that creates the consumer project's `e2e/` directory tree on the first proposal in a
  project, and appends a `.gitignore` snippet so run records stay local while their `README.md` stays tracked.
- Dual-write of every artifact into both the OpenSpec change directory (audit trail) and the `e2e/testing/` working
  tree (the asset future runs consume).
- `Concurrency` section in proposal and test-spec templates with three fields: `Mutates:` (resources the test writes
  to, deletes from, or invalidates), `Conflicts with:` (other tests this one cannot run alongside even when
  `Mutates:` lists do not overlap), `Serial:` (literal `true` or `false` — `true` means the test must run alone).
- `INTEGRATION.md` sections explaining how the main agent schedules a sweep. It asks the user how many runners to
  spawn in parallel, then groups tests by what they touch so two specs with non-overlapping `Mutates:` lists can run
  at the same time, overlapping specs run one after the other, and `Serial: true` specs run alone.
- Companion `e2e-runner` subagent in [Lukk17/agent-standards](https://github.com/Lukk17/agent-standards) at
  `.claude/agents/e2e-runner.md` and `.opencode/agents/e2e-runner.md` — the recommended delegate for driving a
  sweep. Owns the runner contract (Start/End UTC, Duration, tokens, Verdict) and the run-record dual-write into
  `e2e/testing/runs/`.
- CI pipeline (`.github/workflows/validate.yml`) that validates every schema, round-trips a `new change` for each,
  and checks template structure (section count, Concurrency fields present) on every push and PR. Manual release
  workflow (`.github/workflows/release.yml`) that publishes a GitHub Release with notes extracted from this file.
