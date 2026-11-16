# Card 4 — machine-name-sweep

**Create:**
- `plan:internal/machinecli#HubSweep` -> `type HubSweep struct`
- `plan:internal/machinecli#sweep` -> `func sweep(deps Deps, name string, replace []string, hubs []string) []HubSweep`
- `plan:internal/machinecli#recoveries` -> `func recoveries(name string) []string`
- `internal/machinecli/sweep_integration_test.go`
- `plan:internal/machinecli#TestName_SweepsRealHubs` -> `func TestName_SweepsRealHubs(t *testing.T)`

**Edit:**
- `plan:internal/machinecli#Deps`
- `plan:internal/machinecli#productionDeps`
- `plan:internal/machinecli#setName`
- `plan:internal/machinecli#SetResult`
- `plan:internal/machinecli#TestName`
- `internal/machinecli/doc.go`

**Uses:**
- `internal/boardengine#SetOwners`
- `internal/boardengine#OwnerRequest`
- `internal/fabricengine#Bolt.HasUnpushed`
- `internal/hubforge#NewHub`
- `plan:internal/machinecli#runWith`

**Intent:**
Run D6's owner sweep after the file write, so owners recorded under this machine's former names follow the rename on every swept hub, and report each hub's outcome with the recoveries D6's window bound lists.

`Deps` gains `HasUnpushed func(hub string) (bool, error)` and `SetOwners func(hub string, req boardengine.OwnerRequest) (boardengine.OwnerWrite, error)`.
`productionDeps` fills them from the hub's Bolt `HasUnpushed` and `boardengine.SetOwners` over the hub's board directory, matching card 3's `ReadOwners` wiring.

`setName`, after the file write or the no-change rewrite, calls `sweep` over the same `hubsToSweep` list with the new name and the replace set: this machine's names from card 3's step 4 other than the new name, empty entries left out.
For each hub, `sweep` reads `HasUnpushed` first, then makes one `SetOwners` call with `OwnerRequest{Machine: name, Replace: replace}`, `Takeover` unset, and records a `HubSweep`:
- a write error: the hub, the error and the way forward `lym machine name <name>`; the other hubs go on;
- entries in `OwnerWrite.Lost`: each entry with the owner that stands and the same way forward, adding `lym adopt` for an entry whose standing owner is not one of this machine's names;
- `Recorded` non-empty: `swept` with the entries;
- nothing recorded: `nothing to sweep`, or `pushed` when `HasUnpushed` read the Bolt ahead before the write.
  A `HasUnpushed` error leaves the hub reported by its write alone.
The verb exits non-zero when any hub has a write error or a `Lost` entry, after printing every hub's result.
Bound: a hub neither live in the registry nor holding the working directory is not swept; `doc.go` names running the verb from inside it as the way forward.

`SetResult` gains the hubs' sweeps and `recoveries(name)`, the lines D6 lists, printed after any rename that changed the name.
They cover a run under an owner pause (the dobby's probe lifts its own pause after the sweep; otherwise `lym tender resume <slug>`; a hub that committed but did not push needs a re-run of `lym machine name <name>`).
They cover a Finalize halted by its owner check (`lym loom resume` in the run's pair) and a worktree skipped as owned elsewhere (`lym beam start <slug>`, or `lym beam start root` for the prime).
They end with lifting a pause with `lym tender resume` only after that hub's sweep reports done.

`TestName` (tier0) gains rows over the fake hubs, whose `SetOwners` records the request it got and answers from the row:
the sweep sends the former names and the hostname rule's name as `Replace`, without the new name or an empty entry; a failing hub is reported with the re-run while the next hub is swept and the exit is non-zero; a `Lost` entry is reported per entry and exits non-zero; a hub with nothing recorded and `HasUnpushed` true reports `pushed`; and a re-run with the name already set writes no file yet sweeps again.
A rename's envelope carries the recovery lines.

`TestName_SweepsRealHubs` in `internal/machinecli/sweep_integration_test.go` (tier1) builds two hubs with `hubforge.NewHub`, records under the old name a beam project's owner and the prime owner on one, and an owner under another machine's name on the other.
It runs `lym machine name osl46` through `runWith` with a machine file under `t.TempDir()` naming `osl-1046` and those two hubs as the live list.
It asserts each origin's board records `osl46` for the old name's owners, keeps the other name's owner, and the file's `names` read `osl-1046`, the hostname name the seam hands in, then `osl46`.
Read `internal/hubforge`'s exports for the board fixture and how its other callers seed owners.

`internal/machinecli/doc.go` gains the sweep, its per-hub outcomes, the recoveries and D6's bounds.

**ImpactSummary:** the verb from card 3 now writes owners on every swept hub's Bolt; no other package changes.

**Commit:** 4: lym machine name sweeps owners across a rename and names the recoveries
