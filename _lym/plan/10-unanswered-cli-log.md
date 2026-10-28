# Card 10 — unanswered-cli-log

**Create:**
- `plan:internal/creelcli#isChaseWait` -> `func isChaseWait(d time.Duration) bool`

**Edit:**
- `internal/creelcli#postCmd`
- `internal/creelcli#logEntryFields`
- `internal/creelcli#TestCreelCLI`

**Extends:**
- `internal/creelcli/doc.go`

**Uses:**
- `plan:internal/creelengine#OutcomeUnanswered`
- `internal/creelcli#newHookRig`
- `internal/creelcli#rig.startDaemon`

**Intent:**
The CLI shows a chased message's path and a waiting post's `unanswered` outcome (D10, D11).
`post --wait-reply` whose outcome is `unanswered` prints its envelope with `outcome: unanswered` and the message under `unanswered`, and exits 1 with the note "the message got no reply: creel chased its holder and recorded it unanswered; way forward: read lym creel log --id <id> for its path, then ask again or decide without the answer".
`log`'s entry fields gain `chase` on a `chased` entry; the `unanswered` entry's reason already rides `reason`.
`internal/creelcli/doc.go` names the outcome and its exit.

`TestCreelCLI` gains rows over the in-process daemon and its fake clock, with `isChaseWait` matching the chase timer's waits, from one minute to eight, the values of card 7's unexported `agedPersistInterval` and `chaseProbeCeiling`, as card 7's own `isChaseWait` does, so they stay apart from the probe and sweep waits; this row keeps the holder live, so its chase waits stay at one minute (the existing `isSweepWait` is unchanged; no existing row posts a marked message, so no chase wait reaches it): a `post --wait-reply` from `newHookRig`'s strand to itself, in its own goroutine, the strand reading the message with `creel read`, then the daemon clock advanced by whole minutes until the outcome; `post --wait-reply` exits 1 with `outcome: unanswered`, and `log --id <id>` lists `posted` with `expect_reply`, `read`, two `chased` entries with `chase` 1 and 2, and `unanswered` with its reason.

**ImpactSummary:** One new outcome branch in post's waiting path and one log field; every other post and log output is unchanged.
