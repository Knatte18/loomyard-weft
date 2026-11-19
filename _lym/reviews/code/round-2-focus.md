---
round: 2
focus:
  - "README.md: does it describe cloning a one-repository hub (both clone forms), what the layout puts on code branches, the `--single-repo` hazard on a two-repository hub's code repository, and the switch, reset and stash hazards in the prime, as docs/getting-started.md does?"
  - "internal/fabricengine/add.go, the one-repository `anchorTo` branch of `Topology.Add`: when a fork is added at another anchor, do the fork point's tracked `_lym` records move to the new anchor and land in the first records commit with the Anchor-Change trailer, as the two-repository path does?"
  - "internal/fabricengine/recordsduringmerge.go, `commitRecordsAroundMergeWith`: when `newPaired` or `saveMergeState` fails after `update-ref` moved HEAD, does the merge record's `WarpStart` still name the old HEAD, so that a later `MergeAbort` drops the records commit?"
  - "pattern/PATTERN-run-fsm.md: does the background agree with `IsPushOwed` and `FailedPushesOwedBy` on a `protected` prime push (not owed), and with the act rule's refusal wording, which names the owed record's own way forward?"
  - "internal/runfsm/act.go, `Judge`'s `aside.PushOwed` block: are `PushOwed`, `PushOwedError`, `PushOwedWorktree` and `PushOwedWayForward` cleared together?"
  - "internal/battencli/overview_test.go, `TestLoadOverviewThresholds`: do its no-warning rows still fail on any WARN, as at the review base, unless a `Loosened-Check:` trailer names a measured cause?"
  - "One-repository tests added or changed for the fixes in this round: does each pin a behavior the existing tests of the same function do not already cover?"
---

Focus targets the fixes expected after round 1; it limits no severity and widens no range.
