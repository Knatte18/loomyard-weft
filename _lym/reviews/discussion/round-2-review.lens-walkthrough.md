```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end with the data each step actually receives, and checked the end state against the decision's purpose.

**Walkthrough list**
- **D1, met.** Case: a loom Finalize whose parent moved and touched task files, then a conflict. The merge goes through the resolver under the parent lock and `gateAndFix` runs over the merged tree. A failing gate runs the finalize-mode fixer under its caps. `fz.halt` returns a Stuck, and the parent lock is released on every way out (`finalize.go:253`, `finalize.go:800`), so a halt holds no lock. The case lands with no reroute.
- **D2, met.** Case: a darn Code-Bouncer re-entry after the overlap note is removed. The shared `fixerReportInput` reads `Parent-Align-fixer-report.md`, falling back to `Merge-In-fixer-report.md`. A darn run has neither file, so it answers not found as D2 says (`internal/loomcli/subjects.go:235-245`). The Finalize fixer report is `Finalize-fixer-report.md`, a separate file the seam never reads.
- **D3, met.** Case: a loom fixer `escalate` at Finalize. `haltReason` carries the new prefix and `fz.halt` makes a Halt Stuck. `stuckReason` only collapses the text to one line, so the persisted Error and the envelope Reason both open with the prefix (`shedengine/run.go:45-51`, `run.go:397-404`). From there:
  - `landingshed.HandsBack` is false, so the loop's `hand_back` mark is false.
  - `arbiterstop.Check` finds no landing halt and no decision halt, so the stop is eligible.
  - Batten's `HandBack` seam then skips the stop through the new predicate.
- **D3 on pawl, met.** Case: a pawl escalate with `EscalateJudgedFirst` false. The halt keeps `LandingStopPrefix`, so the HandsBack rule row fires and the stop hands back. That is a mechanism, not an agent's choice.
- **D4, envelope-built stop, met.** Case: a loom Finalize escalate.
  1. `facts` passes `JudgesFirst` the producer and the Reason, and the arbiter-first row fires.
  2. `settleFresh` admits the stop.
  3. The arbiter's `lym loom decision add --by arbiter` appends to `DiscussionDecisionRecord`. On a darn run this is the same file `landingDeps` hands the fixer (`decision.go:748`, `landingdeps.go:245`, `darnwiring.go:251`).
  4. The arbiter resumes with goto Finalize, and Finalize re-runs: merge, failing gate, a fresh fixer that reads the new decision.
- **D4, status-built stop on recheck, met.** Case: a moved run rechecked at a halted escalate. `stopFromStatus` leaves Reason empty and sets Error. D4 says the seam reads Error for such a stop, so the arbiter-first row fires.
- **D4, driver start, broken.** Case: a driver that starts at a halted escalate stop. Details in the finding below.

**Findings**

1. LOW · consistency · `decision-record.md`, Acceptance criteria, the driver bullet ("a driver that starts or re-checks at a halted escalate stop, built from the status, also sends it to the arbiter with no tackler call").
   - **Issue:** When a driver starts at a blocked stop and has no open attempt, it never decides a stop built from the status. It runs `checkBlockedStart`, then calls `Step`.
   - Shedengine does not short-circuit a blocked status; it calls the current producer again. So Finalize re-merges, the gate fails, and a new fixer session runs on the attempt the escalate gave back.
   - The arbiter sees the stop only if the fixer escalates again. That stop comes from a fresh envelope, not from the status.
   - At start, the only stop built from the status is the one for the StartCheck refusal, and that one hands back without any decision.
   - So the criterion's "starts" half describes a path that does not exist, and a test written to it pins nothing reachable. It also conflicts with D4's own Bound, which says a new driver re-steps the escalate stop before the arbiter sees it.
   - A driver restarted with an open arbiter attempt rebuilds the stop from the saved envelope, where Reason is set. That case is covered by the envelope path, not the status path.
   - **Fix:** limit the status-built clause to the re-check of a moved run. Have the criterion state that a starting driver re-steps the halted row, and that only a re-escalation, built from the envelope, reaches the arbiter.
   - **Evidence:** `internal/sheddriver/drive.go:181-195` (`run`: `checkBlockedStart`, then `Step`); `drive.go:615-630` (`checkBlockedStart` builds from the status only to hand back); `internal/shedengine/run.go:221-223` (a blocked state does not short-circuit, so the producer is called again); `drive.go:643-690` (`resumeOpenAttempt` rebuilds the stop from the envelope).

Tool calls:
- `shedengine/run.go` blocked-persist branch and `Finalize.halt`: needed to confirm the halt text persists with no added prefix and how the parent lock is released.
- `decision.go` record path and `loomengine.DiscussionDecisionRecord`: needed to check that the arbiter's darn `decision add` writes the file the fixer reads.
- `stuckReason` and the Finalize lock-release lines: needed to confirm the prefix survives into the persisted reason and that a halt holds no lock.
- The conflict session's `ParentName` wiring: needed to see where a loom conflict session now escalates under D1 (only the standard parent directive; no finding).
- `ReviewInput` and `overlapNoteInput` wiring: needed to trace darn's review input once its override is removed.
- `fixerReportInput` body: needed to confirm the shared seam answers not found on a darn run.
