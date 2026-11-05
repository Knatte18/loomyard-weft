# Card 15 — raddle-store-replay

**Create:**
- `plan:internal/boardengine#raddleStorePath` -> `func raddleStorePath(path string) (sketch string, ok bool)`

**Edit:**
- `internal/boardengine#FabricReplay.Claims`
- `internal/boardengine#FabricReplay.Apply`
- `internal/boardengine#replayRun.applyDesignFile`
- `internal/boardengine#TestFabricReplay`
- `internal/boardengine/doc.go`

**Uses:**
- `internal/boardengine#replayRun.addEntry`
- `plan:internal/boardengine#RaddleStoreDir`

**Intent:** D10: the store reaches origin with the filed entries through the board's Bolt replayer.
`raddleStorePath` reports whether a Bolt-root-relative path is a file directly in a store directory, `raddle/<sketch>/<name>` with no deeper segment, and returns its Sketch slug.
`Claims` claims such a path beside the board's root files.
`applyDesignFile` admits a store path as well as a plain file name, writing or removing it by the same rule; a path of any other shape is still refused, and a store file origin does not hold lands, so a concurrent board write from another machine, which writes no file of this Sketch's store, replays beside it.
`Apply` sets aside a store directory's files, reporting them with the entry, when it refuses or loses the add of any entry that directory's `store.json` records as filed, so origin never holds the done marker of D9 step 3 without the entries it marks.
The report names the Sketch and that its staged store and part records stay in the raddle run's own committed `_lym/raddle/` records, from which the operator re-files the parts that did not land with `lym board upsert` once the clash is settled; the replay itself files nothing.
`TestFabricReplay` gains rows: a commit adding a store directory and its filed entries replays both onto origin; the same onto an origin a concurrent write from another machine changed (both land); a commit whose filed entry's add origin already holds under that slug (lost), and one whose filed entry's add the replay refuses (its `depends_on` names an entry origin does not hold), each setting the store files aside and reporting them; a store path one level deeper (not claimed).

**ImpactSummary:** The replayer claims one new path shape and sets aside a store whose entries do not land; every existing claimed path replays as before.
