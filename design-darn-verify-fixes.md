# Darn-Write stops looping on verify failures it cannot own, and the two flaky reed tests behind them

`darn-verify-fixes` · Task · bug, enhancement, batten, gate, loom, shed, reed

# Goal

A verify failure that Darn-Write cannot own, one in a tier its session may not run or one the parent has already fixed, no longer loops through Darn-Write and escalates; and the two flaky reed tests behind both reports are made deterministic.

# Part 1: a tier the writer cannot run

Imported from [issue #714](https://github.com/Knatte18/loomyard/issues/714).

## What happened

`lyx loom` halted `blocked` (anomaly `escalation-to-human`) at Publish.
`publish_verify` failed after merging the parent branch with `internal/reedengine` `TestBoot_TmuxConf/AppendingFileIsSourcedOncePerServerStart/TwoWorktreesBootedConcurrently` reporting `run list-commands: exit status 1: server exited unexpectedly`.
The task's diff did not touch `internal/reedengine`, so the failure looks like a race in booting two tmux servers concurrently, not a defect of the change.

## Why it is abnormal

- The test is flaky and fails an unrelated task at the Publish gate.
- The Darn-Write agent could not reproduce or fix it: the test carries the `tmux` build tag, and tmux-tier go runs are refused in a darn session.
  The agent handed off with no commit, so the run escalated to a human with a failure the darn step can never resolve.

## What to do

- Make `TwoWorktreesBootedConcurrently` deterministic, or retry or isolate the tmux server start it races on.
- Consider whether a darn session facing a verify failure only in a tier it cannot run should get a way to rerun it, or should be told up front to escalate, rather than looping through the Darn cycle first.

Provenance: task `stencilstore-dev-warning-noise`; notes `Darn-Write.md`, `loom-halt.md`.


# Part 2: a failure fixed on the parent

## Seen (2026-10-20, arbiter-verdict-quoting)

Publish failed on `TestOpenWindow_RealTmux`, a bug outside the branch, and batten sent the run back to Darn-Write.
The fix then landed on main through `openwindow-exit-race`, but Darn-Write's verify kept running the failed test on the task branch, which takes main only at Merge-In, so it failed deterministically (verify-7).
The orch had the writer run `lyx fabric merge-in main` by hand.

## Idea

When a verify failure is sent back to Darn-Write and the parent has moved since, merge the parent in before the verify re-runs, or route a failure in a package the branch does not touch to Merge-In instead of Darn-Write.

## From issue #765

Imported from [issue #765](https://github.com/Knatte18/loomyard/issues/765).

What happened: a loom run for a task unrelated to reed halted blocked at Publish.
`publish_verify` failed after merging `main` into the task branch.
The verify log shows a single failing test: `internal/reedengine` `TestOpenWindow_RealTmux/ClosesWhenItsCommandExitsAtOnce`, with the message `reed: the opened window did not read remain-on-exit off: window @3 read "on", want nil`.
Every other package in the verify run passed.
A later `lyx loom goto --to Darn-Write` and `lyx loom resume` then halted Darn-Write blocked with "gate did not pass after 3 attempts", and the `gate-findings.md` file the halt note names was no longer on disk.

Why it is abnormal: the failing test and the reed window code are outside what the task changed, so a task agent cannot fix it.
The halt's way forward sends the run back to Darn-Write, which spends gate attempts on a failure the task does not own.
The test also reads as timing- or merge-dependent: recent `main` history contains a reed `OpenWindow` read-back change and its revert.

What to do:
- Check whether `main` itself fails this test, and whether it is flaky under the load of a full-module verify.
- Fix or stabilise the test or the `OpenWindow` read-back.
- Consider making the Publish halt's way forward distinguish a failure in code the task did not touch from one it did.
- Keep the gate findings file until the halt is cleared, so the halt note's path stays readable.

Provenance: task `arbiter-verdict-quoting`; notes `loom-halt.md`, `loom-halt-2.md`.

## Note `verify-diff-scope-unrelated-pkg`

Folded in on 2026-10-22 (operator decision): the same failure class, a verify gate failing in a package the task did not touch.
Commit `7ac8070a8` already made the watchdog stop marker work without inotify; what stays open here is why the impacted set picked `internal/reedcli` for a `planglyph`-only diff.

Imported from [issue #785](https://github.com/Knatte18/loomyard/issues/785).

The verify gate for a change confined to `internal/planglyph` ran `internal/reedcli` tests.
`TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping` failed on every rerun, although the package's test closure contains no changed file.

The test consumes the watchdog's stop marker only through an inotify watch.
The host was near its `fs.inotify.max_user_instances` limit (about 119 of 128), so a failed watch open leaves the marker unread and the stop subtests time out.

Two things are abnormal:

- The diff-scoped verify selected a package with no changed file in its closure, so an unrelated package can block a task.
- The test fails on host inotify exhaustion instead of skipping or reporting the cause, so the failure reads as a code defect.

The writer could neither raise the host limit nor change an unrelated package in the task, so the finding went to the parent.

What to do: check why the diff-scoping selected `internal/reedcli` for a `planglyph`-only diff, and make the watchdog test skip or fail with a clear message when the inotify watch cannot be opened.

Provenance: task `create-target-transient`, note `Darn-Write.md`.
