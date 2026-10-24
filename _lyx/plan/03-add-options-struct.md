# Card 3 — add-options-struct

**Edit:**
- `internal/fabricengine#AddOptions`
- `internal/fabricengine#Topology.add`

**Create:**
- `internal/fabricengine#AddOptions.syncOptions`

**Intent:**
Turn `AddOptions` from an alias of `SyncOptions` into a struct of its own, so `Add` can take the new options of cards 4 and 5 without widening `SyncOptions`, which every records helper takes.
The struct keeps `SyncOptions`' three fields under the same names, `SkipGit`, `SkipPush` and `BackoffSleep`, as plain fields rather than an embedded struct, so every existing composite literal such as `fabricengine.AddOptions{SkipPush: true}` keeps compiling unchanged.
`syncOptions` returns the three as a `SyncOptions`, and `Topology.add` passes `opts.syncOptions()` wherever it handed `opts` to a records helper (`CommitRecordsPaths`).
This card is a pure refactor with no behaviour change; the existing `Add` tests cover it.

**ImpactSummary:** Type change confined to add.go; every caller passes a composite literal whose field names are kept, so none needs editing.
