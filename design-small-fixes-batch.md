# Small fixes: darn prompt paths, recipe doc line, stale reconcile default, destructive-verb confirm, index note audience, carry-over on disk

`small-fixes-batch` · Task · bug, enhancement, stencil, batten, config, fabric, board, loom · running · Webster

Six small, independent fixes, merged from notes on 2026-10-11.

## Darn prompt paths (was `darn-prompt-paths`, issue #531)

The darn agent prompt contains two instructions that fail when followed literally.

- It says to read the task with `lyx board get <slug>`, but `board get` takes a JSON object, so the bare slug fails with `invalid json`.
  `lyx board get '{"slug":"<slug>"}'` works.
- It names the final-summary spec at `<hub>/_board/_lyx/specs/final-summary-spec.md`, which does not exist.
  The spec lives in the repository at `contracts/specs/final-summary-spec.md`, and the agent had to find that copy itself.

Fix the darn prompt stencil to emit the JSON-object form of `lyx board get` and the `contracts/specs/final-summary-spec.md` path.

## overview.md recipe line and the batten window marker (was `overview-recipe-line`)

From the ide-opt-in PR-Gate review (PR #530, 2026-10-10): `docs/overview.md` around line 49 says a board entry's recipe "must be `loom` or empty", but the darn recipe has landed.
Also: batten's terminal-window marker is deleted when the child has no status file, so a status file vanishing mid-run (seen once in cli-surface) opens a second window.

## Reconcile keeps a stale default (was `reconcile-stale-default`)

Seen (2026-10-10): after `daemon-wakeups` raised `poll_interval_ms` in the shuttle template to 1000 with a floor of 1000, `lyx config reconcile` kept the prime's and every pair's old default `500`, so every `lyx` command in them logs "shuttle config: poll_interval_ms is below the floor and is raised to it".

Idea: reconcile updates a value that still equals the old template default, or at least one that the new floor would raise, and says so in its report.

## Destructive no-argument verbs (was `destructive-verb-confirm`)

In the #512 incident (2026-10-10) a reviewer probing the CLI ran `lyx fabric unwire` and `lyx board retire-legacy` bare in a task pair; both acted at once, unwiring the pair and deleting legacy board files.
57243019b closed the route it took (global `--json` never runs a command), but any bare call of these verbs still acts.

Idea: give destructive verbs that take no argument (`fabric unwire`, `board retire-legacy`, `board prune`) an explicit confirmation flag, so a bare call prints what it would do and changes nothing.
Every caller of these verbs in lyx itself (stencils, skills, the orch role, Go callers) passes the flag in the same change.

## Board group index note audience (was `index-note-audience`)

Webster-Review in cli-surface (PR #519) found the board group's help-index note also prints under the internal audience; the run disputed it as planned.
Idea: scope a group's index note by audience like its commands.

## Carry-over check on disk (was `carryover-verify-on-disk`)

`WriteCarryOver` checks the splice on the produced lines before the atomic rename; re-reading the file after the rename and checking what is on disk would also catch a concurrent writer.
