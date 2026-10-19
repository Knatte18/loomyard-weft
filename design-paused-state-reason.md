# A paused run records who paused it, a goto included

`paused-state-reason` · Note · bug

Imported from [issue #759](https://github.com/Knatte18/loomyard/issues/759).

## What happened

The Go driver found a run in state `paused` with no recorded reason, current producer `Plan-Bouncer`, 32 history entries.
It wrote a `go-driver-paused` friction note pointing at a `drive-reports/drive-envelope-*.json` envelope and nothing else.
This came right after the same run halted `failed` at `Plan-Burler-Review` (a reed "did not track strand" liveness failure) and the history shows a `Plan-Bouncer` stuck entry as the last record.

## Why it is abnormal

A paused run must say who or what paused it.
With no reason, neither the operator nor the driver can tell an intended pause from a lost handoff, and the driver cannot decide whether to resume.

## What to do

- Make every transition into `paused` record a reason (operator request, hibernate, budget, crash-resume, ...), and have the driver refuse or label a paused state that lacks one.
- Check whether the failed-then-paused sequence at the same loom run (`Plan-Burler-Review` failure, then a `Plan-Bouncer` pause) is a path that writes `paused` without a reason.

Provenance: task `ubuntu-2404-support`, notes `go-driver-paused.md`, `loom-halt.md`.
