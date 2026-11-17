```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the Card 11 fix agrees with the text around it (fix-consistency lens).

no findings

The new Intent text in `_lym/plan/11-stalemate-settlers.md` quotes both sentences, and each quote matches `internal/loomcli/doc.go:471` and `:577` exactly. `internal/loomcli/doc.go` is still under Card 11's `**Edit:**`. No other card says "arbiter line". Card 12 also lists `internal/loomcli/doc.go` under `**Edit:**`, but its Intent is about wiring and config and does not touch the arbiter's cause sentences, so the two cards don't conflict.

Tool calls:
- grep of `_lym/plan` for "arbiter line" and loomcli doc.go references: checks whether another card repeats the wording that was fixed; my context only held Card 11.
- grep of Card 12 for doc.go lines: Card 12 also lists `internal/loomcli/doc.go`, and its text was not in my context.
- grep of Card 12 for loomcli, arbiter and stalemate: checks whether Card 12's Intent changes the same sentences.
