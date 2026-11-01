# Fixer-report, round 1

## Changes

- F1 (MEDIUM), fixed as proposed, commit `6b03dcdd`: `TestRunNotice_Conditions` gains the row `PausedTimeDoesNotCountTowardTheStartGrace`.
  It fails with `pause.Intervals` replaced by `nil` at `internal/dobbyengine/runnotice.go:251` and passes with the code as it is.
- F2 (LOW), fixed as proposed, not committed because `_lym/landing/summary.md` is under `_lym`: the summary now says the grace applies when dobby sees the child enter running, and that a halt and resume between two probe ticks gets no grace.
- F3 (LOW), fixed as proposed, commit `439fce75`: `internal/battenshed/doc.go` names the loom bootstrap lock and the minute of awake time from the running episode's since.
- F4 (NIT), fixed as proposed, commit `6407363c`: `internal/dobbyengine/doc.go` says the takeover's driver-dead notice also waits the start grace from the first sight.

`lym gate test ./internal/dobbyengine` passes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
