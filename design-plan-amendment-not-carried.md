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


## From issue #170

Imported from [issue #170](https://github.com/Knatte18/loomyard/issues/170).

## What happened

A Webster row that finds a begun batch waiting on a plan edit halts as `blocked` and sends the run back to `Plan-Burler-Review`.
Over one run this round trip repeated five times (`Webster` stuck, `Plan-Burler-Review` goto, review stuck, `Plan-Burler-Fix` stuck, `Plan-Bouncer` done, `Drift-Await`/`Drift-Expect` skipped, `Webster` stuck again).
Each time the Webster halt reason asked for a concrete plan edit, and the next Webster halt asked for an edit again:

- One halt asked for card 14 to list three additional targets. After a full review/fix/bouncer pass, the next halt asked for the same amendment, naming "plan fixer or `lym webster rebaseline --card 14`" as the way forward.
- A later halt asked for a re-sign arrow head in card 15 to be corrected, then `lym webster rebaseline --card 15` run.
  `lym webster record-batch 14` had refused with `resign-not-done/15-dp-heuristic-full-size[blocking]` and pointed to `lym webster recover-batch 14`.
- One halt came from the auto-rebaseline guard: `00-overview.md changed outside its Card Index`, with the way forward "restore 00-overview.md, or fix the plan".

The history shows `Plan-Burler-Fix` recorded as `stuck` and `Plan-Bouncer` as `done` in the passes that followed, with no sign the requested edit was made.

## Why it is abnormal

The review/fix/bouncer pass completes without acting on the edit Webster asked for, and Webster does not pass its halt reason to the plan fixer as a required change.
Each unresolved request costs a full plan-review pass plus an operator or parent intervention, and the loop only ends when a parent edits the plan by hand.

## What to do

- Hand the Webster halt reason to `Plan-Burler-Review` and `Plan-Burler-Fix` as a finding that must be resolved, and refuse `Plan-Bouncer` done while that finding stands.
- Where the way forward is a `lym webster rebaseline --card N` call, let the loop run it after the fix instead of halting on it.

## Provenance

Task `lagrangian-gap-measurement`; notes `loom-halt-2.md`, `loom-halt-3.md`, `loom-halt-4.md`, `loom-halt-5.md`, `loom-halt-6.md`, `loom-halt.md`, `webster-refusal-record-batch.md`.
