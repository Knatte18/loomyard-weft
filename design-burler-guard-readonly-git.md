# Burler fork guard halts a run on a read-only git command

`burler-guard-readonly-git` · Task · bug, burler, shed · running · Drift-Verify

Imported from [issue #898](https://github.com/Knatte18/loomyard/issues/898).

## What happened

A `lym loom` run halted `failed` (anomaly `producer-hard-failure`) at producer `Discussion-Burler-Review`, round 2 attempt 2.
The halt reason from `shedadapters` said the burler review fork "ran a git-mutating command".
The command it flagged was a chain of read-only git inspections: `git branch -a --contains <sha>`, `git merge-base --is-ancestor <sha> HEAD` and `git show --stat <sha>`, piped through `head` and `grep`.
None of these changes repository state.

## Why it is abnormal

The fork guard that rejects state-mutating git commands misclassifies read-only commands as mutating.
Likely causes: it matches on the `branch` verb without checking for a mutating flag, or it treats the whole compound line (pipes, `&&`, `||`, `2>/dev/null`) as unsafe.
A false positive here is a hard failure, so a review segment that was working normally ended the run.

## What to do

Make the fork git guard classify by verb and arguments:
allow `branch` with listing flags (`-a`, `-r`, `--contains`, `--list`), `merge-base`, `show`, `log`, `diff`, `status` and `rev-parse`, and judge each simple command in a pipeline or `&&`/`||` chain on its own.
Add a regression test with the command line above.
Consider downgrading a guard hit from `producer-hard-failure` to a retried attempt, since the fork changed nothing.

Trace file: `.lym/logs/trace-20261028T105548Z-7bfaff2abe252dc3-993265.log`.

Task slug `validator-checks`; source note: `loom-halt.md`.
