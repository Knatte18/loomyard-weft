## Fixer-report, round 1

### Changes

- **F1** (fixed as proposed, commit `453639b54`): `atOrBelow` in `internal/battencli/overview.go` tracks visited slugs and returns an error naming the repeated slug, so `orchsAtOrBelow` leaves that orch out and logs it at Debug.
  `TestOrchsAtOrBelow` gained `loop-x`/`loop-y` (each other's parent) and the row `ACyclicParentChainIsLeftOut`; the prime's row lists both.
- **F4** (fixed as proposed, same commit as F1 because both edit `overview.go` and `overview_test.go`): dropped "its own first" from the `orchs` field comment and the `orchLines` comment.
- **F2** (fixed as proposed, commit `0d5014627`): the orch row of `docs/overview.md` says `start` ensures the `runs` strand, which runs `lyx batten overview` as the beam's window named `beam: prime` or `beam: <slug>`, and names the beam window first among the session's windows.
- **F3** (fixed as proposed, commit `c195823ec`): `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` plants one run (`run-x`) under the prime's dobby and asserts its line with state and stage cells, plus the beam's `no runs driven` line.

`lyx gate test --tier 1 ./internal/beamcli ./internal/battencli` and `lyx gate test ./internal/battencli` are green.

### Self-check

- fix-consistency, LOW: `internal/loomcli/bootstrap.go`, `internal/loomcli/doc.go` and `internal/orchcli/runs.go` still say "the session's overview window"; not fixed, premise false — the loomcli text names the loom checklist strand, and the `runs.go` title is a parameter, so neither is stale (reviewer's own note: "not wrong").
- fix-affected-set, LOW: `internal/orchengine/doc.go:13` already agrees with the new row; no change needed.
- fix-affected-set, LOW: the same loomcli and `runs.go` comments; same disposition as above.
- fix-affected-set: F3 set left unchecked by the fork; no other `DobbyLines` caller is in the review's scope.
- fix-code-correctness: no findings.
- fix-proposal: no findings.

### Disputed

None.

### Deferred

None.
