```yaml
lens: forced-edits
outcome: reported
```

Lens forced-edits: what each card's changes force outside the members it lists.

**BLOCKING — Card 6 (layout-mutation-record): the record value's spellings outside fabric are not all covered**
- Card 6 changes the type of `fabricengine.Mutations` used as a value, and moves `MutationRecord` out of the engine.
- Its Edit list covers `internal/fabricengine#`, `internal/fabriccli#`, `internal/stencilcli#`, `internal/pairteardown#`, `internal/hibernatecli#` and `internal/hubforge#`. The Intent names those packages as the complete set.
- The tree has non-pointer `fabricengine.Mutations` or `fabricengine.MutationRecord` spellings in two more files:
  - `internal/gatecli/scopes_integration_test.go` (1 spelling)
  - `internal/hubgeom/geometry_integration_test.go` (4 spellings)
- Once the engine's `MutationRecord` goes and `Snapshot` returns `fabriclayout.Mutations`, both test files stop compiling.
- `caller-uncovered` does not catch this, because the card deletes no member glyph and re-signs none.
- Evidence: grep for `fabricengine\.(Mutations|MutationRecord)\b`, excluding `*fabricengine.Mutations`, finds them at those paths.
- Fix: add the member glyphs whose bodies hold those spellings (or `internal/gatecli#` and `internal/hubgeom#`) to card 6's Edit.

**LOW — Cards 7, 8, 10 and 11: `internal/fabricengine/doc.go` keeps describing types that leave the package**
- Each of these cards holds `internal/fabricengine/doc.go` under Uses only.
- The doc's text names, as this package's own:
  - `OriginReport`, `BranchOutcome`, `EntryOutcome` and `ReconcileScope.Run` (the origin reconcile section)
  - `PairPush.LockWait` (the commit and push section)
  - `PullResult`, `UnwireVerbResult.WeftContent`, `MergeResult.Committed` and `DiffResult.NoWeftCorrespondence`
  - `HealthReason`
- `PATTERN-code-conventions` says a package's `doc.go` documents only that package.
- Evidence: the doc.go text in context (the Origin reconcile, Commit and push, Pairs, merge surface and public API boundary sections).
- Fix: list doc.go under Edit on each moving card, naming the `fabriclayout` home where a sentence names a moved type.

**LOW — Card 2: two test comments name the deleted unexported names, and card 2 lists neither**
- `internal/fabricengine/structuraldirs_test.go:24` names `classifyPaths`. It sits in a comment in a file card 2 does not list beyond `wiredNamesFromConfig` and the `TestHubReservedNames…` test.
- `internal/fabricengine/junctionnames_test.go:253-254`, inside `TestIsReservedHubName`, names `structuralCommittedDirs` and `structuralNeverCommittedDirs`. Card 2 removes both; card 3 moves the test as "today's table" without saying the comment changes.
- Fix: name `fabriclayout.ClassifyPaths` and `StructuralCommittedDirs()` / `StructuralNeverCommittedDirs()` in both comments, on card 2 and card 3 respectively.

**LOW — Card 4: the prose describing the narrower weftname-import subset is not updated**
- Card 4 adds `internal/fabriclayout` to the `internal/lymcwd#weftnameImportOwners` map.
- The surrounding prose and owner assertions in `internal/lymcwd/enforcement_test.go` describe that subset as fabricengine, fabriccli, gitkit and hubforge: the map's own doc comment at lines 577-579, and the import-rule comment near line 784.
- After the card that list is incomplete.
- Fix: card 4's Edit of `weftnameImportOwners` also updates the line-784 comment, which may need its enclosing member glyph listed.

**Calls made**
1. `lym quarry resolve` over the cards' glyphs: to confirm each glyph resolves. The output filter was wrong and showed nothing useful.
2. `lym quarry resolve` again with a corrected filter, plus a grep of `.md` and `.yaml` files for moved names: every glyph resolved. Docs hits outside `docs/research` are all covered by cards 2, 3 and 10.
3. `lym quarry resolve` on one glyph: to learn the output format for the filter.
4. The same resolve-and-grep pass again: to see doc references the pre-read lacked.
5. Grep for `//lym:seam` markers on fabricengine types, the `testOnlyCodeAllowed` keys, and the `destructiveguard_test.go` result-type tables: the pre-read lacked the table rows and the seam list. Only `MergeResult` and `MergeOptions` carry seams, and card 10 covers them.
6. Grep for value-type `fabricengine.Mutations` / `MutationRecord` spellings, `stringSlicesEqual` and `classifyPaths` users, fabricengine entries in `testOnlyCodeAllowed`, and watch markers: the pre-read lacked the record value's full caller set.
