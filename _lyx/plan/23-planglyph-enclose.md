# Card 23 — planglyph EncloseAt

**Create:**
- `internal/planglyph#EncloseAt`
- `internal/planglyph#TestEncloseAt`

**Edit:**
- `internal/planglyph/doc.go`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`
- `docs/overview.md`

**Uses:**
- `internal/planglyph#openRepo`

**Intent:**
D9 step 2: quarry's `EncloseAt`, which `go.mod`'s quarry v0.4.0 already ships, is reached only through `internal/planglyph`, which owns the quarry repo handle.

`EncloseAt(worktreeRoot, rev, targets)` opens the repository through `openRepo` and returns `(*quarry.Repo).EncloseAt(rev, targets)` unchanged, one answer per target; an empty `rev` answers the working tree.
A failed call is wrapped in `ErrQuarryUnavailable`, as the package's other quarry boundaries are, and an answer whose length differs from the targets' is the same wrapped error, the positional guard `ensureResolveCoverage` gives `Resolve`.
It is the package's one call of `EncloseAt`, beside its one `Resolve` and one `DeltaGit`.

`pattern/PATTERN-glyph-conversion-chokepoint.md` gains a bullet: quarry's `Enclose` is a location-to-glyph conversion done by quarry, and `planglyph.EncloseAt` is its one sanctioned call; the overview line is unchanged.
`internal/planglyph/doc.go` lists `EncloseAt` among the quarry wrappers.
`EncloseAt` goes in `repo.go` beside the other query wrappers, and that file's header comment, which lists `TOC`, `Glyphs`, `Resolve` and `Expand`, names it too.
`docs/overview.md`'s line calling planglyph the sole owner of every `quarry.Repo` call (`Open`, `Resolve`, `DeltaGit`) adds `EncloseAt`.

`TestEncloseAt`, in package `planglyph` and tagged like the package's other git-backed tests, commits a Go file with one function and an import into a hermetic repository (`gitkit.HermeticGitEnv()`), then asserts: a line inside the function at the commit answers found with that function's glyph; the import line answers `not_found` with the file's self glyph as `Unit`; a `.md` file location is a rejection; an unknown revision is an error wrapping `ErrQuarryUnavailable`.

**ImpactSummary:** additive: one new wrapper, and one background-file bullet naming it; no existing caller or the PATTERN overview changes.
