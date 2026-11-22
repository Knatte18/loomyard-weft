---
verdict: BLOCKING
findings:
  - id: F18
    severity: BLOCKING
    class: scope
    location: _lym/plan/14-goto-clears-pr-review.md:63
    summary: "card 14's prReviewGotoAdmitted builds stopLiveSession from c.reed.Status and c.runner.StopStrand, but `lym loom goto` is wired by wireLightweight, which leaves c.reed and c.runner nil, so the main goto path panics whenever the record names a strand"
    fix: "In card 14's `prReviewGotoAdmitted` bullet, replace the clause 'stopLiveSession stopping the strand through the shuttle runner's StopStrand only when sessionLiveness(c.reed.Status) reads it live, the same liveness and runner the shared bootstrap and the landing wiring give the PR-Review row' with: stopLiveSession stops the strand through the runner `c.stepStopRunner()` answers, resolved inside the closure at call time and never captured when the hook is built, since `goto` runs under `wireLightweight`, which leaves `c.reed` and `c.runner` nil, and `stepStopRunner` returns `c.runner` when `wire` built one and builds the pair's runner the way `wire` does otherwise; `StopStrand` does nothing for a strand reed does not track, so the seam needs no separate liveness read. In **Uses:**, replace `internal/loomcli#sessionLiveness` with `internal/loomcli#loomCLI.stepStopRunner`. The plan gate admits this swap: `lym loom validate-plan --overlay` with the swapped card 14 reports the baseline's informational findings only."
prior:
  - id: F1
    status: resolved
    answer: "Card 6 now lists DriftVerify.passCommand under Edit, runs the endedUnanswered check after runAgent and before storeCaseRulings, takes reaskParent on an open mark, and adds DiffCommand rows to TestDriftVerify; passCommand's structure in verify.go matches the described flow."
  - id: F2
    status: resolved
    answer: "Card 15 lists validPawlDeps and simpleEntryCases under Edit and says what each gains; the other builders of BeamAwait's Deps (world.deps behind TestNewProducers_RefuseAnUnsetSeam, and the entries_pawl wiring) are on the card."
  - id: F3
    status: resolved
    answer: "Card 7 leaves the discussion_review_parent_timeout_min sentence as it is and adds one pr_review_parent_timeout_min sentence after the pr_review_ask_parent line."
  - id: F4
    status: resolved
    answer: "Card 6's reaskParent wait starts at one second and doubles to a minute, the card extends PATTERN-daemon-wakeups, and the Intent states what a watchdog kill leaves and that the re-entry continues the wait through reaskWaiting."
  - id: F5
    status: resolved
    answer: "Card 11 has wake send the stays-open mail after resumeSession to the strand name the record holds then, adds PRReview.wake under Edit, and adds a TestPRReview_Wait row for a resumed session under a new strand name."
  - id: F6
    status: resolved
    answer: "Card 14 stops a recorded strand before the clear and returns a stop error without clearing, with test rows for both. The wiring of that stop is a new defect of the changed hunk, raised as F18."
  - id: F7
    status: resolved
    answer: "internal/driftengine/doc.go is under card 6's Edit and the Intent names the three statements it gains."
  - id: F8
    status: resolved
    answer: "Card 7's Intent carries a Bound paragraph naming what the forcing lets through and what bounds it."
  - id: F9
    status: resolved
    answer: "Card 14 runs the clear through an Admitted hook that Goto and Relaunch call after the status write, with the target passed in; Spec, RunGoto, specFor, loomConsumeAttempt and goDriverDepsFor are listed, and the guard no longer resets. The hook's own wiring defect is F18."
  - id: F10
    status: resolved
    answer: "Card 14 lists Goto and Relaunch under Edit and has both comments name the Admitted hook as the one exception."
  - id: F11
    status: resolved
    answer: "Card 14's prReviewEntryReset resets on any target of PR-Review, with no atOrPastPRReview seam and no halted-before row."
  - id: F12
    status: resolved
    answer: "Card 12's TestParentReviewAnswerVerb rows state the newer ask ended timeout for the replaced-ask row and add a row with the newer ask open that keeps the existing 'a newer ask, ask N, awaits an answer' text."
  - id: F13
    status: resolved
    answer: "Card 5's ClearParentAsk checks no state, returns a caller error naming the number for a missing ask with no test row, and TestClearParentAsk holds two rows."
  - id: F14
    status: resolved
    answer: "Card 1 lists internal/beamsetting/beamsetting.go under Edit and names autonomy in its package doc, and its Intent no longer promises tasksetting_test.go's file comment."
  - id: F15
    status: resolved
    answer: "Card 10's first TestPRReview_Session row asserts Interactive, the record's autonomy and the one ask, with no prompt-wording assertion."
  - id: F16
    status: resolved
    answer: "Card 13 lists internal/loomcli/prcontinue.go under Edit in place of the two member glyphs and names the header rewrite in its Intent."
  - id: F17
    status: resolved
    answer: "internal/burlerengine/doc.go is under card 5's Edit and the Intent states that a cleared ended ask answers nothing."
---

### [BLOCKING] Card 14's session stop reads reed and runner that the goto verb never wires

**Location:** `_lym/plan/14-goto-clears-pr-review.md:63`

**Issue:** The card has `prReviewGotoAdmitted` build `stopLiveSession` over `sessionLiveness(c.reed.Status)` and `c.runner.StopStrand`, and has `specFor` set the hook as `spec.GotoAdmitted`.
`armAt` in `internal/loomcli/arm.go` wires the `goto` verb with `wireLightweight` (`verbUsesLightweightWiring` in `internal/loomcli/cli.go` lists `goto`), which sets only `c.location`, `c.cwd`, `c.shedPaths` and a few `c.env` fields.
`c.reed` and `c.runner` are assigned only by the full `wire` (`internal/loomcli/wiring.go`), so under `lym loom goto --to PR-Review` both are nil when `specFor` builds the hook.
`reedengine.Engine.Status` takes its op lock through the nil receiver, and `shuttleengine.Runner.StopStrand` reads `r.toldErr` first, so a record that names a strand panics the verb after the move is written, and the clear never runs.
The tackler and arbiter relaunch sites run after the full wiring, so only the verb's path, the one the operator and the orch type, breaks.
The loomCLI already has `stepStopRunner` (`internal/loomcli/looparm.go`) for this case: it returns `c.runner` when `wire` built one and builds the pair's runner otherwise.
`StopStrand` ends through `removeStrandIfPresent`, which does nothing for a strand reed does not track, so the liveness read the card adds is not needed to keep the stop from acting on an ended session.

## Rejected

- Card 6's three bullets after the `pattern/PATTERN-daemon-wakeups.md` bullet are indented under it though they describe `reaskParent`: a readability slip, not raised in an incremental round.
- Card 6's Intent does not add `Sleep` to `DriftVerify.Call`'s required seams: `driftDeps` in `internal/loomcli/driftwiring.go` and `envkit` already set it, so no wiring is missed.
