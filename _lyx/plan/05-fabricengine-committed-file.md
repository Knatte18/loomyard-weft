# Card 5 — fabricengine-committed-file

**Create:**
- `internal/fabricengine/committedfile.go`
- `internal/fabricengine/committedfile_integration_test.go`

**Uses:**
- `internal/fabricengine/commitweftpaths.go`
- `internal/gitrepo/blobread.go`

**Intent:**
Give loom a read-only way to read an anchored `_lyx` file as committed at HEAD, without naming the side of the pair that holds it (Fabric Git Invariant: read-only verbs are exempt from the commit seam, but still go through `internal/fabricengine`).
`CommittedAnchoredFile(l *lyxcwd.Location, relPath string) (data []byte, found bool, err error)` is the read-only twin of `CommitAnchoredPaths`:
it resolves the same commit target from `l` (`WeftWorktree(l)` and `l.AnchorRel`, joined through `ScopedPathspec` so the anchor join lives in one place), reads that repository's HEAD SHA, and returns `gitrepo`'s `FileAtRevision` for the joined path.
`gitrepo.ErrPathNotAtRevision` maps to `found == false` with a nil error, as does an unborn HEAD;
every other failure is an error.
The doc comment states the vocabulary-neutral contract the rework row relies on: `relPath` is anchor-relative, the shape `LoomReworkDirRel` and `planparser.PlanDirRel` return.
The integration test builds a wired hub fixture through `internal/hubforge`, commits an anchored file, then modifies it in the working tree and asserts the committed bytes come back;
a path never committed returns `found == false`.

**Verify:** go test -tags integration -run CommittedAnchoredFile ./internal/fabricengine/
