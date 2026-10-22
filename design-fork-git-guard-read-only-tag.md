# Fork git guard flags read-only 'git tag --points-at' as a state-mutating command

`fork-git-guard-read-only-tag` · Task · bug, shed

Depends on: [`run-process-ownership`](design-run-process-ownership.md)

Imported from [issue #819](https://github.com/Knatte18/loomyard/issues/819).

## What happened

A `lyx loom` run halted `failed` (anomaly `producer-hard-failure`) at the `Plan-Burler-Fix` producer, round 2 attempt 2.
The shedadapters check reported that the twill fixer fork "ran a git-mutating command" and named this command line:

`<sed -n read of a source file>; git -C <target> log -1 --format='%h %d' ; git -C <target> tag --points-at HEAD`

Every part of it is read-only: `sed -n` reads, `git log -1` reads, and `git tag --points-at HEAD` only lists the tags pointing at a commit.
The fork wrote nothing, yet the halt message says "a fork must never run a state-mutating git command" and tells the operator to revert what the fork wrote.

## Why this is abnormal

The guard appears to classify `git tag` as mutating by subcommand name, without looking at its flags.
`git tag --points-at`, `--list`, `-l` and `--contains` are read-only forms.
A false positive halts the run as a hard failure and sends the operator after a revert that has nothing to revert.
It also wastes the review rounds already spent: the history shows the plan segment had been through several Bouncer, Burler-Review and Burler-Fix rounds.

## What to do

Make the fork git-mutation check distinguish read-only `git tag` forms (`--points-at`, `-l`/`--list`, `--contains`, `--merged`, `--no-merged`, `-n`) from creating or deleting forms (`git tag <name>`, `-d`, `-a`, `-s`, `-f`).
Check whether other multi-purpose git subcommands (`branch`, `stash list`, `remote`, `config --get`) have the same flaw.
Add a test for the command line above.
Also consider that the halt message's way forward ("revert what the fork wrote") is misleading when the working tree is clean.

Provenance: task `quarry-csharp`, note `loom-halt.md`.

## Operator decision (2026-10-22)

The fork git guard was made subcommand-only on purpose (commit 82cd60938); it now lets read-only `git tag` forms through (`--points-at`, `-l`/`--list`, `--contains`), while a `git tag` that creates or deletes a tag stays refused.
