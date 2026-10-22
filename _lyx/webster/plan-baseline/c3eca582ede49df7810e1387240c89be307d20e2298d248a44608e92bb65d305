# Card 17 — wake-and-orch-owner-pause

**Edit:**
- `internal/hibernatecli#wakeReconcileFromReport`
- `internal/hibernatecli#TestWakeReconcileFromReport`
- `internal/hibernateengine#WakeRun`
- `internal/hibernateengine#holdRuns`
- `internal/hibernateengine#runStep`
- `internal/hibernateengine#TestWatch`
- `internal/hibernateengine#fakeWorld.pause`
- `internal/hibernateengine/doc.go`
- `internal/orchcli#orchCLI`
- `internal/orchcli#Command`
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli#TestStart_ReconcilesTheHubFirst`
- `internal/orchcli/doc.go`
- `internal/cleanupcli/doc.go`
- `cmd/lyx#newRoot`

**Create:**
- `internal/orchcli#InstallOwnerPause`

**Uses:**
- `internal/fabricengine#EntryOwnedElsewhere`
- `internal/pauserecord#OwnerCaller`
- `internal/tendercli#PauseOwnedElsewhere`
- `internal/boardengine#SetOwners`
- `internal/boardengine#ReadOwners`
- `internal/machinename#Name`

**Intent:**
D4's callers of the owner outcome that hold a run: the hibernate wake and `lyx orch start`; cleanup only reports it.
`wakeReconcileFromReport` maps each owned_elsewhere entry, which always has a local pair since the reconcile classifies only recorded local pairs, onto a `WakeRun` with the new `Owner` field set.
`holdRuns` pauses such a run through `Deps.Pause` with `Caller` `pauserecord.OwnerCaller`, so a slug entry already standing adds none, and `runStep` words its notice step "<slug> is owned by machine <owner>: `lyx adopt <slug>` from this machine's prime takes it over here"; a failed owner pause is named, as any failed pause is, with "the next wake or `lyx orch start` pauses it again", never with `lyx tender pause`, whose entry would carry no owner label.
The pause holds a run whose child holds the parent lock once its batten releases that lock (card 12's `OwnerHeld` release).
`InstallOwnerPause` stores the process's owner pause, which `cmd/lyx`'s `newRoot` pre-run installs as `tendercli.PauseOwnedElsewhere`, since `tendercli` imports `orchcli` and the reverse import would cycle; `Command` copies it onto `orchCLI`'s new `pauseOwned` field.
`orchCLI` also gains `claimPrime func() (claimed, lostTo string, err error)`, which `Command` fills over `boardengine.ReadOwners`, `boardengine.SetOwners` with `Prime` and `machinename.Name`, writing only when the board records no prime owner; a nil `claimPrime` or `pauseOwned`, as a hand-built start harness has, is skipped.
`startCmd`, after its reconcile, first calls `claimPrime` (D2's first act), reporting the claim, a lost claim with the other machine's name, or a failure under the reconcile warnings without refusing; it then, before it touches any strand, pauses each owned_elsewhere entry of the report through `pauseOwned`, reports the paused slugs with their owners under a new `owner_pauses` envelope key and a failed or unwired pause under the reconcile warnings naming "re-run `lyx orch start`, which pauses it again", and still never refuses; the start prompt's reconcile note carries the summary's owned-elsewhere lines.
`cleanupcli`'s doc states that `lyx cleanup` only reports an owned_elsewhere entry, through the summary, and tears no such pair down, since the reconcile's teardown covers ended entries alone (card 6's row pins it).
`hibernateengine`'s and `orchcli`'s docs document the owner pause.
`fakeWorld.pause` records the request's `Caller` on the entry it adds, falling back to `PauseCaller` when the request carries none.
`TestWakeReconcileFromReport` gains a row: an owned_elsewhere entry becomes a run with its owner.
`TestWatch` gains rows: a wake whose reconcile reports a run owned elsewhere adds a pair pause with the owner label, lifts the daemon's own entry, and its notice names the owner and `lyx adopt`; a failed owner pause is named with the next wake or orch start.
`TestStart_ReconcilesTheHubFirst` gains rows: a hub with no prime owner calls the claim once and reports it; a lost claim reports the other machine; a failing claim warns and the launch goes on; with a fake `pauseOwned`, an owned_elsewhere entry is paused once, named in `owner_pauses`, and the launch goes on; a failing pause warns naming the re-run of `lyx orch start` and the launch goes on; with no `pauseOwned` wired the entry is named under the warnings and the launch goes on.

**ImpactSummary:** the hibernate wake, orch start and the root pre-run gain the owner pause; cleanup changes doc only.

**Commit:** 17: wake and orch start pause a run owned elsewhere
