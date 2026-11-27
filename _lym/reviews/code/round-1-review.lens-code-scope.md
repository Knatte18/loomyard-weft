```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced every change in the diff to a goal point of the board entry, and the reverse.

**Decisions** (the board entry's `## Goal` points; the record has no `Added after Discussion` entries):
- **G1 (#271):** do not blame the fixer when the new tip is a commit that existed before the session started, such as the task HEAD at session start. Report only commits the session could have added.
- **G2 (#271):** word the halt as "the remote ref moved during the fixer session" unless the pusher is known.
- **G3 (#271):** every push of a task code branch leaves a trace line.
- **G4 (#272):** keep the run dir when the run does not end `fixed`, or name something that survives cleanup, such as the session id.

**Per change:**
- **C1. Ancestry test in the fixer loop → G1.** Covers the start-head `TaskHead` read, the `IsAncestor(remoteAfter, startHead)` test, the log line in place of the pushed end, and the new `fixerOutcome` fields (`remoteRef`, `remoteTip`, `startHead`). Evidence: `internal/landingshed/fixerrun.go:145-148` and `:189-203`.
- **C2. New `Deps.IsAncestor` seam and its wiring → G1.** Evidence: `internal/landingshed/deps.go` (diff 003 lines 12-14) and `internal/loomcli/landingdeps.go:346-352`.
- **C3. Pushed-halt wording and the `startHead..remoteTip` way forward → G2, and the range part to G1.** Evidence: `internal/landingshed/fixerrun.go:419-420`.
- **C4. Separate wording for a deleted remote ref → G2.** At base a deleted ref already ended `pushed` and said "pushed"; this only rewords it. Evidence: `internal/landingshed/fixerrun.go:414-417`.
- **C5. `sessionRef` and the `sessionLog`→`session` rename in the fixer, fix review and Finalize halts → G4.** Evidence: `fixerrun.go:229-246`, `fixreview.go:233` and `:274`, `finalize.go:1144`.
- **C6. The `gitrepo: push landed` trace line in `recordLandedPush`, plus its doc sentence → G3.** Evidence: `internal/gitrepo/push.go:183` and `gitrepo/doc.go` (diff 002 line 9).
- **C7. Seam lists in `battencli` and `loomcli` and the `envkit` fixture → forced by C2.**
- **C8. Comments forced by C1, C3 and C5:** the `TaskRemoteTip` comment in `deps.go`, the `doc.go` paragraph in `landingshed`, and the `fixerEndPushed` and `run` doc comments.
- **C9. Tests forced by C1, C3, C5 and C6:** `fixerrun_test.go` rows and fixture, `finalize_gate_test.go` assertions, and `TestPushBranch_LeavesTraceLine`.
- **C10. Commit 9ae2d2b9 → forced by C1–C8.** It only rewraps comments the earlier commits added.

**Per decision:**
- G1 → C1, C2, C3
- G2 → C3, C4
- G3 → C6
- G4 → C5

**Findings:** none. Every change traces to a goal point or is forced by one that does, and every goal point has a change that builds it. No `Pattern-Departure:` trailer was needed.

Tool calls: none.
