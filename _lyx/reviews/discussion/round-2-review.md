---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:123
    summary: "The Open risks bullet names the Plan-Review rubric and the PR-Gate reviewer as the catch for an unreviewed fix, but neither is told to read a carry-over entry, and D2 changes no stencil that would"
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:33
    summary: "D1 leaves the lighter rule's earlier-open set silently empty when an earlier ledger is unreadable or unparseable, so a degraded read converges a round the rule means to continue, and the bound does not say so"
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:46
    summary: "D2 identifies an entry by segment name but fixes no entry format that marks where it starts and ends in a hand-written section, and the Bouncer holds only the row name, which its config documents as never compared"
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:53
    summary: "D2 bounds the Go write by a failed Validate, but Validate checks heading presence only and needs a support log path the Bouncer is not told, and the judge-written key text is unconstrained so an entry can spill past its own lines"
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:110
    summary: "The docs constraint omits the discussionparser doc.go, whose text names AppendDecision as the one write path into the record and which D2 falsifies"
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:44
    summary: "D2 does not fix the path form written into the committed record or where the rounds come from, and the facts code already distrusts the judge-written rounds list"
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:47
    summary: "D2 says a later Done supersedes an entry, but a skipped Plan-Review settles Done with no ledger and a failed or abandoned generation never reaches Done, so a stale entry stays and its lifetime is unstated"
---

### [MEDIUM] The named catch for an unreviewed fix does not read carry-over entries

**Location:** `_lyx/discussion/decision-record.md:123` (Open risks), with `:52` (D2) and `:6` (Goal).
**Issue:** The first Open risks bullet says a wrong fresh MEDIUM `design` fix "reaches the next producer unreviewed; the carry-over names it, and the Plan-Review rubric and the PR-Gate reviewer are where it is caught".
Neither is wired to do that.
`contracts/stencils/loom/loom-rubric-plan-review.md` measures the plan against "every Decision and every Constraint" of the decision record and calls the record "the measuring stick and never the subject", so a wrong Discussion-Review fix in the record becomes the answer key and is not caught there.
The Webster-Review rubric treats the plan the same way.
The Describe stencil (`contracts/stencils/landing/landing-template-describe.md`) reads the record only for "the design decisions a reviewer needs to know" and says "Never describe the pipeline's process: no ... review rounds", which argues against surfacing a carry-over entry in the PR body that D2 says the PR-Gate reviewer reads.
The Plan-Write stencil reads the record whole, but D2 makes an entry "informational ... never a decision to implement" and no stencil says what Plan-Write does with one.
D2's In list (Scope) names no stencil change, so the Goal's "reach the next producer through the decision record" is a passive write nobody is told to act on, and the Open risks guard does not exist as stated.
**Fix:** State, per segment, which consumer is told to read the carry-over entry and what it does with it (Plan-Write, the Plan-Review rubric, Describe, the rework stencil), and add those stencil changes to Scope; or narrow the Goal and the Open risks bullet to what the write alone achieves and name a guard that exists.

### [MEDIUM] D1's earlier-open set weakens silently on a degraded ledger read

**Location:** `_lyx/discussion/decision-record.md:33-39` (D1 Decision and Bound).
**Issue:** D1 builds the earlier-open list from the ledgers before the latest round.
`ledgerHistories` in `internal/shedadapters/bouncerfacts.go` documents that "an unreadable or unparseable ledger contributes nothing", and `judgeCall` in `internal/shedadapters/bouncer.go` falls back to `(none)` for an unparseable previous ledger.
Today this only thins CIRCLING evidence.
Under D1 the same read feeds the convergence guard, so a missing or malformed earlier ledger empties the list and the judge converges over a MEDIUM `design` finding on a key that was open before.
The rule already says a parse-error review row means `CONTINUE`; nothing says what an unreadable ledger means.
D1 removes a guard from round 2 on and its Bound lists three things the rule never lets through, but not this bypass.
**Fix:** State in D1 what an unreadable or unparseable earlier ledger yields (for example the rule falls back to round 1's rule, or the facts file carries a parse-error row for that ledger and the judge rules `CONTINUE`), and add the case to the acceptance criteria.

### [MEDIUM] D2 fixes an entry's identity but not its format or extent

**Location:** `_lyx/discussion/decision-record.md:46-47` (D2).
**Issue:** D2 says "Each segment owns at most one entry, identified by its segment name: a settle replaces that entry in place".
It never says how an entry is marked, so the write cannot find where an entry starts and ends inside `## Open risks`, which also holds the Discussion producer's hand-written bullets.
A later generation of Discussion-Review also reviews a record that already holds the entry, and a fixer may edit it, which makes "replace in place" and "every other byte unchanged" ambiguous without a fixed shape (heading, marker line or fenced block).
The identity source is also open: `BouncerConfig.Name` in `internal/shedadapters/bouncer.go` is the row name (`Discussion-Bouncer`) and is documented as "a log-field and error-text identity only, never compared, parsed, or used for control flow"; the segment name (`Discussion-Review`) is a recipe field the Bouncer is not told.
**Fix:** Fix the entry's on-disk shape and how its end is found, and name the told value that carries the segment identity (a new config field, not `Name`).

### [MEDIUM] D2's bound leans on Validate, which cannot detect the failure that matters

**Location:** `_lyx/discussion/decision-record.md:53-55` (D2 Bound).
**Issue:** The Bound says a record that "fails `discussionparser.Validate` after the write, is left as it was".
`Validate` in `internal/discussionparser/validate.go` checks that the two files exist and that seven H2 heading lines appear anywhere in the record, fenced blocks included.
A write inside `## Open risks` cannot make a heading vanish, so this check fires only if the write itself removes a heading, and it would not notice an entry that spills over a section boundary.
The entry carries judge-written text: `key` is an unconstrained YAML scalar in the ledger, and D2 does not say it is rendered single-line or escaped.
`Validate` also takes the support log path, and the Bouncer's config carries only `DecisionRecordPath`.
**Fix:** State the actual bound: the entry's text is built from single-line, escaped fields, the write is checked to leave every byte outside the entry unchanged, and name where the support log path comes from or drop `Validate` from the bound.

### [LOW] The docs constraint omits discussionparser's doc.go

**Location:** `_lyx/discussion/decision-record.md:110` (Constraints).
**Issue:** The package doc of `internal/discussionparser` calls itself the "SOLE appender" and names `AppendDecision` as "the one write path into the record", and `append.go` repeats it.
D2 adds a second write path to that package.
The Constraints list `doc.go` for `shedadapters`, `burlerengine`, `loomengine` and `loomcli` and leaves out this one.
**Fix:** Add `discussionparser` to the docs-land-in-the-same-commit list.

### [LOW] D2 leaves the entry's path form and rounds source open

**Location:** `_lyx/discussion/decision-record.md:44` (D2).
**Issue:** The entry lists "the round's review and fixer-report paths".
The record is committed and read in other worktrees, where an absolute worktree path is wrong; the plan-review rubric already treats the decision-record path as anchor-relative.
The same sentence lists the entry's `rounds`, but `ledgerHistories` in `internal/shedadapters/bouncerfacts.go` takes rounds from the ledgers' own numbers "never the entry's judge-written rounds list".
D2 also does not say what an open entry with no class or severity is (the parser allows it).
**Fix:** State the path form (anchor-relative), take the rounds from the ledgers as the facts code does, and state how an unlabelled open entry is treated.

### [LOW] The lifetime of a stale entry is unstated

**Location:** `_lyx/discussion/decision-record.md:47` (D2).
**Issue:** D2 says a later generation's `Done` supersedes the entry.
`approveWithoutReview` in `internal/shedadapters/bouncer.go` settles a skipped Plan-Review as `Done` with no ledger, so the previous generation's entry neither updates nor goes away.
A `CONVERGED` settle that fails after the write is cleared and re-seeded, and if that generation never reaches `Done` its entry points at an archived run directory.
The record states neither case, so the plan writer cannot tell whether either is a defect.
**Fix:** Say whether a skipped settle leaves, removes or rewrites the entry, and that an entry from a generation that did not reach `Done` stays until the next `Done`.

## Focus checks with no finding

- **Focus 1, D1's source of the earlier-open set:** D1 names a new list computed from every ledger before the latest round, not the recurring list, and says so; `ledgerHistories` already records `openRounds`, so a key open in exactly one earlier ledger is in it.
  The third Open risks bullet's guard (the judge mapping against the earlier-open list and the previous ledger) exists, because `judgeCall` passes `previous_ledger` and the facts path.
  The first bullet's guard is the one that does not (F1).
- **Focus 2, generations and ordering:** D2 states that the entry is written before the accept's settle write, that a failed accept stays pending and retries, that a failed `CONVERGED` settle is cleared and re-seeded, and that a later `Done` replaces the entry by segment, so "a replay writes the same bytes" has a meaning (the accept retry and the entry-time attach path).
  What remains open is the identity's shape (F3) and the stale-entry lifetime (F7).
- **Focus 3, commit route per segment:** D2 names one new Bouncer seam wired for all three segments that commits the decision record alone, with no existing pathspec widened, and the acceptance criterion names that seam; this settles `commitDiscussion`, `CommitPlan` and the missing Webster-Review seam.
- **Focus 5, Goal against D1, Auto-mode and Acceptance:** the four state one rule, from round 2 on, with round 1 on today's rule; no disagreement found.
- **D3, budget arithmetic:** with `review_max_bounces` and `review_circling_checkpoint` both 3, `settleUndecided` checks `budgetReached` before the CIRCLING branch, so a round-3 `CONTINUE` or `CIRCLING` escalates with cause `budget` as D3 and its acceptance criterion say.
