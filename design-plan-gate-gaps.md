# Plan and Webster checks out of step with the code a plan changes

`plan-gate-gaps` · Task · bug, webster, loom, quarry · running · Plan-Burler-Fix

## From `webster-glyph-rename`

### Seen (2026-10-14, `tender`, batch `07-proc-process-listing`)

Webster rejected cards 8–10 because their `plan:internal/pausecli#…` and `plan:internal/pauseengine#…` glyphs did not resolve at HEAD.
Card 12, the plan's last card in the same batch, renamed those packages to `tendercli`/`tenderengine` on purpose, so those glyphs can never resolve against the batch's final tree.
All 21 rejected glyphs resolve under the new package names (checked by the recovery strand with `lyx quarry resolve`).
The recovery strand found the work complete at `5348618b4` and could not clear the rejection; there is no verb to accept a batch by hand.

The batch's gate command had the same flaw: it named `./internal/pausecli` and `./internal/pauseengine`, which no longer existed after card 12, so `lyx gate test` refused it and recovery ran the gate over the packages that exist by hand.

### The bug

Webster's glyph check resolves every card's glyphs against the batch's final HEAD, so a plan whose later card renames or moves what an earlier card's glyphs name fails the check though the work is right.

### Ways out to weigh

- Resolve each card's glyphs at that card's own commit.
- Or follow renames: accept a glyph that resolves at the card's commit, or under a package the plan itself renames.
- Plan side: Plan-Review or `validate-plan` flags a rename card whose glyphs earlier cards depend on, or the Batchifier puts a rename in its own later batch.

## From `plan-pinned-tests`

Imported from [issue #594](https://github.com/Knatte18/loomyard/issues/594).

**What happened.** A card added a method to the `r.runChecked` method set.
`cmd/lyx/gitrepoboundary_test.go` pins that set by name, so the change breaks it.
The plan named no card edit for that file, and the break surfaced only when the agent ran `./cmd/lyx` by hand.

**Why abnormal.** A card's declared `Edit:` list and gate steps did not include a test the change necessarily affects, so the per-card gate stayed green while a cross-package boundary test failed.

**What to do.** Have plan or card validation flag cards that change a symbol pinned by name in a boundary test, or run the boundary-test package in the card gate so the break is caught at the card that causes it.

Provenance: task `convergence-plan`; note `06-board-and-archive-readers.md`.

## Seen as well (2026-10-14, `tender`)

Batch 01's gate did not run `./cmd/lyx`, so `TestTierPurity_UntaggedTestsSpawnNothing` (`PATTERN-test-speed`, a `time.Sleep` of 1s or more not resolvable to a constant) failed only later.
It failed on two untagged tests that cards 2 and 3 touched: `internal/githubclient/githubclient_test.go` (`time.Sleep(hold)`) and `internal/selfreportengine/inbox_test.go` (`time.Sleep(holdTime)`).
This is the same class as the pinned-test case: a card's gate does not run the package whose repo-wide scan the change breaks.

Also, batch 07's gate command named `./internal/pausecli` and `./internal/pauseengine`, which no longer existed after card 12's rename, so `lyx gate test` refused it.

## Scope to settle

1. The done-check resolves each card's plan handles at that card's commit, or follows renames the plan makes.
2. Batch gate commands survive a rename inside the batch.
3. The card or batch gate (or validate-plan) catches tests that pin a symbol by name and repo-wide scans in `cmd/lyx` that a change can break, weighing the cost of running `./cmd/lyx` per batch against flagging at plan time.
