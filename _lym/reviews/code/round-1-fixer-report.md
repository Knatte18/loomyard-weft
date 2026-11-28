---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

# Code review round 1 — fixer report

## Changes

- F1 (`4e3e3487f`): the `weaverengine` package doc names every `weaver.yaml` key in full.
  The one-line key list would have passed the line limit with full names, so a later commit replaced it with a lead-in, and the per-key sentences name each key.
- F4 (`4e3e3487f`, same commit as F1): the `AttemptTimeoutMin` comment and the `template.yaml` comment count the bound from the spawn, or from an attach for a strand a crashed call left.
  The package doc's `weaver.attempt_timeout_min` sentence gained the same fact, which put `doc.go` in this group.
- F2 (`520d06c38`): the scripted verify advances the run's `fakeClock` by one minute and the fake fixer's wait by five, through `Sleep`, its one way to advance.
  The fixer-clears and exhausted-ladder tests assert each span's kind, start and end.
- F3 (`cbf8dc378`): `runFixer` records the rework span, starting once `SpawnOrAttach` returns without error.
  It appends by `defer`, so a spawned fixer keeps its span when a later state save or wait fails.
  No test was added, since the finding is not a coverage gap, so no test covers the failed-spawn path.
- F5 (`b51cf7f37`): the attached-fixer resume row asserts the verify's gate span alone.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: one finding, LOW, the F1 rewording grouped `weaver.ladder` under batch attempts only, though verify fixers climb it too. Fixed as proposed (`df86307ef`).
- fix-affected-set: no findings.

A later check found the F4 rewording of the `AttemptTimeoutMin` comment breaking a sentence across lines.
`22cd05584` rewrites it as one sentence per line.

The test gate over `./internal/weaverengine` and `./internal/shedadapters` passed on the final code, that commit included.

## Disputed

## Deferred

## Rulings
