# heddle set/remove: render file targets into every live worktree

`lym heddle set` and `lym heddle remove` now render every file target into every live worktree of the hub after the save.
Before, `CLAUDE.local.md`, `AGENTS.override.md` and `.claude/settings.local.json` stayed stale in every existing worktree
until a take-in or a hand `lym heddle render`.
An agent in a running task worktree therefore kept working without an instruction entry that exists because it must be followed now.

Closes #103

## What changes

- After a successful write, both verbs render into the prime, beams and task pairs, in the order git lists them.
  This holds for `--layer lym` overrides too.
- The verbs print `rendered` (worktrees rendered with no file left unwritten) and `failed` (each worktree whose render failed or left a file unwritten, with its cause and the `lym heddle render <worktree>` way forward).
  `render_warnings` carries the warnings of the rendered worktrees, each opening with its worktree's name.
  It also carries the one warning given when the hub's worktrees do not list, in which case nothing is rendered.
- `--no-render` on both verbs saves without the fan-out, for all but the last of a run of edits.
  The output then carries no `rendered` or `failed` field.
- A running Claude session takes the change at its next compaction, as after any file render.
- Skipped as not live: a worktree git marks prunable, one still being made in a worktree staging directory,
  and one whose path is not a direct child of the hub.
- `ChangeStore` is unchanged for other modules' verbs and renders no worktree.
- The doc comment, help text and `pattern/PATTERN-heddle-instructions.md` describe the new behavior.

## Design decisions

- A failed worktree does not fail the verb, since the change is already saved.
  The failure is reported and exits as success.
- The renders run after the store lock is released and before the detached Bolt sync starts, so no render waits behind the sync's hold of the lock.
  A test pins the prime's `CLAUDE.local.md` as present when the sync starts.
- `heddleCLI.syncWarning` is removed.
  `syncAfterSave` returns the warning instead, so the state no longer lives on the struct.
- `heddle render` and the fan-out share `renderInto`, so both seed the exclude lines the same way.
- The hub comparison in `samePath` resolves symlinks and folds case on Windows.

## To check by hand

- Failed-worktree branch and `render_warnings`: confirm the failure path of `renderEveryWorktree` and the `render_warnings` output behave as the new integration test `WriteRendersEveryLiveWorktree` pins them (`internal/heddlecli/cli_integration_test.go`).
- `--no-render` and the override layer: confirm the fan-out and `--no-render` also cover `set --layer lym` and `remove --layer lym` in `internal/heddlecli/verbs.go`.
- Staging and out-of-hub worktrees: confirm `renderEveryWorktree` in `internal/heddlecli/verbs.go` skips them as intended.
- The regression check did not run for this task, and the change was not exercised against a hub with real running sessions.
