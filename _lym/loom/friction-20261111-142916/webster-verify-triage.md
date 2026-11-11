Integration-suite triage: non-regression verify failures

These identities failed the first verify run and passed or vanished on rerun (flaky):

- github.com/Knatte18/loomyard/internal/gatecli.TestGateTest_DetachedRun/a_passed_lease_ends_a_--wait_gate_line_through_its_child,_which_kills_its_tiers_and_frees_its_slots
- github.com/Knatte18/loomyard/internal/gatecli.TestRedundancyStep/ABrokenLymBuildAndACorruptMapAreErrorLinesThatLeaveTheOtherTestsJudged

