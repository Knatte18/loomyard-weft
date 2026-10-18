# Card 7 — tender-orch-revive-inject

**Create:**
- `internal/tenderengine#orchInjected`
- `internal/tenderengine#orchPending`
- `internal/tendercli#tenderProgram`
- `internal/tendercli#orchReviveTTL`
- `internal/tendercli#revivePending`
- `internal/tendercli#TestRevivePending`

**Edit:**
- `internal/tenderengine#Deps`
- `internal/tenderengine#OrchRevive`
- `internal/tenderengine#Revive`
- `internal/tenderengine#reviveOrch`
- `internal/tenderengine#env`
- `internal/tenderengine#env.deps`
- `internal/tenderengine#TestRevive`
- `internal/tendercli#HubDeps`
- `internal/tendercli#reviveCmd`
- `internal/tenderengine/doc.go`
- `internal/tendercli/doc.go`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/tenderengine#orchQueued`
- `internal/tenderengine#orchAlreadyQueued`

**Intent:**
Tender's orch revive becomes a checked text inject through creel (decision record D7).

`Deps` replaces `QueueOrchRevive` with `OrchRevivePending func() (bool, error)`, which reports whether a revive inject of tender's own to the orch strand stands with no outcome yet, and `InjectOrchRevive func(line string) (int64, error)`, which injects the line and answers its id; `OrchWatcherLive` stays for the pause sequence.
`reviveOrch` reads the orch's state as today and acts only while it reads `idle-stalled` with cause `api-error`; it then runs inside a `pauserecord.Update` of its own that changes no entry, the lock that serialises tender's task-session revive sends: under it a live hub entry answers `paused`, a pending revive answers `pending` and injects nothing, and otherwise `InjectOrchRevive` injects `ReviveLine` and the result is `injected` with its id.
So two concurrent revives inject at most once.
`OrchRevive` drops `Conditional` and `WatcherLive` and gains `InjectID`; its results are `injected`, `pending`, `paused` and `skipped`.

`HubDeps` wires both seams over `creelcli.NewProgramClient(prime, tenderProgram)`, signing as `TENDER` with no strand claim, on the orch strand's name from `orchcli.PrimeOrchStrand`:
`OrchRevivePending` lists `GoClient.Log` from tender's program to that strand and answers `revivePending` of the entries: true when an inject's `posted` event has no `injected`, `unconfirmed` or `expired` event for its id after it;
`InjectOrchRevive` calls `GoClient.Inject` with `orchReviveTTL`, one minute.
The orch revive runs only in the hub scope, which `tendercli` already refuses to an agent's pane, so the claim comes from the orch's pane or from outside any strand pane, both of which creel accepts.
Bound: a pause, resume or watcher record write now also waits at most one revive's creel lookup and inject, each capped by the Go client's call timeout; the condition is read before the inject instead of at typing, so a session that leaves the stall within the TTL can still get the line typed, which starts one extra turn, and the one-minute TTL caps that window.

Tests:
`env` replaces `orchQueued` and `orchAlreadyQueued` with a mutex-guarded fake creel journal, safe for the concurrent row: `env.deps` answers `OrchRevivePending` from it and `InjectOrchRevive` records the line and marks a pending inject with a fresh id, and each seam answers a scripted error when one is set.
`TestRevive` also gains rows: an error from `OrchRevivePending` and one from `InjectOrchRevive` are returned naming the orch revive, with nothing injected for the first.
`TestRevivePending`, a new untagged test in `internal/tendercli`, feeds `revivePending` log entries: none, a posted inject with no outcome (pending), and a posted inject followed by each of `injected`, `unconfirmed` and `expired` (not pending); `cmd/lyx/testdata/test-budget.yaml` raises `internal/tendercli` by one.
`TestRevive`'s orch row asserts: a stalled orch gets one inject of `ReviveLine` reported `injected` with its id, and no other prime session is read or reported; a second revive while that inject has no outcome reports `pending` and injects nothing; once the fake settles the inject a revive injects again; a live hub entry reports `paused` and injects nothing; a slug revive reports no orch part; an orch that reads another state is `skipped` with the state.
A new row runs two revives concurrently over a stalled orch and asserts exactly one inject, which the record lock serialises.
`internal/tenderengine/doc.go` restates the orch revive and its lock bound, `internal/tendercli/doc.go` and `reviveCmd`'s help say the orch gets a checked inject reported with its id, and `docs/overview.md`'s tender row says the same.

**ImpactSummary:** Tender's orch revive path and its report shape only; task-session revives, pause and resume are unchanged, and the one engine test that asserted the old notice fields changes.
