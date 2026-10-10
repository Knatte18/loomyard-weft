# Webster audit and recovery: pathless findings, read-only classifier, rebaseline Stuck

`webster-audit-gaps` · Task · bug, enhancement, webster, fabric · running · Publish

## Goal

Close the webster audit and recovery gaps that `cli-surface` batch 06 hit and that `webster-replan-state` (5a8be0274) left open, in `internal/websterengine` and `internal/fabricengine/refscanner.go`.

## 1. recover-batch fails on a pathless finding (#513)

In run `cli-surface` (2026-10-10), batch 06 failed `record-batch` on a fabric-reference audit finding whose only `uncheckable` entry is a shell command (`for v in ...; do lyx fabric $v --help --json ...; done`).
`lyx webster recover-batch 06` then errored with "websterengine: resolve links in .../fabric-reference: lstat ...: file name too long".
`RecoverSpawnOrAttach` passes `prior.Uncheckable` to `contractWritesFor` (`contractevidence.go`), whose `contractCanon` calls `canonicalPath` (`audit.go`), which runs `filepath.EvalSymlinks` on the finding text; ENAMETOOLONG is not `ErrNotExist`, so it is returned.
5a8be0274 added an early exit when every pathless finding is read-only (`recoverbatch.go`), but a pathless finding that is not read-only still reaches `contractWritesFor` → `contractCanon`.

Expected: `contractWritesFor` and `splitContractPaths` skip pathless entries, so recover-batch returns its documented refusal naming `accept-audit --batch NN`.

## 2. The fabric-reference audit fails help-only lyx calls (#514)

Batch 06's implementer fork read help with `lyx fabric $v --help --json`; the audit (`ClassifyViolation`) classed it as a correctness violation, and `fabricengine.IsReadOnlyCommand` rejected it, so `accept-audit` could not keep the batch's six committed cards (workaround: `reset --to batch-start`, `accept-audit`, redo the batch).
5a8be0274's `readOnlyLyxSegment` admits a lyx help call only when `--help` or `-h` is its last word.

Expected:

- a trailing `--json` (`lyx ... --json`, `lyx ... --help --json`) is read-only too, since 57243019b no command runs under the global `--json`;
- a quoted or globbed argument to an admitted reader (`git log --format='%h %s'`) does not on its own make a command non-read-only;
- a loop or a `$v` expansion stays non-read-only on purpose.

## 3. Reader flags that write (Webster-Review finding `d8-reader-write-flags-unbounded`, BLOCKING at landing)

The reader allow-list in `refscanner.go` admits git reader commands without bounding their flags, so a reader given a writing flag could pass as read-only.
Expected: check each admitted git reader's flags and reject any flag that writes (for example an output-file or config-writing flag).

## 4. Auto-rebaseline merge-base failure (finding `card4-merriambase-failure-escapes-stuck`)

An error computing the merge base during auto-rebaseline may surface as a hard failure instead of Stuck.
Expected: map it to Stuck with a way forward, and test it.

## 5. begin-batch's terminal-dead refusal text (finding `d12-begin-batch-dead-refusal-stale`)

The refusal predates the second recovery.
Expected: it names the second-recovery route and the `recovery_exhausted` state that 5a8be0274 added.

## 6. Thin test rows (findings `card14-classifier-rows-miss-glob-and-tilde`, `thin-test-rows-cards-13-11-17-21`)

The classifier tables lack glob and tilde cases, and the rebaseline and shell-wait tables are thin.
Expected: add those rows alongside items 2-4.
