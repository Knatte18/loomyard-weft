# Run parentage from the origin worktree; batten waits through halts

A run in a task pair now finds its parent from the worktree the pair was created from, not from the caller's `LYX_STRAND_NAME`.
`lyx batten run <slug>` from a plain shell in the prime reaches the prime's orch session.
Batten no longer exits `failed` when the inner run halts, so the pair keeps a watcher that lands and tears it down.
Parent-review reviews every rewrite after a reject, up to a cap, and escalates there instead of letting the rewrite through unseen.

Closes #341
Closes #338

## What changes

- `fabricengine`'s origin record gains a write-once `parent_worktree`, written with `parent_branch` when `lyx fabric add` (and batten's Worktree-Create) creates a pair.
  The legacy origin write in `lyx loom start`'s bootstrap leaves it empty.
- `hubgeom` resolves a pair's parent agent name at use, through `internal/agentname`, as the orch role's name in the parent worktree.
  It feeds reed's `LYX_PARENT` and the parent-review reviewer.
  The name is never stored and no liveness is checked.
- No code path writes `shedrun.Seed.Parent` any more: `lyx batten run`, Seed-Child and the loom bootstrap dropped it.
  The field stays so old seeds still decode, and the resolver falls back to it when the origin record has no `parent_worktree`.
- Parent-review opens a new round after each consumed reject.
  The entry's `attempts: 3` is the cap on rejected rounds, counted from the round store so an attach or restart never resets it.
  At the cap, Discussion-Write fails terminally and the run halts `blocked` with a reason naming the slug, rounds, latest review file and the way out.
- `lyx loom review approve <slug>` on a run halted at the cap supersedes the cap's reject, and the next `lyx loom start` lets the rewrite through.
  The help text states the reject and approve bounds.
- While the reviewer is not live in the parent worktree, the gate holds its delivery prompt without spending a prompt count, within `parent_review_wait_min`.
- Batten's Run-Shed row treats `paused`, `blocked` and `failed` children as budget-exempt waits, with one Warn per halt episode.
  This also covers the stale-`blocked` read right after `lyx loom start`.
- `CONSTRAINTS.md` states the parentage rule, package docs and recipe comments are updated, and CLAUDE.md's TEMPORARY items for #341 and #338 are gone.

## Design decisions to check

- `shuttleengine` gains a terminal failure on `GateResult`, and a `GateEntry` field that lets a must-pass entry return `Pending` and `Final`.
  Only the parent-review wiring sets it.
  `Final` fails on an unconsumed reject instead of passing with a Warn.
- `lyx loom review reject` and `approve` check no caller identity.
  A reject can now halt a run, but only after three rounds and only into `blocked`.
  The supersede path applies only to a run halted at the cap.
- The reject cap of 3 is a pick, not a settled number.
- A message to an absent orch is not queued, and nothing was added for it.
- Batten sends no message on a halt: the child's driver already messages `LYX_PARENT`.
  A `child_driver: go` child therefore reaches the parent only through `lyx batten status`.
- The batten wait has no time limit; `lyx batten pause <slug>` stops it.
- A nested pair resolves its parent to an orch name in a pair, where `lyx orch` never runs.
  Its reviewer never becomes live, so its parent-review always ends in the timed pass with a Warn.
- `failed` is waited on too, one state past the board's "paused and blocked", since `lyx loom start` resumes it the same way.

## Not run by hand

- End to end: `lyx batten run <slug>` from the prime with `LYX_STRAND_NAME` unset, then pause and resume the inner run and confirm the watcher survives and the pair is torn down after done.
- A real reject loop through the cap and `lyx loom review approve` against a live orch.
- The board's `pair-lifecycle` task still lists `batten-resume-sees-stale-blocked`; drop it after landing.
