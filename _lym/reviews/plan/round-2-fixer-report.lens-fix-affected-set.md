```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set: I re-ran each finding's search to check the fix covers every member of the set, not only the members the review named.

Unchecked findings: none.

- **F1** (every path that adds a program sender to `programs`, the set of known mailboxes): I found four places that write to `programs`.
  - `fold.go:303` (`applyPosted`): the plan already keeps `CREEL` out here.
  - `fold.go:347` (the `applyCarried` posted branch): the fix now closes this one.
  - `fold.go:321` (`applyRead`): this needs a read by `CREEL`, and no verb makes one.
  - `verbs.go:231` (a program's first `Read`): this needs `CREEL` to call `Read` itself.

  The posted-inject paths return before the `programs` write at `fold.go:302`, and the carried-inject branch never writes `programs`.
  The new `TestJournal_Retention` row covers the carried path.
  The fix covers the whole set.
- **F2** (every row and sentence that names when a chase step lands): the fixer's own grep found only card 7 and the overview's `chase-clock-cadence`, and both now name tick counts.
  My grep of cards 08–19 found no remaining "running time", "after 15" or tick claims.

no findings

Tool calls:
1. Grepped `programs[` writers in `internal/creelengine` and tick and timing phrases in cards 08+, because my context held only `fold.go:340-360` and no search of the later cards.
2. Read `fold.go:296-325` and `verbs.go:225-236`, because my context lacked the bodies of the `applyPosted` inject branch, `applyRead` and `Read` that write `programs`.
