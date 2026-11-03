```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1.** I found no case where routing `presentTaskWorktreePath` through `taskWorktreePresent` is wrong.
  - Every `taskWorktreeLocation` caller already propagates its error. Those callers are `arm.go:1023`, `goto.go:19`, `cancelcmd.go:396` and `:411`, `dobbywire.go:19`, `readopt.go:78` and the locator.
  - A stray directory used to surface as a bare resolver error from these callers and now surfaces as the by-name refusal, which is never worse.
  - The cached-locator branch kept its stat-only check, which the `.git`-hiding test in `lifecycle_integration_test.go` needs. Its stat-only path now skips the stray check, but the first resolution already ran it.
- **F2.** The new subtest fails with the defect in place. Dropping `ReachedParent: hubChildReachedParent(location, slug)` leaves the seam nil, so `promoted` stays true and `refuse` returns the beam-mark refusal. `childReachedParent` returns true for `CurrentProducer == NameWorktreeTeardown` whatever the child reads. The subtest mutates no shared state, so it does not disturb the sibling subtest that uses `promoted`.

Further tool calls:
1. `grep` for the `taskWorktreeLocation(` and `taskWorktreeLocator(` callers, because the context only showed `wire.go`.
2. `Read` of `readopt.go:60-100`, to confirm the `SpawnPush` seam's error handling.
