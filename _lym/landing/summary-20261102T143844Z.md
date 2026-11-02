# Render heddle's file targets at the worktree's anchor, not the git root

Fixes [issue #13](https://github.com/Knatte18/loomyard/issues/13).
In a hub whose worktrees have an anchor, such as `src/csharp/NORCE.Models`, `lym heddle render` wrote `CLAUDE.local.md` and `AGENTS.override.md` at the git root.
That put them apart from the repository's own `CLAUDE.md` and outside the VS Code folder and every session's working directory.
File targets now render under the worktree's anchor.
In a worktree with no anchor, the anchor is the root, so such a hub renders exactly as before.
No config key is added.

## What changed

- **Fabric's worktree event carries the anchor.**
  `fabricengine.WorktreeEvent` gains `Anchor`, filled by `EventFor` from `Location.AnchorPath()`.
  This is a general geometry field on the seam and names no heddle file, so fabric stays isolated from heddle.
- **The render works at the anchor.**
  `heddleengine.RenderRequest.Root` becomes `Anchor`.
  The target files, the `.heddle` temporary directory, the claude-file check for `CLAUDE.md` and `.claude/CLAUDE.md`, its `AGENTS.md` imports and the codex-file's opening `AGENTS.md` all resolve against it.
  `Renderer.Show` and `lym heddle show` read from the anchor too.
- **Take-in re-renders follow the anchor.**
  `Seam.TookIn` joins each target's anchor-relative `ReRender` paths onto the anchor's path within the worktree before matching the event's root-relative changed paths.
  In an anchored worktree a change to `src/app/AGENTS.md` re-renders, and a change to an `AGENTS.md` above the anchor no longer does.
- **The warp exclude lines name the anchor.**
  `warp_exclude` entries are now relative to the anchor.
  Wiring and `SeedWarpExclude` seed `/<anchor>/<path>` lines, or `/<path>` for anchor `.`, so the rendered files never show as untracked.
  Lines are still only added, never removed, so root lines an older binary seeded stay harmless.

## Design decisions

- **The temporary directory moved to the anchor as well.**
  The design allowed `.heddle` to stay at the root.
  Keeping it beside the target files means one exclude rule covers every listed path, and the rename onto each target stays inside one directory tree.
- **The anchor comes from fabric's event, not from heddle.**
  Both render callers, the worktree seam and `lym heddle render`, already build their request from `fabricengine.EventFor`, which holds the resolved `Location`.
  Carrying `AnchorPath()` on the event keeps heddle an engine that is handed absolute paths and never imports `lymcwd` (PATTERN-cwd-resolution).
- **Old root renders are not swept.**
  A marked `CLAUDE.local.md` that an earlier render left at an anchored worktree's root is not removed.
  The affected hub held its heddle migration until this fix, so no such file is expected.

## Docs

The package docs of `heddleengine` and the warp-exclude paragraph of `fabricengine` now name the anchor, as does the `warp_exclude` comment in fabric's template.
`PATTERN-heddle-instructions` and `PATTERN-durable-vs-ephemeral-state`, with its `PATTERN.md` line, now place the rendered files and `.heddle` at a warp worktree's anchor.

## Tests

- `TestSeam` runs Made and TookIn for a worktree with no anchor and for one anchored at `src/app`, including a changed path above the anchor that must not re-render.
- `TestRender_WritesAtTheAnchorAndLeavesNoUntrackedFile` (tier 1) renders into a hubforge prime with anchor `.` and with anchor `backend`.
  It checks that both files land at the anchor and none at the root, and that `git status` stays empty.
- `TestSeedWarpExclude_SeedsWarpOnlyOnce` gains a subtest that an anchored hub's exclude lines name the anchor and that none names the root.
- The render unit tests run unchanged against the anchor directory.
