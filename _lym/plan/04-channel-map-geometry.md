# Card 4 — channel-map-geometry

**Create:**
- `plan:internal/slackengine#ChannelMapFileName` -> `const ChannelMapFileName = "channels.json"`
- `plan:internal/slackengine#InboxDirName` -> `const InboxDirName = "inbox"`
- `plan:internal/slackengine#SocketLockFileName` -> `const SocketLockFileName = "slack-socket.lock"`
- `plan:internal/slackengine#ChannelMap` -> `type ChannelMap struct`
- `plan:internal/slackengine#ChannelEntry` -> `type ChannelEntry struct`
- `plan:internal/slackengine#ReadChannelMap` -> `func ReadChannelMap(path string) (ChannelMap, error)`
- `plan:internal/slackengine#UpdateChannelMap` -> `func UpdateChannelMap(path string, mutate func(*ChannelMap) error) error`
- `plan:internal/slackengine#ChannelMap.OrchOf` -> `func (m ChannelMap) OrchOf(channelID string) (string, bool)`
- `plan:internal/slackengine#ChannelEntry.Advance` -> `func (e *ChannelEntry) Advance(ts string)`
- `plan:internal/slackengine#ChannelEntry.AdvanceThread` -> `func (e *ChannelEntry) AdvanceThread(thread, ts string)`
- `plan:internal/slackengine#ChannelEntry.ThreadStart` -> `func (e ChannelEntry) ThreadStart(thread string) string`
- `plan:internal/slackengine#ChannelEntry.PruneThreads` -> `func (e *ChannelEntry) PruneThreads(now time.Time)`
- `plan:internal/slackengine_test#TestChannelMap` -> `func TestChannelMap(t *testing.T)`

**Edit:**
- `internal/slackengine#Geometry`
- `internal/hubgeom#SlackGeometry`
- `internal/hubgeom#TestSlackGeometry`
- `internal/slackengine_test#newGeometry`

**Extends:**
- `internal/hubgeom/doc.go`

**Uses:**
- `internal/machinename#Name`
- `internal/slackengine#compareTS`
- `internal/slackengine#slackTimestamp`
- `internal/slackengine#recordLockPath`

**Intent:**
Add the hub's channel map (D5) and the geometry of the push route's files (D1, D2, D5).

`Geometry` gains four fields: `Machine`, the `internal/machinename` name; `ChannelMapPath`, `.lym/slack/channels.json`; `InboxDir`, `.lym/slack/inbox/`;
and `SocketLockPath`, the machine socket lock `slack-socket.lock` beside `slack.json` in the credential directory.
`hubgeom.SlackGeometry` fills them from `machinename.Name()`, returning that function's error unchanged for a machine whose name cannot be read, and from the constants above.
That error already names its way forward, giving the machine a hostname, and it is a propagated read of `os.Hostname`, so no row reaches it.
`internal/hubgeom/doc.go`'s `SlackGeometry` paragraph gains the machine name and the socket lock beside the token file in the credential directory.
`TestSlackGeometry` (tier0) pins the four new fields, the machine read through `machinename.Name()` in the test itself;
`newGeometry` in the slackengine tests sets them under its temp directory, with `Machine` `neve`.

The map is `ChannelMap{Channels map[string]ChannelEntry}`, keyed by the orch's creel name (`ly:orch`, `ly:payments:orch`).
A `ChannelEntry` holds `ID`, `Name`, `Cursor` (the newest top-level timestamp handled), `Threads` (thread timestamp to newest reply handled),
`ThreadFloor`, `Operator` (the operator it invited, empty for none) and `Archived`.
`ReadChannelMap` reads the file through `internal/state`; a missing file reads as an empty map, and a decode failure is an error naming the file and wrapping `state.ErrDecode`.
That error's way forward is moving the file aside, after which the relay ensures the hub's channels anew and adopts its own channels on `name_taken` (card 9).
`UpdateChannelMap` is the one writer: a read-modify-write under the map's `internal/state` lock, so a key an older binary does not know survives, and no caller makes a Slack call inside `mutate`.
`OrchOf` returns the orch whose entry holds the channel id, archived or not.

Cursors only move forward: `Advance` and `AdvanceThread` keep the later of the stored and the given timestamp, by `compareTS`, so two relays' concurrent updates never move one back.
`ThreadStart` returns the thread's cursor, or, with none, the later of the thread's timestamp and `ThreadFloor`, so a reply the channel handled is not forwarded again.
`PruneThreads` drops each thread cursor older than 24 hours before now and raises `ThreadFloor` to 24 hours before now, so a reply no read caught before the prune is not recovered (D2's bound).

Tests, tier0, `TestChannelMap`, one table over a temp file:
a missing file reads empty; an entry written reads back, and a key the type does not know survives an update;
twenty goroutines each `Advance` the same entry to their own timestamp through `UpdateChannelMap`, and the stored cursor is the newest;
`AdvanceThread` to an older timestamp leaves the thread cursor; `PruneThreads` drops a 25-hour-old thread, keeps a one-hour-old one and raises the floor to 24 hours before now;
`ThreadStart` of a thread with no cursor answers the later of its timestamp and the floor; a map file holding `{` is an error naming its path and moving it aside.

**ImpactSummary:** `Geometry` gains fields every builder sets, `hubgeom.SlackGeometry` and the slackengine test helper; the map is new and has no reader until later cards.
