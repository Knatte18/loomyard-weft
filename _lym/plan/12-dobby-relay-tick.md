# Card 12 — dobby-relay-tick

**Create:**
- `internal/beamcli#TestHubDobbyDeps_PrimeTickEnsuresRelay`

**Edit:**
- `internal/beamcli#hubDobbyDeps`
- `internal/beamcli/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/beamcli#TestHubDobbyDeps_PrimeTidyShuttle`
- `internal/slackengine#EnsureRelayWhileAway`
- `internal/slackengine#ErrSpawnUnderTest`
- `internal/slackengine#WriteToken`
- `internal/slackengine#WriteSetup`
- `internal/slackengine#WritePresence`
- `internal/hubgeom#SlackGeometry`

**Intent:** Have each tick of the prime's dobby bring back a relay that died while the operator is away, without a terminal (D3).

In `hubDobbyDeps`'s prime branch (`geom.Slug == ""`), `deps.Tick` runs the hibernate carrier's tick as now and then `slackengine.EnsureRelayWhileAway(hubgeom.SlackGeometry(prime.HubPath), prime.WorktreeName)`, returning `false` and both errors joined; a beam's dobby gets no relay step.
`internal/beamcli/doc.go`, which says the prime's `Deps.Tick` is the tick of the `hibernateengine.Carrier`, says it runs the carrier's tick, then `EnsureRelayWhileAway`, and its Bound names the relay start beside the carrier; `docs/overview.md`'s beamcli line names the relay beside the hibernate request.
`EnsureRelayWhileAway` starts nothing while presence is `terminal`, so after `lym slack stop`, which sets `terminal` first, no tick restarts the relay; a hub with no Slack setup answers `not-configured` with no error, so every hub without Slack ticks as before.
A `gave-up` outcome, card 9's cap after five spawns in a row whose relays each lived under ten minutes, a spawn that fails to start included, is logged at Warn once per episode naming the relay log and `lym slack start`, and returns no error, so the tick goes on.
The dobby loop's own handling of a tick error, logging, stays as it is.

`TestHubDobbyDeps_PrimeTickEnsuresRelay` is a new tier1 test in `internal/beamcli/dobbydeps_integration_test.go`, on a `hubforge.CopyHub` prime as `TestHubDobbyDeps_PrimeTidyShuttle` builds it; it redirects the credential directory with `t.Setenv`, process-global state, so it does not run in parallel.
It writes the token, the setup file and the presence through slackengine at the hub's `SlackGeometry`, and rows: with presence `slack` and no relay live, the prime's `Tick` returns an error matching `slackengine.ErrSpawnUnderTest`, which shows the tick reached the spawn; with presence `terminal`, the state `lym slack stop` leaves, `Tick` returns false and nil; the beam's `Tick` stays nil.
`TestHubDobbyDeps_PrimeTidyShuttle` keeps its assertion that the prime's tick with no hibernate request and no Slack setup answers false and nil.

**ImpactSummary:** One closure in the prime branch of hubDobbyDeps grows by one call, which `beamcli/doc.go` and `docs/overview.md`'s beamcli line name; beams and hubs without Slack setup tick unchanged.
