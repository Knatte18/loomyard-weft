---
dispositions:
  - id: F1
    disposition: fixed
---

## Changes

- F1: `_lym/landing/summary.md`, `## Merge danger`: the Door line is now `one-way`, followed by one sentence saying `runplan.VerifyGateReport` no longer writes the optional `triage` key of `verify-gate.yaml`, which lym reads back in a later process, and that tolerant decoding keeps old and new records readable both ways.
  Code unchanged, as the finding's `fix` key says.
- No commit: the only file changed is under `_lym`, where this round runs no git command.

## Self-check

- fix-code-correctness: no findings (the fix changed no code).
- fix-consistency: LOW, the added sentence sits between the Door and Blast radius lines; the spec does not require adjacency and the parser matches by line prefix. Disposition: disputed in effect, nothing to change; the fork found no concrete breakage and fix-proposal confirmed the parser at `internal/summaryparser/description.go:170` reads both lines.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Gate findings

The gate's reach notes name tests that reach the changed functions, and one gap: no mapped test reaches `internal/gatecli.execution.printReport`.
The change to `printReport` only removed its `e.addTriage(inHub)` call; it still forwards the failed runs to `printFailureReport` unchanged.
`TestRenderFailureReport` pins the rendering, and the subprocess integration tests of `lym gate test` run `printReport`, which the coverage maps cannot see.
A unit test of `printReport` would need a full `execution` with a git-backed `touchedFiles`, and would pin no behavior the removal altered.
No test added; the turn ends with the code unchanged.

## Disputed

None.

## Deferred

None.

## Rulings

None.
