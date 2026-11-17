# Card 10 — Remove the adopt record package

**Delete:**
- `internal/adoptrecord/record.go`
- `internal/adoptrecord/watch.go`
- `internal/adoptrecord/doc.go`
- `internal/adoptrecord/record_test.go`
- `internal/adoptrecord/watch_test.go`
- `internal/adoptrecord/testmain_test.go`

**Intent:**
After cards 5, 6 and 9 no package imports `internal/adoptrecord`: the orch watcher and the dobby read the hold, `lym beam status` shows the hold, and adopt writes no record.
The package and its tests go whole, so no code path reads or writes `adopt.json` or `adopt.lock`; a file a retired worker left in the adopt scratch directory stays unread.
A `grep -rn adoptrecord --include=*.go` over the tree returns nothing after this card.

**ImpactSummary:** one package removed with no remaining importer.

**Commit:** 10: Remove the adopt record package
