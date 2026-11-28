---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Merge danger
    summary: The two-way door is called on a diff that adds an outward records push on the sync-failure path, and the blast radius names only webster although that push now writes a failed-push record that holds the run.
    fix: "In `_lym/landing/summary.md` under `## Merge danger`, keep or change the door, but name the outward change. Write that the reset now pushes the records side to origin when its sync fails after the records commit landed, and that a failed push there writes a failed-push record. Widen `**Blast radius:**` from `webster` to webster plus the run lifecycle, because `ReadRun` reads that record into `PushOwed` and the act rule holds the run on it."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/webstercli/reset.go:200
    summary: On the sync-failure path a hand-over shows up only in `push_error`. The refusal's `error` keeps `fabricSyncWayForward`, which tells the caller to commit locally or let the next bracket verb's sync carry the state, on a run that has been handed to another machine.
    fix: "In the `committed` branch of `internal/webstercli/reset.go`, check `errors.Is(pushErr, fabricengine.ErrHandedOver)` before the generic `push_error` envelope. When it matches, refuse with a message that names the hand-over first, as the success path's `webster: the branch was reset, and the run was handed over: %v` does, and keep the sync error in that message instead of appending `fabricSyncWayForward`. Keep `push_error` in the envelope."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/webstercli/reset_test.go:189
    summary: The failure branch's other outcomes have no row. No row checks that a sync failing before any commit landed refuses with no push and no `push_error`, and none checks that a hand-over there is reported in `push_error`.
    fix: "In `internal/webstercli/reset_test.go`, add a row that fails the sync before the records commit lands, for example with a fabric open that fails or a refused trailer. Assert the refusal, no `push_error` and no owed record for the run. If the fixture can stage another machine's takeover on the run, add a `syncFailsAfterCommit` row that expects `push_error` to name the hand-over. Fold F2's message change into that row's assertion."
---

### [MEDIUM] Merge danger names no outward push and too small a blast radius

**Location:** `_lym/landing/summary.md`, `## Merge danger`

**Issue:** The rubric's Merge-danger bullet makes a `two-way` call a finding when the diff pushes anything outward.
`internal/webstercli/reset.go:200` adds a synchronous records push on the branch where `fabricSync` fails after the commit landed.
Before this change that branch made no push of its own.
When that push fails, it writes a failed-push record, which `pattern/PATTERN-run-fsm.md:46-48` says `ReadRun` reads into `PushOwed`, and that holds the run.
The blast radius `webster` leaves out the run lifecycle, which this new record can now stop.

### [LOW] A hand-over on the sync-failure path keeps a way forward that no longer applies

**Location:** `internal/webstercli/reset.go:200`

**Issue:** When `pushResetRecords` returns `fabricengine.ErrHandedOver` on the `committed` branch, the envelope's `error` is the sync refusal plus `fabricSyncWayForward` (`internal/webstercli/beginbatch.go:27`).
That way forward points at more local commits, and the next bracket verb, for a run whose records nothing more will push (`internal/fabricengine/takeover.go:22`).
The success path makes the hand-over the error itself (`reset.go:214-215`).
The change description records the `push_error` placement as a choice, but leaves the misleading way forward unaddressed.

### [LOW] No row for the failure branch's not-committed and hand-over outcomes

**Location:** `internal/webstercli/reset_test.go:189`

**Issue:** The diff adds the `!committed` guard (`reset.go:196-198`) and a hand-over outcome inside the committed branch (`reset.go:200-205`).
The only new row reaches just `committed == true` with an unreachable origin.
No reset test refuses on a sync failure before the commit lands, and none in `internal/webstercli` asserts `ErrHandedOver`.

## Rejected

- code-tests, a reachable-origin `syncFailsAfterCommit` row checking that the commit reaches origin: the detached push that `Fabric.commit` spawns whenever a commit lands can carry the same commit to origin, so the row would also pass without the fix and would pin nothing the fix adds. The lens raised this caveat itself.
- code-tests, `SkipPush` and the success-path hand-over having no row: both branches moved unchanged into `pushResetRecords`, and neither had a row at the review base, so this is no gap the diff opens.
- code-correctness case 6, `Committed()` being true on a warp-only landing: the reset's pathspec is `_lym` alone, so no warp file is staged and the case cannot arise from this verb.

The rest of the focus was driven with no finding.
`committed` is true exactly when the weft commit landed and recording it failed, through `commitBothSides`'s partial path (`internal/fabricengine/commit.go:212`).
`SkipGit`, a nil opener (standalone) and a pre-commit refusal all give `committed == false`.
`SkipPush` skips the push on both paths alike.
The success path keeps its skip guard, its hand-over failure, its warning log and `push_error` after the extraction.
The new row asserts the refusal, `push_error`, the owed record and the `Run-Transition` commit, as the change description says it does, and it would fail without the fix.
The existing rows keep their ok assertion.
The help text, the `reset.go` header, `doc.go` and `pattern/PATTERN-run-fsm.md` agree with the code.
The `Root-Cause:` trailer names the early return before `c.pushRecords` that ignored `committed`, which is the defect the diff removes.
