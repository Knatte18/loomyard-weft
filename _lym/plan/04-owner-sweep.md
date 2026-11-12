# Card 4 — The per-hub owner sweep

**Create:**
- `internal/machinecli/sweep.go`
- `internal/machinecli/sweep_test.go`
- `plan:internal/machinecli#hub` -> `type hub struct`
- `plan:internal/machinecli#sweptHubs` -> `func sweptHubs(d deps, cwd string) ([]hub, error)`
- `plan:internal/machinecli#hubSweep` -> `type hubSweep struct`
- `plan:internal/machinecli#sweepHubs` -> `func sweepHubs(d deps, hubs []hub, name string, replace []string) (sweeps []hubSweep, clean bool)`
- `plan:internal/machinecli#recoveries` -> `func recoveries(name string) []string`
- `plan:internal/machinecli#TestSweptHubs` -> `func TestSweptHubs(t *testing.T)`
- `plan:internal/machinecli#TestSweepHubs` -> `func TestSweepHubs(t *testing.T)`

**Edit:**
- `plan:internal/machinecli#deps`
- `plan:internal/machinecli#productionDeps`

**Extends:**
- `internal/machinecli/doc.go`

**Uses:**
- `internal/boardengine#SetOwners`
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#OwnerWrite`
- `internal/boardengine#OwnerWritePrime`
- `internal/hubregistry#Live`
- `internal/hubregistry#Entry`
- `internal/standalonestate#UserStateDir`
- `internal/fabricengine#BoltDir`
- `internal/fabricengine#NewBolt`
- `internal/fabricengine#Bolt.HasUnpushed`
- `internal/fabricengine#ReadShortname`
- `internal/lymcwd#ResolveWorktree`
- `internal/lymcwd#ErrNotAGitRepo`

**Intent:** Add the owner sweep of D6 as functions the set path of card 5 calls, with its per-hub report and the recovery lines.
No command reaches them on this card, so no file or owner is written by a run of the verb yet.

`deps` gains these seams, and `productionDeps` fills them:

- `RegistryHubs func() ([]hubregistry.Entry, error)`: `hubregistry.Live` over `standalonestate.UserStateDir`, as `internal/slackcli` reads it.
- `CwdHub func(cwd string) (hubPath string, ok bool, err error)`: `lymcwd.ResolveWorktree(cwd)`, which sets no cwd gate, so any directory of a worktree answers.
  `lymcwd.ErrNotAGitRepo` reads as no hub.
  A resolved hub path counts only when `fabricengine.ReadShortname` finds a shortname record in its Bolt directory, the test `hubregistry.Live` applies.
- `HasUnpushed func(boardDir string) (bool, error)`: `fabricengine.NewBolt(boardDir).HasUnpushed`.
- `SetOwners func(boardDir string, req boardengine.OwnerRequest) (boardengine.OwnerWrite, error)`: `boardengine.SetOwners`.

`hub` names one hub to sweep: its path, its shortname when the registry gave one, and its Bolt directory, `fabricengine.BoltDir` of the path.
`sweptHubs` returns every hub `RegistryHubs` lists, then the hub `CwdHub` answers when no listed hub has its path, compared as cleaned paths.
A failed registry read is returned as it is, since `hubregistry` names its way forward.

`sweepHubs` makes, for each hub in order, one `HasUnpushed` read and then one `SetOwners` call with `{Machine: name, Replace: replace}`:
no slug, no prime and no `Takeover`.
`replace` is this machine's former names, and the caller hands in no empty name and never `name` itself.
A hub's outcome never stops the later hubs. Each `hubSweep` carries the hub, an outcome, and what the outcome needs:

- `failed`: `SetOwners` returned an error, as on an unreachable origin or a refused push.
  It carries the error and the way forward, a re-run of `lym machine name <name>`.
  The same holds for a failed `HasUnpushed` read, and the write is then not attempted.
- `lost`: the write returned entries in `OwnerWrite.Lost`.
  It carries each entry with the owner that stands, `OwnerWritePrime` shown as the prime, and the entries it recorded.
  Its way forward is the re-run, which takes back an entry whose standing owner is a former name;
  for an entry under another name it says the entry stays that machine's and `lym adopt` takes it back.
- `swept`: the write recorded entries and lost none; it carries them.
- `pushed`: the write recorded nothing, lost nothing, and the Bolt was ahead of origin before it:
  the write only pushed a commit an earlier run left unpushed.
- `nothing`: the write recorded nothing, lost nothing, and the Bolt was not ahead: nothing to sweep.

`clean` is false when any hub is `failed` or `lost`; card 5 exits non-zero on it.
`hubSweep` marshals to the JSON the verb prints, with snake-case keys.

`recoveries` returns the fixed lines the verb prints after a rename, each naming one recovery of D6's window bound:

- a run paused as owned elsewhere: after its hub's sweep the dobby's probe lifts its own owner pause and drives the run;
  a pause the probe does not lift is lifted with `lym tender resume <slug>`;
- a hub whose sweep committed but did not push: a re-run of `lym machine name <name>` pushes the commit;
- a Finalize halted by its owner check: `lym loom resume` in the run's pair;
- a worktree a dobby or orch skipped or refused as owned elsewhere: `lym beam start <slug>`, or `lym beam start root` for the prime;
- lift any pause with `lym tender resume` only after that hub's sweep reports done, since an earlier lift leaves pairs unpushed;
- a hub neither recorded nor holding the working directory is not swept: run the verb from inside it.

Tests, in `internal/machinecli/sweep_test.go`, tier0 and parallel, over fake `deps` that record their calls and touch no hub:

- `TestSweptHubs`: registry hubs alone; the cwd's hub added; the cwd's hub already listed, kept once; a cwd in no hub; a registry error returned.
- `TestSweepHubs`: one row per outcome above, asserting the whole `hubSweep` and `clean`.
  A further row puts a failing hub first and asserts the later hub's request and outcome, so a failure stops no other hub.
  Every row asserts the request each hub got: the name, the replace set, no slug, no prime and `Takeover` false.
  The row set also asserts `recoveries(name)` names each command above with the name filled in.

`internal/machinecli/doc.go` gains the sweep: the hubs it covers, the one write per hub, the outcomes and the bound on unswept hubs.

**ImpactSummary:** Adds unexported sweep functions and four seams to `internal/machinecli`; no command calls them yet, so no behavior of `lym machine name` changes.

**Commit:** 4: machine: add the per-hub owner sweep
