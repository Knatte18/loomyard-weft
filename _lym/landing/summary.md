# A gated run is not done while a background shell it never read runs

A gated shuttle run, such as webster's Master, could end `done` while a background shell it started was still running.
The turn end listed the shell as outstanding, but because the label matched no awaited prefix and both output files existed, shuttle judged it as a plain turn end and finalized at once.
In issue #7 the shell chained `git add`, `git commit`, `git log` and `lym gate test --wait`, so the run reported `done` with a commit that may never have landed and a gate verdict nobody read.

## What changes

At a gated turn end whose only outstanding work is shells the session does not await, with every output file present, shuttle no longer counts the run Done while such a shell runs:

- It types one line into the agent's pane naming each such shell, telling the agent to wait for each one, read its result and act on it before it ends its turn.
  Each shell is named once per wait, so an agent that ends its turn again over the same shell is not told a second time.
  If the send fails it is logged at Info and not retried, because the shell's completion starts the session's next turn anyway.
- The turn end is no arrival for the gate.
  The run goes on to its next turn end, which the shell's completion or the line starts, and that turn end is judged as usual.
- If every such shell has been outstanding for `background_shell_wait_min`, the waiting turn end counts as Done through the gate, as before.
  The shells then land in `Result.EndedShells`, and webster's warning and friction note record them as they do today.
  This bound keeps a shell that never ends, such as a watch loop, from holding the run until its deadline.

A gated run with an output file missing still holds the turn end at once, and forks and awaited shells keep a gated turn end waiting as they did before.

The change is in `internal/shuttleengine/wait.go`.
In `pollEventsTick`, the gated branch now queues the line and defers to `expiredTurnEnd`.
`expiredTurnEnd` gained the gated expiry to Done, and both its expiries share the new `unawaitedShellsPastBound`.
`queueShellNudge` and `sendShellNudge` build the line and send it from the wait loop through the gated send.
The shuttle and webster package docs, the `BackgroundShellWaitMin` field comment and the `background_shell_wait_min` template comment now describe the gated expiry.

## Design decisions

- **Refuse Done instead of stopping the shell.**
  The issue offered a second option: shuttle stops outstanding shells at run end and records their results.
  Shuttle cannot do that reliably, because the provider reports no pid for a background shell.
  Making the run wait, and telling the agent to read the result, means the commit and the gate verdict reach the agent before the run ends.
- **Gated runs only.**
  An ungated run with every output file present still finishes at a waiting turn end, whatever is outstanding.
  Ungated runs have no gate verdict to lose, and some, like a driver's binary watch loop, hold shells that never end.
  For those, a line telling the agent to wait for the shell would be wrong.
- **A bounded wait, not an indefinite one.**
  `background_shell_wait_min` is already the bound at which an unawaited shell counts as long-running.
  Reusing it caps the extra wait without adding a new setting.
- **Gate slots need no change.**
  A `lym gate test --wait` cut off with its shell holds its slots through OS file locks, which the process's death releases.
  `TestAcquire_HolderProcessDeathFreesSlot` in `internal/gateslot` already pins this.

## Tests

In `internal/shuttleengine/waiting_test.go`:

- The gated rows of `TestPollEventsTick_OutstandingBackgroundWork` now expect no Done inside the bound and Done at the bound.
  A new staggered row pins that two shells finish only once the later one passes the bound.
- The gated row of `TestWait_RecordsEndedShells` reaches Done only past the bound and records the shell's full time outstanding.
- The new `TestWait_GatedShellNudgeNamesEachShellOnce` pins three things:
  - the first waiting turn end sends exactly one line, naming the shell;
  - a second waiting turn end over the same shell sends nothing;
  - the next plain turn end finishes Done through the gate with no shell left over.
