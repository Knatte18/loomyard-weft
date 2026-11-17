```yaml
lens: fix-consistency
outcome: reported
```

I checked card 23's two new Delete lines and its reworded bring-up sentence against the rest of the card and every other card in `_lym/plan`.

No findings.

- No other plan card names `ownerPause`, `ownerPauseMu`, `InstallOwnerPause` or `ownerpause.go`, so the fix has no twin left unchanged elsewhere.
- The Delete list now names both functions and both vars, and the Intent sentence names the same four. The two agree.
- The ImpactSummary doesn't list the deleted symbols, so it doesn't go stale.
- `internal/orchcli/doc.go` already appears in the Edit list, which covers its mention of `InstallOwnerPause`.
- `ownerpause.go` is left out of the targets on purpose, as the review explains. Once the vars go, nothing uses the `sync` import, and the compiler forces its removal.

Tool calls:
- grep of `_lym/plan` for `ownerPause`, `ownerpause.go` and `InstallOwnerPause`, because my context held only card 23 and not the other cards.
