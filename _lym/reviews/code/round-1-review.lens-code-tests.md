```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** For each behaviour the diff adds or changes, the test row that pins it, or none.

| Behaviour | Pinning test | Verdict |
|---|---|---|
| A non-`plan:` export holding `#` is refused as `export-existing` | `check_test.go` row "export-existing, a member glyph" | pinned |
| An export of a file or directory the worktree holds is refused, in a spot | `check_test.go` row "export-existing, a file the worktree holds" | pinned |
| The same refusal in a part, through the real `os.Stat` closure and `worktreeRoot` join in `Validate` | `check_test.go` `TestValidate` row "an exported path the worktree already holds" | pinned |
| A new file path or new handle export passes | `check_test.go` base rows with valid parts, and `TestValidate` "a valid file and record" | pinned |
| `Check` runs `CheckExportName` on a `plan:` export, with the `part "core": ` prefix and `Part` label | `check_test.go` row "a handle export's name check" (`fabric-vocabulary-name`) | pinned |
| `handle-malformed` for a handle with no unit, and for a handle with a file as its unit | `exportname_test.go` rows 2 and 3 | pinned at `CheckExportName` |
| `CheckExportName` finding shape: `Ref` is the handle, no `Card`, Detail has a way forward | `exportname_test.go` loop assertions | pinned |
| `ExportMet` no longer counts an Edit re-sign | `partcheck_test.go` row "export a card only re-signs" now expects `part-export-missing` | pinned |
| `ExportMet` no longer counts a Delete target | none | **gap** |
| `join.go` `checkJoinable` refuses a part whose export is only re-signed or deleted | none | **gap** |
| `fabricVocabularyBreak` extraction leaves `CheckFabricVocabularyName` unchanged | the existing `CheckFabricVocabularyName` tests, which I did not re-read | pinned by the existing tests, from the diff alone |
| `handleMalformedCheck` constant replacing the literal | the existing `handle-malformed` tests | pinned |
| `NewPartsGate` hands `worktreeRoot` to `Validate` | none; its three tests pass `t.TempDir()` and none holds a file | **gap** |
| `resolvePlainGateClosure` requires an absolute `WorktreeRoot` for `parts` | `entries_gate_test.go` row `PartsRelativeWorktreeRoot` | pinned |
| `validate-parts` passes `c.env.WorktreeRoot` to `Validate` | `validateparts_test.go:95` sets `WorktreeRoot`, but I found no row where an existing export is refused | **gap**, weak |
| Spot exports go through the same name checks, with the `shared spot` owner prefix | none; the handle row uses a part, and the spot row is file-only | **gap**, minor |

Findings:

1. **MEDIUM, class `design`.** The `ExportMet` Delete branch has no row. The diff removed the `Delete` case from `ExportMet` in `internal/planparser/join.go:162-186`, which was half the "re-sign or delete" change. The only changed test row is the re-sign one, `internal/loomshed/partcheck_test.go:210`. The `join_test.go` delete row was changed, not kept as a refusal: at `join_test.go:64` the `gamma` part's Delete card lost its export (`internal/c/gone.go`), and no row was added in its place. Restoring `case CardTypeDelete` in `ExportMet` would still pass every test. A row in `partcheck_test.go` or `join_test.go` with a Delete-only export expecting a refusal would pin it.

2. **MEDIUM, class `design`.** `join.go:116-121`, the join's own refusal of an unmet export, has no row for the new rule. `join_test.go:161` ("an export no card delivers") predates the change and uses a card that creates nothing. No row checks that a part exporting a member it only re-signs is refused at join. The `partcheck_test.go` row covers `ExportMet` through the part check, not the join's own message, which this diff edited.

3. **LOW, class `design`.** `NewPartsGate`'s `worktreeRoot` argument is unpinned. The gate tests at `partcheck_test.go:147,157,170` pass an empty `t.TempDir()`. Passing `anchorPath` where `worktreeRoot` belongs, or `""`, would still pass every test. A gate row with a file in the worktree and an export naming it, expecting a failed gate with `export-existing`, would pin it.

4. **LOW, class `design`.** The `validate-parts` verb's `WorktreeRoot` pass-through (`validate.go:426`) has no existing-export row. `validateparts_test.go` builds a repo fixture at line 95 but I found no refusal case for it.

5. **NIT, class `design`.** Spot exports have no handle-name-check row, and no row asserts the `shared spot %q` owner prefix. `checkExports` is shared with the part loop, so the risk is low.

The diff adds no per-helper redundant test. The `exportname_test.go` rows are distinct from `check_test.go`'s fabric row, which pins the wiring. I raise no duplication finding.

Tool calls (1):
1. grep over `join_test.go`, `partcheck_test.go` and the `loomcli` tests, because my context held the diff hunks of those tests but not their full row lists or the `validate-parts` fixture.
