# Lens review

- test-economy, LOW, card 1, the three session-start rows reach one branch: fixed, one source row stands for all three.
- test-economy, LOW, card 5, the wiring subtest only checks for non-nil, and its BurlerFix half cannot fail: fixed, the subtest drives `Forked()` to show it is a plan-gate closure, and the BurlerFix half is dropped.
- criteria-coverage, MEDIUM, card 1, `restarted` is untested through `sessionReadiness` and `Run.startHoldReleased`: fixed, with a `reading: true` session-start row in `TestSend_WaitsForIdleSession` and a session-start row in `TestGate_WaitsForIdleWriter`, plus `SESSIONSTART` parsing in `clearableBoxEngine`.
- criteria-coverage, LOW, nothing pins that the veto logs nothing per poll: rejected. The veto adds no log call, and the release-Warn count rows already pin the one Warn on every release path.
- criteria-coverage, LOW, nothing asserts that the PR-Rework `rework-plan` entry carries no `Forked`: rejected. `resolveGateSpec` never sets `Forked`, so the assertion could never fail (test-economy's own point on card 5).
- criteria-coverage, LOW, the claim that the flag-less mode ignores `lens-review.md` is pinned only by the file being absent: fixed, with the parity row "LensReviewEmptyIgnoredWithoutTheFlag".
- forced-edits, MEDIUM, card 3, `runExitAuditCrossCheck` hard-errors on a nil audit for an old-record Master: rejected. `internal/websterengine/runlevel.go:702` spawns Master fresh through `deps.Starter.StartMaster(spec, gate)`, and no Attach call exists in `internal/websterengine`, so a Master's record always comes from its own spawn. Card 3 now says so.
- forced-edits, LOW, card 3, the `//testtiming:keep` reason of `TestRunner_Start_HappyPath` is not updated: fixed.
- forced-edits, LOW, card 3, the run-started log reads `spec.ForkSubagents`: rejected. At spawn the value equals the record.
- forced-edits, LOW, card 6, the rubric's overlay wording: rejected. The reviewer itself says it stays true.
- row-reachability, LOW, card 3, the new attach rows must set `gated: true`: fixed.
- row-reachability, LOW, card 6, the Rename mechanic conflicts with the table rewrite: fixed, the rename lands first as a surgical edit and the table rewrite is a separate edit after it.
- row-reachability, LOW, card 7, the wanted fragment is ambiguous: fixed, it now ends with "until it passes".
