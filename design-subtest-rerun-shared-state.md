# Round gate re-runs a failing subtest alone and breaks subtests that share state

`subtest-rerun-shared-state` · Note · bug

Imported from [issue #497](https://github.com/Knatte18/loomyard/issues/497).

Seen (publish-recovery, 2026-10-10): the Webster-Burler round gate failed three times on `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` with "probe run lock: ... .lyx/shed/loom-smoke-task/run.lock: no such file or directory".
`publishFailureCommand` (`internal/loomshed/publishfailure.go:88`, from 7648e8b27) re-runs each failing test from `publish-failure.yaml` alone, with a `-run` filter anchored on the subtest (`^TestSmokeBootstrapLifecycle$/^died_driver_...$`).
That subtest depends on the scratch dir and run lock that the earlier subtests' `loom start` bootstraps create, so run alone it always fails, while the whole test and the full tmux tier pass.
The fixer looped on a failure its own fix could not touch, and the run needed an investigation and a parent decision.

Workaround: a parent decision that the failure is a gate artifact, then resume.

Expected: the re-run names the failing test's top-level test (the first `-run` segment), never a subtest alone, since subtests may share state.
