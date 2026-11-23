# Reviews ask the parent instead of settling past what it must rule on

Three gaps let a review segment settle past something only the run's parent (or its arbiter) may rule on.
This change closes all three by routing each case through an escalation the parent must answer.

## 1. A held decision finding gates the Code-Review perch

The Code-Review rows routed a `decision`-class finding to the parent by a one-line notice only.
The fixer left it alone (held), but the Bouncer had `escalate_decisions` off, so the judge could rule past it and the budget could auto-accept it.

- The Code-Bouncer rows of `darn-recipe.yaml`, `tapestry-recipe.yaml` and `sampler-recipe.yaml` now set `escalate_decisions: true`.
  The Code-Burler-Fix rows set `decisions_escalated: true`.
- That reuses the existing decision route (`internal/shedadapters/bouncerdecision.go`).
  A judged round whose review holds a decision finding escalates with cause `decision`, whatever the judge ruled and whatever the budget.
  The arbiter settles it where the board entry and record answer it, and the parent settles it otherwise.
  A parent `continue` runs the next round, which carries the ruling out.
  A held finding the next review carries forward escalates again.
- The Plan-Review rows are unchanged. They keep the fix row's parent notice.

## 2. A spent budget with an open BLOCKING finding asks the parent

`BouncerConfig.BlockingEscalates` (`blocking_escalates`) used to send such a round to the soft-cap rule.
That rule accepts any round without a design-wrong entry, so BLOCKING findings still landed.

- In `settleUndecided`, a round in the soft-cap window whose ledger holds an open BLOCKING entry, or cannot be read, now escalates with cause `budget`.
  The detail names the open keys.
  This check runs before the auto-accept and before the soft-cap rule.
  The parent accepts, which carries the findings into `## Open risks`, or continues, which buys one budget-exempt round.
  A `budget` escalation goes to the parent, never the arbiter, as `arbiterstop` already rules.
- The Code-Bouncer rows set `blocking_escalates: true`.
  The `code_auto_accept` comment in `internal/loomengine/template.yaml`, the escalation brief's `budget` cause and the shed-recipe spec now describe this.

## 3. A hardener round never converges with its focus undriven

Under the shared review format, an incremental round may raise only BLOCKING findings on changed hunks.
An undriven or WARN focus entry therefore had no finding to block on, and an APPROVED review converged.

- The hardener rubric now has the review raise each focus entry it left undriven or marked WARN as a finding.
  The finding carries class `decision`, severity BLOCKING and `origin: focus`.
- `burlerengine.CheckReviewRound` admits a new BLOCKING `decision` finding with `origin: focus` outside the changed files (`FocusOrigin`).
  A finding of another class with that origin gets no exemption.
- Harden-Bouncer sets `escalate_decisions` and Harden-Burler-Fix sets `decisions_escalated`.
  The arbiter rules first: it orders another round, or accepts with a reason per entry through `lym loom decision add`.
  The parent rules when the arbiter cannot.

## Design decisions

- Part 3 reuses the decision escalation instead of adding a new escalation cause.
  The arbiter is already eligible for cause `decision` and budget goes to the parent, so the routing the operator asked for needs no `arbiterstop` change.
  As a result, every hardener round with an undriven focus entry stops for the arbiter, not only a round that would otherwise converge.
- A held focus finding that a later round drove is resolved by quoting the title of the decision record entry that ordered the drive.
  The round check already requires that quote for every held finding.
- No config reinterpretation ships.
  `code_auto_accept` keeps its value and meaning (auto-accept at a spent budget).
  The recipe's new `blocking_escalates` key carves the BLOCKING case out of it.

## Tests

- `TestBouncer_AutoAccept`: BlockingEscalates now escalates an open BLOCKING entry with cause budget, with and without auto-accept.
  It leaves such an entry alone below the budget, and a parent continue runs another round.
- `TestBouncer_SoftCapRule` drops its `blocking_escalates` path, since that path no longer reaches the rule (recorded as `Loosened-Check`).
- `TestBouncerEntry_BlockingEscalatesAndColorSegment`: the `blocking_escalates` row now expects Awaiting.
- `TestCheckReviewRound`: `origin: focus` rows cover an admitted decision finding, a refused other class and a refused non-BLOCKING finding.
- `TestDecisionSwitches_SetOnTheRowsThatAskTheParent` pins `escalate_decisions`, `decisions_escalated` and `blocking_escalates` across the tapestry, darn, sampler, arras and hardener recipes.
- `tapestry-rows-before-motif.yaml` carries the same new Code rows' keys.

## Merge danger

**Door:** two-way
**Blast radius:** shed
