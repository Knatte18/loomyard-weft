# Card 9 — pairteardown-composite

**Create:**
- `plan:internal/pairteardown#Teardown` -> `type Teardown struct`
- `plan:internal/pairteardown#New` -> `func New(prime *lyxcwd.Location) (*Teardown, error)`
- `plan:internal/pairteardown#Request` -> `type Request struct`
- `plan:internal/pairteardown#RemoveQuietWait` -> `const RemoveQuietWait = 2 * time.Minute`
- `plan:internal/pairteardown#Teardown.EndSession` -> `func (t *Teardown) EndSession(ctx context.Context, req Request) (SessionResult, error)`
- `plan:internal/pairteardown#Teardown.RemovePair` -> `func (t *Teardown) RemovePair(req Request) (fabricengine.RemoveResult, error)`
- `plan:internal/pairteardown#Teardown.Run` -> `func (t *Teardown) Run(ctx context.Context, req Request) (Result, error)`
- `internal/pairteardown/doc.go`
- `internal/pairteardown/teardown_test.go`
- `internal/pairteardown/teardown_integration_test.go`
- `internal/pairteardown/testmain_test.go`
- `internal/pairteardown/testmain_integration_test.go`

**Uses:**
- `plan:internal/fabricengine#Topology.RemoveRefusal`
- `plan:internal/fabricengine#ErrPairNotFound`
- `plan:internal/reedengine#EndSessionByName`
- `plan:internal/reedengine#Engine.MarkRetiring`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#WorktreePath`
- `internal/reedengine#Engine.Down`
- `internal/reedengine#Engine.Status`
- `internal/reedengine#StrandStatus`
- `internal/reedengine#LoadConfig`
- `internal/reedengine#ServerName`
- `internal/reedengine#SessionName`
- `internal/hubgeom#ReedGeometry`
- `internal/loomengine#IsDriverStrand`
- `internal/shedrun#RunLock`
- `internal/shedrun#ParkMarker`
- `internal/shedrun#SelfRunID`

**Intent:**
Implements D1–D4: the one pair-teardown composite both `lyx fabric remove` (card 10) and batten's Worktree-Teardown row (card 11) call.
It is a new package because `reedengine` imports `fabricengine`, so fabric cannot end a session itself without an import cycle.
It sits outside the Fabric Vocabulary owner set: identifiers, strings and comments say "pair" and "task worktree", and it never names a `RemoveResult` field whose identifier carries a side's name.
It passes `fabricengine.RemoveResult` through whole.

`New(prime)` wires the production substrates from the hub's prime location.
`Teardown` keeps each substrate behind an unexported function field (quiet probe, refusal probe, session end, removal, sleep, clock) so the untagged tests drive the sequence with fakes.
`Request` carries `Slug`, `Force`, `Remote`, `QuietWait time.Duration` and `RefuseWhenBusy bool`.

`EndSession(ctx, req)` runs three steps in order:
1. **Quiet wait (D2).** It probes until the pair's loom driver is quiet: the run lock (`shedrun.RunLock(task, shedrun.SelfRunID)`, probed non-blockingly and released at once) is free.
   In addition, the driver strand (`loomengine.IsDriverStrand` over `Engine.Status`) must be absent, dead, `Retiring`, or parked (`shedrun.ParkMarker` present).
   An absent reed session (`reedengine.ErrNoSession`) satisfies the strand clause only; the run-lock probe still runs whenever the task worktree is present.
   A pair whose task worktree is gone is quiet, by the bounded exception stated under Bound below.
   "Gone" means the task worktree path is not a registered linked worktree, the same test card 2 uses for `stray_path`.
   It polls on a fixed interval constant, with the attempt count capped from `QuietWait` and the interval (Live-Substrate Spawn Observability), until `QuietWait` is spent.
   On a spent bound with `RefuseWhenBusy`, it returns an error wrapping a new exported `ErrDriverBusy`, leaving the pair, session and strands untouched.
   That error names the live driver strand and the pair's attach command (`lyx reed attach`, run from the task worktree's anchor path), and says "retry once the driver has written its report, or end it with `lyx reed down` in the pair".
   On a spent bound without `RefuseWhenBusy` (batten's zero wait), it proceeds and reports `DriverWasLive` on the result.
2. **Refusal probe (D3).** `Topology.RemoveRefusal(prime, slug, force)`.
   A refusal is returned unchanged with the session and strands untouched.
3. **Session end (D4).** With the task worktree present it calls `Engine.Down` on an engine built from `hubgeom.ReedGeometry` and the task's reed config, which also tears down the hub's tmux server and socket after the last session.
   With the worktree gone it calls `reedengine.EndSessionByName` with `ServerName(hub)` and `SessionName(WorktreePath(prime, slug))`, using the reed config resolved from the prime's anchor, so no path under `<hub>/<slug>` is recreated.

`SessionResult` reports `Ended`, `AbandonedSession` (from `Down`) and `DriverWasLive`.

`RemovePair(req)` loads the hub's fabric config, builds the topology and calls `Topology.Remove(prime, slug, force, remote)`.
Its doc comment states it runs only after `EndSession` succeeded for the same request; `Run` and batten's one-row producer are its only sequencers.
`Run(ctx, req)` is `EndSession` then `RemovePair`, returning `Result{Session SessionResult; Removal fabricengine.RemoveResult}`; a failed `EndSession` never reaches `RemovePair`.
`RemoveQuietWait` is the `fabric remove` bound, a constant rather than config.

`doc.go` states the sequence and rationale, the quiet rule, the two caller policies, and that `Remove` repeats the probe's checks, so a refusal raised only by `Remove` means the pair changed between probe and removal and the session is already ended.

Bound: with the task worktree present, the composite never ends a non-quiet driver when `RefuseWhenBusy` is set.
Only a caller passing a zero wait without it (batten, after its own `driver_exit_grace_s`) ends one.
A parked driver is ended without refusal, since its park command already committed its records.

The one exception is a gone task worktree, on both caller paths: the session is ended by name with no wait, even when a driver strand in it may still be live.
Quiet cannot be probed there, since the run lock and reed's strand table both lived under the gone worktree, and a refusal would name `lyx reed down` in a pair that no longer exists.
The exception is bounded:
- it applies only when the task worktree path is not a registered linked worktree;
- it ends only the session named `SessionName(WorktreePath(prime, slug))`, by exact match;
- nothing such a driver writes after its worktree is gone can reach the archive, because its sync and commit resolve from the gone worktree and its writes are the recreated paths D4 exists to stop;
- the records already in the sibling worktree are committed before the archive by `Remove`'s pending-record commit (D6).

Tests:
- `teardown_test.go` (untagged, fakes):
  - A busy driver at the bound with `RefuseWhenBusy` returns `ErrDriverBusy` and never calls the refusal probe, session end or removal; the message names the strand, `lyx reed attach` and `lyx reed down`.
  - A driver turning retiring on a later probe proceeds after the counted sleeps.
  - A zero wait without `RefuseWhenBusy` proceeds and reports `DriverWasLive`.
  - A refusal probe error never ends the session.
  - A session-end error never removes.
  - `Run` calls quiet, probe, end and remove in that order.
- `teardown_integration_test.go` (hubforge pair, a real reed session in the pair with a stub driver strand):
  - A probe refusal for task-side dirtiness leaves the session and strands live.
  - A probe refusal for sibling-side changes outside the record pathspec leaves session, strands and sibling branch untouched.
  - After `Run`, the session is gone and `<hub>/<slug>` does not exist.
  - With the task worktree removed by hand and the session still up, `Run` ends the session by name and finishes the pair.
  - With a stop report committed on the sibling branch and the stub driver strand live, `Run` with `RefuseWhenBusy` waits; the strand is marked retiring through `Engine.MarkRetiring` during the wait, `Run` proceeds, and the archive tag contains the report.
- `testmain_test.go` (`!integration`) and `testmain_integration_test.go` call `gitkit.HermeticGitEnv` and run under `tmuxkit.Main`, in `internal/battencli`'s two-file shape.

**Commit:** `9: pairteardown composite ends the session before removing a pair`
