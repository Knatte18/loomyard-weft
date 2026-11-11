# creel post warns when the receiver is not live

`lym creel post` to a strand with no live pane queued the message and printed plain success.
A sender then told the operator the request was on its way, though nothing would read it until the strand ran again.

A post still stores the message.
It now also says whether the receiver was live:

- The answer envelope gains `receiver_live` (`true` or `false`).
- A receiver that is not live also prints one `creel: warning: ` line on stderr, before the answer line.
  The line names the receiver and the command that starts it:
  `lym beam start <slug>` for an orch, `lym beam start root` for the prime's orch, and `lym reed resume` in the worktree for any other strand.
- `--wait-reply` prints the warning as soon as the post is stored, not when the wait ends.

Closes #96

## Design decisions

- `creelengine.Store.PostWith` now returns a `PostResult` (id, interrupted, waiter, `ReceiverLive`, `Warning`) instead of a tuple.
- Liveness comes from the same strand listing the receiver check already reads.
  A listed strand answers its own `Live` flag.
  An orch name of this hub that no strand lists counts as not live.
- A post to a program or to a strand of another hub answers no `receiver_live` and no warning, since creel cannot tell whether either runs.
- The wire `Response` gains `ReceiverLive *bool` and `Warning`.
  A waiting post sends its warning in a progress response right after the pending one, through a new `CallWatching` on the client.
- The `post` help text and the package docs describe the new field and the warning line.
  The help text is also trimmed to stay under the size cap.

## To check by hand

- A `--wait-reply` resumed after a dropped connection answers no `receiver_live`; confirm that is acceptable.
- Post to a stopped orch on a live daemon and see the warning and `receiver_live: false`; the tests do not run against a real crashed session.
