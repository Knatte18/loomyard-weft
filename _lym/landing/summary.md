# Creel holds the bell while the turn an answered ring woke has not ended

When creel rings an idle hook strand's waiter and the ring is acknowledged, more mail can arrive before the woken turn ends.
Creel used to type the fixed bell line, `[creel] Mail is waiting for you. …`, for that mail once 5 seconds passed with no waiter live.
On Codex the woken turn is queued through `codex queue` and can take several seconds to start.
The bell then started a turn of its own ahead of the queued one: a wasted model turn, and the first mail reached the model about twice as late (17 s against 8 s in the creel trial's burst case).
The woken turn's Stop already hands out any mail that arrived meanwhile, so the bell was never needed there.

## What changes

`wakeHookStrand` in `internal/creelengine/deliver.go` now types no bell within the 30-second ring window while either of these holds:

- the last ring stands unanswered, as before;
- no turn end after the ring has been read, which is new: the turn the ring woke is still queued or running, and its Stop delivers the mail.

Past the window an unanswered ring is followed by the bell at once, as before.
In every other case, the 5-second grace with no waiter live still applies.
A strand whose woken turn ended after the ring, as a recorded turn end shows, gets the bell after the grace as before.

## Design decisions

- The hold is bounded by the existing 30-second ring window rather than lasting until a turn end.
  A woken turn records no prompt submit, a user interrupt fires no Stop, and a reading the pane decided carries no turn end.
  Holding until a turn end could keep the bell back forever after such an interrupt, the case the bell exists for.
  `doc.go` records this as a bound: in those cases the bell for mail that arrived after the ring waits until the window ends.
- The rule is engine-neutral.
  It rests on the ring mark and the hook-derived turn-end reading creel already has, so a Claude strand gets the same hold.

## Tests

`TestScheduler_HookWake` in `internal/creelengine/deliver_test.go` gains two rows:

- after an acknowledged ring, with no turn end read after it, a new post gets no bell before the 30-second window;
- once a turn end after the ring is read, a new post gets the bell after the 5-second grace and before the window.

The existing rows for unanswered, released and cancelled rings and for the grace keep their expectations.
