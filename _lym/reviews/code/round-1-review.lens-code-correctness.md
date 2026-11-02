```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff's changed functions and branches. Each case below was run by hand against the code in context.

**List: function or branch, case run, correct or not**

1. `partsparser.Validate` (`check.go`, the new `exists` closure), with a worktree holding `internal/x/newfile.go` and the part exporting it: it returns `export-existing`. Correct. With an absent file it returns no finding. Correct. `loomcli` passes `c.env.WorktreeRoot`, which the lightweight wiring sets at `wiring.go:312`, so `validate-parts` and the gate read the same root. Correct.
2. `Check` signature with `exists` (`check.go:53`): the only callers are `Validate` and `check_test.go`, and neither passes nil. Correct.
3. `checkExports`, handle branch, with `plan:internal/x#NewThing`: `CheckExportName` returns no finding and the loop continues. Correct. With `plan:internal/x#&lt;side&gt;Thing` it returns `fabric-vocabulary-name`, with `Part` set to the label and `owner: ` prefixed to the Detail. Correct.
4. `checkExports`, non-handle branch:
   - `internal/x#Existing` is refused through the `#` test. Correct.
   - `internal/x/newfile.go` is accepted when the file is absent. Correct.
   - A path to an existing directory is refused. Correct.
   - A new glyph with `#` that is not a `plan:` handle is also refused. This is intended, since the way forward tells the writer to spell a new member as a handle.
   - A spot's export uses `spot.Path` as its `Part` label, like the other spot findings. Correct.
5. `CheckExportName`, handle with no `#` (`plan:internal/x`): `handleUnit` fails, it returns one `handle-malformed` and skips the vocabulary check. Correct. `fabricVocabularyBreak` would also return `""` for it, so the early return is consistent.
6. `CheckExportName`, `plan:internal/x/x.go#NewThing`: it raises the file-unit `handle-malformed`, then runs the vocabulary check as well, so a handle can yield two findings. Correct. The rule is applied without the `fileUnitRuleApplies` gate that `checkHandleMalformed` uses; that is right for a Create declaration, which the function states it judges. The gate returns true for no claim or a Create claim.
7. `CheckExportName` under the default language: `&amp;Plan{}` has an empty `Language`, so `planLanguage` returns Go and `GoToolchain` is true. A part that declares `language: csharp` is judged as Go, so the `.go` file-unit rule and the Go-only external-test pseudo-unit rule apply to it. Correct for the common case, and the report notes the limit as a design choice. Finding 1 below covers the C# edge.
8. `fabricVocabularyBreak` (extracted from `CheckFabricVocabularyName`): the body is the old logic moved. The unit, member or identifier failures and an empty unit return `broken == ""`, as the old `continue` did. `CheckFabricVocabularyName` still skips on `broken == ""`. Behaviour is unchanged. Correct.
9. `handleMalformedCheck` constant in `validate.go`: the value equals the old literal `"handle-malformed"`. Correct.
10. `ExportMet` (`join.go`), a `Delete` target export or an `Edit` re-sign export: now `false`. Correct, as intended. A `Create` handle export is met exactly, or through a canonicalized spelling via `refsMatch`. Correct. A `Rename` New-side export, handle or path, is met. Correct.
11. `checkJoinable` and `checkPartPlan` (the only two `ExportMet` callers): both use the narrowed `ExportMet`, and both messages now say "creates". Correct.
12. `NewPartsGate` and `resolvePlainGateClosure`: a relative `WorktreeRoot` is refused by `requireAbsRoot` before the closure is built, and the new test row covers it. Correct.

**Findings**

- LOW, `design`, `internal/planparser/exportname.go:498`. `CheckExportName` always judges under the default language, so a part whose plan sets `language: csharp` has its exports checked as Go. A C# export `plan:Foo/Bar.cs#X` passes, and the Go-only vocabulary rule applies to it. The report's design notes accept this because a parts file carries no language. The cost is that the two paths (this check and `validate-plan --part`) can disagree on a non-Go plan.
- LOW, `design`, `internal/planparser/exportname.go:509`. The file-unit refusal runs without the `fileUnitRuleApplies` gate that `checkHandleMalformed` uses at `validate.go:715`. A handle that a part creates only as a Rename New side with a `.go` unit is refused at Parts-Write, but the part gate would not refuse it. The case is rare, since the Goal's exports are Create declarations.

No other defects found. Failure paths: `Validate` still returns read errors from the record and parts file unchanged, and `exists` swallows `os.Stat` errors as "absent", which is the intended meaning for an export check.

**Tool calls**
1. `grep` for `WorktreeRoot` in `internal/loomcli`, plus the `HandleIdentifier` and `ParseRef` definitions: my context lacked the lightweight wiring's env and those two bodies.
