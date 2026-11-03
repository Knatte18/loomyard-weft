```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the F1, F3, F5 and F6 fixes for sites the fixer missed, using the affected-set lens.

**Findings**

**MEDIUM — F1 left the battencli seam list without `Landing.DriftVerdict`**
- Changed text: `internal/testkit/envkit/envkit.go`, the `Landing.DriftVerdict` entry removed from `nilLegal`.
- `internal/battencli/wire_seams_test.go:207` runs `envkit.NilSeams(c.env)` against its own `intentionallyNil` map. That map lists every `Landing.*` seam that batten leaves nil, with the reason "loom-only seam, batten lands no branch" (for example `Landing.EnginesRun` at line 81).
- The fix added `Landing.DriftVerdict` only to `internal/loomcli/wiring_seams_test.go:22`. `nilLegal` used to skip it, so battencli never needed an entry.
- Batten leaves `Landing` unfilled like its other `Landing.*` seams, so I expect `NilSeams` to report `Landing.DriftVerdict` as nil there. I did not run battencli's test.
- The fixer's gate ran `./internal/driftengine`, `./internal/landingshed`, `./internal/loomcli` and `./internal/testkit/envkit` at tier 0 and `./internal/loomcli` at tier 1. `./internal/battencli` was never in the set.
- The other `NilSeams` callers are not affected. `internal/loomcli/wiring_beam_test.go` only flags `BeamDescribe*` and `Pawl*` paths. `internal/loomcli/wiring_darn_test.go` only flags a fixed list of darn seams.

No finding on the other affected sets:
- **F6:** every `requireCompareRecorded` caller (`verify_test.go:369`, `403`, `423`) uses the new signature.
- **F1 builders:** the two local `Deps` builders in landingshed (`prreview_test.go:590`, `publish_test.go:67`) leave `DriftVerdict` nil on purpose, and nil is still valid there.

Calls not made: I did not look for other `Deps.PullRequest` refresh sites for F5. The search for `openPullRequest` callers is the only one I skipped.

**Calls made**
1. A grep for `requireCompareRecorded`/`BaselineHead` in `internal/driftengine` tests, to check the F6 callers. My context had only the edited lines.
2. A grep for `Landing.DriftVerdict`/`nilLegal`/`NilSeams(` across the repo, to find other lists and callers. My context had only the loomcli ones.
3. A grep for `EnginesRun:` to find other `Deps` builders. My context did not show them.
4. A grep and two reads of `internal/battencli/wire_seams_test.go`, `wiring_beam_test.go` and `wiring_darn_test.go`, to see what the other `NilSeams` callers assert. My context lacked these files.
