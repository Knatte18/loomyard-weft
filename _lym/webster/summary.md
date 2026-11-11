# Adopt runs as a detached, step-recorded worker that pauses every adopted run

All four batches reached `done`. No batch needed recovery, no digest carried `departures` or `renames`, and no digest carried warnings.

## Batch 01: adopt-pause-caller

Cards 1–3. Added `pauserecord.AdoptCaller`, read as held wherever the legacy hold was read.
`orchcli.BeamUp` can now start the dobby without the orch.
Added `internal/adoptrecord`: the record, its path, its lock and its lock-free read.
No deviations. Head `345a8f3a`.

## Batch 04: green-end-reports

Cards 4–5. The green adopt check now only removes the fix task and resets the try count.
The held run states are replaced by `paused_by_adopt`.
Deviations: `internal/adoptcli/green_test.go`, `internal/adoptcli/runreport.go`. Head `78fd6ddf`.

## Batch 06: adopt-step-order

Card 6. Adopt runs as ordered steps (reconcile, pairs, pause, owner, deploy, beam, check, done), with the adopt pause and the owner write before the deploy.
The parent decision on a failed step is implemented: the part it gathered before failing is kept in the record and merged into the outcome.
Deviations: `internal/adoptcli/adopt.go`, `adopt_test.go`, `pause.go`, `steprun.go`. Head `5a65d87d`.

## Batch 07: adopt-worker

Cards 7–10. Added the hidden worker verb that runs the steps under `adopt.lock`, records them and resumes.
`lym adopt` spawns the worker and follows the record, with `--no-orch` and `--discard`.
`lym beam status` shows an unfinished adopt.
Made the `PATTERN-dobby-runs` change and updated the adopt package doc.
The fork reported one limit: the reconcile report's own `mutations` field comes back empty in the end envelope, while the top-level `mutations` lists every entry.
Deviations: `internal/adoptcli/adopt_test.go`, `caller.go`, `caller_test.go`, `cli.go`, `deps.go`, `follow.go`, `follow_test.go`, `steprun.go`, `worker.go`. Head `acd4eada`.

## Scope guard

Every deviation sits in `internal/adoptcli`, the package the cards edit. The scope guard listed them as informational only.


## Integration suite triage

The `final` gate failed, but webster's triage did not attribute the failure to this run.

Flaky (passed on rerun):

- `github.com/Knatte18/loomyard/internal/pattern`
