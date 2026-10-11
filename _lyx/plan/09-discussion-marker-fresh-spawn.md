# Card 9 — discussion-marker-fresh-spawn

**Create:**
- `plan:internal/loomengine#RemoveDiscussionReadyMarker` -> `func RemoveDiscussionReadyMarker(l *lyxcwd.Location) error`
- `plan:internal/loomengine#TestRemoveDiscussionReadyMarker` -> `func TestRemoveDiscussionReadyMarker(t *testing.T)`

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#discussionWriteEntry`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe/entries_discussionwrite_test.go`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/wiring_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/loomengine#DiscussionReadyMarker`

**Intent:**
Make the ready marker never stale (decision record D7): Go removes a marker left by an earlier writer spawn before a fresh spawn, after the attach probe has proved no live writer exists, so an attach to a live writer and a relaunch of a dead one keep it.

`RemoveDiscussionReadyMarker`, in `config.go` beside `DiscussionReadyMarker`, removes the marker and treats a missing one as success.
`TestRemoveDiscussionReadyMarker` is a new untagged test over a temp-dir location: an existing marker is gone afterwards, an absent one returns nil, and a marker under a regular file returns the error; `test-budget.yaml` raises `internal/loomengine` by one for it.

`Env` gains the field `PrepareDiscussionSpawn func() error`, documented as the closure every fresh Discussion-Write spawn runs after the attach probe and before the spawn, read by the `DiscussionWrite` entry; it arrives as a closure for the same reason `DiscussionSpec` does.
`discussionWriteEntry` requires it through `requireSeam` and composes the producer's fresh-spawn preparation: the closure runs first, then, when the gate holds an enabled `parent-review` entry, the round preparation as today; the preparation returns an empty amendment and the first error.
Its header comment states the order.
`newTestEnv` fills the seam with a closure returning nil.
`entries_discussionwrite_test.go` extends its existing top-level tests rather than adding one, since the package sits at its budget row: the construction-failures table gains a missing-seam row, and the parent-review round test gains cases proving a fresh spawn runs the closure (a counter or a file the closure removes) and an attach does not, with and without a parent-review gate.

`loomCLI.wire` sets `PrepareDiscussionSpawn` to a closure calling `RemoveDiscussionReadyMarker(location)`, whichever producer is configured, since removing an absent marker is a no-op.
`wiring_test.go` extends the wiring test that asserts the discussion seams, adding that a marker planted at `DiscussionReadyMarker(loc)` is gone after the closure runs.

**ImpactSummary:** `Env` gains one required seam; the one production builder in `loomcli` and the one test fixture in `shedrecipe` fill it, and the entry's fresh-spawn order is otherwise unchanged.
