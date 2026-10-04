---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/14-vocab-sweep-tests-core.md, _lyx/plan/16-vocab-sweep-tests-rest.md
    summary: Cards 14 and 16 list two test files whose only weft/warp hits are the vocab-exempt token PairWarpWorktree, so there is nothing to reword
---

Focus items 1 to 3 settled with no defect; item 4 produced F1.

### [LOW] Two sweep-card files carry only vocab-exempt hits

**Location:** `_lyx/plan/14-vocab-sweep-tests-core.md` (`internal/hubgeom/reedgeom_integration_test.go`), `_lyx/plan/16-vocab-sweep-tests-rest.md` (`internal/orchcli/smoke_live_test.go`).
**Issue:** The `vocab-sweep` rule sweeps hits "outside the `vocab-exempt` identifiers".
I stripped the exempt tokens from every `_test.go` file the three cards list and counted remaining case-insensitive `weft`/`warp` substrings.
Every listed file keeps a hit except these two.
`reedgeom_integration_test.go` has one hit, `h.PairWarpWorktree(slug)` at line 41, and `smoke_live_test.go` has one hit, `f.hub.PairWarpWorktree(slug)` at line 399.
Both spellings are in the `vocab-exempt` table, so the card has nothing to change there and the Edit entry is a no-op.
The reverse direction is clean: every non-owner test file with a remaining hit is listed by card 14, 15 or 16, except `internal/lyxcwd/{anchor,docslink,enforcement,geometry}_test.go`, which card 18 owns.
**Fix:** Remove both files from the Edit lists.

## Settled focus questions

- **Repo root in a subpath-anchored hub (card 2, `directive-root`).**
  The plan gives it a value: a told `RepoRoot` on both engines' `Geometry`, filled with `l.WorktreePath()` by `hubgeom`'s burler and webster constructors and with the target by `standalonegeom`'s.
  `WorktreeRoot` keeps its anchor-path meaning, which `internal/hubgeom/hubgeom.go:100` and `internal/hubgeom/webstergeom.go:18` require.
  The card lists `hubgeom_test.go`, `webstergeom_test.go` and `standalonegeom_test.go` assertions for the field, including the subpath case where it differs from the anchor path, and subpath-anchored burler and webster directive tests.
  Not a defect.
- **Parameter handling in `RenderMasterPrompt` and `RenderRecoveryPrompt` (card 2).**
  In `internal/websterengine/render.go`, `anchorRoot` feeds `pattern.Directive` alone in both functions, `worktreeRoot` feeds only `masterPlanDirDisplay`, and `promptWorktreeRoot` feeds the card pointers and `{{.worktree_root}}`.
  The card replaces `anchorRoot` with `repoRoot` in both, keeps `worktreeRoot` implicitly and `promptWorktreeRoot` by name, and names the callers `runlevel.go` and `recoverbatch.go` plus `template_test.go`'s calls.
  A grep for both function names finds no other caller.
  Not a defect.
- **Whitespace-only `PATTERN.md` (card 2 Intent).**
  The Intent states what is skipped and the bound: only whitespace-only content is inactive, any other content is inlined, and card 6's real-file test fails an emptied loomyard overview.
  Not a defect.
- **Card 18 scan boundaries.**
  The card states the `internal/configsync` test-file carve-out (literals and comments allowed, identifiers not), that every other owner directory is skipped whole, and that the repo-root `sandbox/` tree is matched on the path's first segment so `tools/sandbox/` is scanned.
  Not a defect.
