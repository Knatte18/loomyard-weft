```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the fix's changes agree with the sibling cards, criteria and cross-references they touch.

**LOW: card 2's ImpactSummary is stale after the fix to F7.** Card 2 now lists `cmd/lym#testOnlyCodeAllowed` under Edit, and its Intent removes the allowlist entries card 1 added. The ImpactSummary still reads "Testcover-local deletes with no production caller left after card 1; no other package references the removed symbols." That is no longer true: cmd/lym's allowlist keys name those symbols, and card 2 edits it.
Evidence: `_lym/plan/02-testcover-planning-delete.md:30`, against the new Edit entry and Intent sentence in the same card.

**LOW: card 1's ImpactSummary leaves out the new allowlist edit.** It still says "outside gatecli only `cmd/lym`'s wait list and docs change". The fix for F7 added `cmd/lym#testOnlyCodeAllowed` to card 1's Edit list, with an Intent sentence that gives it entries.
Evidence: `_lym/plan/01-gate-test-map-mode.md:182`, against the new Edit entry and the sentence after "The `internal/testcover` planning code is left unused here".

No other twin was left stale:
- Nothing in the plan still names `collidesWithAnother`, `testMapCollideWait`, `ABadBase` or the Crowded tests.
- `notests` now agrees across card 1's fixture bullet and its `NamedPackageWithNoTrackedGoFileIsSkipped` bullet.
- The subtests that need `linked/inner` name it as an explicit target, so the hand-made maps keep it in the package set, consistent with card 4's restated subtest.
- The `testmapstamp_test.go` header changes run in order: card 1 drops the selector, then card 3 widens the header.

Tool calls:
1. I grepped the plan for leftover names (`collidesWithAnother`, `ABadBase`, Crowded, `notests`) and for the ImpactSummary lines, and read `internal/gatecli/testmap_integration_test.go:1700-1730`. My context did not hold the ImpactSummary lines of the changed cards, or that subtest's targets.
