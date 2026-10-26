# Dobby dies at a deploy instead of re-executing onto the new binary

`dobby-reexec-on-deploy` · Note · bug, workspaces, batten

Reported as [issue #880](https://github.com/Knatte18/loomyard/issues/880).

## Seen (OSL-1046, 2026-10-26)

The landing of `beam-start-and-ownership` (e6fbc14c) ran the hub's deploy, which replaced the lyx binary at 15:17:37Z.
The prime's dobby log ends at 15:17:25Z, its process (pid 96674) is gone, and `lyx beam status` reports the prime's dobby not running.
Its state record still holds the board-tidy agent it spawned at 15:17:39Z, with a deadline, so the dobby died without ending cleanly, and the tidy agent's reed window stayed open until the orch removed the strand by hand.
The cause is not found: the re-exec onto the new binary failed, or the landing or teardown ended the dobby's process.

## Expected

A dobby re-execs onto a changed lyx binary between ticks, as `internal/dobbyengine` documents, and goes on running through every deploy, so a landing never leaves the hub without its dobby.
A failed re-exec is logged with its error in `dobby.log` and posted to the orch, never a silent exit.
A test replaces the binary under a running dobby and asserts the dobby re-execs and keeps its tidy in flight.
