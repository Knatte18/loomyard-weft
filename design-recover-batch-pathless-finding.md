# recover-batch treats a pathless fabric-reference finding as a path and fails with ENAMETOOLONG

`recover-batch-pathless-finding` · Note · bug, webster

Imported from [issue #513](https://github.com/Knatte18/loomyard/issues/513).

## Seen

In run `cli-surface` (2026-10-10), batch 06 failed `record-batch` on a fabric-reference audit finding whose only `uncheckable` entry is a shell command (`for v in ...; do lyx fabric $v --help --json ...; done`).
`lyx webster recover-batch 06` then errored with "websterengine: resolve links in .../fabric-reference: lstat ...: file name too long".
`RecoverSpawnOrAttach` (`internal/websterengine/recoverbatch.go:377-378`) passes `prior.Uncheckable` to `contractWritesFor` (`contractevidence.go:136-137`), whose `contractCanon` calls `canonicalPath` (`audit.go:463-485`), which runs `filepath.EvalSymlinks` on the finding text; ENAMETOOLONG is not `ErrNotExist`, so `audit.go:474` returns it.

## Workaround

`lyx webster reset --to batch-start --batch 6`, then `lyx webster accept-audit --batch 6`, then `lyx webster run`, discarding the batch's commits.

## Expected

`contractWritesFor` and `splitContractPaths` skip pathless entries, so recover-batch returns its documented refusal naming `accept-audit --batch NN`.
