---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:33-37,117
    summary: "D1 tells the judge to use keys the facts list as open in an earlier round, but the facts file lists only keys open in two or more earlier ledgers, so at round 2 it is always empty and a key open once earlier reads as fresh; the Open risks claim that the recurring-key list guards this is false"
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:42-46,123-126
    summary: "D2 idempotency rests on an entry naming segment and round, but a re-entered segment archives its run directory and restarts at round 1, and a CONVERGED round is never replayed, so the entry identity collides across generations and a failed Approve or Commit never reaches a replayed settle"
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:48,128
    summary: "D2 and the acceptance criteria claim the appended bytes are committed by the end of Done in all three segments but name no mechanism for Plan-Review, whose commit seam stages only the plan directory, and the discussion directory sits outside it"
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:40-49
    summary: "D2 adds a Go write path into a hand-written section of the record that later reviews use as their answer key, yet states no bound, unlike D1, D5 and D6: placement inside Open risks, preservation of existing bytes, behaviour on a missing heading, and growth across generations are all left open"
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:6
    summary: "Goal states the convergence rule with no round floor, so read alone a round-1 review with only MEDIUM design findings converges, which D1, the Auto-mode assumptions and the Acceptance criteria all forbid"
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/support-log.md:12
    summary: "What a carry-over entry denotes (a finding the round's fixer addressed and no fresh reviewer has seen, versus an unfixed one under a parent accept) is stated only in the support log, which Plan-Write never reads; D2 lists status open entries without saying so"
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:106-109
    summary: "Auto-mode assumptions 1, 3 and 4 restate D1's rationale, D2's accept and sink clauses and the support log's rejected alternatives, so the same reasoning is said in up to three places"
---

## Verdicts on the focus entries

- D2 and the writers of `## Open risks`: findings F2, F3, F4.
- D1 against Goal and Acceptance criteria: findings F1, F5.
- D5 and D6: not defects, see below.
- Cross-statement consistency (D4 empty-inherits, D3 budget, Webster-Review commit claim): D3 and D4 not defects; the Webster-Review claim is F3.
- Relocation in both directions: findings F6, F7.

### [MEDIUM] D1's "facts list as open in an earlier round" cannot tell a fresh key from a recurring one

**Location:** `_lyx/discussion/decision-record.md` D1 (Decision and Bound) and the third Open risk.
**Issue:** D1 makes the round-2-or-later rule hinge on "a key the facts list as open in an earlier round".
`computeRoundFacts` in `internal/shedadapters/bouncerfacts.go` fills the facts' recurring-key list from `recurringKeys(runDir, round-1)`, which returns keys open in two or more distinct ledgers among rounds 1 through N-1.
The judge's own ledger for round N does not exist when the facts are rendered.
At round 2 only ledger 1 exists, so the list is always empty ("No ledger key recurs across rounds").
At round 3 a key open in round 2 alone, and re-raised in round 3, is also absent.
So the facts file marks as "fresh" exactly the keys D1 means to continue on, and D1's Bound ("never lets through ... a recurring or reopened MEDIUM `design` key") does not hold for a key that was open one round earlier.
The third Open risk names "the facts' recurring-key list and the lossless ledger carry-forward rule" as the existing guard; the list is not a guard at the first round the new rule applies.
The judge does read the previous ledger, and `circlingEvidence` already computes "open in this ledger and an earlier one" after the judge writes, but D1 names neither as the source of the "open in an earlier round" test.
**Fix:** State in D1 where the "open in an earlier round" set comes from (for example a facts list of keys open in any earlier ledger, rendered by Go, or the judge comparing against the previous ledger) and say whether the facts change is in scope.
Correct the third Open risk to name the guard that does exist.

### [MEDIUM] D2's idempotency key and retry story do not match how the Bouncer re-enters

**Location:** `_lyx/discussion/decision-record.md` D2 (fourth and fifth sentences) and the Acceptance criteria on a replayed settle.
**Issue:** D2 says the entry "names its segment and round, and a re-call over a round whose entry already exists appends nothing", and the criteria require that "a replayed settle of the same round appends nothing a second time".
In `internal/shedadapters/bouncer.go` `Call`, a CONVERGED verdict on disk at entry is a settled generation: the run directory is archived and the segment re-seeds from round 1 ("a CONVERGED replay no longer exists"), and the same clear applies to a settled accept, which the doc comment says covers "a resume after a failed Commit seam".
Two consequences the record does not settle:
- A failed Approve or Commit after the append is followed by a clear and a new generation, not a replayed settle, so the entry from the abandoned generation stays in the record and the new generation's round 1 either collides with its identity (appends nothing, losing the new carry-over) or duplicates it, depending on the unstated identity rule.
  The same collision follows any rework generation or `goto`, where Plan-Review and Webster-Review restart at round 1.
- On a circling accept, `settleUnconverged` writes the settled record before Approve.
  If the append runs after that write and fails, the next call sees a settled accept, clears and re-seeds, and the carry-over is lost; if it runs before, a replay is a real pending accept.
  The record says only that the append "runs before Approve and Commit".
**Fix:** State the entry's identity so that it distinguishes generations (for example including the archived-run or generation marker, or the review file's path), where the append sits relative to the accept's settle write, and what the criterion "replayed settle" means given the clear-and-re-seed behaviour.

### [MEDIUM] D2's commit claim has no mechanism for Plan-Review

**Location:** `_lyx/discussion/decision-record.md` D2 (last sentence) and the criterion "committed by the end of the `Done` transition in Discussion-Review, Plan-Review and Webster-Review".
**Issue:** The record flags that Webster-Review "has no commit seam today" but treats Discussion-Review and Plan-Review as covered.
`commitDiscussion` stages the whole discussion directory, so Discussion-Review is covered.
Plan-Review's seam is `CommitPlan` in `internal/loomcli/wiring.go`, whose pathspec is `planparser.PlanDirRel()` only; the decision record lives under the discussion directory and the append would stay uncommitted dirt after Plan-Review's `Done`.
The Webster-Review row has no seam by construction, and its fixer commits source only.
The record leaves open whether each segment gets a new seam, a widened pathspec, or one shared commit of the discussion directory, and the acceptance criterion cannot be met without that choice.
**Fix:** Add a Decision or Constraint naming the commit route per segment (which seam stages the decision record, or that Go commits it in the settle path), so Plan-Write does not have to discover that two of three segments lack it.

### [MEDIUM] D2 introduces a write edge with no stated bound

**Location:** `_lyx/discussion/decision-record.md` D2.
**Issue:** D1, D5 and D6 each carry a `Bound:` line; D2, which adds a Go write path into a file that was hand-written and approved, carries none.
After Discussion is approved the record is the answer key of Plan-Review, the sole input of Plan-Write, and context for Describe and rework.
The record does not state:
- where inside `## Open risks` the entry lands and that every other byte is unchanged (only the no-entry case is pinned byte-identical);
- what happens when the heading is absent or the result fails `Validate` (the Notes point at `AppendDecision`, which restores the prior bytes, but Notes are non-binding and `Constraints` say only that the append is a `discussionparser` function);
- that entries are informational and not requirements, so a later Plan-Review does not treat a carried finding as a decision to implement;
- whether entries accumulate across rework generations without pruning, which is the discussion-file growth the rubric warns about.
The `discussion gate` section contract itself is not at risk: `Validate` checks heading presence only, and the append adds no heading.
**Fix:** Add a `Bound:` line to D2 covering placement and byte preservation, the missing-heading and failed-validate behaviour, the entries' informational status, and what bounds growth.

### [MEDIUM] Goal omits the round floor that D1 requires

**Location:** `_lyx/discussion/decision-record.md` Goal, first paragraph.
**Issue:** Goal says a segment converges "once its latest round carries no BLOCKING finding and no MEDIUM-or-worse `design` finding on a key already open in an earlier round".
In round 1 no key is open in an earlier round, so by the Goal alone a first round with only MEDIUM `design` findings converges.
D1, the first Auto-mode assumption and the first Acceptance criterion say round 1 keeps today's rule.
The Goal is the plan writer's first read and states a different rule.
**Fix:** Add "from round 2 on" to the Goal's rule, naming round 1's unchanged rule in the same sentence.

### [LOW] The meaning of a carry-over entry lives only in the support log

**Location:** `_lyx/discussion/support-log.md` Interview item 4, against `_lyx/discussion/decision-record.md` D2.
**Issue:** Item 4 records that the reviewed round's findings "are already fixed when the judge rules", so a carry-over lists fixed-but-unreviewed findings.
Plan-Write never reads the support log, and D2 says only "every `status: open` entry".
The judge stencil defines `open` as "still applies", which does not say fixed, and under a parent accept an entry may be unfixed.
A reader of `## Open risks` cannot tell which the entry means.
Per the completeness test the requirement moves into the record first; the interview narrative stays.
**Fix:** Add one clause to D2 stating what an entry denotes in each of the two `Done` paths, and leave item 4 as the deliberation.

### [LOW] Auto-mode assumptions repeat Decisions and support-log content

**Location:** `_lyx/discussion/decision-record.md` Auto-mode assumptions 1, 3 and 4.
**Issue:** Assumption 1 restates D1's rationale and the support log's first rejected alternative.
Assumption 3 restates D2's "or a recorded circling `accept`".
Assumption 4 restates D2's closing paragraph and the support log's rejected "separate carry-over sink per segment".
The assumptions are worth keeping as flags of auto-picks, but the reasoning belongs once, in the Decision, with the rejected-alternative trail in the support log.
**Fix:** Cut each assumption to the pick itself and a pointer to its Decision; keep assumptions 2, 5 and 6, which add information.

## Checked, not defects

- D3: the bounce count at the judge of round N is the Bouncer row's episode stuck count (seed plus N-1 CONTINUE), so a budget of 3 escalates at round 3, and `settleUndecided` makes a spent budget win over CIRCLING; the stated outcome and the unchanged checkpoint hold.
- D4: empty-inherits is stated alike in Scope, D4 and the Acceptance criteria; `validateModelSpecList` rejects an empty list but accepts the one-entry `""` the template carries.
- D5: the bound ("only delays the fixer's start; skips no gate", no fixer when the reviewer half ends without an accepted review) is enough, and the ordering clause of `PATTERN-review-round` is preserved.
- D6: the new edge is bounded to a Bouncer-row await with an unsettled decision file, gives no decision and spawns no driver; the other refusals keep their way forward.
- D7 and the Constraints cover the amended `PATTERN-review-round` clause; the stencil ban on exact signatures and `file:line` citations was not applied against the record, and the Notes for the plan writer are optional.
