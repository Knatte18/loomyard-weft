```yaml
lens: code-tests
outcome: reported
```

**Per behaviour — the test that pins it**

1. `driveNotices` sends the run-id line at a watched blocked stop an agent judged; nothing at watched unjudged blocked; nothing at watched judged awaiting — `TestDriveNotices` three new rows (`internal/sheddriver/drivehandback_test.go:45-47`).
2. `settleStop` sets `d.judged = true` before the tackler/arbiter call (`drive.go:314`) — `TestGoDriver_Settle` two watched rows (`drive_test.go:425-443`, `wantNotifies: 1`; without the flag a watched hand-back sends nothing).
3. `handBack` copies `d.judged` into `h.Judged` (`drive.go:1002`) — same two `TestGoDriver_Settle` rows.
4. `run` resets `d.judged` at a fresh stop (`attempt == ""` branch, `drive.go:250`) — **none** (mutation survivor confirms).
5. `overviewFrameEditedFindings` reports an overview edited outside its Card Index after a begun batch — `TestNewPlanGate/OverviewEditedOutsideTheCardIndexAfterABegunBatchFails` (`gates_test.go:733`).
6. Reports nothing for unedited overview / no begun batch / no frame — matching rows (`gates_test.go:734-736`).
7. An edit confined to the Card Index after a begun batch passes (the gate agrees with `Rebaseline`, which accepts it) — **none**.
8. Unreadable overview is a returned error from `ValidatePlan`; nil record reports nothing — **none** for either.
9. `CommitWithTrailers` puts caller trailers after `Warp-SHA` on the weft only — `TestCommitWithTrailers/TrailersEndTheWeftMessageOnly` (`commit_integration_test.go:226-249`).
10. `CommitWithTrailers` refuses an invalid key / multi-line value before committing anything — `TestCommitWithTrailers/AnInvalidTrailerCommitsNothing` (`commit_integration_test.go:251-271`).
11. `commitWeftLocked` appends trailers on an unborn warp too ("on a born and an unborn warp alike", `recordsgit.go:221`) — **none**.
12. `Commit` delegates to `commit` unchanged — existing `Commit` tests (reach report, 78+).
13. `fabricSync` passes trailers through; reset to batch-start carries `Run-Transition: <worktree>` and `Warp-SHA: <new tip>` — `TestResetCmd_RecordsTheRunTransitionAtTheNewTip` (`reset_test.go:161-189`).
14. `wireHub` sets `c.runID = shedrun.ResolveRunID(loc, shedrun.SelfRunID)` (`wiring.go:217`) — **none**; the reset fixture injects `runID: loc.WorktreeName` directly (`reset_test.go:69`), and no webstercli wiring test names `runID`.
15. Reset's other paths (to start with archive; archive-only) carry the trailer — **none**.
16. `wireStandalone` sets `c.runID = ""` — not worth a row (standalone has nil opener, `fabricSync` never commits).

**Findings**

- **MEDIUM / design** — `internal/sheddriver/drive.go:250` — The judged-flag reset at a fresh stop is untested.
  Evidence: mutation report survivor at `drive.go:250`; every `TestGoDriver_Settle` row that hands back judged either judges the stop in hand or never judged. No row has a watched run where an agent judges stop A, the run resteps with no attempt, and a later blocked stop B hands back by rule — which with the reset dropped would send an extra run-id notice.
  Fix: add that `TestGoDriver_Settle` row (watched, agent judges A then restep, B hands back by rule) with `wantNotifies: 0`.

- **MEDIUM / design** — `internal/loomshed/gates_test.go:733-736` — No row edits only the Card Index after a begun batch and expects the gate to pass.
  Evidence: an implementation that hashed the whole overview instead of its frame would pass all current rows yet refuse the edit `Rebaseline` accepts (`rebaseline.go` doc: "A change to 00-overview.md is accepted only when it is confined to the Card Index"); the decision record asks that the two agree in both directions.
  Fix: add a row that changes a Card Index line after a begun batch and expects `Passed: true`.

- **MEDIUM / design** — `internal/webstercli/wiring.go:217` — Hub wiring filling `c.runID` is untested.
  Evidence: `reset_test.go:69` injects `runID: loc.WorktreeName` directly; grep finds no other webstercli test reading `runID`. Dropping the `wireHub` line would lose the trailer in production with every test still green.
  Fix: have the reset test wire through `wireHub`, or assert `c.runID` in an existing `TestWire_*` hub test.

- **LOW / design** — `internal/loomshed/gates.go:400-403` — Unreadable overview → returned error from `ValidatePlan` is untested.
  Fix: add a row making the overview unreadable via `permkit.Chmod` and expect the gate to return an error.

- **LOW / design** — `internal/fabricengine/recordsgit.go:236-238` — Trailers on an unborn warp are untested though the doc claims it.
  Fix: add a `TestCommitWithTrailers` subtest on an unborn-warp fixture asserting the message ends with the caller's trailer and holds no `Warp-SHA`.

- **LOW / design** — `internal/webstercli/reset.go:177-185` — Only the batch-start reset path asserts `Run-Transition`; reset to start (archive) and archive-only commit through the same `fabricSync` call untested.
  Fix: assert the trailer in the corresponding `TestResetCmd` steps, or add a start-reset row to the new test.

- **NIT / consistency** — `internal/sheddriver/drivehandback_test.go` `TestDriveNotices` — Listed contained by `TestGoDriver_Settle`, but its assertions differ: it pins the exact notice text (`generic`) and the no-agent-judged / judged-awaiting rows, which `TestGoDriver_Settle` checks only as `wantNotifies` counts.
  Fix: keep it and mark it `//testtiming:keep exact hand-back notice text and the judged/state matrix` directly above its `func Test…` line.

**Tool calls**
1. Grep for `runID` in webstercli tests and the loomshed overview fixture — context lacked whether any wiring test asserts `c.runID` and whether the plan fixture's overview holds framing outside the Card Index.
