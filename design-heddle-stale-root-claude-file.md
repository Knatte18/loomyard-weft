# heddle render leaves the old CLAUDE.local.md at the worktree root after the anchor fix

`heddle-stale-root-claude-file` · Note · bug, heddle

Imported from [issue #16](https://github.com/Knatte18/loomyard/issues/16).

Seen in hub Models (worktree anchor src/csharp/NORCE.Models), after the fix for heddle-render-ignores-anchor (issue 13).

- `lym heddle render` now writes CLAUDE.local.md to the anchor, as intended.
- The copy it wrote before the fix stays at the worktree root (loomyard-setup/CLAUDE.local.md, duplication-and-import-followups/CLAUDE.local.md, Models/CLAUDE.local.md). Render reports `removed: []`.
- Claude Code loads CLAUDE.local.md from every ancestor of the working directory, so a session at the anchor loads both: the stale root copy and the current anchor copy, which duplicates lym's own instructions and keeps any entry removed since.
- Existing worktrees that were not re-rendered (the prime Models, the beam duplication-and-import-followups) have only the stale root copy and no anchor copy. A worktree created after the fix (drift-baseline-harness) got the anchor copy at creation.

Expected: render removes a claude-file it rendered earlier at a path that is no longer the target (it carries lym's 'rendered by lym heddle' marker), and existing worktrees get the anchor copy without a manual `lym heddle render <worktree>`, or the upgrade says to run it.

Workaround: none applied; the stale root files are left for the operator to delete.