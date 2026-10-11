# Card 10 — discussion-marker-fresh-spawn

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#discussionWriteEntry`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe/entries_discussionwrite_test.go`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/wiring_test.go`
- `internal/testkit/envkit#FullEnv`
- `internal/battencli/wire_seams_test.go`

**Uses:**
- `plan:internal/loomengine#DiscussionReadyMarker`
- `plan:internal/loomengine#RemoveDiscussionReadyMarker`

**Intent:**
Make the ready marker never stale (decision record D7): Go removes a marker left by an earlier writer spawn before a fresh spawn, after the attach probe has proved no live writer exists, so an attach to a live writer and a relaunch of a dead one keep it.

`Env` gains the field `PrepareDiscussionSpawn func() error`, documented as the closure every fresh Discussion-Write spawn runs after the attach probe and before the spawn, read by the `DiscussionWrite` entry; it arrives as a closure for the same reason `DiscussionSpec` does.
`discussionWriteEntry` requires it through `requireSeam` and composes the producer's fresh-spawn preparation: the closure runs first, then, when the gate holds an enabled `parent-review` entry, the round preparation as today; the preparation returns an empty amendment and the first error.
Its header comment states the order.
`newTestEnv` fills the seam with a closure returning nil.
`envkit.FullEnv` fills it the same way, beside `DiscussionSpec`, so `TestFullEnv_FillsEverySeam` stays green and the full loom-recipe builds in `internal/shedbuild` and `internal/loomrecipe` that start from `FullEnv` pass the new `requireSeam`.
`wire_seams_test.go`'s `intentionallyNil` table in `internal/battencli` gains `"PrepareDiscussionSpawn": "loom-only seam, batten has no discussion segment"` beside `DiscussionSpec`, since batten's `wire` leaves the field nil.
`entries_discussionwrite_test.go` extends its existing top-level tests rather than adding one, since the package sits at its budget row: the construction-failures table gains a missing-seam row, and the parent-review round test gains cases proving a fresh spawn runs the closure (a counter or a file the closure removes) and an attach does not, with and without a parent-review gate.

`loomCLI.wire` sets `PrepareDiscussionSpawn` to a closure calling `RemoveDiscussionReadyMarker(location)`, whichever producer is configured, since removing an absent marker is a no-op.
`wiring_test.go` extends the wiring test that asserts the discussion seams, pinning `RemoveDiscussionReadyMarker` through its one caller: a marker planted at `DiscussionReadyMarker(loc)` is gone after the closure runs; running the closure again, with the marker absent, returns nil; and with a regular file planted where the marker's parent directory belongs, the closure returns the error.
No new top-level test is added, so no budget row moves.

**ImpactSummary:** `Env` gains one required seam; loom's production builder in `loomcli`, the `shedrecipe` fixture and `envkit.FullEnv` fill it, batten's builder leaves it nil and its seam table says so, and the entry's fresh-spawn order is otherwise unchanged.
