# Card 2 — shedrecipe-block-registry

**Create:**
- `internal/shedrecipe#EngineBlock`
- `internal/shedrecipe#blockEntry`
- `internal/shedrecipe#blockProducer`
- `internal/shedrecipe#blockProducer.Call`
- `internal/shedrecipe#BlockGate`
- `internal/shedrecipe#BlockGates`
- `internal/shedrecipe#BlockGateAlways`
- `internal/shedrecipe#blockGates`
- `internal/shedrecipe#LookupBlockGate`
- `internal/shedrecipe#BlockGateNames`
- `internal/shedrecipe#TestBlockGates`

**Edit:**
- `internal/shedrecipe#registry`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#TestSimpleEntries_HappyPath`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe_test#coverageGuardAllowedUnreachableEngines`
- `PATTERN.md`

**Extends:**
- `internal/shedrecipe/doc.go`
- `pattern/PATTERN-shed-recipe-registry.md`

**Intent:** Add the block's producer and the closed table of block gate checks to the one registry package (D3, D5), in a new file `internal/shedrecipe/entries_block.go`.

- `EngineBlock` is the engine name a block row carries once expanded (PATTERN-named-constants); the registry maps it to `blockEntry`, keyed by the constant.
- `blockEntry` takes no Config key (`configRejectUnknown`, as `stubEntry` does) and returns a `blockProducer`, whose `Call` returns `shedengine.Done` with an empty `OutputPointer` at once: the block row's Done marks entering the block.
- `BlockGate` builds a block row's `shedengine.EntryGate` from the told `Env`; `BlockGates` is the closed table type keyed by check name, and `blockGates` its one value, holding `BlockGateAlways`, whose gate always holds with an empty reason.
- `LookupBlockGate` returns the check a name selects and refuses any name the table lacks with `shedrecipe: unknown block gate %q; way forward: name one of: <BlockGateNames joined by ", ">`; `BlockGateNames` returns the names sorted in a fresh slice, as `Names` does.
  shedbuild's Build reaches the refusal for a recipe row whose `block_gate` names no check, asserted there (card 5, `TestBuild_Errors`).

The registry's doc comment and `registry.go`'s file header comment, which both say the implementer table is the one table outside the registry, name `BlockGates` as the second closed table.
`doc.go` gains a `# The Block row` section describing the producer and the block gate table; the pattern background file gains a `## Block gates` section saying a block gate check is a row of `BlockGates` in `internal/shedrecipe` and never code per recipe.

`PATTERN.md`'s `PATTERN-shed-recipe-registry` entry is rewritten to the declared text:

```markdown
- `PATTERN-shed-recipe-registry` — Adding producer, recipe row or block gate: one `Constructor` or `BlockGates` map in `internal/shedrecipe`, bar `Implementer`'s; a row's value looked up by key, a side action a `Command` row, a row off by one config key its `entry_gate`, one whose off reads more its own call. — [background](pattern/PATTERN-shed-recipe-registry.md)
```

Tests, all tier0:
- `TestSimpleEntries_HappyPath` gains a `Block` row: the entry builds with an empty Config and its producer returns Done with no error.
- `TestBlockGates` (new): `LookupBlockGate(BlockGateAlways)` builds a gate that holds with an empty reason, and `BlockGateNames` lists `always`.
- `TestNames`' exact list gains `Block`, and `minimalEntries` gains `{newTestEnv, emptyConfig}` for it so `TestEveryEntry_RejectsUnknownConfigKey` covers its unknown-key refusal.
- `coverageGuardAllowedUnreachableEngines` gains `Block`, since no embedded recipe names it until card 6, which removes the entry.

**ImpactSummary:** One registry entry and one new closed table in `internal/shedrecipe`; no recipe names `Block` yet, so no build changes, and the guard tables that enumerate the registry are updated on this card.
