```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the search behind each set-shaped finding (F1, F2, F7, F8). F1, F2 and F7 are covered; F8 has one sibling left stale.

**Findings**

- **MEDIUM — F8, the plan-spec edit.**
  - Changed text: `contracts/specs/loom-plan-spec.md:128`, which now says "the worktree anchor's git-ignored `.scratch/`".
  - Summary: the spec now names the anchor's `.scratch/`, but the done-check code and doc comments that implement the skip still say the worktree root's `.scratch/`.
  - Evidence: `internal/planglyph/donecheck.go:62` says "under the worktree root's .scratch/". `internal/planglyph/doc.go:137` says "under the worktree root's git-ignored `.scratch/`". `scratchDirPrefix = ".scratch/"` at `donecheck.go:29` is a root-relative prefix match, and `DoneChecks` takes only `worktreeRoot`, so no anchor reaches it. At a subpath anchor a draft at `backend/.scratch/x` therefore still gets `create-not-done`, which contradicts the spec's "Webster's `create-not-done` check skips it". The review's fix told the fixer to check this skip for a root assumption, and it does assume the root. The fix either needs to carry the rule through to the code, or the spec edit needs to name the root as the code does and be flagged as an open decision.

- **LOW — F7 and F5 test helper comments.**
  - Changed text: `scratchlink_integration_test.go:107`, "removeRootLink removes the prime pair's root link, leaving the root absent", and `:93`, "replaceLinkWithRealDirectory removes the root link".
  - Summary: both helpers now run on the anchor's `.scratch` at `backend`, but their comments and many "root .scratch" failure messages still say "root".
  - Evidence: the `Leaves_*_Anchor_backend` rows pass the anchor link to `removeRootLink` and `replaceLinkWithRealDirectory`. This is cosmetic.

**Checked, no gap**

- **F1:** `scratchLinkedInSibling` is called from one place, `unseedScratchExclude`. That function's two callers are `scratchlink.go:222` and `anchorchange.go:475`, and both go through the new `Lstat` census. No other consumer still uses `IsLink` for it.
- **F2:** I found no other prose that still describes `remove` as unlinking only a root link. The remaining "root link" hits in `scratchlink.go` and its test comments describe the legacy root link correctly.
- **F7:** every `scratchShapeCases` loop (wire and reconcile), the adopt row and the reconcile adopt row now run at both anchors. The rows still at `"."` are the refused-after-classify rows and the CorrectLink rows, which the review did not name.

**Left unchecked**

- F3 (cleanup), F4 (anchor-change exclude), F5 (`Remove` of a legacy root link) and F6 (non-fabric root entries) are single-site coverage gaps, not set-shaped. I did not rerun a search for them.

**Tool calls**

1. A grep for `scratchLinkedInSibling` and `unseedScratchExclude` callers, plus root-scratch wording across go and md files. It covered the F1 and F2 sets, which my context lacked.
2. A grep for `Anchor: "."` rows in `scratchlink_integration_test.go`, plus other tests that touch `.scratch`. It covered the F7 set.
3. A grep for prose naming the repository or worktree root's `.scratch`. It covered the F8 set and turned up the `planglyph` siblings.
