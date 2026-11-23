# Every darn escalates to the parent when round 1 fixed a BLOCKING finding

`fixed-blocking-escalates-parent` · Note · bug, loom

## Seen

Since `code-bouncer-asks-parent` landed (8cf7f4775), four darn runs on 2026-11-23 stopped at Code-Bouncer with cause `budget` after round 1:
`amend-plan-loses-webster-batches`, `pr-findings-open-fabric-nil`, `beam-pr-describes-itself` and `godocreflow-worsens-findings`.
In each, the fixer had fixed every finding, BLOCKING included, committed it and passed its gates;
the ledger still held the BLOCKING entry `open`, because no later round re-reviewed the fix, and a darn's budget is spent after one round.
The orch then read the fixer report and the commit and accepted each time, which cost each run a parent turn and a wait.

## Idea

A BLOCKING finding the fixer reports `fixed`, with a commit and a green gate, is verified without a full review round before the Bouncer judges the budget:
for example, a short verification of only those findings against the fix commit, which closes the entry or keeps it open.
The parent is asked only when a BLOCKING finding stays open after that check, or the fixer did not fix it.
