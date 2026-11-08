```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. For every change a card makes, I looked for callers, tests, comments and docs outside the members that card names.

**Findings**

1. **MEDIUM, Card 11 (D8 line routing): re-sign arrows missing on `outbound.refused` and `outbound.reportStale`.**
   - Card 11's Intent posts the dropped/refused line "to that message's channel" and the stale line "to the prime's channel".
   - Today both methods take `setup Setup` and post to `setup.ChannelID` (`internal/slackengine/outbound.go:136`, `:184`, `:197`, `:141`).
   - After the change the setup holds no channel, so each method needs a channel argument in place of `setup`. The same probably applies to `outbound.post` and `outbound.postMessages`, which pass `setup` through (`outbound.go:115`, `:152`).
   - Card 11 lists all four under `**Edit:**` without the arrow. Either add arrows with the new heads, or state that the signatures stay and say where the channel comes from.

2. **LOW, Card 2 / Card 1 (D11, `OpenConnection`): comments that describe a bot-token-only `API` are not listed.**
   - Card 2 builds an `API` with the app-level token, but `internal/slackengine#API` ("calls Slack's Web API with one bot token", `api.go:33`) and the `api.go:1` header still describe a bot-token-only client.
   - Neither card lists `internal/slackengine#API` or `internal/slackengine#NewAPI`.

3. **LOW, Cards 5 and 6: file-name constant comments go stale.**
   - `internal/slackengine#TokenFileName` ("the machine's bot token file", `geometry.go:13`) is unlisted, though Card 5 makes the file hold two tokens and the app id.
   - `internal/slackengine#SetupFileName` ("the hub's channel and operator record", `geometry.go:15`) is unlisted, though Card 6 makes it operator-only.
   - Card 4 edits `Geometry` but not these consts.

4. **LOW, Card 12 (two loops become one outbound loop plus wakes): `backoff.go` comments go stale.**
   - The file header and `internal/slackengine#sharedFloor` describe "the relay's two loops" and "both loops" (`backoff.go:1-2`, `:18`).
   - Card 12 deletes `runInbound`, so those comments become false. No card lists `sharedFloor` or the file.

5. **LOW, Card 7 (last-healthy stamp): the `starter.ensure` doc comment goes stale.**
   - The comment says the relay is judged by "the last successful Slack call minus the spawn time" (`start.go:163-164`).
   - Card 7 rewrites that meaning on `lifetime` and `RelayState` but does not list `internal/slackengine#starter.ensure`.
   - Card 5 lists `starter.ensure` only for the token check, and it comes before Card 7, so it does not cover the rewording.

6. **LOW, Cards 5, 10, 11 and 12: file-header comments of rewritten files go stale.**
   - The affected headers:
     - `credentials.go:1` ("the machine's bot token file")
     - `inbound.go:1` ("the hub's Slack channel")
     - `outbound.go:1` ("posts … to the channel")
     - `relay.go:1-3` ("runs the inbound and outbound loops")
   - These headers are no member glyph, so listing the members does not cover them. Each card's Intent should say it rewrites its file's header.

7. **LOW, Card 6 (D10: setup names only the operator): `template.yaml`'s comment goes stale.**
   - The comment says "the channel, the operator and the token are never config, since `lym slack setup` and `lym slack login` write them" (`internal/slackengine/template.yaml:1`).
   - After Card 6, setup writes no channel. No card lists the file.

8. **LOW, Card 4 (`Geometry` gains `Machine` and `SocketLockPath`): `hubgeom/doc.go` sits under `Uses:` though the card makes it incomplete.**
   - Its `SlackGeometry` paragraph names only the token file in the credential directory (`internal/hubgeom/doc.go:23-26`).
   - It now also needs the socket lock there and the machine name. Move it to `Extends`.

9. **LOW, Card 10 (`RelayDeps.LiveHubs` added): `newRelayRun` is not listed on Card 10.**
   - `internal/slackengine_test#newRelayRun` builds `RelayDeps` by field name (`relay_test.go:95-104`). Card 10 lists `TestRunRelay` but not this helper, which only Card 12 edits.
   - Between Cards 10 and 12 the relay tests run with a nil `LiveHubs` that the keeper calls on its join path.

10. **LOW, Card 12 (D2 wake loop): the file holding the wake loop is not named.**
    - `wakeLoop` and its on-disk fallback are unbounded waits.
    - The Intent says only that `relay.go` stays on `unboundedWaitFiles` (`cmd/lym/daemonwakeups_test.go:26-58`). It does not say whether `wakeLoop` goes in `relay.go`.
    - If it lands in a new file, that file is an unlisted registration. The card should name the file, or add it to the list.

**Outside this lens**
- Card 8's `writeHint` creates its temp file in `.lym/slack/`, not through `lymtemp` (`PATTERN-lym-files`: "temp under `lymtemp`"). The decision record mandates that placement, so the plan should record the departure.

**Tool calls**
- Grepped the repository for stale Slack wording (`slack setup`, `last_slack_ok`, answer hint, `ReadToken`/`WriteToken`), for `//lym:seam` markers, and resolved the `Uses` glyphs: my context lacked uses outside the two packages and any seam markers.
- Resolved `UserStateDir`, `RecordFileName` and `machinename.Name` and searched the reference docs: my context lacked whether those glyphs and any CLI reference docs exist.
- Read `internal/hubgeom/doc.go` lines 18-30 and the leaf-package and `machinename` imports: my context lacked what the `hubgeom` doc says about `SlackGeometry`, and whether `hubgeom` may import `machinename`.
