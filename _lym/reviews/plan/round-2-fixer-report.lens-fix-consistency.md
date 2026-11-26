```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes to cards 8 and 9 agree with the text around them and with their twins elsewhere.

**LOW — card 8's ImpactSummary and doc sentence don't mention the `boardCriticalSection` split**
- **Changed text:** `_lym/plan/08-board-run-settings-carry.md`. The fix added `Board.boardCriticalSectionRollingBack` to **Create:** and `Board.boardCriticalSection` to **Edit:**, and its body text says the old method's body moves into the new one.
- **What is stale:** the ImpactSummary still reads "Adds one board write path, one configreg wrapper and one walk step after the hub-wide commit". It never says that `boardengine`'s shared write scaffolding is split.
- **Same gap in the doc sentence:** "`doc.go` of each package gains a sentence naming its new function" now covers two new boardengine functions under a singular "its new function".
- **Evidence:** the card's **Create:** list now holds two `boardengine` entries, and the ImpactSummary line was not changed.

Everything else I checked agrees:
- Card 9's "the dobby's deps and three other load sites" matches its three-site bullet: `hubLandDeps`, `wireHub` and `loomConfigOf`.
- Card 8's `EnsureFailingRunSettingsCarry` row now says "rollback", which matches the new restore path.
- The renamed tier0 test is used consistently in **Create:** and Tests.
- `00-overview.md` names neither changed symbol nor any render-failure claim.
- The only other mention of the old test name is in `lens-review.md`, which is a historical record.

Calls:
- One grep of `00-overview.md` and the plan cards for twins of the changed text: render, rollback, dobby, `boardCriticalSection`, the old test name and D5/D7. My context did not hold the overview or the other cards.
