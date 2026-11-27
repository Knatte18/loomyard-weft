# Run the PR-Review session tests over a recorded tmux below real reed

The PR-Review session tests filled the session liveness and registration seams with closures over a hand-kept strand table, and drove a fake reed.
Both stand in for repository modules, which `PATTERN-test-stand-ins` forbids.
The rule admits a stand-in only for something outside the repository, below the one module that reaches it, answering recordings of the real thing.
This change builds that stand-in for tmux and moves the tests onto it, so the real session runner and the real reed engine run under them at tier0.

## What changes

- **`reedengine.NewOver` and `Outside`.**
  An engine can be built over an `Outside` whose set fields replace the tmux binary, the process table's pane age, the user manager's memory cap and the strand guid source.
  `New` builds over the real ones, as before.
  `PaneProcessAge` is exported so a recorder can pass reads through to the real process table.
  While `Outside.Tmux` is set, reed's server start goes through it too, as one `new-session` call without the server flags, instead of a detached exec.
- **`reedengine.SessionLiveness` and `SessionRegistrar`.**
  The two reads a row keeping one session makes of the strand table move from `loomcli` into `reedengine`, unchanged apart from their error prefix.
  The wiring calls them from there, and their table test moves with them.
- **`internal/testkit/tmuxrec`.**
  A `Recorder` passes reed's calls to real tmux, the process table and the user manager, and keeps every answer.
  A `Replay` answers reed from one recording, and `PlaySession` builds a real runner over a real reed over a replay.
  A recording keeps paths, the user name, the host name and process ids as placeholders.
  A replay binds pids to values above any kernel's pid range, so a replay never reads or signals a process of the machine it runs on.
  A call the recording holds no answer to fails the test, naming the call.
  The kit's tier0 test replays each scenario over its recording and compares what reed read.
  Its tier2 test drives each scenario over real tmux, compares what reed read with the stored recording, and writes the recording again with `-tmuxrec.write <dir>`.
  The scenarios are a start, stop, resumed start and stop; a start whose pane's process then ends; and a start whose provider never comes up.
- **The PR-Review session tests.**
  The session world in `landingshed` replays those recordings.
  Its liveness and registration seams are reed's own reads, and registrations go to the real `runproc`.
  A wrapper counts the row's starts and stops and passes each to the real runner.
  The failures the rows scripted by hand are now provoked through state the real modules read:
  - an unwritable run directory fails a start;
  - a file where the registration directory goes fails a registration;
  - an unreadable reed state directory fails a status read;
  - a recorded pane whose process ended reads dead;
  - a recorded start whose provider never came up is the transient start failure.

## Design decisions

- The entry asked for a recorded reed and creel session stand-in.
  Under `PATTERN-test-stand-ins`, reed and creel are repository modules and take no stand-in, so the stand-in sits below reed, for tmux and the process table.
- The `Outside` seam covers more than tmux and the pane age.
  Pane creation also moves the pane's shell into a memory-capped scope of the systemd user manager.
  That would call the user bus from a tier0 test, so the memory cap is a field too, recorded like the rest.
  The guid source is a field so a replay mints the guids its recording names.
  A replay needs no stand-in for the kill and descendant reads of a removal: they read the kernel's process table by pid, and a replayed pid names no process.
- A replay answers a call that changes nothing on the server from the latest recorded answer to it, and a call that changes something only from the next one.
  So a row may read the strand table more or fewer times than the scenario did, while every change it makes must come in the recorded order.
- The `MailAgent` closures, one of which returns a made-up "no such strand" error, stay as they are, and so do envkit's session closures.
  A real creel post goes through a daemon process that no tier0 test can run, and envkit is being changed by another branch in flight.
  Both are filed for a follow-up.
- The kit's real tmux exec lives in its tier2 test file, since a kit's non-test files may not import `os/exec`.

## Merge danger

**Door:** two-way
**Blast radius:** `reedengine`
