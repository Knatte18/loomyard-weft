# Card 21 — tender-every-orch

**Edit:**
- `internal/tenderengine#Deps`
- `internal/tenderengine#ReviveReport`
- `internal/tenderengine#Revive`
- `internal/tenderengine#reviveOrch` -> `func reviveOrch(deps Deps) ([]OrchRevive, error)`
- `internal/tenderengine#OrchRevive`
- `internal/tenderengine#orchCase`
- `internal/tenderengine#isOrchStrand`
- `internal/tenderengine#TestRevive`
- `internal/tenderengine#TestPause`
- `internal/tendercli#HubDeps`
- `internal/tendercli#orchReviveTarget` -> `func orchReviveTarget(worktree *lyxcwd.Location) (*creelcli.GoClient, string, error)`
- `internal/hibernateengine#refuseCaller`
- `internal/hibernateengine#TestSubmit`
- `internal/orchengine#Watcher.settledWithoutInterrupt`
- `internal/orchengine#Watcher.orchNetworkOperationInFlight`
- `internal/orchengine#TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine`
- `internal/orchengine#Watcher`
- `internal/orchcli#orchCLI.watchCmd`
- `internal/tenderengine#env.deps`
- `internal/tenderengine/doc.go`

**Create:**
- `plan:internal/tenderengine#Orch` -> `type Orch struct`
- `plan:internal/orchcli#HubOrchs` -> `func HubOrchs(location *lyxcwd.Location) ([]*lyxcwd.Location, error)`
- `plan:internal/orchengine#Watcher.SetOrchName` -> `func (w *Watcher) SetOrchName(name string)`

**Extends:**
- `internal/orchcli/doc.go`

**Uses:**
- `plan:internal/orchcli#OrchStrandOf`
- `internal/orchcli#PrimePaths`
- `internal/fabricengine#ReadOriginFor`

**Intent:**
D8: the readers that today read only the prime's orch read every orch of the hub, the prime's and each beam's; this card covers tender's pause, revive and status reads, hibernate's caller rule and the orch watcher's own pause checks.
`orchcli.HubOrchs` lists the prime's location and each local pair whose origin record carries the beam mark, the worktrees an orch may run in.
`tenderengine.Deps` replaces `OrchStrand`, `OrchRevivePending`, `InjectOrchRevive` and `OrchWatcherLive` with `Orchs func() ([]Orch, error)`, one `Orch` per worktree recording a strand: its strand name, the `Pair` whose session holds it, whether its watcher is live, and its own revive-pending and inject seams.
`reviveOrch` revives each orch stalled on an API error by the existing rules and `ReviveReport` carries `orchs`, one `OrchRevive` per orch naming its strand; `orchCase` reads a hub pause as delegated when any orch's watcher is live; `isOrchStrand` recognises any orch of the hub (the orch role, whatever its slug), so a beam orch running `lyx tender pause` reads as the orch that ran it.
`tendercli.HubDeps` fills `Orchs` over `HubOrchs`, each strand by `OrchStrandOf` and each revive target by `orchReviveTarget` over that worktree.
`hibernateengine.refuseCaller` admits any orch strand, a beam's included; the orch watcher's `settledWithoutInterrupt` and `orchNetworkOperationInFlight` match an orch caller or marker by its own strand name rather than by an empty slug, so a beam's watcher judges its own orch under a hub pause.
`Watcher` gains `orchName`, the agent name of the orch it watches, set through `SetOrchName`; the zero value keeps today's match of an orch name with an empty slug, so a hand-built watcher reads as the prime's.
`orchCLI.watchCmd` sets it to the acting worktree's orch name (`<shortname>:orch` in the prime, `<shortname>:<slug>:orch` in a beam).
`env.deps`, the tender test harness, fills `Orchs` in place of the four removed seams, and the tender package doc, which reads "the orch" as the prime's, is rewritten to name every orch of the hub.
Test rows: `TestRevive` revives a stalled beam orch beside a healthy prime orch, reporting both; `TestPause` reads a beam orch's live watcher as delegated and a beam orch caller as the orch that ran the pause; `TestSubmit` accepts a beam orch caller and still refuses a driver; the watcher's hub-pause test settles on a beam orch's own strand.

**ImpactSummary:** Tender's orch seams become a list and its revive report a list; tendercli is their one production filler, and the orch-name rules widen from the prime's orch to every orch.
