# Plan accepted a gate step naming a nonexistent external-test package path

`plan-gate-external-test-paths` · Note · bug

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


## From issue #577

Imported from [issue #577](https://github.com/Knatte18/loomyard/issues/577).

A task card's gate line listed a path ending in `_test` that names the external test package of a Go package (for example `./internal/<pkg>_test`) and is not a directory.
`lyx gate test` reports `[setup failed]` for that argument.
The implementing agent worked around it by running the gate line without that path; every other package passed.

This is abnormal because the bad argument is only discovered at implementation time, after the plan was approved by the Plan bouncer and review rounds.
A card's gate line should be checked when the plan is written or reviewed, or `lyx gate test` should map an external test package path to its package directory with a clear message.

What to do: validate gate-line package paths against existing directories in plan validation, and/or make `lyx gate test` name the missing directory in its setup-failure output.

Provenance: task `merge-review`; notes 18-merge-in-row.md.
