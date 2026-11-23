---
dispositions:
  - id: F13
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
---

## Changes

- **F13** (`internal/shedadapters/bouncerdecision.go`, `bouncer.go`): `settleDecisionAccept` now holds an admitted accept over a CONTINUE or CIRCLING verdict when the parsed ledger has an open BLOCKING entry of another class.
  Outside the soft-cap window `holdAcceptOverOpenEntries` starts the next round, with no `BudgetExempt` since that is false whenever the window is closed.
  Inside the window it retires the accept and escalates with cause budget, naming the open keys.
  The check reads the ledger `settleDecisionAccept` already holds through the new `openBlockingKeysOf`, so it has no read-error branch.
  Doc comments of `routeDecisions` and `settleDecisionAccept` name the rule.
  `TestBouncer_AutoAccept` gained the below-budget row (Stuck, judge outputs kept) and the spent-budget row (Awaiting, cause budget), and now fails a held accept that settles.
  The decision brief stencil `bouncer-fragment-decision.md` no longer says an accept passes every listed BLOCKING finding.
- **F12** (`bouncer_escalation_test.go`): the decision-over-BLOCKING row also asserts `the review also holds decision findings (D1)` through the new `wantDetailAlso`.
- **F2** (`reinterpret.go`, `configreg.go`, `reinterpret_test.go`, `_lym/landing/summary.md`): `FlagCodeAutoAccept` flags a file that sets no key, a header-only file and nil bytes, and stays quiet only for `false`.
  The loom entry sets `RunsWithoutFile: true`.
  The test rows `OtherKeysOnly` and `HeaderOnly` now expect a flag, and `AbsentFile` was added.
  The summary says one-way and widens the blast radius to the burler review round check and the hub config reconcile notice.
- **F11** (`entries_burlerhalves_test.go`, `validate_integration_test.go`): `TestBurlerReviewEntry_ProfileMapping` builds a harden_round and a plain row over a judged round 1 and runs `CheckReviewRound` on the built round context with a new focus finding outside the changes.
  `TestValidateReviewVerb_RoundChecks` accepts it in the hardener run directory and refuses it in the code run directory.
- The landing summary also records that the hold applies to every row that sets `escalate_decisions`, motif and arras included.

Commits: one for F13 and F12 (shared test file), one for F2, one for F11, and one follow-up for F13 from the self-check.
`_lym` files were edited and not committed.
Verified with `lym gate test` tier 0 on the touched packages, and tier 1 on `burlercli`, `hubreconcile` and `configreg`.

A gate repair then rebroke two comments at sentence boundaries, the `focusReview` field comment in `validate_integration_test.go` and the `settleDecisionAccept` doc comment, with no change of meaning.

## Self-check

- fix-consistency, MEDIUM: the decision brief stencil still said an accept passes every listed BLOCKING finding. Fixed.
- fix-consistency, LOW: the `template.yaml` `code_auto_accept` comment. Disputed, premise false: line 53 already says "true, the default" and that an open BLOCKING round escalates to the parent.
- fix-code-correctness, LOW: the read-error branch of `holdAcceptOverOpenEntries` was untested. Fixed differently: the branch is removed and the check reads the ledger already parsed.
- fix-code-correctness, LOW: tier 1 was not run for `hubreconcile` and `configreg`. Fixed: both pass at tier 1.
- fix-affected-set, LOW: the landing summary did not say motif and arras rows get the hold. Fixed.
- fix-proposal, LOW: a hub created with no `loom.yaml` also receives the one-shot notice. Disputed, every fix worse: the review asks for the absent file to be flagged, and telling a new hub from an old one needs a hub-age marker outside this write surface.

## Disputed

- fix-consistency LOW on `internal/loomengine/template.yaml:53`: ground `premise false`; the comment reads "true, the default, makes the Code-Review perch accept ..." and states the escalation for an open BLOCKING round, so a hub on the default is already told.
- fix-proposal LOW on the absent-file notice: ground `every fix worse`; evidence `internal/hubreconcile/reinterpret.go:64`, where the walk runs the check with nil only through `RunsWithoutFile`, the mechanism F2 asks for.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached me.
