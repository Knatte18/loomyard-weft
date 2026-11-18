```yaml
lens: code-scope
outcome: reported
```

**Task:** code-scope lens. Trace each change in the diff to a goal point of the board entry. The record has no `Added after Discussion` entries.

**Decisions** (the goal points under `## Goal`; `## Decisions` is empty):
- **A1.** Find why the strand was untracked at re-prompt time.
- **A2.** When the strand is gone, relaunch or resume the producer instead of halting, or fail fast.
- **B1.** On a re-step after `interrupted-stop`, reconcile a shuttle run that reads `running` with a gone waiting process before the step starts.
- **B2.** Check that the interrupted-step record names the step in flight, consistent with the history.
- **B3.** If a clean stop can look like this, record the cause where the loop exits.

**Changes and what they trace to:**
- **C1.** Gate re-prompt resume: `relaunch.go` (new), `wait.go` `handleGatedBoundary` routing on `errStrandGone`, and `run.go` `errStrandGone`/`strandGoneError` wrapping in `liveStrand`. Traces to A2.
- **C2.** `strandAddSpec` extracted in `run.go` and shared by `launch` and the resume. Forced by C1.
- **C3.** `TestGate_RepromptToGoneStrand` and `resumableFakeEngine` in `gate_test.go`. Forced by C1 (its test).
- **C4.** `shuttleengine/doc.go` gate paragraph and the `wait.go` doc lines on resume. Forced by C1 (docs it makes stale).
- **C5.** `LoopSpec.OnStepOrphaned`, called from `runChild`'s no-envelope exit, from `stepUntilStop`'s panic recovery and from `deadLoopStop`. Plus the loomcli wiring: `arm.go`, and `looparm.go` `orphanedStepNote` with the shared `stepRunsNote`. Traces to B1.
- **C6.** `StopStepRuns` now records a tied run as stopped when its strand is dead or untracked (`stepstop.go`). Traces to B1, since after a machine stop every strand of the step is dead or untracked.
- **C7.** `shedverbs/doc.go` and the `shuttleengine/doc.go` step-stop paragraph. Forced by C5 and C6.
- **C8.** Test changes in `loop_test.go`, `loopwait_test.go`, `looparm_test.go` (rename and table over both hooks) and `stop_test.go`. Forced by C5 and C6.
- **C9.** Commit `fea250c5`, comment line-break fixes. It touches only files C1–C8 already change (`git show --stat`), so it is forced.
- No commit in the range carries a `Pattern-Departure:` trailer.

**Each decision and the changes that build it:**
- **A1:** no change. `_lym/landing/summary.md` "Design decisions" says the evidence is gone and the change recovers from a gone strand whatever removed it. A search found no `loom-halt*.md` under the hub for `launch-manifest-parity` or `lym-retro`, which fits that claim. An investigation can't be satisfied by changing the diff, and A2 is the remedy the entry asks for, so no finding.
- **A2:** C1, C2, C3, C4. This is the entry's "resume instead of halting" branch, falling back to the earlier failure with both causes named.
- **B1:** C5, C6, C7, C8. Reconciliation happens at the interruption's own stop, not at the re-step (finding 1).
- **B2:** no change. The summary claims that after `Code-Bouncer` bounces back the status names `Code-Burler-Review` while the history ends at `Code-Bouncer`, so record and history agree. The entry asks for a check, not a change, so that is no scope finding as long as the claim holds. Its truth belongs to the handler's or the correctness review.
- **B3:** no change. The entry's condition is "if a clean stop can look like this". The summary argues a clean stop leaves the loop's signal mark and the keeper's exit mark, so a markless exit is a `SIGKILL` or a machine stop. That answers the condition, so no finding.

**Findings**

1. **Severity:** LOW. **Class:** scope.
   **Location:** `internal/shedverbs/loopwait.go:583` (`deadLoopStop`) and `internal/shedverbs/loop.go:609` (`runChild`).
   **Summary:** B1 asks for the reconcile on the re-step, before the step starts. The diff instead stops the step's runs once, inside the stop that reports the interruption. Nothing in the re-step path does it again. So a run whose stop failed is still left `running` for the re-step to trip on. That covers `report.Failed`, a reed status error in `StopStepRuns`, or a hook that ran while reed was down. A run left by an interruption before this change is caught the same way. The interrupted cause names the failure, but nothing retries it.
   **Evidence:** the board entry, B1's bullet (decision-record.md:53); `internal/shuttleengine/stepstop.go:57-99`, where a failed stop is listed under `Failed` and nothing else happens; `internal/loomcli/looparm.go` `stepRunsNote`, where a failure only becomes note text; and the summary's claim "Because the dead-loop stop runs before the waiter reports the halt, the leftover is settled before any re-step".
   **Fix:** have the re-step settle the step's leftover runs before it starts, or file a parent decision that settling them at the stop is enough.

**Tool calls**
- `git show --stat` of the three range commits: my context lacked which files the comment-fix commit `fea250c5` touches, needed to judge it as forced or as scope.
- `find` for the `loom-halt*.md` notes: my context lacked whether the A1 evidence still exists, needed to judge the summary's "the halt notes are gone" claim behind the A1 trace.
