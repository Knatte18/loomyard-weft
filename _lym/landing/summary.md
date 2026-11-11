# Clear the green adopt-check mark; reed attach/up refuse non-worktrees

Two reed faults, fixed together.

## Adopt check mark

After `lym adopt` ran its check, the prime's reed status bar kept the check's mark, and it took a lot of space.
A green check left a long line of every tier with its time, and the line stayed until the next check or the end of the session.

- A green check now renders the zero mark, which clears the status bar.
  The check record, the orch's notice and `lym beam status` already hold the result.
- A failed check stays shown as one short item: `adopt check of <subject> failed at tier<N>; lym beam status shows why`.
  When the record has no failed tier, the item reads `failed; lym beam status shows why`.
- `primeMark` leaves a cleared own mark out of the ` | ` join, so another subject's running check still shows alone.
- The envelope's `suiteMessage` and the `adoptcli` docs now say the bar shows a check while it runs and when it fails.

## Reed attach and up outside a worktree

`lym reed attach` and `lym reed up` run in the hub's Bolt checkout booted a tmux session on the hub's reed socket.
No reed listing of the hub's worktrees showed it.

- The `reedcli` pre-run now refuses `attach` and `up` in fabric's own checkouts, the Bolt checkout and a pair's records sibling.
  It calls `fabricengine.RequireDrivableWorktree`, and does so before any config read or tmux call.
- The refusal exits 1 and names the verb, fabric's reason and the way forward: run it from the hub's prime or a task worktree.
- Every other verb still runs there, so `down` removes a session an older binary booted in such a checkout.
  A test pins that `down` is not refused in Bolt.
- The `attach` and `up` help texts and the `reedcli` package doc state the refusal.

Closes #112
