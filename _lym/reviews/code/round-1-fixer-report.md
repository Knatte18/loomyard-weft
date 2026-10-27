# Burler code round 1 — fixer-report

## Changes

- **F1** (LOW), fixed with my own change.
  The proposed `first` derivation would still log a spurious line: the start heartbeat's failed read leaves `loggedRuns` nil, and the next tick's `logRunChanges` logs every run as first seen before any later heartbeat primes it.
  Instead `heartbeat` sets a `primeOwed` flag when its read fails with the memory unprimed, and the next `logRunChanges` primes the memory silently and clears it (`internal/dobbyengine/passes.go`, `loop.go`).
  `heartbeat` derives priming from `loggedRuns == nil` rather than `lastHeartbeat.IsZero()`, and clears the flag when it primes.
  A flag was needed over priming whenever the memory is nil, because the direct-tick tests of `TestLog_ChangesOnly` rely on a nil memory logging first-seen lines.
  Added a `TestRun_Heartbeat` row where a re-exec's start `Runs` read fails: one unreadable heartbeat, no `run a:` line.
- **F2** (LOW), fixed via the review's stated alternative: `SeedAdoptGate` and `gateslot/doc.go` now say a template-equal `adopt` cannot be pinned while `finalize` is tuned.
  The keep-the-entry change would need a per-name exclusion in `yamlengine.DropTemplateEqualNames`, which takes whole paths only; the review itself names the doc correction as acceptable.
  The Open-risks sentence quoted by the review lives in the board entry, outside this repo; no doc in the repo claims the way back.
- **F3** (NIT), fixed as proposed in `internal/adoptcli/adopt.go:1`, and, from the self-check, the same stale phrase in `adopt.go:98`, `adopt_test.go:3`, `deps.go:192`, `deps.go:195`, `deps.go:200` and the error string at `deps.go:203`.
- **F4** (NIT), fixed as proposed in `internal/dobbyengine/loop.go`.

Commits: one per finding, plus a follow-up commit for the self-check changes.
`lym gate test ./internal/dobbyengine ./internal/adoptcli ./internal/gateslot` is green.

## Self-check

- fix-code-correctness, LOW: `primeOwed` not cleared when a heartbeat primes — fixed as proposed.
- fix-consistency, LOW: stale "test suite" wording beside F3 — fixed as proposed (merged with the affected-set finding).
- fix-proposal: no findings.
- fix-affected-set, LOW: `adopt.go:98` and `deps.go:192`, `203` stale — fixed as proposed; the F2 wider doc sites it left unchecked I grepped (`docs`, `PATTERN.md`, `pattern`) and found no claim that a hub can pin the template `adopt`.

## Disputed

None.

## Deferred

None.
