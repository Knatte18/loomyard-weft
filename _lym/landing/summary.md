# Fix comment cleanup leftovers: stale docs, dead helper, joint edits

Settles the defects the `comment-cleanup` landing (`f7465bd8`) found but left.
The change touches comments, docs and one dead function; no behaviour changes.

## What changes

- `lymcwd.ResolveWithAnchor` and its shared body `resolveWithAnchorCore` are removed.
  Neither had a non-test caller; `ResolveWorktree` remains the ungated variant.
  `docs/overview.md`, `docs/shared-libs/lymcwd.md` and the `internal/lymcwd/gate_test.go` comments no longer mention it.
- `docs/shared-libs/configengine.md`:
  - The layout tree shows the current worktree, local and hub layers instead of the old `discussion.md`/`plan.md` files.
  - The "Migration from old format" and "Exported functions / `LymDirName`" headings now say what their sections hold.
  - The `Load`, `LoadOrTemplate`, `Set` and `Edit` headings carry their current `Dirs`/`Spec`/`Layer` signatures, with the "current signature is" notes dropped.
  - `ConfigDir`, `StagingFile` and `ConfigFileRel` use the `lymdirs.` qualifier.
- Comments repaired where two cleanup workers edited the same span:
  - `cmd/lym/stencilseed.go`: a split sentence.
  - `internal/adoptcli/doc.go`: a false claim about how the `primeMark` marks are sorted.
  - `internal/websterengine/render.go`: the `specsDir`/`gateTiers` requirement and the `plan/00-overview.md` path, both now matching the code.
  - Test-file header comments in `cmd/lym` and `internal/adoptcli`: wrapped or banned-token wording fixed.

## To check by hand

- The stencils under `contracts/stencils/loom/` still reference `{{.specs_dir}}/loom/...` paths (for example `loom-plan-spec.md` and `loom-plan-checks-spec.md`).
  The diff does not touch them; confirm whether those folders exist in the specs repository, or fix the paths.
- The edited comment claims in `internal/adoptcli/doc.go` and `internal/websterengine/render.go` were checked against the code, not by a test.

No regression check ran for this task.
