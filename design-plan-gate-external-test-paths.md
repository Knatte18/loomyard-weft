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
