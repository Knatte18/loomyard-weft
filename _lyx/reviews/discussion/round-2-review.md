---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:33
    summary: D1 matching rule leaves the leading-part length open, so the bound that an extra Enter never lands on a draft of other text cannot be tested
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:31
    summary: D1 does not say when the input box is read after Enter, so a capture taken before the box redraws reads a normal send as still holding the text and sends an extra Enter
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:90
    summary: D9 removes the skills step unconditionally while R5 makes the removal conditional on the plan writer confirming A2, and D9 never says it overrides the present statement in orchengine doc.go
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:83
    summary: D8 specifies the more-than-one case and the exactly-one-and-read case only; a boundary with one later turn end that is not the one being read, or none, is left undefined and has no acceptance criterion
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:143
    summary: D12 and its Constraint claim PATTERN-gate-self-check-parity, whose rule is a gate plus a CLI self-check verb, but the decision adds no verb and does not say how the pattern is met
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:141
    summary: The Constraints say D5's strand removals are logged, but D5 does not say so
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:146
    summary: The Constraints cite D14 for PATTERN-test-isolation and PATTERN-testkit, which D14 does not mention
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:178
    summary: The D6 criterion pins only a driver that died mid-producer, while D6's bound and the support log's rejected alternative name a driver death between steps
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:149
    summary: The test-prune ordering is stated in the Added-after-Discussion decision, again as a Constraint, and again as R4
  - id: F10
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:84
    summary: D8 rationale says a session that ran turns after a compaction has re-read what it needed, which contradicts D9's statement that a compaction loses the role file
---

### [MEDIUM] D1's matching rule has no testable length for the leading part

**Location:** `_lyx/discussion/decision-record.md:33-35`
**Issue:** The rule says the box holds the sent text when its normalized content "contains the sent text or a bounded leading part of it".
The bound is not given, and "or a leading part" makes the whole-text alternative redundant.
The Bound line claims an extra Enter "never lands on ... a draft of other text", which holds only if the leading part is long enough to tell a draft from the sent text.
A test cannot be written against an unstated length.
A text long enough to collapse into a placeholder is never confirmed, so a lost Enter on exactly those long prompts is caught only by D2's pacing; the Decision states the fallback but not that residual.
**Fix:** State the leading part as a fixed number of normalized characters, normalized the way `sendVerified` normalizes its needle, and name the collapsed-placeholder residual as an open risk.

### [MEDIUM] D1 does not say when the box is read

**Location:** `_lyx/discussion/decision-record.md:31`
**Issue:** The box is read "after the text is seen in the pane", with no interval between Enter and the read.
Immediately after Enter the box can still show the text, so a normal send reads as still holding it and gets an extra Enter, onto an empty box or a running turn.
That contradicts the Bound and the acceptance criterion that a box which clears at once gets no extra Enter.
**Fix:** State that each read follows a provider-owned settle after the Enter, the same kind of delay D2 introduces, so the criterion is testable with an injected clock.

### [MEDIUM] D9 and R5 disagree, and D9 does not name the statement it overrides

**Location:** `_lyx/discussion/decision-record.md:88-91` and `:166`
**Issue:** `internal/orchengine/doc.go` states that a clear, a compaction and an auto-compaction each lose the session's skills.
D9 reverses that for compactions on A2, whose only source is the board entry, and updates `doc.go` without saying it overrides it.
D9's Bound accepts that a session may lack a skill until the next `/clear`, which makes the skip final.
R5 then tells the plan writer to confirm A2 against the deployed version "before removing the step", which makes it conditional, and gives no fallback.
The acceptance criterion states the removal unconditionally.
**Fix:** Keep D9 final: add the premise and its source to D9, say it supersedes the `doc.go` statement, and reduce R5 to the accepted risk without the confirm-first clause.

### [LOW] D8 leaves two cases undefined

**Location:** `_lyx/discussion/decision-record.md:82-84`
**Issue:** D8 defines a reload for exactly one later turn end that is the one being read, and a baseline move for more than one.
A boundary with exactly one later turn end that is not the one being read, which a restarted watcher on an old cursor can hit, and a boundary with none are not covered.
The acceptance criteria pin the other two cases only.
**Fix:** State that in those cases the watcher types nothing and leaves the baseline, so a later turn end evaluates the boundary again, and add one acceptance line for it.

### [MEDIUM] D12 claims a pattern whose verb it does not add

**Location:** `_lyx/discussion/decision-record.md:113` and `:143`
**Issue:** `PATTERN-gate-self-check-parity` pairs a gate's closure with a CLI self-check verb and says adding a gate adds its verb in the same task.
D12 adds a gate entry and pairs it with the post-gate parse, not a verb, and the scope forbids new verbs.
Both the Rationale and the Constraint present the pattern as met.
**Fix:** State the actual position in D12: the gate and the post-gate parse share `ParseReview`, no standalone verb is added because of the no-new-verbs rule, and the in-session re-prompt carries the same parse error.
Reword the Constraint so it claims only the shared function.

### [LOW] D5 does not say its removals are logged

**Location:** `_lyx/discussion/decision-record.md:63-65` and `:145`
**Issue:** The `PATTERN-spawn-observability` constraint says strand removals in D4 and D5 are logged, and only D4 says so.
**Fix:** Add "and is logged" to D5.

### [LOW] Constraints cite D14 for patterns it does not state

**Location:** `_lyx/discussion/decision-record.md:146`
**Issue:** D14 names `PATTERN-test-speed` and `PATTERN-test-economy` only.
**Fix:** Cite `PATTERN-test-isolation` and `PATTERN-testkit` in D14, since the CLI-wiring tests need the hermetic and tmux entry points.

### [LOW] The D6 criterion is narrower than D6's bound

**Location:** `_lyx/discussion/decision-record.md:71` and `:178`
**Issue:** The support log rejects dropping the trigger because that "hides a real driver death between steps".
D6's bound and the criterion cover only a driver that died mid-producer.
**Fix:** Widen the criterion to a driver that died before it recorded the handoff voucher, mid-producer or between a finished step and the voucher.

### [NIT] The prune ordering is stated three times

**Location:** `_lyx/discussion/decision-record.md:130-134`, `:149`, `:165`
**Issue:** The decision carries the whole requirement; the Constraint bullet and R4 repeat it.
**Fix:** Delete the Constraint bullet and R4.

### [NIT] D8's rationale contradicts D9

**Location:** `_lyx/discussion/decision-record.md:84`
**Issue:** The rationale says the session has "already re-read what it needed", while D9 says the compaction loses the role file and the pointer step restores it.
**Fix:** Rewrite the rationale to say that a reload typed after turns that ran without it no longer matches the compaction it answers; R3 already covers the role file reaching the session later.

## Focus verdicts

- D9: states what it lets through and its bound; the premise conflicts with `doc.go` and R5 (F3).
- D4: the removal set is stated exactly in the probe's own verdicts (respawn-eligible, with a live strand); a start the probe refuses, through an errored candidate or several attachable ones, removes nothing.
  Not a defect.
- D1: bound stated, matching not testable (F1, F2).
- D8: agrees with the criteria and R3 for the restarted watcher and several turn ends at once; two cases are open (F4).
- D3, D11: bounded; D6 bounded, criterion narrow (F8).
