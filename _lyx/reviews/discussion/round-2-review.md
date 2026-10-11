---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D5 Bound)
    summary: D5's bound rests on an unsettled premise and states the wrong outcome; a Merriam turn end with no fork outstanding and output files missing is held by shuttle until master_timeout_min with a parent hold notice, never a run end
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D5, fixer-fork step 4)
    summary: the fixer-fork guard cannot keep Merriam off the verify gate; once the fixer's interim notice clears it from the outstanding list, Merriam's no-tool turn end with outcome and summary already on disk is a gated arrival that runs verify mid-fix
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D9 Bound)
    summary: D9's held-call reading cannot tell a held tool call from an abandoned one, so a fork stopped mid-call reads active for the rest of the run and disables the idle kill and the quiet notice, a case D1's ceiling does not bound
---

### [BLOCKING] D5's bound states an outcome shuttle does not produce

**Location:** `_lyx/discussion/decision-record.md`, D5's `Bound:` paragraph.

**Issue:** D5 bounds its new edge with two claims.
First, "a final notice misread as interim leaves Merriam's turn ending with no fork and no backgrounded call outstanding, which ends the run with nothing done".
Second, "an interim notice whose fork Claude Code no longer counts as outstanding: the run ends instead of hanging".
Shuttle does neither.
In `internal/shuttleengine/wait.go`, `pollEventsTick` returns a held turn end for a non-waiting turn end with output files missing, and `Wait` logs "turn end held, output files missing" and keeps polling ("A turn end without every output file never ends the run").
Master's spec in `internal/websterengine/runlevel.go` sets no `HoldDeferral`, so each held turn end also sends Merriam's parent a hold notice through `notifyHeld` (`internal/shuttleengine/hold.go`).
The held run ends only at `master_timeout_min` (480 in `internal/websterengine/template.yaml`), the liveness check or, with D9, loom's step idle kill.
The Master template's "ending a turn with no running background work and no outcome file ends the run with nothing done" is the sentence D5 inherits, and the code contradicts it.

The premise the focus asks about is also unsettled: whether a fork that has sent a finished notice still counts as outstanding.
In `internal/shuttleengine/claudeengine/background.go`, the transcript fallback counts a fork launch as matched by any later `<task-notification>` naming its id, interim or final.
The payload path counts only entries whose `status` is `running`.
The board entry quotes Claude Code reporting the fork as `finished` at each interim stop.
Both signals point to the fork reading as *not* outstanding after an interim notice.
In that case, the "rare" interim case D5 exists for does not take the bounded path D5 describes.
Instead, each interim notice in the batch loop becomes a held turn end, and each held turn end sends a hold notice to Merriam's parent.
That moves the noise the goal removes to the parent instead of removing it.
A misread final notice leaves the batch unrecorded for up to 480 minutes, or until the step idle kill, with no prompt to Merriam.
The plan writer cannot implement a correct guard from D5 as written, because it describes neither case's outcome.

**Fix:** Settle the premise in a Decision: a fork that sent an interim notice is no longer in shuttle's outstanding list (from `background.go`'s matching rule and the payload status), or cite evidence that it stays.
Then restate D5's bound against shuttle's real behaviour.
An interim-notice turn end is held, and the fork's final notice starts Merriam's next turn.
State whether the parent hold notice per interim stop is acceptable, or carry a remedy as a requirement.
A misread final notice is held until `master_timeout_min` or the step idle kill, whichever comes first.
If D5 relies on the Master template's safe-turn-end sentence, add correcting that sentence to Scope.

### [BLOCKING] The fixer-fork guard does not keep Merriam off the verify gate

**Location:** `_lyx/discussion/decision-record.md`, D5's third sentence on the fixer fork's step 4.

**Issue:** D5 says an interim notice from the fixer ends Merriam's turn "without rewriting the outcome and summary files, so it never re-arrives at the gate while the fixer works".
Whether a turn end is a gate arrival is shuttle's call, not Merriam's, and skipping the rewrite does not change it.
Merriam wrote `outcome_path` and `summary_path` before the turn end that reached the gate, so both files exist while the fixer works.
`pollEventsTick` returns `OutcomeDone` for a plain Stop with every output file present.
`Wait` treats a gated `OutcomeDone` as the writer's boundary and calls `handleGatedBoundary`, which runs the verify gate.
Only an `EventWaiting` turn end with the fixer still listed keeps the run waiting (`expiredTurnEnd`: a gated run never treats a waiting turn end as an arrival).
By F1's evidence, an interim notice takes the fixer off that list.
After that, Merriam's no-tool turn end is a gated arrival at the verify gate while the fixer is mid-edit.
The gate then runs verify on a dirty or half-fixed tree and spends one of `verify_gate_attempts`.
It also re-prompts Merriam with new findings, and that re-prompt has Merriam spawn a second fixer beside the first.
The batch loop does not hit this, because there the output files are missing and the turn end is held.
The fixer path needs its own treatment.

**Fix:** Add a Decision that bounds the fixer case on shuttle's real arrival rule.
One way is to rule out an interim notice from the fixer by construction: D3's flat background ban plus D1's ceiling, with the residual case (a call the Bash tool backgrounds at the ceiling) named.
That residual case still needs a stated outcome: an arrival that spends one verify attempt on a dirty tree, and a second fixer.
State that outcome as the accepted bound or carry a remedy, and drop the claim that skipping the rewrite prevents the arrival.

### [MEDIUM] D9's held-call reading has no bound for an abandoned call

**Location:** `_lyx/discussion/decision-record.md`, D9's `Bound:` paragraph.

**Issue:** D9 reads a fork "whose newest entry is a tool call with no result yet as active at the time of reading".
Its bound is "a held tool call reads active only while it is held, and D1's ceiling ends or backgrounds it within an hour".
A transcript ending in a `tool_use` with no result does not prove the call is still held.
A fork stopped or interrupted mid-call leaves the same shape, and so does a call that Claude Code ends without writing a result.
`internal/websterengine/doc.go` already treats a fork "stopped and resumed across a no-report call" as a case that occurs.
D1's ceiling bounds only a live call, not a transcript whose writer is gone.
Such a fork reads active from then on, so loom's `step_idle_timeout_min` kill and batten's `notice_quiet_min` notice never fire for that Merriam.
The run is then bounded only by `master_timeout_min`.
That reverses D9's own rationale for rejecting the shuttle-side option: "it would read a hung fork as live until the run deadline".
This is the bypassable-guard case the rubric's attack-surface question names.

**Fix:** Bound the held-call reading in D9 as a requirement.
For example, read a pending tool call as active only until the D1 ceiling has passed since the `tool_use` entry was written, then fall back to its write time.
Restate the Bound to cover a call whose writer is gone.

## Focus verdicts

- Focus 1, the interim-notice Decision: D5 names a bound for both cases, but the bound is wrong and its premise unsettled (F1).
  By `background.go`'s matching rule and the board's evidence, a fork that has sent a finished notice does not count as outstanding.
- Focus 2, activity under foreground calls: D9 settles the premise for forks and carries the remedy as a requirement rather than an investigation, apart from the abandoned-call gap (F3).
  A main session's own foreground gate reads quiet from its `tool_use` write onward, the same quiet stretch a backgrounded gate gives today through its waiting Stop line, so no new gap there.
- Focus 3, the verify-fix fixer: D3 carries the same background ban and gate-rule pointer into `webster-body-verify-fix.md`, matching the implementer body.
  The completion-notice guard is not equivalent to the batch loop's, because the fixer runs with output files present (F2).
