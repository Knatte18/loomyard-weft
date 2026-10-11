---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:86
    summary: D5's fixer bound misstates the outcome of an interim-notice arrival; a dirty tree fails the gate at once, so the budget burns in a few Merriam turns, and the stuck end removes the strand with the fixer fork mid-edit
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:116
    summary: D9 reads a held call only when the fork transcript's newest entry is the tool call, which misses a held call followed by a later entry such as a parallel sibling call's result
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:117
    summary: D9 returns a read-time value through shuttleengine's TurnEndActivity.TranscriptModTime, whose provider-neutral contract says transcript last write, yet no Decision or Scope line updates that contract
---

### [MEDIUM] D5's fixer bound understates what an interim-notice arrival does

**Location:** `_lyx/discussion/decision-record.md:86`–`88`, and the matching Open risk at line 148.

**Issue:** The premise holds: once `outcome_path` and `summary_path` exist, a plain Stop is `OutcomeDone` in `pollEventsTick`.
On a gated run `Wait` hands that to `handleGatedBoundary` whether or not Merriam rewrites the files, as the Decision says.
The stated consequences, though, do not match the verify gate's code (`internal/websterengine/verifygate.go`, `newVerifyGate`):

- On a dirty tree the closure fails at once from `s.dirty()` and never runs `verifytree.Verify`.
  So each arrival lasts one re-prompt and one empty Merriam turn, and a fixer with uncommitted edits sees the whole `verify_gate_attempts` budget spent within seconds.
  "Unless the fixer's final notice comes first" then practically never happens while the fixer is mid-edit.
- Each failed evaluation also rewrites the verify-gate report, which is the fixer's input file named in its prompt, with a dirty-paths report.
- A spent budget goes to `finalize(OutcomeDone)`, and without `KeepPane` that calls `RemoveStrand`.
  This kills Merriam's pane and the fixer fork running in it, leaving the tree in whatever half-fixed, possibly dirty state the fixer had reached.
  "Two fixers never edit the tree at once" is true, but the bound never states that the one fixer is torn down mid-edit.

Under D1 alone this fires only after a one-hour hang, where ending stuck is defensible.
Under the Open risk at line 148 (ceiling ignored) it fires for every fixer whose integration gate runs past ten minutes, a legitimate fixer.
That is more than "spends verify attempts early": the run ends stuck within a minute and the working fixer is killed.

**Fix:** Restate the fixer bullet with these three effects (an instant dirty-tree failure per arrival, the rewritten report, strand removal with the fixer live).
Then either mark the outcome explicitly as accepted because D1 makes the trigger a one-hour hang, or name a remedy.
Correct line 148's consequence to match.

### [MEDIUM] D9's held-call test keys on the newest transcript entry

**Location:** `_lyx/discussion/decision-record.md:116`; the acceptance criterion at line 159 states it more broadly.

**Issue:** D9 reads a fork as active when its transcript's "newest entry is a tool call with no result yet".
A held gate need not be the newest entry.
A fork that issues parallel tool calls writes one assistant line per `tool_use`.
When a quick sibling call returns while the gate is still held, its `tool_result` line (and the attachment lines Claude Code writes after results, seen in a real fork transcript under `~/.claude/projects/*/*/subagents/`) is newer than the pending gate call.
The rule then falls back to the last write time, so a fork holding a 35-minute foreground gate reads quiet: batten's `notice_quiet_min` (30) notices it, and a gate near the ceiling approaches loom's `step_idle_timeout_min` kill.
These are the failures D9 exists to prevent.
The acceptance criterion at line 159 ("holds a tool call open") is satisfied by the broader reading, so the Decision and its criterion disagree.

**Fix:** State the rule as "the fork transcript holds a `tool_use` with no matching `tool_result`", keeping the cap measured from the transcript's last write.

### [LOW] D9 changes a provider-neutral contract it says stays untouched

**Location:** `_lyx/discussion/decision-record.md:117`, and the Scope's package-docs bullet.

**Issue:** `ReadAgentActivity` takes the transcript part from `TurnEndActivity.TranscriptModTime`.
Both that field's doc in `internal/shuttleengine/activity.go` and `AgentActivity.LastActivity` ("the newer of the transcript's last write and …") define it as the transcript's last write, and `internal/shuttleengine/doc.go` repeats it.
D9 makes claudeengine return the read time for a held fork call and the newest write across fork transcripts.
The code outside claudeengine stays the same, as D9 says, but the documented contract it reads becomes false.
The Scope lists only claudeengine's and websterengine's `doc.go`.

**Fix:** Add a Decision line, or extend the Scope, so the plan updates the shuttleengine contract (field doc, `LastActivity` doc, `doc.go` activity paragraph) to mean "the session's last activity, forks included".

## Focus verdicts

- Interim-notice premise: settled, not a defect.
  `outstandingBackgroundTasks` counts only payload entries with `status` `running`, and the transcript fallback matches a launch on any later notification naming its id, so an interim-notified fork is not outstanding.
  The Bound states the batch-loop outcome correctly: Master's spec sets neither `HoldDeferral` nor `QuietHold`, so `Wait` calls `notifyHeld` once per held turn end, and the run ends at `master_timeout_min`, the liveness check or the step idle kill.
- Fixer-fork guard: the gated-arrival premise is right; the bound's description of that arrival is F1.
- D9 held-call reading: the Bound says a pending call cannot be told apart from an abandoned one and caps it at the ceiling past the last write.
  The cap is what ends the active reading for a fork whose writer is gone, and that is adequate.
  The detection rule's gap is F2.
