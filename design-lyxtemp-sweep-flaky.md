# Flaky lyxtemp TestSweep/two_concurrent_sweeps_both_succeed fails first verify run

`lyxtemp-sweep-flaky` · Note · bug

Imported from [issue #584](https://github.com/Knatte18/loomyard/issues/584).

The integration-suite triage in a verify run reported `internal/lyxtemp.TestSweep/two_concurrent_sweeps_both_succeed` as flaky: it failed the first verify run and passed or vanished on rerun.

Why abnormal: a test of concurrent sweeps that fails nondeterministically points at a race or an unbounded retry in the sweep itself, and flaky tests cost a verify rerun and can block a Publish step (the same run halted blocked at Publish with `publish_verify failed after merging parent branch`, exit code 1, escalation-to-human; the notes do not show whether this test caused that failure).

What to do: reproduce by looping `lyx gate test -run 'TestSweep' ./internal/lyxtemp` under load, find the race in the concurrent-sweep path, and make the test deterministic or the sweep race-safe.

Provenance: task `runs-overview`; notes `webster-verify-triage.md`, `loom-halt.md`.
