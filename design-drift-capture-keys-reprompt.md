# Drift gate re-prompts on per-commit capture metadata keys

`drift-capture-keys-reprompt` · Note · bug, drift, loom

Imported from [issue #82](https://github.com/Knatte18/loomyard/issues/82).

> Built by task `drift-provenance-stops-runs`: capture provenance now goes to a `{meta_dir}` sibling that drift never compares, the expectation takes a `metadata` block of file globs and keys, and a failure whose cells no code change can move ends the gate at once as a Terminal failure, with no re-prompt.
> Check whether this note still names a gap before building anything from it.

## What happened

The drift gate reported the `_source.txt` keys `commit` and `captured_at` as unexpected cells on two consecutive rounds after the squash commit.
Both values change with every commit and every capture, so no code fix can return them to the baseline, and the sealed expectation written by `Drift-Expect` cannot list a value that is unknown before the commit exists.
Each re-prompt was a fix round with nothing to fix.
The producer `Darn-Write` then ran out its verify budget: the run halted `blocked` with anomaly `escalation-to-human` after "gate did not pass after 3 attempts".

## Why it is abnormal

The gate fails on values that are non-deterministic by construction, so the loop cannot converge and consumes the verify budget regardless of the agent's work.

## What to do (original proposal)

Either make the drift check skip capture-metadata keys such as `commit` and `captured_at`, or let the sealed expectation declare keys whose values are exempt from comparison.

Provenance: task `support-for-dop-rheology-in`; notes `Darn-Write.md`, `loom-halt.md`.
