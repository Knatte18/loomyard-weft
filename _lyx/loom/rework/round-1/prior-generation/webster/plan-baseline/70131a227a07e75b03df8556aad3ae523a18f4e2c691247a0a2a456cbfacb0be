# Card 13 — creelcli-delivery-adapter

**Create:**
- `internal/creelcli/delivery.go`
- `internal/creelcli/delivery_test.go`
- `internal/creelcli/testmain_test.go`
- `plan:internal/creelcli#hubDelivery` -> `type hubDelivery struct`
- `plan:internal/creelcli#worktreeOps` -> `type worktreeOps interface`
- `plan:internal/creelcli#newHubDelivery` -> `func newHubDelivery(hubPath string, engineFor func(shuttleengine.Config) shuttleengine.Engine) *hubDelivery`

**Edit:**
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Implement creel's `Delivery` seam over the hub's worktrees, reed's persisted state and shuttle (D1), the one place creel's keystrokes leave for a pane.
`hubDelivery.Strands` lists the hub's worktrees with `fabricengine.List`, skipping prunable ones, resolves each with `lyxcwd.ResolveWorktree`, and reads its reed state with `reedengine.LoadState(<anchor>/.lyx)`: every persisted strand is a `creelengine.Strand` (name, guid, worktree, slug empty for the prime).
A worktree that does not resolve fails `Strands` naming it, unlike `internal/tendercli`'s skip, since a skipped worktree's strands would read as gone to the removed-holder sweep (D15).
Each worktree is reached through `worktreeOps`, the four calls the adapter makes on it: `Status` (reed's), and `SessionIdle`, `Send` and `Interrupt` (a shuttle runner's).
`hubDelivery` holds one unexported seam field, `wire`, from a resolved worktree to its `worktreeOps`; `newHubDelivery` sets it to build what `internal/tendercli`'s `newShuttlePair` builds (the worktree's reed and shuttle config through `fabricengine.ConfigDirs` and `hubgeom.ReedGeometry`, a `reedengine.Engine` and a `shuttleengine.Runner` built with `engineFor(shuttleCfg)`), and a test replaces it with a stub answering each call as scripted.
Production passes `claudeengine.NewFromConfig` as `engineFor`, adding one hardwired construction site that `engine-registry` later folds in; `Runner.SessionIdle`, `Runner.Send` and `Runner.Interrupt` take record-less strands since card 5.
Liveness and pane pid come from that worktree's reed `Status`; `reedengine.ErrNoSession` reads every strand of that worktree dormant, and any other `Status` error fails `Strands` naming the worktree.
A failed worktree list, an unreadable reed state or a failed status is returned as an error naming the worktree and file, so creelengine never mistakes an unreadable strand for an absent one.
Errors map onto creel's sentinels: `shuttleengine.ErrSessionBusy` to `creelengine.ErrBusy`, `shuttleengine.ErrSubmissionNotLanded` to `creelengine.ErrNotLanded`, `shuttleengine.ErrNoRunningTurn` to `creelengine.ErrNoRunningTurn`; any other error passes through.
`Paused` reads `pauserecord.ReadCoverage` at the record path `hubgeom.PauseGeometry` gives, with the strand's slug, so a prime strand is covered by a hub entry only (D14); an unreadable record is returned as an error, and creelengine's worker then types nothing for that probe.
Worktree wirings are built at each call, so a pair added or removed is seen at the next sweep.
`testmain_test.go` stays untagged and calls only `gitkit.HermeticGitEnv()` and `tmuxkit.Main`, so every tier of the package has its `TestMain`.
`delivery_test.go` carries `//go:build integration`, so `TestHubDelivery` runs in tier 2: its `hubforge` hub and `fabricengine.List`'s `git worktree list` are spawns the untagged tier forbids (`PATTERN-test-speed`).
`TestHubDelivery` (`delivery_test.go`), over a hub fixture from `hubforge` with a pair, reed states seeded on disk and `wire` replaced by the stub: strands of the prime and of the pair are listed with slugs, liveness and pane pids from the stub's `Status`; a corrupt `reed.json` fails `Strands` naming the file; a stub `Status` answering `reedengine.ErrNoSession` reads that worktree's strands dormant, and one answering another error fails `Strands` naming the worktree; a worktree directory whose `.git` entry is removed fails `Strands` naming it; stub `Send` and `Interrupt` errors wrapping `shuttleengine.ErrSessionBusy`, `ErrSubmissionNotLanded` and `ErrNoRunningTurn` come out as `creelengine.ErrBusy`, `ErrNotLanded` and `ErrNoRunningTurn`, and another error unchanged; `Paused` answers true under a hub entry for a prime strand and under the pair's entry for a pair strand, false for a prime strand under a pair entry, and fails on an undecodable pause record.
`cmd/lyx/testdata/test-budget.yaml` gains an `internal/creelcli: 1` row.

**ImpactSummary:** New creelcli package file implementing the seam; no existing caller; one new budget row.
