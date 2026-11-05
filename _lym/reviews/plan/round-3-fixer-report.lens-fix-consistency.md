```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: checked whether the fixed cards still agree with their sibling cards, the overview and cross-references.

Findings:

- **LOW.** Changed text: the new **Create:** group in `_lym/plan/16-drift-verify-lines.md`, which adds `darnWriteRow` and `websterRow`.
  Summary: the card's `**ImpactSummary:**` still lists only the Deps fields, the Drift-Recheck engine switch and the helper replacement. It doesn't mention the two driftengine row constants the card now creates, and card 19 now uses them.
  Evidence: card 16's `ImpactSummary` reads "Replaces Drift-Verify's body and its private helpers inside driftengine, switches Drift-Recheck's engine in both recipes, adds two Deps fields wired in loomcli; …". Card 19's Uses now lists `plan:internal/driftengine#darnWriteRow` and `#websterRow`.

Twins I checked that agree:
- **Card 12:** Intent line 72 ("a record with no `awaited_at` is a Stuck the same way") matches the new row and the "three Stucks" prefix sentence.
- **Card 16:** the Intent's interim `command` Stuck (line 104) matches the new row and the prefix rule at line 127. Card 17's Intent ("replacing card 16's interim Stuck") matches its new TestDriftVerify sentence.
- **Card 19:** the Intent's "card 16's `darnWriteRow` and `websterRow`" matches its Uses lines and card 16's Create group.
- **Overview:** the route derivation (line 63) and the counted-capture decision (line 92) match cards 12 and 16.
- **Cards 5 and 6:** the new test rows match their Intent refusal lists.

Tool calls:
- One grep of `_lym/plan/00-overview.md` for the route, row-constant, interim and never-counted text. My context held the overview only as grep fragments, not its Shared Decisions on the fix route and the counted capture.
