# heddle set and remove re-render every live worktree of the hub

`lym heddle set` and `lym heddle remove` saved the store and rendered `HEDDLE.md`,
but left every existing worktree's rendered files (`CLAUDE.local.md`, `AGENTS.override.md`, `.claude/settings.local.json`) stale until a take-in or a hand `lym heddle render`.
An entry an agent must follow now therefore never reached agents in running task worktrees, and nothing named the stale worktrees.
Fixes issue #103.

## What changes

- After a successful save, both verbs render every file target into each worktree git lists for the hub: the prime, beams and task pairs.
  Each render seeds the worktree's exclude lines first, as the hand render does.
  A worktree git marks prunable is skipped, since its directory is gone.
- The output gains `rendered`, the worktrees rendered whole, and `failed`, one `{worktree, error}` per worktree whose render failed or left a file unwritten.
  A failure's error carries its cause and the `lym heddle render <worktree>` way forward.
  Warnings of a rendered worktree appear in `render_warnings`, each prefixed with the worktree name.
- A failed worktree does not fail the verb: the change is saved, and the operator heals the named worktree by hand.
- `--no-render` on `set` and `remove` skips the fan-out, for all but the last of a run of bulk edits; the output then carries neither field.
- Running Claude sessions take the change at their next compaction, as they already did after a file render.

## Design decisions

- The fan-out lives in `internal/heddlecli`, beside the hand render it shares `renderInto` with; rendering stays in `heddleengine`, and fabric only lists worktrees and seeds exclude lines.
- The Bolt sync now starts after the renders rather than before.
  Each render reads the store under the Bolt write lock, and the detached sync takes that lock to commit, so starting it last keeps the renders from waiting behind it.
- `ChangeStore`, the write path another module's verb uses (`lym orch rules`), renders no worktree.
  Its entries are prompt targets, which reach a session at launch and need no file render.
- The `failed` list is always present, empty when nothing failed, so a caller can test it without a presence check.

## Tests

`TestHeddleCLI/WriteRendersEveryLiveWorktree` drives a hub with a prime and a pair through a set, a `--no-render` set, a remove where the pair holds an unmarked `CLAUDE.local.md` (reported in `failed` while the prime renders), and a set after the pair's directory is removed (skipped as prunable).

## Docs

The `heddlecli` package doc and verb help, the `heddleengine` seam comment and `pattern/PATTERN-heddle-instructions.md` name the write's render as a way file targets reach a worktree.
