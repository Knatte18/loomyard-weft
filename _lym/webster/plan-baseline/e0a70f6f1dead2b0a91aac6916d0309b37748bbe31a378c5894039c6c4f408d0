# Card 21 — sync-status-bar

**Extends:**
- `internal/fabriccli#Command`

**Edit:**
- `internal/reedengine#bindingLines`
- `internal/reedengine#Engine.bindingLinesLocked`
- `internal/reedengine#Engine.pinBindingsLocked`
- `internal/reedengine#statusFormatSessions`
- `internal/reedengine#TestPinBindingsLocked_PinsTheToggleLinesOnlyWhenEverythingResolves`

**Create:**
- `internal/pushledger#SyncStatus`
- `internal/pushledger_test#TestSyncStatus`
- `internal/fabriccli#SyncStatusVerb`
- `internal/fabriccli#syncStatusCmd`
- `internal/reedengine#syncStatusCommandOption`
- `internal/reedengine#syncStatusFormat`
- `internal/reedengine#composeSyncStatusCommand`

**Uses:**
- `internal/fabriccli/doc.go`
- `internal/reedengine/doc.go`
- `internal/pushledger/doc.go`
- `internal/fabriccli#addWeftVerbs`
- `internal/reedengine#composeSlackStateCommand`
- `internal/reedengine#Geometry`
- `internal/pushledger#List`
- `internal/pushledger#LastClear`
- `internal/pushledger#Entry.Owed`
- `internal/fabricengine#Config.PushTiming`
- `internal/pushledger#Update`
- `internal/reedengine#TestBar_LineTwoShowsTheCheckMarkBeforeTheClock`

**Intent:** D4: every session bar of a hub shows `sync: <minutes>m · <seconds>s`.

`pushledger.SyncStatus` renders the field: the minutes since the hub last had no owed target, `LastClear`, and 0 while no entry is `Owed`;
the seconds the longest interval in force across owed entries, or `base` while nothing is owed.
An entry holding only `HeldRecords` or `Dropped` is left out of both, since it is held, not owed (D3).
A refused target's entry stands, so its minutes keep counting (D3).
`syncStatusCmd` is the hidden `lym fabric sync-status --hub <dir>` verb, registered in `Command` with the internal audience annotation as `teardownChildCmd` is.
It loads the hub's `fabric.push_interval_s` through `Config.PushTiming` over the prime's config, prints `SyncStatus`'s line, reads the ledger, writes nothing, spawns nothing and touches no network,
so a tmux job may run it every redraw; a failed read prints `sync: ?` and exits zero, so the bar never shows an error.

The field is a `#()` job, as the Slack toggles are:
`composeSyncStatusCommand` composes `<lym> fabric sync-status --hub <hub>` over the geometry's `HubPath`, doubling every `#` as `composeSlackStateCommand` does;
`bindingLines` gains `syncStatus`, `bindingLinesLocked` composes it, and `pinBindingsLocked` pins it in the server-global `syncStatusCommandOption` when the hub's server starts, unsetting it when the line does not resolve.
`statusFormatSessions` shows `syncStatusFormat` before the check mark, so every session of the hub draws it; an unset option draws nothing.
`bar.go`'s file comment names the new server-global option and the field on line 2 beside the Slack state line.
`TestBar_LineTwoShowsTheCheckMarkBeforeTheClock` reads line 2 with the option unset, so the field draws nothing there and the test stands as it is.

Tests, tier0, adding under a second to one tier0 run.
`TestSyncStatus` pins `sync: 0m · 60s` for an empty ledger, and `sync: 7m · 240s` for two owed entries with intervals of 60 s and 240 s, the clear time written through `pushledger.Update`'s `now` seven minutes before;
a refused target's entry alone keeps the minutes counting; an entry holding only `HeldRecords`, and one holding only `Dropped`, each alone reads `sync: 0m · 60s`; and an entry that does not decode returns an error, which the verb prints as `sync: ?`.
`TestPinBindingsLocked_PinsTheToggleLinesOnlyWhenEverythingResolves` gains the sync-status line among the lines pinned when everything resolves, and its unset when the binary does not resolve.
The bar's rendering in every hub session bar is checked by review.

**ImpactSummary:** Adds one hidden verb and one bar field with its pinned line; the bar's other fields and options are unchanged.
