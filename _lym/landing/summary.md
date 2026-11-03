# shuttle: don't count a gated run done over an unread background shell

A gated run whose turn ended with a background shell outstanding was counted Done at once when its output files existed.
The shell could hold a `git commit` or a `lym gate test --wait` whose result the session never read, so `done` guaranteed neither a commit nor a gate verdict.

Now `internal/shuttleengine` withholds Done in that case and tells the agent to read the shell first.
A gated waiting turn end counts only when every outstanding task is a shell the session does not await, meaning no `Spec.AwaitedShellPrefixes` entry names it:

- With an output file missing, the turn end is held at once, as before.
- With every output file present, the run keeps waiting and types one line into the pane naming each such shell.
  The line tells the agent to wait for each shell, read its result and act on it before ending its turn.
  The shell's completion starts the next turn, and a turn start cancels the recorded waiting turn end so no expiry counts a turn already under way.
- Once every such shell has been outstanding for `background_shell_wait_min`, the turn end counts Done through the gate as at any turn end.
  This bounds a shell that never finishes.
- A fork or an awaited shell still keeps a gated turn end waiting without expiry, as before.

Closes #7

## Design decisions

- I chose the issue's first option, refusing Done and telling the agent, over having lym stop the shell.
  The provider reports no pid, so lym still kills no shell.
  Strand removal ends only the shells inside the pane's process tree, and `Result.EndedShells` still records whatever is outstanding at run end.
- Each shell is named once per `Wait`.
  A shell counts as named only after the send succeeds, so a line dropped under a pause or lost to a failed send is queued again by the next waiting turn end over that shell.
  A failed send is logged at Info and not retried in place.
- A turn end that expires into Done is not nudged, because the line would claim the run is not done.
- `background_shell_wait_min` now also bounds the gated Done, in addition to the ungated hold and the wait-marker threshold.
  `Spec.AwaitedShellPrefixes` keeps a shell out of the nudge and the expiry too.
- The code is in `wait.go` (`expiredTurnEnd`, `unawaitedShellsPastBound`, `queueShellNudge`, `sendShellNudge`), with new fields on `Run`.
  Docs are updated in `shuttleengine`, `websterengine` and `docs/overview.md`, and `waiting_test.go` covers the nudge, the expiry and the turn-start clearing.

## To check by hand

- The issue's third item, that a gate run abandoned this way releases its slot, is not changed here.
  The change makes abandonment rarer for gated runs by holding Done, but a run that ends at its deadline or by a stop still leaves the shell unread, and I did not check slot release for that path.
- The nudge was exercised only through the engine's tests.
  It was not run against a live Master session with a real `lym gate test --wait` shell.
