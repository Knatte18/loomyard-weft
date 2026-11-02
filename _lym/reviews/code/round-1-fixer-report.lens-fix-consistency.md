```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set for consistency with its siblings and twins (lens fix-consistency).

No findings.

- **`CheckExportName` doc comment:** the added sentence matches the code. The file-unit test still runs only under `plan.GoToolchain()` on the default `Plan{}`, and the existing language sentence already says so. The comment now describes the lack of a `fileUnitRuleApplies` gate. No sibling comment contradicts it.
- **`join_test.go` rows:** both new rows sit in the table that held "an export no card delivers", so they are not new test functions. They expect the refusal text `join.go:119` emits ("no card creates its export"). The re-sign row reuses `resignArrow` from the enclosing scope. Its export `internal/b#Member` matches the arrow's glyph.
- **`partcheck_test.go` rows:** the Delete row sits beside "export a card only re-signs" with the same `wantCheck`. The new `TestNewPartsGate` subtest follows the neighbouring lowercase-sentence naming and the shared `recordPath`. It writes the file under its own `worktreeRoot`, which is separate from `anchorPath`. A caller that passed the anchor path or an empty string would therefore fail it.
- **`validateparts_test.go` row:** the `strings.Replace` target matches `splitPartsText`. The exported file `internal/x/x.go` exists in the fixture's `plankit.Repo` worktree and not under the anchor. The row uses PascalCase names like its siblings. It reuses the existing `wantErrMsg` "loom: parts file is not yet valid" and the `export-existing` body check.
- **No twin left behind:** the Delete and re-sign cases of the `ExportMet` narrowing are now pinned at both join and part check. The gate and verb root choices are each pinned once. `checkHandleMalformed` and `CheckExportName` still share only `handleMalformedCheck`. That matches the review's conditional "if the finding is to stay in step" for F2, so I do not count it as a missed twin.

Tool calls made: none; my context already held every changed file and the review.
