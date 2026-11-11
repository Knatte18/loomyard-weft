# Clear a green adopt check's mark; reed attach and up refuse _bolt

Two reed faults the operator hit.

## The adopt check's mark stays after a green check

The "adopt-check tab" at the bottom of the terminal is the check mark that `lym adopt check` sets on line 2 of the prime's reed status bar.
At a green end it read `adopt check of <subject> green: tier0 40s · tier1 104s · tier2 …` and stayed until the next check or the end of the session, taking much of the line.

- A green end now clears the mark: `checkMarkOf` renders the zero mark, which `SetCheckMark` turns into an unset of the session options.
  The result stays in `check.json`, in the green notice to the orch, and in `lym beam status`, which names `suite.log`.
- A failed end keeps a mark for the operator, shortened to one item: `adopt check of <subject> failed at tier<N>; lym beam status shows why`.
  `lym beam status` lists the tiers and the log, so the mark does not repeat them.
- `primeMark` leaves a cleared own mark out of its ` | ` join.
  When other subjects' checks are still running, their marks alone show, and with none running the bar clears.
- The adopt envelope's per-check line now says the bar shows the check "while it runs and when it fails" instead of "its progress and result".

The check runs as a detached process and opens no pane or window, so the mark was the only thing to change.

## reed attach and up booted a session in the Bolt checkout

`lym a` run in the hub's `_bolt` checkout booted a tmux session named `_bolt` on the hub's reed socket, and `lym reed list` did not show it.
`lymcwd.Resolve` accepts any git checkout under the hub, so the reed pre-run built an engine for it.

The reed pre-run now refuses `attach` and `up` when `fabricengine.RequireDrivableWorktree` refuses the location: the Bolt checkout and a pair's records sibling.
The refusal comes right after the location resolves, before any config read or tmux call.
It names the verb and fabric's reason, and gives as the way forward the hub's prime or a task worktree under the hub's path.
Every other reed verb still runs there, so `lym reed down` still removes a session an older binary booted in such a checkout.

## Design decisions

- The refusal names the hub's path rather than the prime's directory name.
  From the Bolt checkout, `fabricengine.PrimeName` lists the board repository's own worktrees, so no prime name of the warp repository can be read there without new fabric geometry.
- The refusal is limited to `attach` and `up`, the two verbs the issue names and the ones that boot a session.

## Tests

- `TestRunCheckMarks` pins the cleared mark at a green end and the one-item mark at a failed end.
- `TestPrimeMark` gains rows for a cleared own mark: with another check running it shows that one alone, and with none running it clears.
- `TestRunCLI_AttachAndUp_RefuseFabricCheckouts` runs `attach` and `up` in a Bolt checkout and a records sibling built as bare `.git` directories, with no config and no git process.
  It pins the exit code, the verb, fabric's reason and the way forward naming the hub.
