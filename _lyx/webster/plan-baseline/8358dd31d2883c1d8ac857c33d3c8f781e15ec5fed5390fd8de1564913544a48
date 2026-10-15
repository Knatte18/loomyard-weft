# Card 4 — offline-record-readers

**Create:**
- `internal/shuttleengines#RecordReaders`
- `internal/shuttleengines#readsOf`
- `internal/shuttleengines#transcriptsAdapter`
- `internal/shuttleengines#transcriptsAdapter.LocateSessions`
- `internal/shuttleengines#transcriptsAdapter.ReadSession`
- `internal/shuttleengines#pricerAdapter`
- `internal/shuttleengines#pricerAdapter.PriceMessage`
- `internal/shuttleengines#instructionsAdapter`
- `internal/shuttleengines#instructionsAdapter.InstructionFiles`
- `internal/shuttleengines#TestRecordReaders`
- `internal/shuttleengines#readingEngine`
- `internal/shuttleengines#readingEngine.LocateSessions`
- `internal/shuttleengines#readingEngine.ReadSession`
- `internal/shuttleengines#readingEngine.PriceMessage`

**Edit:**
- `internal/shuttleengines/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `pattern/PATTERN-shuttle-provider-seam.md`

**Uses:**
- `internal/shuttleengines#registry`
- `internal/shuttleengines#SessionNamers`
- `internal/shuttleengines#forksWithoutAuditEngine`
- `internal/shuttleengine#Engine`
- `internal/shuttleengine#InstructionFileLister`
- `internal/shuttleengine#RecordReads`
- `internal/shuttleengine#TranscriptReader`
- `internal/shuttleengine#MessagePricer`

**Intent:** `internal/shuttleengines` hands out every registered engine's read capabilities for offline record reading, built without a shuttle config (D6).

`RecordReaders` builds one engine per registry entry with `shuttleengine.Config{}` and `configengine.Dirs{}`, as `SessionNamers` does, and returns `readsOf` over them.
`readsOf` maps each engine name to a `shuttleengine.RecordReads` whose `Transcripts`, `Pricer` and `Instructions` are set only when the engine implements `shuttleengine.TranscriptReader`, `shuttleengine.MessagePricer` and `shuttleengine.InstructionFileLister` respectively, and nil otherwise.
Each set field holds an adapter, one per capability and not one per engine: `transcriptsAdapter`, `pricerAdapter` and `instructionsAdapter` each keep the capability value in an unexported field and forward their one or two methods to it.
The adapter's dynamic type is the adapter, so no type assertion on a value in the set recovers a `shuttleengine.Engine` that can start, open, send to or stop a run; the set skips nothing a run could use, because it exposes reading alone.

`TestRecordReaders` pins both halves:
- `RecordReaders()["claude"]` has all three capabilities set, and none of the three values type-asserts to `shuttleengine.Engine`; the claude entry's `Instructions.InstructionFiles()` equals the engine's own list, showing the adapter forwards.
- `readsOf` over a map holding `forksWithoutAuditEngine{}` gives that name an entry whose three fields are nil.
- `readsOf` over a map holding a test-local `readingEngine`, an embedded `shuttleengine.Engine` whose `LocateSessions`, `ReadSession` and `PriceMessage` return fixed answers, gives adapters that return those answers, so every adapter is shown to forward.

The package's row in `cmd/lyx/testdata/test-budget.yaml` rises by one for the new top-level test.
`doc.go` gains a paragraph on `RecordReaders`: the zero config, the per-capability adapters and why they hide the engine.
`pattern/PATTERN-shuttle-provider-seam.md` gains a bullet beside the `SessionNamers` one: an engine's record-reading capabilities reach a reader outside the engines only through `shuttleengines.RecordReaders()`, as adapters that expose reading alone; the `PATTERN.md` entry line is unchanged.

**ImpactSummary:** additive: a new constructor and three unexported adapters in the registry package, whose allowlisted imports are unchanged, and one budget row raised.

**Commit:** 4: shuttleengines hands out each engine's read capabilities behind adapters
