# Commit run transitions locally and push them from a background pusher

No batten or loom run transition waits for the network any more.
A transition commits its records, starts a background pusher for the worktree and returns,
so the next row starts at once, also while origin is slow or unreachable.
The driver-stop record commits (batten's run records and `lym loom commit-records`) do the same.

## What changes

- **Transitions.** A transition does no push, fetch, origin reconcile or network-gate admission.
  This replaces batten's push-free start rows as a special case: every row behaves that way.
- **The pusher.** The new `internal/runpusher` package, run by the hidden `lym loom pusher` verb,
  holds one lock per records worktree and pushes the records side, then the code side.
  Each commit starts it, as do the dobby's tick and a resume.
- **Failures.** An owed push (transport, deferred, teardown lock) no longer holds the run.
  The pusher retries it from one second, doubling to a minute, writes no failed-push record,
  and posts one notice when the wait reaches a minute.
  A refused push still leaves its record and notice, and stops the pusher until the record is cleared.
- **Loom's pre-push check.** The origin reconcile runs in the pair's pusher before each push, not before the commit.
  A pause covering the run now stops the push, where before it skipped the check and let the push through.
  The check still guards the pull-request record's commit and push and PR-Review's task push.
- **Takeovers.** A pair's pusher hands its run over on a rejected push.
  The prime's pusher hands over each taken-over batten run its unpushed commits name:
  it archives the prime's records tip under an owner tag,
  leaves that run's commits out of the replay and pushes the other runs' commits in order.
- **Worktree-Teardown.** It releases the prime lock after the session shutdown, pushes the pair to completion,
  and retakes the lock to remove the pair only while origin's tracking refs hold both tips.
  A refused push, or tips origin still lacks after three pushes, halts the row and keeps the pair.
  With git or pushes off it removes the pair without pushing.
- **`lym loom commit-records`.** The envelope is `committed` and `pusher_started`.
  It no longer reports `push_skipped`, `push_stopped` or a push error.

## Design decisions

- A takeover re-runs from what origin holds, so redone work is accepted.
  Until the original machine's pusher meets the takeover, both machines may run rows whose acts stay local.
- The pusher is keyed by records worktree, not by run: every batten run shares the prime's.
  A refused push of the prime's records branch therefore holds the pushes of every batten run on this machine.
- The pusher is registered under the hub's owner with the push role,
  so a run's teardown, cancel or hand-over does not end it.
- A read of a run on its own machine no longer waits on a code tip the local pair holds, since its pusher owes it.
  Another machine still sees the tip as missing.
- A push that finds a branch's entry ended on origin deletes the local tracking ref and keeps the upstream setting,
  so local ahead and level reads do not take a stale ref for owed work.
- Pushes that are a row's own act (PR-Review's task push, Finalize, the pull-request record) stay synchronous.
  `webster reset` keeps its detached push.

## To check by hand

- **Not run: a real spawned pusher.** The spawn refuses to start under `go test`.
  In a live hub, confirm that transitions of a batten run and a loom run start `lym loom pusher`,
  that origin receives both sides, and that the pusher exits once level.
- **Not run: timing.** Compare the time between skipped loom rows and between batten's later transitions
  against the figures of the `later-transition-push-wait` entry.
- **Tests of the pusher's lock and start conditions** (`internal/runpusher`) were found missing, severity medium.
  Check that `seams_test.go` and `newdeps_integration_test.go` cover the real lock and condition reads.
- **Tests of the pusher restart calls** (dobby tick, batten and loom resume, in `internal/beamcli`,
  `internal/battencli` and `internal/loomcli`) were found missing, severity medium.
  Check that each wired call is now driven.
- **Tests of the teardown's push and level wiring** (`teardownPushSeams` in `internal/battencli/wire.go`)
  were found missing, severity medium.
  Check that `teardownpush_integration_test.go` covers it.
- **Worktree-Teardown looping with pushes off** (`internal/battenshed/teardown.go` and its wiring) was found,
  severity medium.
  Check that the pushes-off path and the three-push bound close it.

## Merge danger

**Door:** two-way
**Blast radius:** runs

A revert restores the synchronous push.
It does not recover commits that a faulty pusher or teardown had already dropped before removing a pair.
