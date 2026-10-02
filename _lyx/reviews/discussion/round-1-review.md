---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md Decisions 1 and 3
    summary: Bounds of bypass and AllowAgentTool omit that every subagent and fork the orch session spawns (adopted sessions included) runs unprompted, and that AllowAgentTool has no operator off-switch
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md Decision 5
    summary: Bound claims adopt never takes over a session a live process holds, but the fail-open registry path lets exactly that through and the Decision states no bound for it
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md Decision 6
    summary: DEFER contract is underspecified - match rule on the turn-end message, DEFER plus a handoff file, DEFER on a hard or requested cycle, and the deferral hold's start point are all left to the plan writer
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md Decision 6
    summary: No trigger precedence when a reading is at or over both thresholds, so with idle_grace_s at or above soft_idle_s a soft cycle starts at a hard-cap reading and can be deferred
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md Acceptance criteria, strand-name line
    summary: Naming/parent/SendMessage criterion verifies shipped agent-naming behaviour but is not marked as a live-smoke check nor listed in Decision 7's smoke scope
  - id: F6
    severity: NIT
    location: _lyx/discussion/decision-record.md Decision 6 rationale and Auto-mode assumptions
    summary: Hub-review provenance parentheticals belong in support-log.md, and the first auto-mode bullet overstates that every pick is the writer's own
---

### [MEDIUM] Bypass and Agent-tool bounds omit what spawned agents inherit

**Location:** `decision-record.md` Decision 1 Bound, Decision 3 Bound.

**Issue:** Decision 1's Bound says the permission-prompt guard is removed only for a run whose caller sets `bypass`.
It does not say that Claude Code subagents and forks run in the parent's permission mode, so with Decision 3 every typed subagent and fork the orch session spawns also runs every tool with no prompt.
The support log's Security turn says this ("the orch session and its subagents"), but the plan writer never reads it.
Decision 3's Bound says only orch sets the allowance, yet it never states what the operator can still switch off: there is no `orch.yaml` key, so the Agent tool cannot be denied for orch at all, and the operator's only lever is `permission_mode: prompt`, which keeps the Agent tool but makes its subagents prompt in the orch pane.
Neither Bound says that `--adopt` puts an operator session that ran prompt-gated under orch's spec, so adopting upgrades it to bypass and the Agent tool.

**Fix:** Extend Decision 1's Bound to cover spawned subagents and forks and adopted sessions, and Decision 3's Bound to state that the allowance has no switch and that `permission_mode: prompt` is the operator's remaining lever.

### [MEDIUM] Decision 5's Bound is false on the fail-open registry path

**Location:** `decision-record.md` Decision 5, the registry sentence and the Bound.

**Issue:** An absent or unreadable registry warns and proceeds.
On that path a second live Claude process can hold the same session id, and adopt then resumes it anyway: two processes append to one transcript and orch clears and resumes a session another pane is still driving.
The Bound says adopt "never takes over a session a live process holds", which this path contradicts, and the Decision states nothing that bounds the case.
Bounds do exist: adopt is an explicit operator verb naming one id, the transcript check still confines it to a prime session, and the warning is logged.
They are not written down.

**Fix:** Rewrite the Bound to say adopt never takes over a session a live process is positively identified as holding, that a missing or unreadable registry lets an unconfirmed holder through, and what bounds that: the explicit operator verb, the logged warning (also on the envelope), and the prime-only transcript check.

### [MEDIUM] DEFER contract is not specified enough to plan from

**Location:** `decision-record.md` Decision 6, the `DEFER` sentences.

**Issue:** The watcher can tell a DEFER turn end from an ordinary reply only through the turn end's final assistant message (`shuttleengine.Event.Message`), but the Decision does not say that or the match rule (exact word, trimmed, or substring).
Three cases are left open:
- A soft cycle whose session writes a handoff file and also replies `DEFER`.
  Today's handoff phase clears once a turn end follows a written file, so whether the file or the veto wins is a design choice the plan writer would have to make.
- A turn end with neither a handoff file nor `DEFER`.
  The existing behaviour waits until `handoff_timeout_s` and aborts with "handoff timed out", and the Decision should say the soft cycle keeps that.
- `DEFER` on a hard or requested cycle.
  "Cannot be deferred" in fact means the watcher ignores the reply and keeps waiting for the file until the timeout, which the acceptance line "does not defer it" leaves implicit.

The hold is also ambiguous: "waits a further `soft_idle_s`" does not say whether it counts from the DEFER turn end or from the next one.
`toIdle` forgets the newest event, so only a turn end read after the abort can qualify again.

**Fix:** State in Decision 6 that the DEFER is matched against the turn end's final message after trimming whitespace, with exact equality to `DEFER`.
Also state that a written handoff file wins over `DEFER` (the cycle proceeds, since the handoff guarantee holds), and that a turn end with neither keeps the existing wait-then-timeout.
State that `DEFER` on a hard or requested cycle is ignored, and that the hold counts `soft_idle_s` from the DEFER turn end's read time.

### [MEDIUM] No trigger precedence at a reading over both thresholds

**Location:** `decision-record.md` Decision 6.

**Issue:** With the template values (`idle_grace_s` 30, `soft_idle_s` 300) the hard trigger qualifies first whenever the reading is at or over `threshold_tokens`.
Both knobs are operator-settable, though.
With `idle_grace_s` at or above `soft_idle_s`, the soft conditions hold first at a hard-cap reading, a deferrable soft cycle starts and records `soft`, and the session can defer a cycle the hard cap required.
That contradicts "the hard cap cannot be deferred".
The Decision never says which trigger wins when both qualify on one tick.

**Fix:** State that a reading at or over `threshold_tokens` is always a hard trigger: the soft path is considered only below the hard threshold, and a cycle started at a hard-cap reading records `hard` and ignores `DEFER`.
Add the case to the watcher acceptance line.

### [LOW] Strand-name acceptance criterion does not say it is a live check of shipped behaviour

**Location:** `decision-record.md` Acceptance criteria, the line beginning "After adopt, the orch strand carries".

**Issue:** No Decision or Scope item backs this line, and it needs none.
`orchSpec` already sets `Role` and `NameOverride` to `agentname.RoleOrch`, both `buildLaunchCmd` and `buildResumeCmd` already pass `--name "$LYX_STRAND_NAME"`, and `loom start` already records the caller's name as `parent` (commit 218ba2a68).
An adopt launch through `orchSpec` inherits all of it.
The line does not say so, and SendMessage delivery can only be observed live, so the plan writer cannot tell whether this is new work or verification, unit or smoke.
Decision 7's smoke list omits it.

**Fix:** Mark the line as a live-smoke check of the naming shipped with agent-naming, applied to an adopted session, and add it to Decision 7's smoke coverage.

### [NIT] Hub-review provenance in the decision record

**Location:** `decision-record.md` Decision 6 rationale "(hub review, 2026-10-02)"; Auto-mode assumptions "(revised by the hub review)" and the first bullet.

**Issue:** The parentheticals are deliberation history, already recorded in the support log's Review rounds and Question ledger Q9.
They carry no requirement, so removing them passes the completeness-before-leanness test.
The first auto-mode bullet says every pick is the writer's own, but Decision 6's natural-break rule came from the hub review.
The other auto-mode bullets each restate a Decision with its reason, which is what the section is for, so they stay.

**Fix:** Drop both parentheticals.
Reword the first bullet so it credits Decision 6's revision to the hub review.
