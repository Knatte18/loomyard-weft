# creel post says when its receiver is not live

`lym creel post` to a strand that was not running printed only `{"id":…,"ok":true,"sender":…}`.
A sender had no way to tell that its message would sit unread until someone started the receiver (issue #96).

## What changes

A post is still stored for a receiver that is not live, and it still waits in that receiver's mail.
The answer now says whether the receiver is live:

- `creelengine.Store.PostWith` returns a `PostResult` with the id, whether a turn was interrupted, the reply waiter, `ReceiverLive` and `Warning`.
  The result reads liveness from the same strand listing the receiver check already reads, so the post makes no second listing call.
  A listed strand answers its own `Live` flag.
  An orch name of this hub that no strand lists, which creel already accepts as a receiver, answers not live.
  A program or a strand of another hub answers no liveness, since creel cannot tell whether either runs.
- A receiver that is not live gets a warning naming the command that starts it.
  For an orch that is `lym beam start <slug>`, with `root` for the prime's orch.
  For any other strand it is `lym reed resume` in the strand's worktree.
- The daemon's `Response` gains `receiver_live` and `warning`.
  A waiting post (`--wait-reply`) sends the warning in a progress response right after its pending one, so the sender hears it before the wait starts, not when the wait ends.
  The pending and final responses of a waiting post carry `receiver_live` too.
- `lym creel post` adds `receiver_live` to its envelope and prints the warning as one `creel: warning: …` line on stderr.

## Design decisions

- The JSON key is `receiver_live`, not the issue's example `recipient_live`, because creel's protocol, docs and errors call the addressee the receiver everywhere.
- The warning goes to stderr, where creel's other notes go, such as an inject's stall notes, and the envelope keeps only the boolean.
- `Post`, `PostWait` and `PostInterrupt` keep their signatures.
  `PostWith`, whose only callers are the daemon and creelengine's tests, returns the struct instead of a fifth positional value.
- The prime's beam slug `root` is a constant in creelengine, since importing the beam CLI or the board engine just for that name would pull in packages creel otherwise does not need.

## Tests

- `TestStore_Mail` row `PostWith_AnswersWhetherTheReceiverIsLiveAndHowToStartOneThatIsNot` covers each receiver kind: a live strand, a dormant pair strand, a dormant prime orch, a dormant beam orch, an unlisted beam orch, a program and another hub's orch.
  The row asserts the exact liveness and warning for each.
- `TestDaemon_Protocol` row `PostAnswersWhetherItsReceiverIsLiveAndAWaitingPostHearsTheWarningAtOnce` checks the protocol fields and that a waiting post gets the warning as progress before its timeout.
- `TestCreelCLI` checks the envelope's `receiver_live` and the exact stderr line for a live and a dormant receiver.
