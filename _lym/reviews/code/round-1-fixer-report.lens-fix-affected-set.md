```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind the set-premise findings F1, F2 and F3, and check the fix covers every member.

**Findings**

1. LOW — `internal/loomcli/wiring.go:838` (the `driftDeps(..., decisionRecordPath: loomengine.DiscussionDecisionRecord(location))` call, untouched by the fix and not named by the review).
   - It is the one wiring site F3's search ("a site that still reads `_lym/discussion` for a beam") turns up that the new assertions do not cover.
   - Every other site is either already routed through `runRecordOf` and asserted by the new `wiring_beam_test.go` rows, or belongs to darn, discussion or tapestry only (`darnwiring.go`, `plan.go`, `discussiontable.go`, `twill.go`).
   - The pawl rows list in `wiring_beam_test.go` has no Drift row, so this path is unreachable for a pawl run today. This is the reviewer's own "behaviour equivalent today" reasoning for `landingdeps.go`/`arbiterprompt.go`.
   - Nothing pins that, so a Drift row added to the pawl recipe would read the discussion record of a beam.
   - The review did not name this site, so the miss is the reviewer's. I'm reporting it only because it is the same key.

**Checked, no findings**
- F1: `decisionParent` and `decisionReceivers` each have exactly one production caller (`decision.go:869`, `:907`) and the new tests. Both callers were updated for the changed signatures.
- F2: `withProjectDirective` has two production callers (`wiring.go:682` for the discussion table, `darnwiring.go:235` for the darn table). The new table test exercises the function itself, so both callers are covered.
- F3: sites that use `runRecordOf` or the record struct (`wiring.go:299-309`, `:370`, `:523`, `arbiter.go:123`, `landingdeps.go:255`, `beamwiring.go` via `wiring.go:887`) are all asserted by the extended pawl rows. This includes the arbiter prompt, `RecordWrite`, `PatternRecordPath`, `DecisionRecordPath`/`SupportLogPath` and `BeamDescribe.ProjectDecisions`.
- Left unchecked: burler's `PatternRecordPath` at `wiring.go:528`. The fix does not assert it and the search did not reach it; no change set item depends on it.

**Tool calls**
1. `grep` for `decisionParent(`/`decisionReceivers(` across the repo: my context did not hold the post-refactor caller set.
2. `grep` for the `Discussion*` path accessors in non-test code: F3's set is every site reading the discussion path, which my context lacked.
3. `sed` over `wiring.go:822-842`, `subjects.go:40-52`, `peers.go:215-228` and `wiring.go:325-332`: to see whether the remaining discussion-path sites could be reached for a beam.
