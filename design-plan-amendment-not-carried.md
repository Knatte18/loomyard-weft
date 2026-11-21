# Plan amendment to an unchanged card is approved without being carried into it

`plan-amendment-not-carried` · Note · bug, burler, webster, loom

Imported from [issue #168](https://github.com/Knatte18/loomyard/issues/168).

## Seen

On a tapestry run in Webster, the parent recorded `lym loom decision add <slug> --by parent --amends-plan` naming a concrete change to one card (new Edit targets and a changed Intent), then ran `lym loom goto --to Plan-Burler-Review` and `lym loom resume`.
Plan-Burler-Review approved in one round without editing the card.
Its review said the amendment was not carried into the card, but that "the round's check admits a new finding only on a changed hunk in a file the changes file lists", and the changes file listed only `00-overview.md` from an earlier amendment round.
Webster then began the batch again, its fork stopped on the unamended card, and the run blocked at Webster.
An earlier amendment on the same run, which added new cards, did go through the same path.

## Workaround

A second parent decision, without `--amends-plan`, told Webster that the batch's fork follows the amending decision over the card, editing the extra targets, and records the batch without a plan edit.
Then `lym loom resume`.

## Expected

An `--amends-plan` decision routed to `amendment_target` lets the plan review raise findings on the cards the amendment names, even when those cards have no changed hunk, so the fixer edits them.
