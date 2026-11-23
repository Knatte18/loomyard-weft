```yaml
lens: code-scope
outcome: reported
```

**Decisions** (answer key: decision record alone; no `Added after Discussion` entries; the record's other sections are empty):
- G1 — A Webster auto-rebaseline block sends no stop notice to the run's parent; under autonomy the parent must hear of every stop.
- G2 — Plan rows edit `00-overview.md` outside its Card Index after a begun batch; Plan's gate accepts it, Webster's rebaseline refuses it. Either the rows keep to the Card Index or the gate refuses, so the two agree.
- G3 — `webster reset` (and `goto`) should commit a transition naming the new tip; also "none of these stops sent the parent a notice".

**Per change**
- C1 — sheddriver judged-blocked notice (`judged` field + set/reset in `drive.go`, `driveHandBack.Judged`, `driveNotices` condition, doc.go/drivehandback.go comments) → G1 (and G3's last sentence, since the dead-batch-03 Webster block also goes to the arbiter). Tests in `drive_test.go`/`drivehandback_test.go` forced by C1.
- C2 — `overview-frame-edited` finding in `ValidatePlan` (`gates.go` code+doc, `loomshed/doc.go`) → G2 (gate route is one of the two the entry allows). `TestNewPlanGate` rows forced by C2.
- C3 — reset's records commit carries `Run-Transition` (`CommitWithTrailers`/`commit`, `commitBothSides`/`commitWeftLocked` trailer threading, `fabricSync` variadic trailers, `websterCLI.runID` + hub/standalone wiring, `reset.go` transition) → G3 (reset half).
- Forced by C3: `export_test.go` signature update; `TransitionTrailerKey` doc in `trailer.go` (its writer list goes stale); `doc.go` lines in fabricengine and webstercli; `reset.go`/`sync.go` file headers; `TestCommitWithTrailers`, `TestResetCmd_RecordsTheRunTransitionAtTheNewTip` and the fixture's `runID`.
- No change traces to none.

**Per decision**
- G1: C1.
- G2: C2.
- G3: C3 (reset half); the `goto` half — no change; the parent-notice sentence — C1.

**Findings**

1. **MEDIUM — scope** — `internal/webstercli/reset.go:177-183` (transition added to reset only)
   Summary: G3 asks that "`webster reset` (and `goto`) should commit a transition naming the new tip"; the diff builds only the reset half and leaves `lym loom goto --to Webster` unchanged. The record holds no `Added after Discussion` entry narrowing G3; the omission rests only on the writer's own claim in `_lym/landing/summary.md` ("Design decisions: Only `webster reset` writes the transition"), which is not named as a parent correction (the two parent-confirmed calls it names are for defects 1 and 2).
   Evidence: `_lym/discussion/decision-record.md` Goal, defect 3 ("`webster reset` (and `goto`) should commit a transition naming the new tip"); `_lym/landing/summary.md` Design decisions, third bullet.
   Fix direction: either have goto's records commit carry `Run-Transition`, or have the run's parent record the narrowing (`lym loom decision add`) so scope matches the amended entry.

Tool calls:
1. Grep in `internal/loomcli` for goto's handler/records commit — to judge the unbuilt `goto` half; context had no goto code (found only guard/driver wiring, not goto's commit path).
