```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: checking that the change set agrees with its sibling cards, criteria and cross-references.

no findings

Checked and found consistent:
- **Card 10's `prime` repoint and card 12:**
  - Card 12's Intent now says `s` replaces the already repointed `prime`.
  - Card 12's Edit list still holds the yaml, `TestMount_File` and `TestShortcuts_FileAndRootListing`.
  - `shortcutcli/doc.go` and `docs/overview.md` (line 236) name `lyx prime` without naming `orch start`, so neither goes stale between cards 10 and 12.
- **Card 14's `nilLegal` entry `Landing.LandBlockers`:**
  - `envkit.NilSeams` skips `nilLegal` paths, and both wire tests walk `NilSeams`, so neither wire table needs a twin entry.
  - Card 17's "loom and darn leave both unset" matches the `unfilled` map's reason.
- **Card 16's seam fixtures:**
  - `Pawl.Watch` in `nilLegal` agrees with card 15's "a nil `Watch` wakes on the timer alone".
  - Card 17's `TestWire_BeamRecipe` asserts a pawl seed fills `Pawl`, while both wire tables are for a loom or batten seed, so "pawl-only seam" agrees.
- **Card 16's rename to `TestNewPawl` and `TestPawlRouting`:** card 6's lines naming the old names describe that card's own rename, which has already landed as commit 6.
- **Card 15's tier sentence:** it agrees with the `plan:internal/pawlengine#TestMain` Create target. No other card or overview line says card 15's tests are untagged.
- **Card 10's helper deletions:** the Delete list and the Tests paragraph agree. `waitForDiag` and `turnEnds` stay, since other tests still call them.

Outside my lens (fix-proposal): between cards 10 and 11, `lyx prime` runs the pre-card-11 `beam start`, which starts only the dobbies and no orch. So for those two commits the shortcut attaches without starting an orch. Card 10's added sentence only claims the shipped file still resolves, which holds.

Tool calls:
- Read card 17 and grepped the sibling cards for `Pawl`, `LandBlockers`, `intentionallyNil` and `nilLegal`: my context did not hold card 17 or the cross-card references.
- Read the two wire-seam tests and envkit's `nilLegal`: my context held only grep hits, not how the wire tests consume `NilSeams`.
- Grepped the shortcut docs and the overview for `prime` and `orch start`: these docs were not in my context.
- Grepped card 11's Intent for shortcut, args and `orch start` mentions: card 11's body was not in my context.
