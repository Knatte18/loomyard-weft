```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced the diff's changes to the board entry's goal points.

**Decisions.** These are the goal points under `## Goal` / `## Idea` in `_lym/discussion/decision-record.md`; the record has no `Added after Discussion` entries.
- **G1:** a pending record that came from a batten cancel names `lym batten cancel <slug> --remove` as its re-run.
- **G2:** a pending record whose pair is gone names `lym batten cancel <slug> --remove` as its re-run, whoever wrote it.
- **G3:** the re-run is carried the way the `discard` record's `rerun` field already carries it.

**Changes, each mapped to a decision:**
- **C1:** `Topology.Remove` / `removePair` take `rerun` and carry it into the job's `Rerun`. The held-lock refusal also names it (`internal/fabricengine/remove.go:209-534`).
  - Traces to G1 and G3. Naming the rerun in the lock refusal copies what `taskbranchdiscard.go:49-51` already does for a discard, so G3 covers it.
- **C2:** `rerunWayForward` names `job.Rerun`, and `recordsArchiveWayForward` drops its origin condition when `Rerun` is set (`remoteteardown.go:203-249`). Traces to G1.
- **C3:** `pairteardown.Request.Rerun` is added and passed to `Remove` (`internal/pairteardown/teardown.go:73-75,247`). Forced by C1 and needed for G1.
- **C4:** `removePairAndBranch` sets `Rerun: cancelCommand(slug, true)` (`internal/battencli/cancel.go:783`). Traces to G1.
- **C5:** Field doc comments on `remoteTeardownJob.Rerun` and `PendingTeardown.Rerun`, plus `doc.go:241-243`. Forced by C1 and C2.
- **C6:** The chokepoint guard's argument count goes from 4 to 5, with its comments and fixture (`teardown_enforcement_test.go`). Forced by C1's signature change.
- **C7:** Every test call site of `Remove` gains `""`, across `battencli`, `fabricengine` and `ideengine`. Forced by C1.
- **C8:** New tests: the `TestPendingTeardowns` row, `TestRemove_PendingRecordNamesTheRerunItsCallerGave`, and the cancel test's `Rerun` assertion. Forced by C1–C4 and pin G1.

**Decisions, each mapped to the changes that build it:**
- **G1:** built by C1, C2, C3 and C4, with C5–C8 forced.
- **G2:** no change builds it. A record written by `lym fabric remove`, or by any other `Rerun`-less caller, still names `lym fabric remove <slug> --remote` once its pair is gone. The other callers are `fabriccli/remove.go:87,96`, `beamcli/dobbydeps.go:455`, `battencli/beamfork.go:180` and `battencli/wire.go:449`.
- **G3:** built by C1 and C2, which reuse the existing `Rerun` field of the job and the record.

No change lacks a decision, so I found no unrequested scope.

**Findings**

1. **Severity:** MEDIUM. **Class:** scope. **Location:** `internal/fabricengine/remoteteardown.go:206`, `rerunWayForward`.
   - **Summary:** The board entry asks for `lym batten cancel <slug> --remove` as the re-run "when the record's pair is gone, or the record came from a batten cancel". The diff builds only the second condition. A `remove` record with an empty `Rerun` still gets `lym fabric remove <slug> --remote` after its pair is gone, which is the dead end the entry's `## Seen` describes.
   - **The narrowing is unrecorded.** The record has no `Added after Discussion` entry for it. `_lym/landing/summary.md` "## Design decisions" justifies it on the writer's own authority: a `fabric remove` slug may have no batten run, and telling a gone pair apart would cost a network call.
   - **Fix:** leave the code as it is and defer to the run's parent. The parent either records the narrowing with `lym loom decision add`, or rules that the pair-gone case must be built.
   - **Evidence:** `_lym/discussion/decision-record.md:15-16`, `remoteteardown.go:206-214`, `_lym/landing/summary.md` "## Design decisions".

Outside my lens: `doc.go:242` claims the cancel "finishes even once nothing of the pair remains". `refuseNothingLeft` (`cancel.go:752-769`) suggests otherwise; the code-correctness lens owns that.

Tool calls: none.
