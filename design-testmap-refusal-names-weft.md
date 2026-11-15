# lym gate testmap refusal's way forward names a path that also refuses

`testmap-refusal-names-weft` · Note · bug, gate, cwd

Imported from [issue #126](https://github.com/Knatte18/loomyard/issues/126).

## What happened

Run from a task worktree, `lym gate testmap internal/adoptcli` refused because the worktree is not the prime.
The refusal's way forward named the prime's weft directory (`loomyard-weft`).
Run there, the verb refused again: that directory holds no `go.mod`.
The call works only from the prime's code worktree (`loomyard`), which neither refusal names.

The way forward is built from `hubgeom.PrimeWorktreePath`, which returns the weft path where a code path is needed.

## Why it is abnormal

A refusal's way forward must be a call that succeeds.
Here it sends the caller to a second refusal.
The task allowed one build and no repeats, so the misleading way forward nearly cost the pilot its only run.

## What to do

Make the `lym gate testmap` refusal name the prime's code worktree, not the weft.
Check the other refusals that take their way forward from `hubgeom.PrimeWorktreePath` for the same weft-for-code mix-up.

Provenance: task `test-redundancy-candidates`; note `Darn-Write.md`.