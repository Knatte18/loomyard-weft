Lens fix-proposal: one finding, on F4.

**LOW** — F4, the `show claude-file` assertion in `RenderWritesFileTargetsAtTheAnchor`: it does not pin the anchor.
- It checks exit code 0 and "text of rules", which the seeded store yields for any directory.
- If `verbs.go:335` reverted from `AnchorPath()` to the worktree root, the output would not change.
- Fix: make the output depend on the directory, or drop the show lines.

**F2** `anchored_hub_wiring_seeds_the_lines_under_the_anchor`: no finding. A wrong anchor at the wiring call fails the row.

**F3** help text and package doc: no finding. No stale "worktree's root" remains outside `render.go:37`.

Tool calls:
1. grep for leftover "worktree's root" wording in `internal/heddlecli` and `internal/heddleengine`: the context did not show `heddleengine`'s docs.
