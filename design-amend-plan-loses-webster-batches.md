# Plan amendment during Webster loses the record of committed batches

`amend-plan-loses-webster-batches` · Task · bug, webster, loom · running · Preflight

Imported from [issue #185](https://github.com/Knatte18/loomyard/issues/185).

Seen: on a loom run in Webster, lym loom decision add --amends-plan sent the run back to Plan-Burler-Review. Webster's state.json, which recorded four committed batches, was lost. Plan-Burler-Fix then failed with create-already-exists on cards 1-4, because their Create entries had already landed on the task branch.

Workaround: the parent ruled by decision that the landed Creates become Edit entries, and validate-plan then passed.

Expected: an amendment during Webster keeps the record of committed batches, and the plan rework sees which cards already landed, so it neither re-creates their files nor needs a hand ruling.