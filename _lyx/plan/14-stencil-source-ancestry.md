# Card 14 — The stencil-seed caller supplies the build ancestry

**Create:**
- `plan:internal/fabricengine#StencilSource` -> `func StencilSource(worktreePath, sourceDir, revision string) stencilstore.Source`

**Edit:**
- `cmd/lyx#seedStencilsAt`
- `internal/stencilcli#Command`
- `internal/fabricengine/warpforward_integration_test.go`
- `internal/stencilstore/doc.go`
- `docs/overview.md`

**Uses:**
- `plan:internal/fabricengine#HeadContains`
- `internal/buildvcs#Running`
- `plan:internal/stencilstore#Source`
- `plan:internal/stencilstore#BuildAncestry`

**Intent:**
Decision 13: the ancestry check runs in `internal/fabricengine`, only when a drift comparison needs it, and reaches `stencilstore` as card 13's `Source.Build` input.
The Part 1 stamp is not used.

`StencilSource` (in `warpforward.go`, beside `HeadContains`):
- An empty `sourceDir` returns the zero `Source`.
- An empty `revision` (an unstamped binary) returns `Source{Dir: sourceDir}` with a nil `Build`, so today's classification stands.
- Otherwise `Build` is a closure that runs `HeadContains(worktreePath, revision)` at most once per `Source`, memoized with `sync.Once`, so one seed pass with several drifted stencils reads ancestry once.
  It returns `BuildInHead` or `BuildNotInHead`.
  A read error is logged with `logger.Warn` naming the worktree and returns `BuildAncestryUnknown`.
- The closure is never called by a seed pass whose stencils do not drift, so such a pass runs no git.

`seedStencilsAt` builds the stencils pass's source as `fabricengine.StencilSource(worktree, sourceDir, buildvcs.Running().Revision)`.
`stencilcli`'s sync verb builds it the same way from `l.WorktreePath()` and `resolveSourceDir(l)`.
Both specs passes keep `stencilstore.Source{}`.

`internal/stencilstore/doc.go` gains a sentence saying the drift warning's ancestry input is told by the caller and `stencilstore` runs no git.
`docs/overview.md`'s stencil module paragraph changes "classifies a differing board copy four ways" to five, adding `behind`: a worktree whose HEAD lacks the binary's build commit, for which the remedy is syncing with main, not a deploy.

Tests in `warpforward_integration_test.go`, one table test of `StencilSource` on a hubforge prime:
- An empty `sourceDir` gives the zero `Source`.
- An empty `revision` gives a nil `Build`.
- The prime's HEAD sha gives `BuildInHead`.
- An absent well-formed sha gives `BuildNotInHead`.
- Calling `Build` twice on one `Source` returns the same answer.

**ImpactSummary:** The stencil seed pass and `lyx stencil sync` now pass a lazily-read ancestry into the drift warning; a pass with no drifting stencil is unchanged.

**Commit:** 14: stencil seed: tell the drift warning whether the worktree holds the build commit
