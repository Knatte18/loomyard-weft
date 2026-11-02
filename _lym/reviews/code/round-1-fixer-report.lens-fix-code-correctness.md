```yaml
lens: fix-code-correctness
outcome: reported
```

Findings: no findings.

Each changed branch and added test, run on a concrete input:

- **`CheckExportName` doc comment** (`exportname.go`): the change is a comment only. Its claim matches the code, which applies the `.go`-suffix test under the default `Plan{}` language with no `fileUnitRuleApplies` gate.
- **Join rows "an export its only card deletes" and "an export its only card re-signs"** (`join_test.go`):
  - **Delete row:** the Delete group falls through every `ExportMet` case, so the export is unmet and `checkJoinable` refuses with the quoted part name, path and "no card creates its export". Restoring `case CardTypeDelete` makes `refsMatch` return true on the equal string, so the row would fail.
  - **Re-sign row:** an Edit group is likewise not a case in `ExportMet`. The re-sign's left token also stays in the group's `Refs`, per `doc.go:63`, as `internal/b#Member`, which equals the export. A restored re-sign branch would therefore match and the row would fail.
  - **Scope:** `resignArrow` is declared at `TestJoinParts` function scope, so the refusals table can see it.
- **"export a card only deletes" row** (`partcheck_test.go`): `internal/x/x.go` exists in `splitWorktree`, and `ValidatePart` checks only the core part, so the single blocking finding is `part-export-missing`. Restoring the Delete case in `ExportMet` would drop that finding and fail the row.
- **"an export the worktree root holds fails the gate"** (`partcheck_test.go`): the written file is `internal/x/newfile.go` under `worktreeRoot`, and the other exports and the record are the same ones the passing gate subtest uses. A wrong root, such as `anchorPath` or an empty string, would find no file, produce no `export-existing`, and fail the test.
- **"AnExportTheWorktreeHoldsFails" row** (`validateparts_test.go`): the replaced substring `  - \`plan:internal/x#NewThing\`\n` occurs once in `splitPartsText`, and `internal/x/x.go` is in the fixture's `plankit.Repo`. The row gets exit 1, the same envelope message as the over-max row, and the finding in the body. A wrong `WorktreeRoot` in `validate.go` would give exit 0 and fail the row.
- **Callers:** no signature or behaviour changed, so no caller is affected.

Tool calls (1 of 6):
1. `grep Resigns` over `internal/planparser/*.go`: to confirm a re-sign's left token stays in `Refs`, because my context lacked the Edit-group construction.
