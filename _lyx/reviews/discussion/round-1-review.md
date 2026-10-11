---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D5)
    summary: D5 routes an interim fork notice to a turn end with no tool call but states no bound for a fork that never sends a final report or a final notice misread as interim
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/discussion/decision-record.md (D5, D3)
    summary: The verify-fix fixer fork's notice path in the Master template has the same interim-notice hazard D5 guards in loop step 4, and D5 and D3 leave it unguarded
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D1 Bound, Open risks)
    summary: Removing interim notices and allowing hour-long foreground calls collides with loom's 60-minute step idle kill and batten's 30-minute quiet notice, and the record leaves the remedy undecided
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/discussion/decision-record.md (Scope, D7, Acceptance criteria)
    summary: CLAUDE.md line 31 tells agents to run lyx gate test as a background Bash call, which D7's no-production-text claim and the git grep criterion miss
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md (D6)
    summary: D6 leaves open whether the orch copies the gate rule into a fresh agent's brief or points the agent at the orch's system prompt, which that agent cannot read
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D1 Bound, Open risks)
    summary: D1's Bound says the Bash tool ends a call at the ceiling, while Open risks says the tool ends or backgrounds it
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D2, D3, D4, Acceptance criteria)
    summary: D2 calls the steer the one statement of the gate rule, but D3 and D4 restate its cd and quick-check parts, and the criteria omit D4 for verify-fix
---

### [MEDIUM] D5's routing edge has no stated bound

**Location:** `_lyx/discussion/decision-record.md`, D5.

**Issue:** D5 adds a routing edge: a fork notice judged interim from its message text skips `record-batch` and ends Merriam's turn with no tool call.
The rubric's attack-surface rule requires the Decision to state what this lets through and what bounds it, and D5 states neither.
Two cases are unbounded.
A fork that never sends a final report, for example because it died, leaves Merriam idle.
A final notice whose model-written text Merriam misreads as interim leaves the batch unrecorded.
Merriam's safe-turn-end rule in `contracts/stencils/webster/webster-template-master.md:9` relies on an outstanding fork or backgrounded `recover-batch`.
The record does not say whether a fork that has sent a "finished" notice still counts as outstanding in the Stop payload that shuttle's `expiredTurnEnd` (`internal/shuttleengine/wait.go`) judges.
If it does not, the turn end reads idle-stalled rather than waiting.

**Fix:** State in D5 what bounds each case, for example the run deadline and liveness check for a fork that never reports.
State what a misread final notice leads to, and the signal that tells the operator.
If no existing bound holds, decide one, such as Merriam calling `record-batch` on the next notice regardless of its text.

### [MEDIUM] The fixer fork's notice path is unguarded

**Location:** `_lyx/discussion/decision-record.md`, D5 and D3; `contracts/stencils/webster/webster-template-master.md`, "A gate failure: spawn one fixer fork", step 4.

**Issue:** On the fixer fork's completion notification, Master step 4 rewrites the outcome and summary files and ends the turn, which re-arrives at the gate (`internal/websterengine/doc.go:461` says the same).
An interim notice from the fixer fork would make Merriam re-arrive at the gate while the fixer still works, which burns a gate attempt on a half-fixed tree.
D5's rationale ("the fallback … needs a safe reading") applies here equally, but D5 changes only loop step 4.
D3 gives `webster-body-verify-fix.md` only "the same change for its `lyx gate test --tags integration` step".
It does not say whether the fixer also gets the flat `run_in_background` ban and the foreground quick-check rule.
Without that rule, a backgrounded quick check still produces interim notices.

**Fix:** Extend D5 to the fixer fork's step 4, and say in D3 that the verify-fix body carries the same no-backgrounding rule as the implementer body.
Add both to the Acceptance criteria.

### [MEDIUM] Foreground waits meet the idle watchdogs

**Location:** `_lyx/discussion/decision-record.md`, D1 Bound and the third Open risk.

**Issue:** Today each interim notice starts a Merriam turn and writes Merriam's transcript.
After D1–D3, Merriam's transcript stays unwritten from the fork spawn to the final report.
A single fork call may also run for up to an hour.
Activity is read from the main transcript's modification time (`internal/shuttleengine/claudeengine/activity.go`).
A shuttle wait on a fork shows no wait marker, because `shellWaitActive` (`internal/shuttleengine/waitmark.go`) counts only non-awaited shells.
Two consumers act on that reading.
`step_idle_timeout_min: 60` (`internal/loomengine/template.yaml`) kills a `--until-stop` step that shows no activity, through `agentActivityFor` (`internal/loomcli/looparm.go`).
`notice_quiet_min: 30` (`contracts/recipes/batten-recipe.yaml`) sends batten's quiet notice.
The one-hour ceiling equals the step idle kill window, so a slow batch can be killed or noticed as quiet while it works.
I did not verify whether a fork's tool calls write the parent's transcript.
The Open risk names this but leaves the remedy undecided: the plan writer is told to check, not what to do with the result.

**Fix:** Settle the premise in Discussion: does an outstanding fork, or a fork's foreground call, count as activity?
If not, decide the remedy, for example counting an outstanding fork as live activity, or setting the ceiling below the step idle window.
Put the remedy in a Decision so the plan writer has a requirement rather than an investigation.

### [MEDIUM] CLAUDE.md still says to background the gate

**Location:** `_lyx/discussion/decision-record.md`, Scope, D7 and the seventh Acceptance criterion; `CLAUDE.md:31`.

**Issue:** `CLAUDE.md:31` says the pre-push tiers run "each run as its own background Bash call".
Every Claude session in this repository loads it, including the orch's forks that run those tiers and any Webster fork working on loomyard.
It contradicts the new steer, and D7's rationale ("a single leftover instruction re-creates the wake pattern") applies to it.
Scope does not list it, D7's file list omits it, and `git grep "background Bash call"` matches it.
That match means the criterion either fails or depends on an unstated reading of "production text".

**Fix:** Add `CLAUDE.md` to Scope and D7, worded to point at the gate rule.
Alternatively, define in D7 which files count as production text and exclude `CLAUDE.md` with a reason.

### [MEDIUM] D6 does not say what goes into the agent's brief

**Location:** `_lyx/discussion/decision-record.md`, D6.

**Issue:** D6 says a brief "tells it to run it as the gate rule in the orch's own system prompt says".
Its rationale says a fresh non-fork agent "does not see the orch's appended system prompt".
Read literally, the stencil line tells the orch to point the agent at text the agent cannot see.
The other reading is that the orch restates the rule's content in the brief, which D2's "no restating the number" seems to forbid.
The plan writer has to choose between the two readings.

**Fix:** State in D6 that the orch copies the rule's content, including the timeout value, from its own system prompt into the brief.
Note that this is the one sanctioned restatement outside the steer.

### [LOW] D1's Bound and the Open risk disagree on what happens at the ceiling

**Location:** `_lyx/discussion/decision-record.md`, D1 Bound and the first Open risk.

**Issue:** D1's Bound says that after one hour "the Bash tool ends it" and that the gate then kills its tree and releases its slot.
The first Open risk says that at the ceiling a call "ends or backgrounds".
A backgrounded gate is not killed and keeps its slot, so the Bound holds only in the "ends" case.
The gate's own claim holds: `internal/gatecli/child.go` catches SIGINT, SIGTERM and SIGHUP and kills its tied child tree.
The orch's own turn is not at stake, since `contracts/stencils/orch/orch-template-role.md:90` has the orch run no test suite itself.

**Fix:** Make the Bound match the Open risk: name both outcomes at the ceiling and what bounds the backgrounded one, which is D5's fallback.

### [LOW] The gate rule is stated in more than one place

**Location:** `_lyx/discussion/decision-record.md`, D2, D3, D4 and the Acceptance criteria.

**Issue:** D2 is titled "the one statement of the gate rule" and puts the no-`cd` and foreground quick-check rules in the steer.
D3 restates the quick-check rule in the implementer body, and D4 restates a stricter "never `cd`" in both bodies.
Only the timeout value is stated once.
The criteria pin D4 for the implementer prompt but not for the verify-fix prompt, and they omit D4's worktree-relative-path rule.

**Fix:** Either narrow D2's claim to the timeout value, or have D3 and D4 point at the steer for the `cd` and quick-check rules.
Add the verify-fix `cd` rule and the relative-path rule to the Acceptance criteria.

## Focus verdicts without a finding

On the Open risks directive, I found the `BASH_MAX_TIMEOUT_MS` premise correctly contained as a risk with an observable signal, and filed no finding on it.
The activity-reader premise is not contained the same way, so it is filed as F3.
On the relocation directive, I found no deliberation narrative in `decision-record.md` that belongs in `support-log.md`.
Each Rationale supports its Decision or Bound.
