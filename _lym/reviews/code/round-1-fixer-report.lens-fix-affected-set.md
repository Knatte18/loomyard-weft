```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1** (callers of `ExportMet`): the only caller outside `join.go` is `internal/loomshed/partcheck.go:108`, the part check. The join refusal rows now cover Delete and re-sign, and `TestValidatePart` has a Delete row beside the existing re-sign row, so both callers are pinned for both forms. `join.go`'s own caller is `checkJoinable`, which the join rows reach.
- **F2** (copies of the handle-malformed rules): the only copies are `checkHandleMalformed` and `CheckExportName`. `CheckExportName`'s only caller is `partsparser/check.go:194`. The doc comment now states the file-unit and default-language scope. The optional shared helper was not made, and the review asked for it only conditionally.
- **F3** (callers that choose the worktree root for the parts check): the callers are `NewPartsGate`, reached through `shedrecipe/entries_gate.go:212`, and `validate-parts` at `loomcli/validate.go:426`. Both now have a test where the root holds a file that an export names, expecting `export-existing`. The other callers, `ValidatePart` at `validate.go:385` and the `NewPartPlanGate` path, were not part of the finding.

Unchecked: none.

Tool calls:
1. grep `ExportMet(` across `internal`, to list the callers of F1's set; my context held only `join.go` and the test files.
2. grep `WorktreeRoot` in `entries_gate.go` and `validate.go`, to list the callers that choose the root for F3.
3. grep `CheckExportName` and `handleMalformedCheck` across `internal`, to list F2's copies and callers.
