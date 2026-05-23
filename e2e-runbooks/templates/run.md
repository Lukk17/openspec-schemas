# {capability}: run record

Spec: [`../{N}-{capability}-test.md`](../{N}-{capability}-test.md)

<!--
This file is created by copying `../{N}-{capability}-tasks.template.md` to
this location and filling it in. Filename uses ISO-8601 UTC with colons
replaced by hyphens: `runs/2026-05-12T17-23-36_{N}-{capability}-tasks.md`.

Runner contract:

1. Record `Start (UTC)` BEFORE running any prerequisite check. It is the
   first action.
2. Execute each task in spec order. Tick the box on success; record what
   went wrong on failure.
3. After the Verdict line is decided, record `End (UTC)`.
4. Compute `Duration = End - Start` as HH:MM:SS. Wall-clock for the whole
   test (prereqs + reset + run + verify), NOT just the API client call.
5. Fill `Input tokens` and `Output tokens` with the best estimate of LLM
   API tokens consumed. Leave blank if exact numbers aren't available. Do
   not invent.
6. Write the Result summary paragraph and the Verdict (PASS or FAIL).
7. Log anything done outside the spec under "Additional tasks I did".

This template's body is identical to tasks-template.md; both shapes drift
together. Edit the matching tasks-template in the same change.
-->

## Tasks

### Prerequisites

- [ ]

### Reset state

- [ ]

### Run

- [ ]

### Expected

- [ ]

### Verdict

- [ ] Verdict: PASS / FAIL (delete the wrong one)

## Result summary

<!-- One-paragraph narrative anchored to the Expected assertions. Write it
above this line; fill the metrics below. -->

Input tokens:

Output tokens:

Start (UTC):

End (UTC):

Duration:

---

## Additional tasks I did
