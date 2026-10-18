# Round 2 fixer-report — discussion

## Changes

- F1 (MEDIUM, design), fixed as proposed: D4's first sentence now rests on `lyx orch rules` being the only route that commits the hub-wide file, shipped by this task, deployed only after landing, and prime-only.
  D4 adds a card that files the `orch-rules-move` board entry holding the step and the rules verbatim.
  The Open risks line "Nothing at landing reminds the operator" is deleted, and the support log's rejected alternative is reworded.
  The proposal's "backlog board entry" is written as "board note" filed through `lyx board upsert --body-file`, because the board has no backlog kind (self-check).
- F2 (MEDIUM, consistency), fixed as proposed: D3's failed-commit sentence names re-running `lyx orch rules --from <file>` as the way forward, and the false claim about the pull-first skip on a dirty Bolt is gone.
  The failed-write criterion names the re-run.
- F3 (LOW, design), fixed as proposed: D3's restore-inside-the-closure sentence is deleted, and the first plan-writer note now points only at `reportHubWrite` and `PullThenCommitWritten`.
  The twin in the plan-writer Tests note, "the write-failure restore", is cut as well.
  The criterion that a failed write leaves the prior content stays.
- F4 (NIT, consistency), fixed as proposed: D2's second heading-rule statement is cut.
- The support log gains a `### Round 2` entry under Review rounds.

## Self-check

- fix-consistency, MEDIUM: D4's rationale still said the verb is the only route that commits a Bolt file — fixed: it now says the hub-wide file.
- fix-consistency, LOW: the board-note card had no Scope item or criterion — fixed: both added.
- fix-consistency, LOW: `_lyx/discussion/relevant.md:14` still calls `setHubWide`/`rollBack` the restore route the verb follows — deferred, see below.
- fix-proposal, MEDIUM: a promoted run of the note would call the prime-only, undeployed verb from a task pair — fixed: D4 says the note's body states that it is an operator step, run from the prime after the deploy and never promoted, and the criterion requires that.
- fix-proposal, LOW: no criterion for the board-note card — merged with the fix-consistency LOW, fixed.
- fix-proposal, LOW: the re-run races a board write that sweeps the uncommitted file — fixed: D3 states that a board write before the re-run sweeps the file into its own commit.
- fix-proposal, LOW: the Tests note no longer names a failed-write case — fixed: it lists a failed write leaving the prior content.
- fix-claims, LOW: "backlog" is not board vocabulary — fixed: "board note" through `lyx board upsert --body-file`.
- fix-way-forward, LOW: the same "backlog" wording — merged with the fix-claims finding, fixed.

## Disputed

None.

## Deferred

- fix-consistency LOW on `_lyx/discussion/relevant.md:14`: the file is outside this round's write surface, so the fix (drop "and restore" and `rollBack` from the line) is left to the loop owner.
