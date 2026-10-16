# Plan gate paths to external test packages, flaky lyxtemp sweep test, Merge-In halt with no status

`twb-btf-it` · Note · bug · paused · Plan-Burler-Fix

# Plan gate paths to external test packages, flaky lyxtemp sweep test, Merge-In halt with no status

`intake-bugs-1012` · Task · bug, loom, webster, shed

Three bugs taken in from GitHub on 2026-10-12, bundled as one small task.

## Was `plan-gate-external-test-paths`

Imported from [issue #571](https://github.com/Knatte18/loomyard/issues/571).

A plan's card gate step and batch gate both listed a path of the form `./internal/<pkg>_test`, treating an external test package as if it were a directory.
No such directory exists: external test packages live in `internal/<pkg>`.
`lyx gate test` reports `[setup failed]` for that path, so the step cannot go green as written.
Every real package in the same steps passed.
Two separate batches (a fork on the first, the implementing agent on the second) hit the same failure.
A fork does not edit the plan, so it was blocked, and the other agent worked around it by running the steps without the bogus path.

Why abnormal: the plan passed through authoring and validation with a gate path that can never resolve, and the failure only surfaced at execution time in several batches.

What to do: have the plan validation (or the Plan-Write step) check that every package path in a gate step resolves to an existing directory, and reject a `_test`-suffixed path whose directory is absent.
Optionally, make `lyx gate test` point at the missing directory in its `[setup failed]` message.

Provenance: task `twill-plan`; notes 01-fabricvocab-package.md, 06-plan-producer-key.md.


### From issue #577

Imported from [issue #577](https://github.com/Knatte18/loomyard/issues/577).

A task card's gate line listed a path ending in `_test` that names the external test package of a Go package (for example `./internal/<pkg>_test`) and is not a directory.
`lyx gate test` reports `[setup failed]` for that argument.
The implementing agent worked around it by running the gate line without that path; every other package passed.

This is abnormal because the bad argument is only discovered at implementation time, after the plan was approved by the Plan bouncer and review rounds.
A card's gate line should be checked when the plan is written or reviewed, or `lyx gate test` should map an external test package path to its package directory with a clear message.

What to do: validate gate-line package paths against existing directories in plan validation, and/or make `lyx gate test` name the missing directory in its setup-failure output.

Provenance: task `merge-review`; notes 18-merge-in-row.md.


### From issue #580

Imported from [issue #580](https://github.com/Knatte18/loomyard/issues/580).

Several card gate steps and the batch gate listed an external test package (an `_xtest`-style `<pkg>_test` name) as a directory argument to `lyx gate test`.
Such a name is not a directory, so each of those steps exited 1 with `directory not found` and `[setup failed]`, while the real packages passed.

This is abnormal because the gate step is defined by the plan, and the plan was accepted with a step that can never succeed.
The agent had to recognise the setup failure as a gate-definition flaw rather than a card failure.

What to do: have plan validation (`validate-plan`) reject a `lyx gate test` step whose package argument does not resolve to an existing directory, or have `lyx gate test` map an external test package name to its directory; the Plan-Review rubric could also name this check.

Provenance: task `plan-gate-fixes`; notes: `01-shared-reference-delete-order.md`.

## Was `lyxtemp-sweep-flaky`

Imported from [issue #584](https://github.com/Knatte18/loomyard/issues/584).

The integration-suite triage in a verify run reported `internal/lyxtemp.TestSweep/two_concurrent_sweeps_both_succeed` as flaky: it failed the first verify run and passed or vanished on rerun.

Why abnormal: a test of concurrent sweeps that fails nondeterministically points at a race or an unbounded retry in the sweep itself, and flaky tests cost a verify rerun and can block a Publish step (the same run halted blocked at Publish with `publish_verify failed after merging parent branch`, exit code 1, escalation-to-human; the notes do not show whether this test caused that failure).

What to do: reproduce by looping `lyx gate test -run 'TestSweep' ./internal/lyxtemp` under load, find the race in the concurrent-sweep path, and make the test deterministic or the sweep race-safe.

Provenance: task `runs-overview`; notes `webster-verify-triage.md`, `loom-halt.md`.

`board-field-drop` (`78e31aa06`) later bounded the sweep's not-empty retry under a per-sweep budget (its D7); check first whether the flake still reproduces on that code.

## Was `merge-in-status-missing`

Imported from [issue #581](https://github.com/Knatte18/loomyard/issues/581).

`lyx loom` halted a run as `failed` at the Merge-In producer.
The halt note shows reason `status-missing`, anomaly `producer-hard-failure`, `history_entries: 0` and an empty `trace_file`.

This is abnormal because the producer left no status and no history, and no trace was recorded, so the cause of the failure cannot be recovered from the halt note or the run's files.
It looks like a lyx problem (the Merge-In row failing before writing any status or trace), not a task-code failure.

What to do: find out why Merge-In produced no status; make the driver record a trace file and a more specific reason when a producer exits without a status, so the halt note is diagnosable.

Provenance: task `plan-gate-fixes`; notes: `loom-halt.md`.


## From issue #587

Imported from [issue #587](https://github.com/Knatte18/loomyard/issues/587).

A batch's gate lines named `./internal/fabricengine_test`, which is not a directory: the external test package lives inside `internal/fabricengine`.
`lyx gate test` reports `[setup failed]` for that path on every run, although every real package passes.

Abnormal: a plan gate line can name a path that never resolves to a package, and nothing catches it at plan time, so every gate run carries a spurious failure.

To do: validate gate-line package paths when the plan is written or checked (a `_test` suffix names an external test package, not a directory), and make the `[setup failed]` output say the path does not exist.

Provenance: task `alignment-agent`, note `01-loom-alignment-config.md`.

## Was `plan-cli-tree-check`

Imported from [issue #588](https://github.com/Knatte18/loomyard/issues/588).

A card asked for a new `lyx loom peers` verb as a runnable group carrying `AudienceRole`.
`cmd/lyx/clitree_test.go` requires a bare group to print help (exit 0) and a mistyped subcommand to refuse with "unknown subcommand", and `clihelp.Invocable` treats a runnable group with no `Use` token or flag as non-invocable, so such a group must carry no audience.
The implementer had to deviate from the card: the group carries no audience annotation and only the `peers show` subcommand carries `AudienceRole`.

Abnormal: the plan specified a shape the CLI tree invariants forbid, and the conflict surfaced only during implementation.

To do: have plan authoring or plan review check CLI-surface cards against the `clitree_test.go` and `clihelp.Invocable` rules, and document in the CLI-authoring docs that audience annotations belong on leaf verbs.

Provenance: task `alignment-agent`, note `01-loom-alignment-config.md`.
