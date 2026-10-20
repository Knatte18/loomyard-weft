# Darn-Write verify cannot pass a failure fixed on the parent

`darn-verify-parent-fix` · Note · enhancement, batten, loom

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
