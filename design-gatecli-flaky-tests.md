# Flaky gatecli tests fail the first verify run and pass on rerun

`gatecli-flaky-tests` · Note · bug, gate

Imported from [issue #118](https://github.com/Knatte18/loomyard/issues/118).

What happened: the webster integration-suite triage recorded two tests in the gatecli package that failed the first verify run and passed or vanished on rerun, so they were classed as flaky and not as regressions.

- gatecli TestGateTest_DetachedRun, subtest 'a passed lease ends a --wait gate line through its child, which kills its tiers and frees its slots'
- gatecli TestRedundancyStep, subtest 'ABrokenLymBuildAndACorruptMapAreErrorLinesThatLeaveTheOtherTestsJudged'

Why it is abnormal: a verify run should give the same result on an unchanged tree. Both tests exercise process, lease and slot handling and a lym build, which points to timing or shared-state dependence, so they can fail a verify intermittently and cost reruns.

What to do: run both tests repeatedly, including under load and in parallel with other gatecli tests, find the race or shared state, and make them deterministic.

Provenance: task machine-name-verb; note webster-verify-triage.md.