```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F2 change set for agreement with its surroundings (fix-consistency lens).

No findings.

- F2: no other non-test Go comment in `internal` says "Stop block" or "Stop blocks". The reworded `stopBlocks` comment in `pool.go:51` agrees with `stopBlockCap` in `verbs.go:322` and the `ClaimAtBoundary` doc, which say "Stop claims that handed out mail".
- F1 seam: `beforeOutcomeWait` is declared on `Options`, set only in `export_test.go`, and called once in `awaitOutcome`. Its doc follows the `writeFault` test-seam wording ("a test seam, nil in production"), and the `awaitOutcome` doc comment, which does not mention the seam, remains accurate.
- Review F1 part (b), a client closing between the ring write and the ack, is already covered by the existing row `AMarkedHookWaitWhoseClientLeavesBeforeTheAckLeavesTheMailUnread` (`server_test.go:413` before the edit). The new row covers part (a) only. The fixer's report should say (b) is covered by that row, not leave it unmentioned.

One call, one inspection of the diff: I grepped `internal` for stale "Stop block" wording and for every use of `beforeOutcomeWait`, because my context held no grep of other files for either.
