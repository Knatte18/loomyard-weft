## Fixer-report, round 1

Every finding was fixed as proposed. Tier 0 passed for `./internal/consoletitle`, `./internal/reedcli` and `./internal/loomcli`.

## Changes

- F1 (MEDIUM), `d11e5d14`: the `Long` text of `lym reed window` in `internal/reedcli/window.go` now gives the task form `<T#> [<hub shortname>:<P#>] <slug>`, the beam form `ORCH [<hub shortname>:<P#>] <slug>`, and the prime's `ORCH [<hub shortname>:P0] root`. The sentence that a missing part is left out stays.
- F2 (LOW), `48264feb`: the `statusWindowTitle` doc comment in `internal/loomcli/sharedbootstrap.go` gives the example `T4 [ly:P3] tapestry` and says no prime name is passed, so the prime is never titled as the orch.
- F3 (NIT), `340a2497`: `formedTitle` in `internal/reedcli/window_test.go` is now `"T7 [ly:P3] task"`.
- F4 (NIT), `a411def4`: `Title` in `internal/consoletitle/consoletitle.go` appends `name` only when it is non-empty. A `NoName` row in `TestTitle` pins `Title("ORCH", "ly", "P0", "")` as `ORCH [ly:P0]`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
