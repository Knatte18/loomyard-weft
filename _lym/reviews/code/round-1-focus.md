---
round: 1
focus:
  - "internal/runfsm/state.go: does Derive follow D1's precedence order exactly, including the landed rule (batten record done, or no record and board done) and the cancelled record whose task branch is gone from origin reading as no record?"
  - "internal/runfsm/act.go: does Act answer in D1's order (owed push offline, hand_over, code tip missing, landing past parent merge, wait for hold, pause, paused, cancelling, terminal, else act), and is a shed pause an aside as the last Added entry says?"
  - "Cancel path in internal/battencli (cancel.go, cancelmarker.go, cleanup.go, startguard.go): is the cancel intent committed and pushed before anything ends, does the finish set shed state cancelled and never done, and does every pair removal delete the cancelled batten record in the same step?"
  - "internal/battencli cleanup: does it remove a cancelled record only once its pair is gone and refuse one whose pair stands, naming lym fabric remove <slug>, with ClearDoneRun leaving cancelled runs and their task branch alone?"
  - "Callers of runfsm.Act (dobbyengine passes, battencli run and step, loomcli resume, tendercli resume): does each ask the act rule before acting and refuse wait or hand_over with a named way forward, and does each new function have a production caller?"
  - "internal/tendercli and internal/tenderengine: do pause and resume write and clear the run-scope intent, commit and push before returning, refuse a run another machine owns naming that machine, and handle origin unreachable by committing and owing the push?"
  - "PR record (internal/pullrequest, landingshed) and fixer attempt counts: are both written into the pair's tracked records, committed with the transition, and read back on another machine; does the diff leave any `.lym` path still holding them?"
  - "pairteardown ReadRun and teardown changes, and fabricengine origin reads: does reading from origin's branch when no local pair exists match the local read, and is tolerant record reading (PATTERN-tolerant-records) kept for new tracked records?"
  - "pattern/PATTERN-lym-files.md: does the diff land the declared change with its declared text? Also check pattern/PATTERN-leaf-packages.md against the scope, since the answer key does not list it."
  - "Scope: the diff touches many packages beyond the record's Scope list; check each unlisted package's change is licensed by a plan card, and that the origin-owed marker's retirement matches D3 while the bolt-holds-catch-up marker stays."
  - "Test economy: TestCancelMarker_ReadsNothingFromAPrimeWithNoScratchRoot, TestJudgeCancelMarker and TestLog_ChangesOnly are reported contained; compare their assertions with the named covering tests before filing anything."
---

Seed focus for round 1, set from the decision record's Goal, D1, D3, Added entries and Acceptance criteria, and the diff stat of the review range.
