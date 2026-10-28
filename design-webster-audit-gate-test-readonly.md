# webster fabric-reference audit flags the sanctioned lyx gate test as a run-state rewrite, failing the batch

`webster-audit-gate-test-readonly` · Note · bug, webster

Imported from [issue #877](https://github.com/Knatte18/loomyard/issues/877).

## What happened

A Webster implementer fork ran `lyx gate test --wait --gate card --card <card>` and `lyx gate test --wait --gate batch --card <card>`, the gate commands the session's own gate rule tells agents to run, and once chained them after a `git commit`.
`lyx webster record-batch` failed the batch with `fabric-reference violation ... ran a fabric-referencing command ("lyx gate test ...") that can rewrite run state — an implementer fork must never touch the fabric repo directly`.
The finding "names no path"; it is raised from the command text alone.
`lyx webster recover-batch` then refused with `needs_fresh` on the same findings.

## Why it is abnormal

`lyx gate test` is the mandated way to run tests, so the audit makes the normal workflow a violation.
The audit does not distinguish a gate invocation from a command that writes the fabric repo, and a pipe through `tail`/`grep` does not change that.
A whole batch of otherwise valid work was lost to the flag, and the run halted blocked at Webster with an escalation.

## What to do

Treat `lyx gate test` (and its `--wait`, `--gate`, `--card` forms) as read-only for the fabric-reference audit, or make the audit check the paths a command actually writes.
At minimum the finding should say why this specific command can rewrite run state.

## Provenance

Task `umbrella-anchor`; notes: webster-refusal-record-batch.md, webster-refusal-recover-batch.md, webster-refusal-recover-batch-2.md, webster-refusal-recover-batch-3.md, loom-halt.md, loom-halt-2.md, loom-halt-3.md.
