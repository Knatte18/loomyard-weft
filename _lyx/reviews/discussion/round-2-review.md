---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D4 (Bound and skipped-Webster lines), D11 Bound
    summary: The empty-review skip runs no fix gate on the re-review of a round whose fix gate was exhausted, so a target whose gate failed every attempt reaches the judge as clean and can be auto-accepted or converged.
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D4 bullet list
    summary: D4 lists the fix row's exits but not a fixer that dies twice or times out, so whether that exit keeps the accepted review and acceptance record or archives them with the fixer report is left open.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D7 third sentence
    summary: The prose-to-frontmatter match does not make a zero-findings parse trustworthy when an APPROVED review puts its findings under a misnamed key and writes no prose blocks; refusing unknown frontmatter keys at the current-round gate closes it.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: D10's budget-of-1 gate-exhaustion rule, a halt for a human whatever the auto-accept flag, has no acceptance criterion pinning it.
---

### [BLOCKING] The empty-review skip runs no gate after a fix gate was exhausted

**Location:** `_lyx/discussion/decision-record.md` D4 (the `Bound:` line and "A skipped Webster round runs no verify gate, since nothing changed"), and D11's `Bound:` line.

**Issue:** D11 says a fix gate exhausted in a round leaves its "gate-failed edits ... in the target".
D4's gate-exhausted bullet archives the review and fixer report and hands to the Bouncer, which replays the prior round's CONTINUE as a counted Stuck (`settleUndecided` in `internal/shedadapters/bouncer.go`) and sends the same round number back to review, since `ResolveRound` no longer sees that round's report.
If that re-review has zero findings, D4 spawns no fixer and runs none of the row's gates, so the `verify` (or `discussion`/`plan`) gate that failed every attempt never runs again over the gate-failed edits.
"Nothing changed" is false here: the tree changed since the last passing gate.
The review row's own gate (D5) is only the review-parse and `fix` check.
The judge then harvests a round whose fixer report says no findings.
Concrete path with no parent involved: budget N, round N-1's fix gate exhausts, the Bouncer replays round N-2's CONTINUE (count reaches N), round N-1 is re-reviewed with zero findings, the skip writes the Go report, the judge harvests round N-1, `budgetReached` is true, and D11's auto-accept settles Done over a target whose gate failed every attempt and never ran again.
A CONVERGED verdict on that round settles Done the same way without auto-accept.
Today's single row always runs the fixer and its gates after a review, so this is a guard the split removes, and D4's bound does not state or limit it.
D11's harvest-only bound covers the replay after the exhaustion but not the harvest of the re-reviewed round that follows it.
No acceptance criterion covers it.

**Fix:** In D4, replace the zero-findings bullet and the "A skipped Webster round runs no verify gate, since nothing changed" line with: on zero findings the fix row still runs the row's gates over the unchanged target; if they pass it writes the Go-authored fixer report and hands to the Bouncer BudgetExempt; if they fail it spawns the fixer with the gate's findings as its work, held on the same gates as a round with findings.
Extend D4's `Bound:` line to say the skip lets through only an accepted zero-findings review over a target whose row gates pass.
Add an acceptance criterion: after a fix gate is exhausted, a re-review of that round with zero findings runs the row's gates, and a failing gate spawns the fixer instead of reaching the judge as clean, so neither a CONVERGED verdict nor an auto-accept settles over it.
Amend the existing zero-findings criterion from "runs no verify gate" to "runs the row's gates and spawns no fixer when they pass".

### [LOW] The fix row's died or timed-out fixer exit is unstated

**Location:** `_lyx/discussion/decision-record.md` D4 bullet list.

**Issue:** D3 states the reviewer's died-twice exit and D4 lists the fix row's exits (findings present, zero findings, report on disk, gate exhausted, review changed, no acceptance record), but none covers a fixer that dies twice, times out, or ends with an unrecognized outcome.
Today's single row archives both round paths on that exit, which forces a re-review.
Under the split the fix row could instead archive only the fixer report and keep the accepted review and its acceptance record, so a resume re-spawns the fixer without re-running the reviewer, as the restart criterion already does for a killed fixer.
D3's "every archive of a round ... moves it" keeps either choice free of a stale record, but the choice itself is the plan writer's to guess.

**Fix:** Add a D4 bullet: "fixer died twice, timed out, or ended with an unrecognized outcome: hard error, as today, archiving the fixer report only and keeping the accepted review and acceptance record, so a resume re-spawns the fixer" (or, if re-review is wanted, archiving all three paths), and add the matching restart case to the restart acceptance criterion.

### [LOW] A misnamed findings key with no prose blocks still parses as zero findings

**Location:** `_lyx/discussion/decision-record.md` D7, third sentence.

**Issue:** `ParseReview` decodes the frontmatter into `reviewHeader` without known-field checks (`internal/burlerengine/verdict.go`), so findings under a misspelled key decode as zero findings.
D7's one-for-one check catches that only when the reviewer also wrote `### [SEVERITY]` prose blocks.
An APPROVED review with findings under a misnamed key and no prose blocks passes both checks as zero findings, and D4 skips the fixer; a BLOCKING verdict is already refused with zero BLOCKING findings, so the gap is the APPROVED case.
D7's claim that "a zero-findings parse means the reviewer wrote no finding, never findings under a key the tolerant frontmatter decode ignores" does not hold for that case.
Round 1's fixer passed on refusing unknown keys because `verdict.go` records the tolerant decode as deliberate, calling agent-written header metadata "harmless noise".
That rationale predates the skip: once D4 reads zero findings as "spawn no fixer", a misnamed findings key is no longer harmless, and scoping the refusal to the current round's review gate leaves the tolerant decode in place for hydration and every other reader.

**Fix:** In D7, have the current-round review gate also refuse any top-level frontmatter key other than `verdict` and `findings` (prior-round hydration keeps the tolerant decode), and state that this, with the prose match, is what makes a zero-findings count trustworthy for D4.
Extend the misspelled-key acceptance criterion to cover a review with a misspelled findings key and no prose block.

### [LOW] No criterion pins the budget-of-1 gate-exhaustion halt

**Location:** `_lyx/discussion/decision-record.md` `## Acceptance criteria`.

**Issue:** D10 states that under a budget of 1 a fix gate exhausted in round 1 leaves no judged round, so the Bouncer's re-bounce Stuck meets Shed's generic budget block and the run halts whatever the auto-accept flag.
The premise holds: the seed call's Stuck is counted, and with round 1's paths archived `ResolveRound` returns 0, so `Call` takes the re-bounce branch and returns a counted Stuck that `episodeStuckCount >= 1` blocks.
It is the auto-accept bound for the hub's own target setting (`webster_max_bounces: 1`, `webster_auto_accept: true`), but no criterion pins it, so a change that made the re-bounce exempt or auto-accept it would pass every listed criterion.

**Fix:** Add an acceptance criterion: with `webster_max_bounces: 1` and `webster_auto_accept: true`, a fix gate exhausted in round 1 halts the run blocked for a human, with no accept or Done recorded.
