---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D11 and D4 (fix gate exhausted bullet)
    summary: Auto-accept after a fix-gate-exhausted round accepts a target whose gate failed, a bypass D11's bound does not name
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D3 and D4 (archive rules)
    summary: The acceptance record's fate on every archive exit is unstated, so a stale record can make the next review row skip the reviewer
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 (Bound line)
    summary: The empty-review skip trusts a parser that tolerates unknown frontmatter keys, so a misnamed findings key reads as zero findings and skips the fixer
---

### [BLOCKING] Auto-accept after a fix-gate-exhausted round accepts a gate-failed target

**Location:** `_lyx/discussion/decision-record.md` D11, with D4's "fix gate exhausted" bullet.

**Issue:** D4 sends a fix-gate-exhausted round to the Bouncer as a counted Stuck with an empty Path, after archiving the review and fixer report.
The fixer's edits, overlay or committed, stay in the target.
The Bouncer then resolves its round from the review files alone (`ResolveRound`), finds round N-1 judged, and replays its recorded CONTINUE through `settle` → `settleUnconverged` → `settleUndecided` (`internal/shedadapters/bouncer.go`).
When that replay finds the budget spent, today it escalates with cause budget, so the parent sees the state.
Under D11 the same replay has "budget spent and no recorded decision", so it auto-accepts round N-1: carry-over of N-1's findings, approve, commit, Done.
The target accepted is the one round N's fixer left behind after its gate (`discussion`, `plan` or `verify`) failed every attempt, for example a Webster tree whose verify failed.
D11's Bound names the circling cause, unreadable records, a failed seam and the flag, but not this path, so the bypass is unbounded.
Trace with `webster_max_bounces: 2`: seed Stuck (Bouncer count 1), round 1 judged CONTINUE (count 2), round 2 fix gate exhausted, Bouncer replays round 1 with `budgetReached` true, and auto-accept settles Done.
A related gap: with budget 1, a gate-exhausted round 1 makes the Bouncer's re-bounce Stuck hit Shed's generic budget block, so the run halts with no judged round, which neither D10's "1 gives exactly one" nor the `webster_max_bounces: 1` criterion states.

**Fix:** In D11, add to the Bound that auto-accept fires only when the Bouncer judged the round in the same generation with no gate-failed round after it.
For example, it fires only on a harvest or replay whose round is the highest round with a review on disk and no later archived gate-failed round, and otherwise escalates as today.
Name the case in the Bound, add an acceptance criterion ("with auto-accept true, a fix gate exhausted after the budget is spent escalates to the parent rather than accepting"), and state in D4 or D10 that a gate-exhausted round under budget 1 halts on Shed's generic block.

### [MEDIUM] The acceptance record's lifecycle on archive is unstated

**Location:** `_lyx/discussion/decision-record.md` D3 and D4.

**Issue:** D3 makes the acceptance record the deciding fact: "A round whose acceptance record is already on disk spawns nothing and hands on BudgetExempt".
But every archive the record names covers only "both round paths" (D4's fix-gate-exhausted bullet) or "the round's paths" (D3's reviewer gate failure), which today are the review and the fixer report (`archiveRound` in `internal/shedadapters/burler.go`).
If the record survives the archive of round N after a fix gate exhaustion, the next review-row call for round N finds the record, spawns no reviewer, and hands to the fix row, which then holds a record whose review is gone.
D4's "review changed after acceptance: hard error, as today" has the same gap: today that error goes through `failureExit`, which archives, and whether the record is archived with it, or the run wedges on the same digest mismatch at every resume, is not stated.
This is a requirement the plan writer needs and it is in no Decision.

**Fix:** In D3, state that the acceptance record is one of the round's paths: every archive of a round (orphan review, reviewer gate failure, fix gate exhaustion, review changed after acceptance) moves the record with the review and fixer report.
In D4, state that the "review changed after acceptance" hard error archives all three, so a resume re-reviews the round.

### [MEDIUM] The empty-review skip trusts a tolerant parse

**Location:** `_lyx/discussion/decision-record.md` D4, "Bound" line.

**Issue:** D4 bounds the skip to "an accepted review with zero findings of any severity", where the count comes from `ParseReview`.
`ParseReview` decodes the frontmatter without known-field checks (`reviewHeader` in `internal/burlerengine/verdict.go`), so `verdict: APPROVED` with findings under a misspelled key, such as `finding:`, parses as APPROVED with zero findings and passes the review gate.
Today the fixer still reads that review file and sees the findings in its prose.
Under D4 the fixer never spawns, the Go-authored report says there were none, and with auto-accept and a spent budget the round settles with nothing carried into `## Open risks`.
The focus directive names this case; D4 states no bound for it.

**Fix:** In D4 or D7, state that the review gate refuses a frontmatter whose `findings` is absent while the prose holds a `### [` finding block, or that refuses any top-level frontmatter key other than `verdict` and `findings`, so a zero-findings parse means the reviewer wrote none.
Add an acceptance criterion: a review with an unknown findings-like key or a prose finding block and no `findings` entries fails the review gate.

## Verdicts on the focus directive's questions

- Routing (D1, D3, D4): no row returns Done; every BudgetExempt and counted Stuck reaches the Bouncer through the fix row; the review and fix hand-offs are exempt, so the Bouncer's count is its own Stucks only, as `NewBouncer`'s budget rule says today.
  The pass-through and restart cases hold apart from F2.
  The fix-gate-exhausted path is F1.
- Attack surface (D4, D11): the skip is F3; the stale-flag and accept-after-escalation cases are bounded in D11's last paragraph; the Go-authored fixer report and the digest are bounded by D4's digest re-check, apart from F2.
- Completeness before leanness (D6, D7, D9): D6 names the write surface, the commit rule and the dispute rule as prompt contents, and D9 restates the dispute ground against the code; the existing stencils carry their text, which the pointer rule wants left in place.
  Not a defect.
- D10 against the criteria: the Decision, Constraints and criteria agree; darn's `review_max_bounces`, where 0 skips the review, is out of D10's scope by the Scope Out line, so the two key families do not contradict.
- Premises: D1 holds (`episodeStuckCount` in `internal/shedengine/run.go` ends the episode at any segment row's Done); D2 holds (`MultiLLMProducer` returns Done on a chair's success); D8 holds as far as the stencils show, since the channel is the fixer's optional `SendMessage` in `burler-template-fix-orchestrator.md`.
- Leanness: Scope Out, Auto-mode assumptions and Open risks carry scope and assumption statements the plan writer needs, and the support log's Interview and Question ledger hold deliberation only.
  No relocation finding.
