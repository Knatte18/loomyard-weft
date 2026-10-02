# Fixer report — plan, round 1

## Fixed

- **F1** (`02-parent-resolver.md`): a parent worktree with no `.git` entry now resolves to the pair form instead of returning the prime-check error, so a removed parent pair no longer breaks card 8's loom wiring; added a filesystem test case for it.
- **F2** (`05-review-gate-rounds.md`): the final closure's below-cap reason is split: an unconsumed reject says the re-spawned writer is re-prompted with the review, a consumed one says the re-spawned writer's discussion goes to the parent as the next round; the final-closure test asserts each reason.
- **F3** (`07-parent-review-recipe.md`): the Intent states that the built entry's `Attempts` equals `GateConfig.Cap`, why that keeps the in-memory count below the store count (Decision 5), and that `resolveParentReviewClosure`'s doc comment carries the claim; the gate test asserts `Attempts` is 3 and that the third reject is terminal.
- **F4** (`06-review-gate-hold.md`): the liveness-error Warn is bounded to once per round per distinct error text; the hold test asserts one Warn across repeated arrivals.
- **F5** (`10-review-approve-supersede.md`): `reviewWaySettled` and the refusal-spec `review: verdict already recorded` row gain a clause that a run halted at the reject cap proceeds only after `lyx loom review approve <slug>`.

`lyx loom validate-plan` passes after the edits, with only informational `create-new-unit` findings.

## Deferred

None.
