# Card 8 — discussion-ready-marker

**Create:**
- `plan:internal/loomengine#DiscussionReadyMarker` -> `func DiscussionReadyMarker(l *lyxcwd.Location) string`
- `plan:internal/loomengine#RemoveDiscussionReadyMarker` -> `func RemoveDiscussionReadyMarker(l *lyxcwd.Location) error`

**Edit:**
- `internal/loomengine#TestDiscussionPaths`

**Uses:**
- `internal/loomengine#discussionDirName`

**Intent:**
Add the ready marker's two public accessors of decision record D7, which the twill spec (card 9) and the `loomcli` wiring (card 10) both use.

`DiscussionReadyMarker`, in `config.go` beside the other discussion path accessors, returns the marker path under the anchor's `lyxdirs.DotLyxDirName` at the subpath mirroring the discussion directory, `discussionDirName` then `decision-record.md.ready`, spelling neither lyx directory literal itself (`PATTERN-lyxdirs-single-declarer`, `PATTERN-durable-vs-ephemeral-state`).
Its doc comment says the marker is untracked state the twill writer creates once its record passes the self-check, and the condition its lens forks wait on.

`RemoveDiscussionReadyMarker`, beside it, removes the marker and treats a missing one as success;
its doc comment says it runs before every fresh Discussion-Write spawn, so a marker left by an earlier spawn never wakes a later spawn's forks.
Its behavior is pinned by the wiring test card 10 extends, which drives it through the closure that is its one caller.

`TestDiscussionPaths` asserts `DiscussionReadyMarker` sits under the anchor's `.lyx` at `discussion/decision-record.md.ready`, spelled through `lyxdirs.DotLyxDirName`.

**ImpactSummary:** Two new accessors with no callers until cards 9 and 10, and one assertion added to an existing table test.
