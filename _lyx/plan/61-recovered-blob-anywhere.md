# Card 61 — recovered-blob-anywhere

**Edit:**
- `internal/websterengine/suspect.go`
- `internal/websterengine/gitwrap.go`
- `internal/websterengine/recoverbatch_test.go`

**Intent:**
Fix the first item of PR #321 round-4 finding G.
`checkRecoveredSuspects` looks for the flagged blob only at the suspect path, so a recovery strand that `git mv`s the file keeps the flagged content under a new name and passes.

- `gitwrap.go`: `treePathsWithBlob(worktree, commit, blob string) ([]string, error)` lists every path in `git ls-tree -r <commit>` whose object id is `blob`, sorted.
- `suspect.go`: for each suspect path with a non-empty `Blob`, `checkRecoveredSuspects` replaces its at-path lookup with `treePathsWithBlob(worktree, head, Blob)`.
  Each listed path whose `commitBlob` at `bs.StartSHA` is not `Blob` adds `suspect path <p> still holds the content the audit flagged, at <listed path>; revert it to <StartSHA> or re-derive it`;
  a path the start commit already held with the same content is legitimate and is skipped.
  The doc comment says the search covers the whole head tree.

Tests in `recoverbatch_test.go`, over the existing recovery fixture with a batch failed on a Master parent-write to `internal/x.go` holding `forged`:

- `TestPersistRecoveryTerminal_FailsWhenSuspectContentMoved`: the strand `git mv`s `internal/x.go` to `internal/y.go` and commits → `ErrBatchFailed` naming both paths.
- `TestPersistRecoveryTerminal_PassesWhenStartHeldSameContent`: the start commit already holds `forged` at `internal/z.go`, and the strand restores `internal/x.go` → done.
- `TestPersistRecoveryTerminal_PassesReverted` and `TestPersistRecoveryTerminal_PassesRederived` still pass.

**ImpactSummary:** Recovery looks for the flagged content anywhere in the recovery head's tree, not only at the suspect path, so a moved copy fails the batch again.
