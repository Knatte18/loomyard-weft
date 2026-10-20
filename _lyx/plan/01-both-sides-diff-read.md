# Card 1 — both-sides-diff-read

**Create:**
- `plan:internal/fabricengine#Fabric.ChangedPathsBothSides` -> `func (f *Fabric) ChangedPathsBothSides(from, to string) ([]string, error)`
- `plan:internal/fabricengine_test#TestChangedPathsBothSides_NamesRenamesAndRawPaths` -> `func TestChangedPathsBothSides_NamesRenamesAndRawPaths(t *testing.T)`

**Uses:**
- `internal/fabricengine#Fabric.ChangedPaths`
- `internal/fabricengine_test#buildReconcileFixture`
- `internal/fabricengine#CurrentSHAForTest`
- `internal/gitkit#CommitFile`
- `internal/gitkit#Git`

**Intent:** Add the warp read darn's tree classification (D2) runs: `Fabric.ChangedPathsBothSides` returns every repository-relative path that differs between `from` and `to` in the pair's code repository, a rename by both its old and its new path, sorted and deduplicated.
It runs `git diff --name-only -z --no-renames <from> <to>` through `gitexec.Run` in the warp checkout, beside `ChangedPaths` in `internal/fabricengine/parenttips.go`, and splits the output on NUL, so each path arrives verbatim and never in core.quotePath's C-quoted form.
`--no-renames` reports a rename as a deletion of the old path and an addition of the new one, so a rename or deletion out of production code is seen by its old path.
A git failure is returned wrapped with `from` and `to`, as `ChangedPaths` does.
`ChangedPaths` and `diffNames` are left as they are, so the Merge-In overlap record and the peers `changed_paths` line keep their output.

The new integration test, in `parenttips_integration_test.go` beside `TestChangedPaths_NamesExactlyTheCommittedFiles`, builds the fixture with `buildReconcileFixture`, commits a file `docs/ä.md`, then a `git mv` of a committed `internal/x/x.go` to `tools/x.go`, and asserts the read from the first commit to HEAD equals exactly `docs/ä.md`, `internal/x/x.go` and `tools/x.go`, with `ä` spelled raw.
A second assertion reads from HEAD to HEAD and gets no path.
A third reads from an unknown ref and gets an error naming it.
