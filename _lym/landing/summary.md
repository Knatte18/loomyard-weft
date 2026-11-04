# Drift capture fails by name; reconcile migrates a moved drift.yaml dir

Two Drift faults around the baseline capture.

## Capture without a baseline

A baseline capture that exited 0 but wrote no file halted Drift-Await with only the log path as its cause, and a resume reproduced the same halt.
The reason of a failed capture now names the directory the command ran in and the last lines of its output, or that it printed nothing.
This holds for a timeout, a non-zero exit and an exit 0 with no file, in the main capture and in the accepted side capture.
`lym drift status` and the Drift-Await halt carry that reason.

The detached `capture-child` writes the failure to its log as an error envelope and exits non-zero.
`lym drift capture` fails with the reason and its ways forward when the record has already ended failed as the verb returns.
A capture that fails later records the same reason.

Closes #41

## Migrating a reinterpreted `drift.yaml`

The move of Drift's relative `dir` base from the anchored directory to the worktree root (`c896481c`) carried no migration, so a hub's `capture.dir: .` silently pointed at the wrong directory.
`configreg` modules now declare reinterpretations.
`hubreconcile` applies each one once to the hub's base file, after the hub-wide reconcile and in the same board commit, and records a `reinterpreted-<id>` marker on the board so it never runs twice.
`driftengine.RebaseDirsOntoRoot` is the first, `drift-dir-root-base`, for the capture and compare dirs:

- A dir naming a directory under the anchored directory is rewritten to the same directory relative to the worktree root.
- A dir naming a directory under the root alone is kept.
- A dir naming neither is flagged with the `lym config drift --set` way forward.
- An absolute or unset dir is kept, and a hub anchored at its root changes nothing.

Once the commit lands, the rewrites and flags are mailed to the hub's prime orch as one creel notice signed `RECONCILE`.
`orchcli` installs the notifier; it posts only for its own hub's Bolt checkout.
With no notifier or no recorded orch, the notice is logged only.

## Design decisions

- `lym drift capture` stays detached and does not wait for the record to leave running.
  The cause reaches the operator through the child's exit, the failed record, `lym drift status` and the halt reason.
- A dir under the anchored directory wins over one of the same name under the root, since the file predates the move.
- A failed rewrite restores the board's config files and removes markers the pass wrote, as the existing hub-wide reconcile does.

## To check by hand

- No regression check ran, and nothing was run against a live `cs` hub.
  Run a reconcile on a hub whose `drift.yaml` holds `capture.dir: .` and confirm the rewrite and the orch's notice.
- The root cause of the original run's missing file was not reproduced.
  The wrong-base `dir` is the likely cause; confirm it against the `cs` hub's capture log.
- The failure subtest for the capture child, in `internal/driftengine/capture_test.go` and `internal/driftcli/capture_integration_test.go`, depends on timing, and the non-zero exit of the child is not asserted.
- `lym drift capture` exits 0 before it checks the baseline, so a capture that fails after the verb's check is seen only in `lym drift status` and the halt reason.
- `commandReport` in `internal/driftengine/steprun.go` reads the log tail only, so check that the last line of a long log is quoted whole.
- The reconcile notifier in `internal/orchcli/reconcilenotifier.go` has its post, other-hub refusal and root pre-run install tested; the live creel delivery is unchecked.
