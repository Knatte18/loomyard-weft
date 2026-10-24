# Fixer-report — round 1, code

## Changes

- F1 (LOW), fixed as proposed in `5c98b65e4`: the subtest "a running record whose waiter is gone logs once per episode until it reads live" gained a stretch that makes run-a waiterless again, rewrites it with outcome `done` and reads it, and asserts the recovery Info count rises to 2.
- F2 (LOW), fixed as proposed in `5c98b65e4`: the `!found` branch of `ReadAgentActivity` calls `waiterlessRuns.Recover` before `continue`.
  The same subtest gained a stretch that removes run-a's record and asserts the Info count rises to 3, so the branch is covered.
- F3 (NIT), fixed as proposed in `5c98b65e4`: `waiterlessRecovered` now reads "shuttle: run no longer has a gone waiting process".
- Self-check LOW, fixed in a follow-up commit: `doc.go` now lists a missing record beside "reads live again" and "no longer reads running" as an end of the episode.

`lyx gate test ./internal/shuttleengine` passed after the F1–F3 commit.

## Self-check

- fix-consistency, LOW, `doc.go:426` understates the vanished-record recovery: fixed as proposed.
- fix-proposal, LOW, same `doc.go:426` finding (duplicate): fixed as proposed, merged with the one above.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
