---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md D2 (Which rounds ask, the generation parenthetical)
    summary: D2's generation premises are false; round numbers restart at 1 in a new generation, and a goto back to Discussion-Write or a resume on an unsettled perch starts no generation, so a rewritten record can skip the parent
    fix: In D2, replace the parenthetical with what the code does and where it is read. The generation number is the run directory's generation-record.json Number, read by the Bouncer (internal/shedadapters/settlerecord.go), and a missing record reads as generation 1. A new generation starts only when a settled perch's subject changed (startNextGeneration in internal/shedadapters/amendment.go). It archives the run directory and seeds round 1 again, so round numbers restart. A goto or a resume restarts nothing (internal/shedadapters/bouncer.go:238, 367-369). Then change the asking rule so a rewritten record still reaches the parent. Recommended form, a round also asks when the decision record changed outside Burler-Fix since the last ask, for example when Discussion-Write ran after the last ask's round. State where that is read from, with evidence. Add an acceptance criterion for a goto back to Discussion-Write on an unsettled perch after a satisfied parent answer.
    origin: lens:existing-paths
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D3 (ask record keyed by generation and round)
    summary: The Bouncer's clear path re-seeds round 1 under the same generation number, so a (generation, round) key finds the old done ask, sends no ask and replays stale parent findings
    fix: In D3, name the clear path (entryClear in internal/shedadapters/bouncer.go:458-470, which carries generation-record.json through CarryRecords). Key the ask record so a clear cannot reuse it. Recommended form, add the run directory's round-1 seed identity to the key, such as the round-1 focus file's write time, or have the ask record store the decision record's commit and treat an ask on another commit as absent. Add an acceptance criterion for a clear after a done ask.
    origin: lens:existing-paths
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D5 (deadline cap)
    summary: D5 names no cap formula, no room for the chair to consolidate after the wait, and no outcome when the cap leaves no time or review_timeout_min is 0, leaving the cap for Plan to find out
    fix: In D5, state the formula with evidence from CapAssistTimeout (internal/loomengine/review.go:64-84). The chair's timeout is review_timeout_min, or shuttle's run_timeout_min when that key is 0. The deadline is the ask time plus the lesser of discussion_review_parent_timeout_min and that timeout minus the start allowance, one chair keep-warm await cap and a named consolidation margin. State the no-room outcome. Recommended form, the round sends no ask, logs one Warn naming both keys, and records the ask as not sent with that reason, which the review's line names. Add an acceptance criterion for the no-room case.
    origin: lens:bound-breaker
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D3 (no live orch sets no-parent) and acceptance criterion D3 (no parent orch to reach)
    summary: NotifyParent falls back to the prime's orch when no orch above the pair is live and returns posted false with no error when no strand is recorded, so no-parent as D3 states it is never reached and a dead parent leaves the ask open until timeout
    fix: In D3, state that a NotifyParent return of posted false with a nil error sets no-parent, as a post error does. State that a dead orch the post still reaches, the prime's included, ends the ask in timeout at the deadline (internal/orchcli/notify.go:41-49, 193-195). Reword the D3 criterion to cover the case of no recorded orch strand, and add one for an unreachable orch that ends in timeout.
    origin: lens:bound-breaker
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D4 (answer verb writes for the slug's current round) and D5 (ask lifecycle)
    summary: The answer verb names no ask, so a late answer lands on a newer ask about a different record, and nothing closes an ask left open when the chair ends without running await
    fix: In D3 and D4, make the request line carry the ask's generation and round, and have the verb take them as a required flag. The verb refuses an ask that does not match the open one, with a way forward naming lym loom decision add. In D5, have Go set an ask still open to timeout once the review table ends, on success or failure. Add acceptance criteria for a mismatched answer and for an ask the table ended with still open.
    origin: lens:interactions
  - id: F6
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D4 (--findings-file) and D7 (parent severity and class)
    summary: D7 keeps the parent's severity and class, but D4 defines the findings file only as markdown findings naming what to change, so D7's mapping has no defined input
    fix: In D4, define the findings file's shape. Recommended form, one item per finding with a required change and optional severity and class fields in the review format's vocabulary. Have the verb refuse a file with no item. In D6, have the orch-role section tell the fork to use that shape. Extend the D7 acceptance criterion to read a finding's given severity and class.
    origin: lens:interactions
  - id: F7
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D3 (ask through NotifyParent) and D6 (section for mail from shortname LOOM)
    summary: NotifyParent signs as the caller's strand when the process carries a non-orch strand name, such as an LLM driver running the review row, so the ask need not come from the LOOM program the D6 section keys on
    fix: In D6, key the orch-role section on the line's [discussion-review request] opener, from any sender creel stamps as one of the run's own sessions or the hub's LOOM program. Cite the signing rule in internal/orchcli/notify.go:188-218. Alternatively, in D3, post the ask with a program-signed notice that ignores the strand claim, and name it.
    origin: lens:walkthrough
  - id: F8
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md D2 (each round reads the key) and D3 (restart keeps the ask)
    summary: A key flipped off between a round's ask and a restart of its review row leaves the record silent on whether the restarted chair awaits and consolidates the ask already sent
    fix: In D2, state that the key decides only whether a new ask is sent, and that an ask already sent in the round is awaited and consolidated whatever the key now reads.
    origin: lens:interactions
  - id: F9
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md D3 (restart finds the ask record)
    summary: D3 does not order the ask post and the ask-record write, so a crash between them either never mails the parent or mails it twice
    fix: In D3, state the order and its crash outcome. Recommended form, write the ask record with state pending-post first, post, then set it open. A restart that finds pending-post posts again, accepting a possible second mail.
    origin: lens:bound-breaker
  - id: F10
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md D3 and D4 (ask receiver and caller guard)
    summary: The ask's receiver and the answer guard's parent are each resolved by liveness at call time, so an orch that comes up or goes down between ask and answer gets the ask and is then refused
    fix: In D3, have the ask record store the orch strand the ask was posted to. In D4, have the caller guard also admit that recorded strand for that ask. State this as part of D4's bound.
    origin: lens:existing-paths
  - id: F11
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md D4 (caller guard) and D6 (Bound)
    summary: D6's bound implies D4's guard narrows the fork, but the guard admits any caller with the parent's strand name and guards only against a mistaken call, so the fork can run every orch-guarded verb on every run
    fix: In D4's bound, state that the guard checks an unverified strand name and stops only a mistaken call (internal/loomcli/decision.go:11). In D6's bound, state that the fork's reach equals the orch's, including decision replace and decision store on any run, and is limited to the two verbs and the slug by instruction alone.
    origin: lens:bound-breaker
  - id: F12
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md acceptance criterion D4 (refusals)
    summary: D4 refuses an answer to a no-parent ask, but the criterion reaches only the done and timeout refusals, so that refusal has no reaching test as PATTERN-refusal-way-forward requires
    fix: Add no-parent to the D4 refusal criterion.
    origin: lens:interactions
  - id: F13
    severity: LOW
    class: scope
    location: _lym/discussion/decision-record.md acceptance criteria D5
    summary: D5's review line saying the parent review did not arrive has a criterion only for no-parent, none for timeout
    fix: Add to the D5 criteria that a round whose ask ends in timeout writes the review line naming timeout as the reason.
    origin: handler
  - id: F14
    severity: NIT
    class: consistency
    location: _lym/discussion/decision-record.md D5 (A new verb, not lym shuttle await on a marker)
    summary: D5 argues a rejected alternative in the decision record, which belongs in the support log's Rejected alternatives
    fix: Cut the sentence starting "A new verb, not `lym shuttle await` on a marker" from D5. Its requirement is already in D5's bullets, which say the await sets timeout at the deadline. Add a Rejected alternatives bullet to support-log.md for waiting with lym shuttle await on a marker, with that reason.
    origin: handler
---

### [BLOCKING] D2's generation rule rests on false premises

**Location:** `_lym/discussion/decision-record.md` D2, "Which rounds ask".
**Issue:** `startNextGeneration` archives the run directory and `seedCall` seeds round 1 again (`internal/shedadapters/amendment.go:42-59`, `internal/shedadapters/archive.go:67-76`, `internal/shedadapters/bouncer.go:1621-1651`), so round numbers restart rather than carry on.
A goto back to Discussion-Write over an unsettled round, and a resume, start no generation (`internal/shedadapters/bouncer.go:238`, `:367-369`).
After a satisfied parent answer, the rewritten record then reaches a round that does not ask, which breaks D2's own purpose that every rewritten record goes to the parent.
The record also never says the generation is read from `generation-record.json`, which leaves that fact to Plan.
The existing-paths, bound-breaker and walkthrough lenses all found this, and so did the handler.

### [MEDIUM] The Bouncer's clear reuses the ask key

**Location:** `_lym/discussion/decision-record.md` D3.
**Issue:** `entryClear` archives and re-seeds round 1, and `CarryRecords` carries the old generation record forward (`internal/shedadapters/bouncer.go:458-470`, `internal/shedadapters/settlerecord.go:261-282`).
The new round 1 finds the old `done` ask: it sends no ask, and the chair keeps stale parent findings.

### [MEDIUM] D5's cap is left for Plan

**Location:** `_lym/discussion/decision-record.md` D5.
**Issue:** "Capped … as `assist_reviewer_timeout_min` is capped" names no formula.
`CapAssistTimeout` subtracts a start allowance and an await cap, and it has a no-room branch (`internal/loomengine/review.go:64-84`).
D5 leaves no room for consolidation after the wait, and states no outcome for the no-room case or for `review_timeout_min: 0`.
This is a deferred research question.

### [MEDIUM] `no-parent` is unreachable as stated

**Location:** `_lym/discussion/decision-record.md` D3 and its criterion.
**Issue:** `NotifyParent` falls back to the prime's orch, and it returns `posted=false` with no error when no strand is recorded (`internal/orchcli/notify.go:41-49`, `:193-195`).
D3 maps neither case, and a dead parent ends the ask in `timeout`, not `no-parent`.

### [MEDIUM] The answer verb names no ask, and an open ask is never closed

**Location:** `_lym/discussion/decision-record.md` D4 and D5.
**Issue:** The verb answers "the slug's current round", so a late answer to generation 1's mail can land on generation 2's open ask.
Only the await sets `timeout`, so an ask stays open when the chair dies or skips the await, and a late answer is then accepted for a round that has already ended.

### [MEDIUM] The findings file has no shape for D7 to read

**Location:** `_lym/discussion/decision-record.md` D4 `--findings-file` and D7.
**Issue:** D7 maps the parent's severity and class, but D4 gives the file no place for either, so the chair has to guess.

### [MEDIUM] The ask need not come from the LOOM program

**Location:** `_lym/discussion/decision-record.md` D3 and D6.
**Issue:** `notifyPrime` posts as the caller's strand when the strand env names a non-orch strand (`internal/orchcli/notify.go:188-218`).
A row run from an LLM driver's pane then signs the ask as the driver, and D6's section, keyed on `<shortname>:LOOM`, misses it.

### [LOW] A key flip on restart is unspecified

**Location:** `_lym/discussion/decision-record.md` D2 and D3.
**Issue:** When the key is turned off after a round's ask and the review row then restarts, the record does not say whether the restarted chair awaits and consolidates the ask already sent.

### [LOW] The order of the ask post and the ask-record write

**Location:** `_lym/discussion/decision-record.md` D3.
**Issue:** The record does not order the post and the record write, and either order has a crash outcome the record does not state.

### [LOW] The parent is re-resolved between the ask and the answer

**Location:** `_lym/discussion/decision-record.md` D3 and D4.
**Issue:** The ask's receiver and the guard's parent are each resolved by liveness at call time (`internal/orchcli/notify.go:44`, `internal/loomcli/decision.go:685`, `:1695`), so they can differ.

### [LOW] D6's bound overstates D4's guard

**Location:** `_lym/discussion/decision-record.md` D4 and D6.
**Issue:** The guard admits any caller named as the parent and stops only a mistaken call (`internal/loomcli/decision.go:11`, `:693`).
The fork's reach is therefore the orch's whole reach.

### [LOW] The `no-parent` refusal has no criterion

**Location:** `_lym/discussion/decision-record.md`, the D4 criterion.
**Issue:** The `no-parent` refusal has no reaching criterion, which `PATTERN-refusal-way-forward` requires of every refusal.

### [LOW] The timeout review line has no criterion

**Location:** `_lym/discussion/decision-record.md`, the D5 criteria.
**Issue:** The line saying the parent review did not arrive is covered for `no-parent` only, not for `timeout`.

### [NIT] A rejected alternative is argued in D5

**Location:** `_lym/discussion/decision-record.md` D5.
**Issue:** The "not `lym shuttle await`" sentence is a rejected alternative.
The completeness-before-leanness test holds: its requirement already sits in D5's bullets.

## Notes

- Focus item D2 against issue #87: not a defect.
  The departure is stated, and so is its bound: the Bouncer's approval, `discussion_max_bounces: 0`, auto-accept, the hard cap and accept.
- Focus item D4/D6 guard: the bound gap is F11.
  The fact that a fork passes the guard holds: the guard compares strand names (`internal/loomcli/decision.go:693`), and `lym loom` has no fork check.
- D6's `decision add` during Discussion-Review is refused only while Discussion-Write runs (`internal/loomcli/decision.go:893`).
  The darn record check is skipped on the discussion recipes, so the verb works mid-round.
