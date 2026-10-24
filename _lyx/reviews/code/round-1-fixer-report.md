# Fixer-report, code review round 1

## Changes

All three findings were fixed as proposed, in `internal/gitexec/gitexec_test.go`.
They share one file and interleaved hunks, so commit `9b2662c5d` carries F1, F2 and F3 together rather than one commit each.

- F1: a new subtest beside the stall subtests sets the stall bound equal to the deadline, runs a silent ext remote, and asserts one run, one kill report, and a `*GitError` with `Timeout` equal to the deadline and `Stalled` false.
- F2: the remote environment subtest asserts the child sees `GIT_TRACE_PACKET=`, and the local subtest asserts its printed environment lacks `GIT_TRACE_PACKET`.
- F3: the never-answers stall subtest asserts exactly one retry Warn and that lyxtemp's run directory holds no `git-packet-trace-*` file afterwards.
  The remote environment subtest sets an operator `GIT_TRACE_PACKET` with `t.Setenv` and asserts the child does not see it.

Self-check follow-up, commit `5434f37a7`:

- The local subtest is renamed to mention the packet trace.
- The `TestRun_RemoteBounds` doc comment names the stall bound and the process environment among the serial-state reasons.
- The local subtest unsets an inherited `GIT_TRACE_PACKET` (restored by `t.Setenv`'s cleanup), so an operator's exported variable cannot fail it.

`lyx gate test --upto 2 ./internal/gitexec -- -run TestRun_RemoteBounds -count=1` passes.

## Self-check

- fix-consistency, LOW, local subtest name omits the packet trace: fixed as proposed.
- fix-consistency, LOW, doc comment omits process-global state: fixed as proposed.
- fix-code-correctness, LOW, the equality subtest depends on a timer race: not changed; any other bound arms the check legitimately, so no sound change exists (same finding as the fix-proposal one below).
- fix-code-correctness, LOW, inherited `GIT_TRACE_PACKET` fails the local assertion: fixed (merged with the fix-proposal duplicate).
- fix-proposal, LOW, same inherited-variable case: fixed, as above.
- fix-proposal, LOW, the F1 subtest detects a broken guard only at equality, likely but not deterministically: not changed, the equality is the strongest available value.
- fix-affected-set: no findings.

## Disputed

- fix-code-correctness and fix-proposal, LOW, timer race in the F1 subtest. Ground: every fix worse. Evidence: with `stallBound > deadline` the deadline kills first and the stall timer is stopped, so the test could not fail; with `stallBound < deadline` the guard is armed legitimately; equality is the only value that distinguishes the guard.

## Deferred

None.
