# Preflight worktree-clean blocks on lym's own _lym/usage session record

`usage-record-blocks-preflight` · Note · bug, preflight, fabric

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
