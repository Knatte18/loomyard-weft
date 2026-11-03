# The hibernate request is carried by dobby, not a daemon of its own

`hibernate-in-dobby` · Task · enhancement, dobby · running · PR-Review

## Seen (operator, 2026-10-31)

`lym hibernate request` runs a daemon of its own that waits for the runs to land and every session to pause before it hibernates the machine.
Dobby already drives every run of its worktree to its end, and `battenrecipe.PastLand` is the one past-Land test that dobby and the hibernate daemon share (built by `landing-watch`).

## Goal

The hibernate request moves into dobby, so no daemon of its own waits on the runs.

## Decisions (operator, 2026-11-03)

- The prime's dobby carries the request: it reads the runs of every beam, so a request with `--after-all` waits on all of them, and the request is watched and cancelled in one place.
  No beam's dobby carries or votes on it.
- The request is kept on disk: when the prime's dobby is not running or dies while it waits, it takes the request up again when it starts.
- The wait for every session to pause moves over unchanged, the fork rule included; a change to how forks are handled is a task of its own, later.
- A hub pause leaves the dobby that waits on the request outside it, so the pause the request adds never stops the wait.
