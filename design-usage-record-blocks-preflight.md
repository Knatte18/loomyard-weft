# Preflight worktree-clean blocks on lym's own _lym/usage session record

`usage-record-blocks-preflight` · Task · bug, preflight, fabric · starting · Seed-Child

Imported from [issue #190](https://github.com/Knatte18/loomyard/issues/190).

## Seen

A darn run blocked at Preflight on its first step.
The worktree-clean precondition reported `?? _lym/usage/session-<id>.json`, the usage record of one of lym's own sessions, as an uncommitted state change.
The driver's tackler handed the stop back to the parent orch, asking whether to commit the record or to ignore `_lym/usage`.

By the time the orch looked, loom's own `loom: commit run records` commit had committed that record, so `_lym` was clean.
The usage record had been written between loom's records commit and the worktree-clean check.

Stop reason, verbatim:

```
preconditions not met: worktree-clean: uncommitted state changes under `_lym`: ?? _lym/usage/session-db065ed3-54e1-4008-bacd-df683191b516.json; way forward: commit or stash the code changes with git, and commit the _lym changes with `lym fabric commit`, then re-step
```

## Workaround

The orch checked that the record was tracked and `_lym` was clean, then ran `lym loom resume` in the task worktree.
The run's tackler could neither commit nor decide, so the run waited for a parent turn.

## Expected

lym's own usage records never block worktree-clean.
Either worktree-clean leaves out `_lym/usage`, or lym commits the usage records before it runs the check.


## From issue #194

Imported from [issue #194](https://github.com/Knatte18/loomyard/issues/194).

Preflight halts a run as `blocked` because lym's own per-session usage file leaves the worktree dirty.

What happened:
The loom run halted twice at Preflight, anomaly `escalation-to-human`, with this reason (the session id differs between the two halts):

```
preconditions not met: worktree-clean: uncommitted state changes under `_lym`: ?? _lym/usage/session-<id>.json; way forward: commit or stash the code changes with git, and commit the _lym changes with `lym fabric commit`, then re-step; a session lym refuses the verb from reports status: FAILED and the orch runs it
```

The only dirty path is an untracked `_lym/usage/session-<id>.json`, which lym itself wrote for the session that stepped the run.
No code change or agent edit was involved.

Why it is abnormal:
The `worktree-clean` precondition is tripped by lym's own bookkeeping, so a fresh step can block before any work starts.
The way forward asks the session to run `lym fabric commit`, but the same message says a session whose verb lym refuses must report `status: FAILED` and leave it to the orch.
The run therefore escalates to a human for a state lym created.

What to do:
Either exclude `_lym/usage/` from the `worktree-clean` check (or git-ignore it), or have lym commit or flush the usage file before Preflight evaluates the worktree.

Provenance: task `models-wemod-robustness-sanity`; notes `loom-halt.md`, `loom-halt-2.md`.
