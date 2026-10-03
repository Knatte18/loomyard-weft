# Fixer report — plan round 1

Every finding in `round-1-review.md` is fixed in the plan cards.
`go run ./cmd/lyx loom validate-plan` passes over the edited plan, reporting only informational `create-new-unit` findings.

## Fixed

- **F1** (MEDIUM), card 10: `contracts/specs/refusal-spec.md` joins the Edit list.
  The card adds one loom row per circling refusal: the four verb refusals plus the shared resolver's slug-required and unknown-slug refusals under the `circling` group.
  `review_test.go`'s resolver cases also run under `circling`, so each of those two refusals has a reaching test.
- **F2** (MEDIUM), card 6: the `CIRCLING` bullet now carries both of D6's no-progress signals, flat or rising gating counts and gating findings on recurring or reopened keys.
  A new bullet ties a recurring key to the gating class through the latest review's finding the judge maps to that key in its own ledger.
  The claims list pins each signal and the class tie.
- **F3** (MEDIUM), card 4: `settle` reads the verdict through `parseRecordedVerdict`, since the harvest sites have already applied the strict check.
  The legacy-`BLOCKING` replay test asserts the ledger pointer.
- **F4** (LOW), card 4: the card states the re-judge route (degrade, the round producer's unjudged hand-back, then a re-judge) and its cost of one bounce unit on each row, with a block on an exhausted budget.
  It also states that the legacy archive follows a finished run or a waited attach.
  The doc.go bound sentence carries the route.
- **F5** (LOW), card 8: the settled-accept branch in `settle` is described as reachable only through the entry-time attach branch, where it degrades.
- **F6** (LOW), cards 8 and 10: the Awaiting Reason names the task worktree for `lyx loom start` and `lyx loom goto`.
  Card 10's overview sentence notes that batten does not resume a child on a circling decision and that `lyx loom start` in the task worktree does.
- **F7** (LOW), card 10: `contracts/specs/loom-status-spec.md` joins the Edit list, and its `error` comment is widened to name the circling halt.
- **F8** (LOW), card 8: the card states that re-entering after a goto over an undecided `CIRCLING` round halts again, and that `continue` then reviews the rewrite.
  `Call`'s doc comment carries this.
- **F9** (NIT), card 8: the budget-sentence condition is `ok && count >= budget`, and a new test case covers `ok` false.

## Deferred

Nothing deferred.
