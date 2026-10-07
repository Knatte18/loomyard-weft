# Card 2 — planindex-seam

**Create:**
- `internal/planindex/doc.go`
- `internal/planindex/planindex.go`
- `internal/planglyph/index.go`
- `internal/planindex#Index`
- `internal/planglyph#NewIndex`

**Edit:**
- `internal/planglyph/repo.go`
- `internal/planglyph/doc.go`
- `internal/planglyph/delta_integration_test.go`
- `docs/overview.md`

**Intent:**
Decision D8's seam, first half: a cgo-free package that names the code index without linking it.
`internal/planglyph` imports `quarry/quarry`, which links tree-sitter, and every package importing `planglyph` links it too.
`internal/planindex` imports only the standard library and `internal/planparser` (which is cgo-free), and declares:

- `Finding`, `Severity`, `SeverityBlocking`, `SeverityInformational` and `ErrQuarryUnavailable`, moved here from `planglyph/repo.go` with their doc comments and unchanged error text.
  `planglyph` keeps its names as aliases (`type Finding = planindex.Finding`, the constants re-declared from planindex's, `var ErrQuarryUnavailable = planindex.ErrQuarryUnavailable`), so every `planglyph` caller compiles unchanged and `errors.Is` matches across both names.
- `Index`, the interface a package that calls the code index receives instead of importing `planglyph`: `ValidateFormat`, `ValidateRework`, `ValidateDispatch`, `DoneChecks` and `LaterDeleteReferences`, with `planglyph`'s current signatures, and `Delta(worktree, fromSHA, toSHA string) (Delta, error)`.
- `Delta`, the interface over one batch's git delta, so no caller names `quarry.GitDeltaAnswer`: `BindHandles(plan, planDir, cards)`, `ScopeGuard(cards)` and `DetectDrift(fullPlan, completed, planDir, worktreeRoot, sha, now)`, the last computing the pending plan from `completed` as `planglyph.PendingPlan` does today.
  On an `ErrQuarryUnavailable` error `Index.Delta` returns the empty delta, never nil, because `websterengine` calls `BindHandles` on the zero delta after that error today.

`planglyph.NewIndex` returns the real `Index`, each method forwarding to the existing `planglyph` function, with a compile-time assertion that it satisfies `planindex.Index`.
The forwarding is covered by `planglyph`'s existing tests; `delta_integration_test.go` gains the row that `Index.Delta` on an unreadable range returns `ErrQuarryUnavailable` with a usable empty delta.
`docs/overview.md`'s package tree lists `internal/planindex` beside `internal/planglyph`.

**ImpactSummary:** Moves `planglyph`'s finding types into a new package behind aliases; no caller changes in this card.
