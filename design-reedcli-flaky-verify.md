# Flaky verify failure in internal/reedcli on the first integration-suite run

`reedcli-flaky-verify` · Note · bug, reed

Imported from [issue #158](https://github.com/Knatte18/loomyard/issues/158).

The webster verify triage reported one non-regression failure that passed or vanished on rerun: the package `github.com/Knatte18/loomyard/internal/reedcli`.

The package failed the first verify run and was then classed as flaky because the rerun did not reproduce it.
That is abnormal: a verify run should be deterministic, and a flaky package makes verify fail spuriously and costs a rerun.
The note records no failure output, so the cause is unknown.

What to do: run the `internal/reedcli` tests repeatedly, for example with `lym gate test ./internal/reedcli`, to reproduce the failure and find the nondeterminism, such as shared state, ordering, timing or a leaked environment.
Consider having the triage note keep the first run's failure output so a flaky identity can be diagnosed.

Provenance: task single-repo-fabric-mode, note webster-verify-triage.md.