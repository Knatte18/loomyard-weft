# Card 13 — anchor-relay-restart

**Edit:**
- `internal/fabriccli#anchorDeps`
- `internal/fabriccli#anchorResult`
- `internal/fabriccli#anchorResult.fields`
- `internal/fabriccli#moveAnchor`
- `internal/fabriccli#hubAnchorDeps`
- `internal/fabriccli#anchorFake`
- `internal/fabriccli#anchorFake.deps`
- `internal/fabriccli#TestChangeAnchor`
- `internal/fabriccli/doc.go`
- `internal/fabriccli#anchorCommand`

**Uses:**
- `plan:internal/slackengine#EnsureRelay`
- `plan:internal/hubgeom#SlackGeometry`

**Intent:** Start the relay again once a prime anchor change ends, whether it lands or fails and whatever dobbies it starts again (D3).
A prime move's `EndDobbies` ends every `runproc.HubOwner` process but a push or a sync, the relay included, and a failure with the change record standing starts no dobby, so the dobby restart alone cannot bring the relay back.

- `anchorDeps` gains `StartRelay func() error`, which `hubAnchorDeps` fills with `slackengine.EnsureRelay(hubgeom.SlackGeometry(acting.HubPath), primeName)`, the prime's name `hubAnchorDeps` already resolves, whose `not-configured` outcome is no error.
- In `moveAnchor`, once `EndDobbies` has been called for a move whose slug is `deps.Prime`, whether it returned an error or not, since it may have ended the relay before it failed, every return after it calls `StartRelay`: on the success path its error sets `anchorResult.RelayStartError`, which `fields` reports as `relay_start_error`; on each `fail` path, the standing-record and unreadable-record cases included, its error joins the returned error as "start the Slack relay again: <cause>; run `lym slack start`".
  A beam's move, and a prime move that failed before `EndDobbies`, start no relay, since nothing ended it.
- `internal/fabriccli/doc.go`'s anchor-move text and `anchorCommand`'s `Long`, which list what a move restarts and reports, name the relay restart after `EndDobbies` and `relay_start_error` beside `dobby_start_error`.
- `anchorFake` gains a scripted relay error and `deps` logs `start-relay`.
- `TestChangeAnchor` gains rows: a prime move that lands logs `start-relay` after the dobby restart; a prime move whose apply fails logs `start-relay`; a prime move that fails with the change record standing starts no dobby and logs `start-relay`; a prime move whose `EndDobbies` fails (`endErr`) logs `start-relay`; a prime move whose parent lock fails (`lockErr`), before `EndDobbies`, logs no `start-relay`; a beam's move logs no `start-relay`; a relay start error on a landed move reports `relay_start_error` naming `lym slack start` and still succeeds.

**ImpactSummary:** One new dep field called on the prime's post-end paths of moveAnchor and one new result field, both named in `fabriccli/doc.go` and the anchor verb's `Long`; beam moves and existing rows are unchanged, and only hubAnchorDeps wires the field.
