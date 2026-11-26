# Gate fixer interrupted by a machine stop is reported as dead (timeout)

`gate-fixer-reboot-reads-timeout` · Note · bug, gate, shuttle

Imported from [issue #248](https://github.com/Knatte18/loomyard/issues/248).

## What happened

A row gate spawned a cold fixer, and the machine rebooted under a minute later.
The run was re-stepped more than two days after that.
The loop's reconcile had stopped the fixer's shuttle run, but the gate fixer's record still read live.
The restarted driver waited on that record, classified it `dead` with reason `timeout`, and told the writer:

> The cold fixer ended without a reply (timeout); any change it left uncommitted is in the worktree.

The interrupted-stop detail, from the first entry after the reboot:

```
loop exit: not recorded, since no process marked the loop's exit, and the machine has booted since the loop started, so a stop of the machine likely ended it; (...) stopped shuttle runs: ly:<slug>:gate-fixer; left runs whose output files all exist: ly:<slug>:darn
```

The fixer's shuttle `run.json` (excerpt):

```
"createdAt": "2026-11-23T19:19:20Z",
"outcome": "stopped",
"roleKey": "gate-fixer",
"interrupted": true
```

The gate fixer's `records.json` after the resume (excerpt):

```
"spawnedAt": "2026-11-23T19:19:27Z",
"status": "dead",
"deadReason": "timeout"
```

## Why it is abnormal

The fixer did not run out of time: the machine stop ended it, and lym had already recorded its run as interrupted.
`Await` in `internal/gatefixer/await.go` checks the elapsed time since spawn before it checks whether the strand is live,
so any fixer interrupted by a stop that outlasts its timeout is reported as `timeout` instead of `died`.
The writer and anyone reading the record are sent looking for a slow fixer or a timeout that is too short.

The dead record also stays in the visit's set, which `internal/gatefixer/rowgate.go` counts against the fixer budget.
The interrupted fixer's session is not resumed, so the failure it was given falls back to the writer.

## What to do

- Report a fixer whose run is marked interrupted, or whose strand is gone across a reboot, as interrupted or `died`, not `timeout`.
- Consider resuming its session, or spawning a replacement that does not use up the budget, as is done for interrupted run agents.

Provenance: task `sandbox-without-new-kernel`, note `loom-halt.md`.
