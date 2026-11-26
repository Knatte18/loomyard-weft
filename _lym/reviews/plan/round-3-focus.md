---
round: 3
focus:
  - "Card 9 (_lym/plan/09-reconcile-before-load.md), planned tier1 tests: for each test that runs a loom or shed verb on a seeded run, does the verb stop before it starts a tmux server or drives a producer (PATTERN-test-speed), and does it still reach the config load the test is meant to prove, with an assertion that fails without the walk?"
  - "Card 9, walk sites: does every route to a strict loom or tapestry load run the walk first? Include the dobby, where internal/beamcli/dobbydeps.go builds run notices wired in internal/battencli/dobbynotice.go that reach a configreg.SettingLookup of tapestry, and internal/dobbyengine re-execs onto a changed binary with no start verb in between."
  - "Card 8 (_lym/plan/08-board-run-settings-carry.md): boardCriticalSection in internal/boardengine/board.go saves board.json, runs afterSave, then renders. After a failure at any of those steps, does board.json keep its old bytes under the write lock and does the next walk retry the carry, as decision D7 requires, and does a planned test reach the failure?"
  - "Cards 2 and 5, decision D4's --unset removal route: for a config file holding two old keys, can each be removed? setChecked in internal/configcli/configcli.go rolls a write back when the load error after it differs from the one before, and setHubWide runs validateHubFile on the hub default layer. Check what card 2's error reports with several old keys and what card 5's planned tests cover."
  - "Card 8's planned assertion that a second walk writes no board commit: under go test the detached sync resolves selfexec.LymBinary, which returns ErrTestBinary. Can the assertion fail?"
  - "Card 9's TestLoomVerbs_StaleStampReconcilesBeforeACirclingVerbLoads: does the verb it runs load any config, and does the loaded value depend on the key the test seeds?"
---

The last entries concern plan text that may be unchanged since the previous round.
