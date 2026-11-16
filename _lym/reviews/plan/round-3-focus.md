---
round: 3
focus:
  - "Cards 10 and 12: `saveRunState` takes the state-mutation lease itself, and a batch begin holds that lease. Settle whether any save inside a lease-holding section can re-acquire the lease, given `runplan.AcquireStateMutation` takes a blocking flock on `mutate.lock`, and whether a `TestRun` row reaches it against a real state directory."
  - "Cards 12 and 13: `StopRecord` is cleared at a batch begin and in `reenter`. Settle whether any path where a re-entered attempt ends done, or the plan's last batch finishes, leaves `StopRecord` set for the next call."
  - "Card 13: `reenter` and `resumeStrands` hand `Run` a continuation. Settle whether each function the re-entry no-spawn path and the crash-resume path call is a target of some card, and whether the card states the rung, retry allowance, start commit and card set a resumed or re-entered batch continues with."
  - "Cards 12, 16, 18 and 23: `outcome.yaml` is written by `weaverengine.Run` and read by `RunInFlight`, and `weaverEntry` requires `WeaverRun`. Settle whether the writer and reader share one stated key set, and whether `weaverengine.Run` is filled in `loomCLI.wire` or reached only through a default no card wires (PATTERN-test-only-code)."
---

Focus targets the sites round 2 touched.
The reviewer checks each against the rubric afresh and may raise any other finding.
