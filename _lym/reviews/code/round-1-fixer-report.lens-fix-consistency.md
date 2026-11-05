```yaml
lens: fix-consistency
outcome: reported
```

Checked the change set for consistency with its surroundings. Two findings.

**MEDIUM — `contracts/specs/loom-plan-spec.md:128` (F8)**
- **Changed text:** "under the worktree anchor's git-ignored `.scratch/` is a transient draft".
- **Problem:** The spec now names the anchor, but the code it describes still skips only the worktree-root prefix. The sibling texts also still say "root", so the spec contradicts all three.
- **Evidence:**
  - `internal/planglyph/donecheck.go:27-29` sets `scratchDirPrefix = ".scratch/"`, which is matched against a root-relative key at line 104.
  - `internal/planglyph/donecheck.go:62` says "under the worktree root's .scratch/".
  - `internal/planglyph/doc.go:137-138` says "worktree root's ... a `.scratch/` below the root ... keep[s] [its] done-check".
- **Effect:** At a subpath anchor, a draft at `backend/.scratch/x` still gets `create-not-done`, contrary to the spec. The review asked the fixer to check this skip rule, and the change set neither adjusts it nor reconciles the sibling text.

**LOW — `internal/cleanupcli/cleanup_integration_test.go:107`**
- **Changed text:** The new test builds `scratchRoot` from `h.Location.AnchorRel`, but the existing test beside it still uses `filepath.Join(h.PrimeWorktree(), scratchDirName)` with its comment "the prime's .scratch directory".
- **Problem:** The two tests now disagree on how the path is built. The old one is correct only because its anchor is `.`.
- **Effect:** Cosmetic. It is harmless at anchor `.`.

I found no other stale "links" or "root" census wording. The `doc.go` wording for the root exclude line, the anchor-change line and the `lym fabric remove` paragraph now agrees with `scratchlink.go`.

Tool calls (1 of 6):
1. `grep` for stale census and root-scratch wording across `.go` and `.md` files, because my context did not show other mentions outside the files the fixer changed.
