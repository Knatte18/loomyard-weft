## Batch 01-gate-test-map-mode, attempt 1

- Card 2: `testOnlyCodeAllowed` (cmd/lym/testonlycode_linux_test.go) holds one block of testcover entries plus the
  `unreachedMapPlanning` const below it; delete both. `Map.FailedAt` has no entry (the guard did not report it).
- `gate test`'s `Long` text is near the 2000-byte cap of cmd/lym `TestCLIForm_EveryCommand`; the card gate's fixer already shortened it.
- Card 4: remove the "Bound: ... written with no such check" lines in internal/gatecli/doc.go's Coverage maps section.
- `testMapRun.takeSlot` still takes the `acquire` func that only `holdSlot` now passes.
- Hub test fixtures: append to `info/exclude`, never overwrite it, or the prime reads dirty (no green record, whole diff scope).

## Batch 02-testcover-planning-delete, attempt 1

The testcover selector, upkeep planning, declaration compare, Map.FailedAt and Toolchain.GoRoot are gone,
and cmd/lym's test-only-code allowlist holds no map planning entry.
`Map.Failure` and `Map.BuildFailure` stay as record fields; no method reads them together any more.

## Batch 03-map-run-tier-bounds, attempt 1

- Cards 3 to 5 are committed; the map run now ends in one `unmeasured` list, and `skipped` and `queued` are gone from its envelope.
- `dirtyModuleFile` reads `Fabric.CodeWorktreeChanges`: the fabric-vocabulary guard refused filtering `Fabric.Status` by side in gatecli.
- The card gate runs no tier1 test of internal/gatecli; check the map run with
  `lym gate test --tier 1 ./internal/gatecli -- -run 'TestGateTestMap|TestGateRedundant'`.
- A test config whose tier command names targets needs `{targets}` and `all_targets`.
