## Batch 01-gate-test-map-mode, attempt 1

- Card 2: `testOnlyCodeAllowed` (cmd/lym/testonlycode_linux_test.go) holds one block of testcover entries plus the
  `unreachedMapPlanning` const below it; delete both. `Map.FailedAt` has no entry (the guard did not report it).
- `gate test`'s `Long` text is near the 2000-byte cap of cmd/lym `TestCLIForm_EveryCommand`; the card gate's fixer already shortened it.
- Card 4: remove the "Bound: ... written with no such check" lines in internal/gatecli/doc.go's Coverage maps section.
- `testMapRun.takeSlot` still takes the `acquire` func that only `holdSlot` now passes.
- Hub test fixtures: append to `info/exclude`, never overwrite it, or the prime reads dirty (no green record, whole diff scope).
