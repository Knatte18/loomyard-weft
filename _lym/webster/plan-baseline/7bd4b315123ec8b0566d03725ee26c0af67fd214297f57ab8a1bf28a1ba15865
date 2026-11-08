# Card 3 — channel-names

**Create:**
- `internal/slackengine#ChannelName`
- `internal/slackengine#ArchivedName`
- `internal/slackengine#fitName`
- `internal/slackengine_test#TestChannelName`

**Intent:**
Name each orch's channel as D4 says, and the dated name a retired channel takes before its archive.

`ChannelName` returns `<shortname>-<machine>-<orch>`, where `<orch>` is `prime` for an empty slug (the hub's own orch), the slug for a beam, and `prime_beam` for a beam whose slug is `prime`.
Each character of the machine name outside Slack's channel grammar (lowercase letters, digits, `-` and `_`) becomes `-`; the machine name is lowercased first, as `machinename.FromHostname` already does.
The shortname and slug grammars produce only legal characters, so they pass unchanged.
`fitName` cuts a name over Slack's 80 characters: it keeps the first 73 characters and appends `-` and the first six hex digits of the SHA-256 of the whole name, 80 in all; a name of 80 or fewer passes unchanged.
`ChannelName` returns `fitName` of the joined name.

`ArchivedName` returns `fitName` of `<name>-<UTC date>`, the date as `2006-01-02`, for n 1, and `<name>-<UTC date>-<n>` for n 2 and on;
the operator-added entry of the decision record asks for `-2`, `-3` and on after a rename refused with `name_taken`.

Tests, tier0, `TestChannelName`, one table: `ly`, `neve` and the empty slug give `ly-neve-prime`; slug `payments` gives `ly-neve-payments`; slug `prime` gives `ly-neve-prime_beam`;
a machine name `neve+box 1` maps to `neve-box-1`; a name of exactly 80 characters passes unchanged and one of 81 ends in `-` and six hex digits at 80, the digits being SHA-256 of the 81-character name;
`ArchivedName("ly-neve-payments", 2026-11-08, 1)` is `ly-neve-payments-2026-11-08`, n 2 adds `-2`, and a base name of 79 characters is cut to 80 with the hash suffix.
