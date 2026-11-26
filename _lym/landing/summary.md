# Name a kept cd and the way back when the cwd gate refuses a directory

An agent's shell keeps the directory a `cd` moves it to for every later call.
After survey commands ran `cd` into subdirectories, the next `lym creel post` was refused by the strict cwd gate.
The refusal named the cwd and the anchored directory but not the likely cause or the way back,
and it came on a later call than the `cd` that caused it.

Closes #245

## What changes

- The strict cwd gate in `internal/lymcwd` now adds to its refusal that a shell keeps a `cd` for every later command,
  so an earlier `cd` likely moved it.
  It also names the way forward: `cd` back to the anchored directory,
  and reach other directories through absolute paths or `git -C`.
- The refusal is now an `*AnchorGateError`.
  It still wraps `ErrCwdOutsideAnchor` and keeps its opening text.
- The new text reaches the verbs that surface the gate's error, creel among them.
  The exception is `drift capture-child`, whose cwd no shell chose:
  it reports the refusal without the cause and way forward.
- A new shipped heddle entry, `lym-shell-directory`, renders into the agents file after the edit-round entry.
  It says the shell keeps a `cd` across calls,
  and to reach another directory through an absolute path, `git -C <dir>` or `env -C <dir>`.

## Design decisions

- The refusal says more; the gate accepts the same directories as before.
  The issue offered accepting any directory inside the worktree as an alternative,
  which would have loosened the gate for every verb behind it, not for creel alone.
- The message changes at the gate rather than in `creelcli`, because any gated verb can trip over a kept `cd`.

## Tests

- `TestCheckCwdAnchorGate` pins the whole refusal text for a subdirectory of a root-anchored worktree
  and for the worktree root above a subpath anchor.
- `TestDrift_RunsInTheAnchoredDirectory` checks that the capture child's refusal carries no way forward.
- The shipped-entry pin in `TestStore` gains the new entry's row.

## Merge danger

**Door:** two-way
**Blast radius:** agents
