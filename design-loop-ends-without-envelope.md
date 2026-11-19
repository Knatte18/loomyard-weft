# Loop ends without envelope or recorded exit after repeated 'session state disagrees' warnings

`loop-ends-without-envelope` · Task · bug, shuttle, shed · running · Code-Bouncer

Imported from [issue #150](https://github.com/Knatte18/loomyard/issues/150).

## What happened

`lym shed step ... --until-stop` halted as `failed` with anomaly `interrupted-stop` while step `Drift-Await` was in flight.
The note read: `loop-exited: loop ... ended without an envelope`, with `loop exit: not recorded, since no process marked the loop's exit`.
The status file already read `Webster`, so the step's result and the loop's state disagreed.
The history ended at `Drift-Expect / skipped`, after `Plan-Bouncer / done` and `Drift-Await / skipped`.

Before the loop ended, the step stderr carried the same warning three times, at roughly 7 to 8 minute intervals:
`shuttle: session state disagrees ... loop=held state=busy cause=background stands=loop`.
The stopped shuttle run was the `webster` strand.

## Why it is abnormal

The warning repeated for over 15 minutes without lym resolving the disagreement between the held loop and the session state `busy` caused by `background`.
The loop then ended with no envelope and no recorded exit, so the halt note cannot tell a kill, a machine stop or a lym defect apart.
A session reported `busy` by a background cause while the loop holds it looks like a stuck or misclassified state, which may have led to the loop being stopped.

## What to do

- Find out why the shuttle state check reported `loop=held state=busy cause=background` repeatedly, and whether that state should count as a real disagreement.
- Make the loop record its exit, or at least a reason, on every path that ends it without an envelope, so a halt note can name the cause.
- Have the repeated warning escalate or self-resolve rather than recur silently until the loop dies.

Provenance: task `claude-accounts-per-worktree`, from note `loom-halt.md`.
