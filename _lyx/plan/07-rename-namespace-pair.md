# Card 7 — rename-namespace-pair

**Create:**
- `plan:internal/planparser#checkRenameNamespacePair` -> `func checkRenameNamespacePair(plan *Plan, worktreeRoot string) []ValidationError`
- `plan:internal/planparser_test#TestValidate_RenameNamespacePair` -> `func TestValidate_RenameNamespacePair(t *testing.T)`

**Edit:**
- `internal/planparser#validate`

**Uses:**
- `plan:internal/planparser#PathEntryPath`

**Intent:** Refuse a `Rename` pair that names a C# namespace (D3), with the new blocking finding `rename-namespace-pair`.

Under `csharp` a pair of two self glyphs is a file pair and runs `git mv`, so `Glyphchain.Sharp# -> Glyphchain.Core#` would move a path that does not exist.
`checkRenameNamespacePair`, registered in `validate` right after `checkPathMissing`, reports, under `csharp` only, every pair whose old side is a slash-free self glyph whose `PathEntryPath` reading names nothing on disk under `worktreeRoot`, file or directory, and nothing a Create target or Rename destination of an earlier card names, each read through `PathEntryPath`, never through `createTargetsUnion` or `renameTargetsUnion`, whose `diskPathForRef` drops every slash-free self glyph under `csharp`.
It reads the disk rather than the extension, since a dotted namespace has a path extension too.
Its detail names the pair and both ways forward: rename the namespace through Edit cards on the files that declare it and the files that refer to it, or, when the old side is meant as a root file, correct it to a file that exists.
The package comment's check list in `validate.go` names it.

It runs only on pending cards: `ValidateDispatch` and `ValidateFormatAfter` drop a finding against a done card, so a pair whose `git mv` has landed, and whose old side is gone from disk, is never rechecked.
Under `go` the same pair is `path-missing`'s, as today, so this check stays silent there.

Tests: `TestValidate_RenameNamespacePair` writes a temp worktree holding `Program.cs` and runs `Validate` over `csharp` plans: `Glyphchain.Sharp# -> Glyphchain.Core#` is one finding naming the way forward; `Program.cs -> App.cs` is none; a root pair `Tool.cs -> Util.cs` whose old side card 1 creates and card 2 renames is none; a pair `Old.cs -> New.cs` whose old side card 1 renames into from `Program.cs` is none; a pair whose old side `docs#` names a directory the worktree holds is none; a slashed pair `src/A.cs -> src/B.cs` with no such files is none, since the check reads only slash-free old sides; and the first plan under `go` draws no `rename-namespace-pair`.

**ImpactSummary:** one new blocking check, scoped to `csharp` plans' Rename pairs, registered in the shared dispatch list; no existing plan is `csharp`.
