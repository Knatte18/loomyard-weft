## Goal

````markdown
Imported from [issue #149](https://github.com/Knatte18/loomyard/issues/149).

## What happened

The loom run halted `failed` at `Darn-Write` with anomaly `producer-hard-failure`.
The error chain ended in the gate fixer's prompt build:

```
shedadapters: Darn-Write (seats): seat run: seatengine: chair wait: shuttle: gate: gatefixer: build the fixer's launch: gatefixer: fill the gatefix-body-darn prompt: stencil: include "loom-fragment-loosened-check" has no entry in the includes
```

## Why it is abnormal

The `gatefix-body-darn` prompt references the include `loom-fragment-loosened-check`, but the gate fixer's launch supplies no entry for it.
The failure is deterministic prompt wiring, not a transient fault, and it kills the whole run at the point where the gate fixer is first needed.

## What to do

Add the `loom-fragment-loosened-check` entry to the includes the gatefixer passes when it fills `gatefix-body-darn`, or drop the include from the prompt.
Add a test that fills every gatefix body prompt with the production includes map, so a missing include fails in CI rather than mid-run.

Provenance: task `redundant-check-fixes`; note `loom-halt-2.md`.
````

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-19): Recast onto the version-skew cause

Decision: The missing-include wiring is already fixed on main (72c3e192) and the asked-for test exists (loomcli TestFixerSpec, gatecli TestRunWithFixer), so make no change for that part. Instead fix the real cause found: a running lym binary that renders board stencil copies written by a newer binary fails mid-run with a bare template error. Make the stencil read detect that skew (a board stencil newer than the running binary's shipped set, ChangedPins or an equivalent version mark) and fail early with an error naming the skew and the way forward (restart the run's driver on the current binary), or fall back to the binary's own shipped stencil where that is safe. Keep it small; add a test for the skew case.

Rationale: Dropping the run leaves the cause that killed redundant-check-fixes in place; the same skew can hit any long-running driver after update-plugins refreshes the board stencils.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - edited-copy-from-newer-writer-bypasses-detection (class design, severity MEDIUM, open in rounds 1)
  - embedded-bytes-fallback-against-stencil-ownership (class design, severity BLOCKING, open in rounds 1)
  - skew-tested-only-through-unexported-helper (class design, severity BLOCKING, open in rounds 1)
  - writer-ordering-disagrees-with-refresh-ordering (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
