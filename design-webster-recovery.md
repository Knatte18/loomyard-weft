# Webster recovery has a way forward, and its audit accepts a sanctioned gate test

`webster-recovery` · Task · bug, webster · running · Darn-Write

## webster recovery way-forward is circular: recover-batch points to accept-audit, which refuses; reset is refused while run.lock is held (was note `webster-recovery-circular`)

Imported from [issue #878](https://github.com/Knatte18/loomyard/issues/878).

#### What happened

After `lyx webster record-batch` failed a batch on fabric-reference audit findings, `lyx webster recover-batch` refused with `needs_fresh` and a way forward of "`lyx webster accept-audit --batch 6` then `lyx webster recover-batch 6`", or `reset --to batch-start`, or `reset --to start`.
`lyx webster accept-audit --batch=6` then refused, because the batch's commits are kept only when every fabric reference is read-only, and the finding is the very one that made recover-batch advise it. Its only way forward was `lyx webster reset --to batch-start --batch 06`.
`lyx webster reset --to=batch-start --batch=6` was refused with `run is already in progress ... (run.lock held)`, with way forward "wait for it to finish, or check `lyx webster status`".
The Webster session had no verb left that it was allowed to use, so it escalated: the run halted `blocked` with anomaly `escalation-to-human`, and the halt note says the parent has to run the reset after the run stops.

#### Why it is abnormal

- `recover-batch` offers `accept-audit` as a way forward in a case where `accept-audit` is known to refuse; the advice cannot be followed.
- The refusals were retried several times (two `accept-audit` refusals, three `recover-batch` refusals), each with the same text.
- `reset` is refused for a lock the Webster session itself holds, so the one working remedy is unreachable from inside the run that needs it.

#### What to do

- Have `recover-batch` run the `accept-audit` check and drop that option from the way forward when it would refuse.
- Let `reset --to batch-start` run from the Webster session that owns `run.lock`, or have the refusal name who must run it and when.

#### Provenance

Task `umbrella-anchor`; notes: webster-refusal-recover-batch.md, webster-refusal-recover-batch-2.md, webster-refusal-recover-batch-3.md, webster-refusal-accept-audit.md, webster-refusal-accept-audit-2.md, webster-refusal-reset.md, loom-halt-3.md.

## webster fabric-reference audit flags the sanctioned lyx gate test as a run-state rewrite, failing the batch (was note `webster-audit-gate-test-readonly`)

Imported from [issue #877](https://github.com/Knatte18/loomyard/issues/877).

#### What happened

A Webster implementer fork ran `lyx gate test --wait --gate card --card <card>` and `lyx gate test --wait --gate batch --card <card>`, the gate commands the session's own gate rule tells agents to run, and once chained them after a `git commit`.
`lyx webster record-batch` failed the batch with `fabric-reference violation ... ran a fabric-referencing command ("lyx gate test ...") that can rewrite run state — an implementer fork must never touch the fabric repo directly`.
The finding "names no path"; it is raised from the command text alone.
`lyx webster recover-batch` then refused with `needs_fresh` on the same findings.

#### Why it is abnormal

`lyx gate test` is the mandated way to run tests, so the audit makes the normal workflow a violation.
The audit does not distinguish a gate invocation from a command that writes the fabric repo, and a pipe through `tail`/`grep` does not change that.
A whole batch of otherwise valid work was lost to the flag, and the run halted blocked at Webster with an escalation.

#### What to do

Treat `lyx gate test` (and its `--wait`, `--gate`, `--card` forms) as read-only for the fabric-reference audit, or make the audit check the paths a command actually writes.
At minimum the finding should say why this specific command can rewrite run state.

#### Provenance

Task `umbrella-anchor`; notes: webster-refusal-record-batch.md, webster-refusal-recover-batch.md, webster-refusal-recover-batch-2.md, webster-refusal-recover-batch-3.md, loom-halt.md, loom-halt-2.md, loom-halt-3.md.

