# Card 5 — Decode seed.json tolerantly

**Edit:**
- `internal/shedrun#decodeSeed`
- `internal/shedrun#ReadSeed`
- `internal/shedrun#ParseSeed`
- `internal/shedrun#WriteSeed`
- `internal/shedrun/doc.go`

**Delete:**
- `internal/shedrun#seedWire`
- `internal/shedrun#TestSeed_LegacyParentKey`

**Create:**
- `internal/shedrun#TestSeed_UnknownKey`

**Intent:**
D4: `decodeSeed` stops calling `DisallowUnknownFields` and decodes straight into a `Seed` with `json.Unmarshal`, so a seed another binary wrote with a key this one does not know decodes; malformed JSON is still an error.
`seedWire` and its `LegacyParent` field, which exist only so the strict decode accepts the retired `parent` key, are deleted; `decodeSeed` is their only user.
`WriteSeed`'s agreement check on recipe, driver and params stays, and an agreeing existing seed is still left as it is, so the unknown key survives; `shedrun` stays the sole declarer of `Seed` and the run-directory paths (`PATTERN-shed-run-directory`).

Doc comments: `decodeSeed`'s says it decodes data as a `Seed`, accepting an unknown key; `ReadSeed`'s and `ParseSeed`'s "decodes strictly so an unknown key is an error" becomes "accepts an unknown key, which `WriteSeed` never rewrites away"; `WriteSeed`'s gains that an agreeing seed is left byte for byte, unknown keys included.
`shedrun`'s package doc says a seed's retired `parent` key "still decodes; the key is discarded and never re-encoded" and that `ParseSeed` decodes "with the same strictness as ReadSeed"; the first becomes: a seed decodes with any key this binary does not know, the retired `parent` key included, and an agreeing `WriteSeed` leaves the file as it is; the second says `ParseSeed` decodes as `ReadSeed` does.

`TestSeed_LegacyParentKey` pins the retired `parent` key and that another unknown key is refused, both of which D4 replaces, so it is deleted and `TestSeed_UnknownKey` takes its place, reusing `writeLegacySeedFile`:
- a seed file carrying `"parent":"ab:hub"` and `"future":{"x":1}` reads through `ReadSeed` with recipe and driver decoded and no error;
- `WriteSeed` with an agreeing seed over that file returns nil and leaves the file's bytes unchanged.
A disagreeing seed is already pinned by `internal/shedrun#TestWriteSeed_RefusesDisagreeingSeed`.
The budget row for `internal/shedrun` stays, one test replacing one.

**ImpactSummary:** `ReadSeed`, `ParseSeed` and `WriteSeed` accept unknown keys; their callers in shed, loom and batten see no other change.

**Commit:** 5: decode seed.json tolerantly
