```yaml
lens: fix-consistency
outcome: reported
```

Checked against the fix-consistency lens: does the change set agree with its sibling cards and cross-references?

**BLOCKING.** Card 13 deletes `InnerRunDeps.Sleep` and the `pollInterval` field, but `holdSpawnWhileCovered` still reads both until card 14.
- **Changed text:** the new Intent sentence in `_lyx/plan/13-innerrun-short-check.md`: "With `wait` gone here and card 14's spawn hold yielding, nothing in the producer sleeps, so the poll cadence goes with it: `InnerRunDeps.Sleep` is deleted with `NewInnerRun`'s `waitOrCancel` default, and the producer's `pollInterval` field with it".
- **Evidence:** `internal/battenshed/innerrun_pause.go:58` is `p.deps.Sleep(ctx, p.pollInterval)` inside `func (p *innerRunProducer) holdSpawnWhileCovered`.
  Card 13's Edit list does not hold `internal/battenshed#innerRunProducer.holdSpawnWhileCovered`; only card 14 lists it, and card 14 comes after card 13 in `00-overview.md`.
- **Effect:** after card 13, `internal/battenshed` no longer compiles, and card 13's own tier0 gate fails.
  The sentence leans on a card that has not run yet, and the fix broke a card that compiled before.
- **Remedy:** move the deletion of `Sleep` and `pollInterval` to card 14, together with the `nilLegal`, `innerRunEntry`, `shortPollBattenRecipe` and test edits that follow from it.
  Or add `holdSpawnWhileCovered` to card 13 and stop it sleeping there.

No other disagreements:
- No other card mentions `Sleep`, `pollInterval`, `poll_interval_s` or the old reason text.
- Card 17's new sentence about the recipe header agrees with card 13's Run-Shed edit of the same header.
- Card 27's `internal/proc` and card 36's added imports sit on cards that list `dobbyengineAllowedImports`.
- `PassExit` is created in card 33 and used in card 37, in the right order.
- Card 7's ImpactSummary and Tests paragraph agree with its new targets.

Tool calls:
1. `awk` over `innerrun_pause.go` and `grep` over the plan cards: my context lacked which function holds the `deps.Sleep` read at line 58, and whether any sibling card names the deleted seam or the old cadence.
2. `grep` of `00-overview.md` and `sed` of `innerrun_pause.go:50-62`: my context lacked the card order and the hold's loop body.
