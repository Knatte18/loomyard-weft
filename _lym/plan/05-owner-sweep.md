# Card 5 — owner-sweep

**Edit:**
- `plan:internal/machinecli#setName`
- `plan:internal/machinecli#nameDeps`
- `internal/machinecli/doc.go`

**Create:**
- `plan:internal/machinecli#hubSweep` -> `type hubSweep struct`
- `plan:internal/machinecli#sweepHub` -> `func sweepHub(deps nameDeps, hub, name string, replace []string) hubSweep`
- `plan:internal/machinecli#recoveries` -> `func recoveries() []string`
- `plan:internal/machinecli#TestName_Sweep` -> `func TestName_Sweep(t *testing.T)`

**Uses:**
- `internal/boardengine#SetOwners`
- `internal/fabricengine#Bolt.HasUnpushed`
- `internal/fabricengine#NewBolt`
- `internal/fabricengine#BoardDir`
- `internal/hubforge#CopyHub`
- `internal/hubforge#CloneSecond`
- `internal/testkit/boardkit#InstallReplay`
- `plan:internal/machinecli#hubsToSweep`
- `plan:internal/machinecli#TestName_CollisionCheck`

**Intent:** After the file write, sweep every hub `hubsToSweep` returns, so each owner recorded under one of this machine's former names reads the new name (D6).
The sweep runs on every run that passes the collision check, the file written or not, so a re-run with the same name finishes a sweep a hub failed.

The replace set is the written file's non-empty `names` entries other than the new name.
`sweepHub` reads `fabricengine.NewBolt(fabricengine.BoardDir(hub)).HasUnpushed()` before the write.
It then makes one `boardengine.SetOwners` call on the hub's board with `Machine` the new name and `Replace` the replace set, `Takeover` unset, and returns a `hubSweep`.
The `hubSweep` holds the hub, the recorded entries, the lost entries with their standing owners, the error, and whether the Bolt was ahead before.
`nameDeps` gains `BeforeOwnerPush func(hub string)`, nil in production, which `sweepHub` hands to `OwnerRequest.BeforePush`, so a test can make a competing origin write win.
A test cannot reach `OwnerWrite.Lost` otherwise.

Each hub's report in the output:
- an error: the error, and the way forward, a re-run of `lym machine name <name>`; the other hubs go on;
- lost entries: each entry with the owner that stands, and the way forward, that re-run, which takes back an entry whose standing owner is a former name; an entry under any other name stays that machine's, and `lym adopt` takes it back;
- recorded entries: the entries swept;
- nothing recorded: nothing to sweep, or pushed when the Bolt was ahead before the write.

The verb exits non-zero when any hub reports an error or a lost entry.
When the run changed the name, the output carries `recoveries`, the lines D6's window bound lists:
- an owner pause lifts at the dobby's probe after the hub's sweep, else with `lym tender resume <slug>`;
- a hub whose sweep committed but did not push needs a re-run of `lym machine name <name>`;
- a Finalize halted by its owner check resumes with `lym loom resume` in the run's pair;
- a dobby or orch that skipped a worktree as owned elsewhere needs `lym beam start <slug>`, or `lym beam start root` for the prime;
- a pause is lifted with `lym tender resume` only after that hub's sweep reports done.

`internal/machinecli/doc.go` gains the sweep, its replace set, its per-hub reports and the recoveries, with D6's bounds.

`TestName_Sweep` is tier1, beside `TestName_CollisionCheck`, with hubs built under its temporary directory.
Its rows:
- after a rename, the open beam project and the prime recorded under the former name, one of them at origin only, read the new name at origin;
  a done beam project under the former name and a beam project under another name are untouched, and the output carries the recoveries;
- a hand-written `{"name": "osl46"}` file sweeps the owners recorded under the hostname rule's name;
- with two hubs, one whose origin URL is unreachable, the failing hub is reported with the re-run as its way forward and the other is swept, and the verb exits non-zero;
  after the URL is restored, a re-run with the same name sweeps it and exits 0 with no recoveries, the name unchanged;
- a competing origin write through `BeforeOwnerPush` reports the entry lost with its standing owner, and the verb exits non-zero;
- a hub with nothing under a former name reports nothing to sweep, and one whose Bolt held an unpushed commit reports pushed.

**ImpactSummary:** Extends the new verb with the owner sweep; it writes owners only through `boardengine.SetOwners` and only under this machine's former names.
