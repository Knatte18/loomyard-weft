# heddle: render file targets at the worktree's anchor

`lym heddle render` wrote the `claude-file` target to `CLAUDE.local.md` at a worktree's git root.
In a hub whose worktrees have an anchor such as `src/csharp/NORCE.Models`, the repository's own `CLAUDE.md`, the workspace folder and every lym session's working directory sit at the anchor, so the operator never saw the rendered file and it lay apart from the `CLAUDE.md` it complements.
File targets now render under the worktree's anchor.
In a worktree with no anchor the anchor is the root, so such a hub renders as before.

Closes #13

## What changes

- `heddleengine`: `RenderRequest.Root` becomes `Anchor`.
  File targets, the check for the repository's `CLAUDE.md` and `.claude/CLAUDE.md` that decides whether `claude-file` imports the `AGENTS.md` files, and the `codex-file` target's opening from `AGENTS.md` all read and write under the anchor.
  `Target.File` and `Target.ReRender` are now anchor-relative, and `Report` paths are anchor-relative.
- `heddleengine` seam: `TookIn` still receives root-relative changed paths, so it joins each target's `ReRender` paths under the event's anchor before matching.
- `fabricengine`: `WorktreeEvent` gains `Anchor`.
  The warp exclude list (`warp_exclude` in `fabric.yaml`) is now anchor-relative, and wiring and `SeedWarpExclude` seed root-anchored `/<anchor>/<path>` lines into `info/exclude`, `/<path>` for anchor `.`, so the rendered files stay untracked.
- `heddlecli`: `render` passes the event's anchor, and `show` evaluates the targets against `AnchorPath()`.
  Help text and package docs say anchor directory.
- `PATTERN.md`, `PATTERN-durable-vs-ephemeral-state` and `PATTERN-heddle-instructions` are reworded from warp root to warp worktree's anchor.

## Design decisions

- No config key is added; the path stays the literal `File` of a `targetTable` row.
- The render's `.heddle` temporary directory moved to the anchor too, with the rendered files, instead of staying at the root.
  `.heddle` is on the same anchor-relative exclude list, and the rename onto each target stays within one directory.
- The anchor is resolved once, in `fabricengine.EventFor` from the location's `AnchorPath()`, and carried on the event, so heddle's engine never imports `lymcwd`.
- Exclude lines name the anchor subpath explicitly: a slash-free pattern would match at every depth, and a leading slash anchors it at the repository root.
- Existing hubs with an anchor keep a previously rendered `CLAUDE.local.md` at the root.
  The next render does not remove it, and its exclude line stays in `info/exclude`, since fabric never removes a line.

## Tests

Render, seam, exclude-seeding and CLI (`render` and `show`) tests cover a worktree with an anchor and one without.

## To check by hand

- Run `lym heddle render` in a worktree of a hub with an anchor, and confirm `CLAUDE.local.md` appears beside the repository's `CLAUDE.md` and `git status` shows no untracked file.
  This was not run against a real hub.
- A reviewer found that a base path reads the worktree root where it should read the anchor (`merriam-base-reads-worktree-root`); the finding was addressed, and no fresh reviewer has seen the result.
  Check that no remaining heddle read of repository files uses the worktree root.
- A reviewer found that wiring's anchored exclude lines were untested (`wiring-anchored-exclude-lines-untested`); a test now pins them, and no fresh reviewer has seen the result.
  Check that the test in `internal/fabricengine/warpexclude_integration_test.go` covers a non-trivial anchor.
