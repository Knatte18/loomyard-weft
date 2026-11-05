```yaml
lens: fix-consistency
outcome: reported
```

Checked whether the fix's changes to cards 7, 8, 23 and 25 still agree with the cards and references around them.

no findings

- `reportengine` config references: no other card still uses `reportengine#LoadConfig`, `#Config`, `reportengine/template.yaml` or `configreg#TestNames`.
  Card 26 still edits `internal/reportengine#allowedImports`, but only to add `internal/raddleparser`, which has nothing to do with the config module the fix moved.
- Card 7's `Uses:` entry `internal/configreg/doc.go` and its ImpactSummary ("only card 8 reads the key") still match the rewritten card, and card 8 now uses `plan:internal/reportconfig#LoadConfig`.
- The doc.go files moved to `Edit:` in card 23 (`loomcli`, `shedrun`, `loomrecipe`, `shedcli`, `runprofile`) and in card 25 (`pawlengine`) appear in no other card's targets except `internal/shedrun/doc.go` in card 4, which the fix did not touch.
- In card 23, the new sentence giving raddle `light` in `runprofile/doc.go` matches the card's own `runprofile/template.yaml` line, `raddle: light`.

Tool calls:
1. A grep of `_lym/plan` for leftover `reportengine` config and `configreg#TestNames` references, and for the cards that name doc.go files. My context held only cards 7, 8, 23 and 25.
2. A grep of card 26's `allowedImports` lines and of the other cards that name the moved doc.go files, to check for twins outside the cards I had read.
