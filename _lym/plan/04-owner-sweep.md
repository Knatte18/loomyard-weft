# Card 4 — owner-sweep

**Create:**
- `plan:internal/machinecli#hubSweep` -> `type hubSweep struct`
- `plan:internal/machinecli#sweepHub` -> `func sweepHub(hubPath, machine string, replace []string) hubSweep`
- `plan:internal/machinecli#recoveries` -> `func recoveries() []string`
- `plan:internal/machinecli#TestNameSweep` -> `func TestNameSweep(t *testing.T)`

**Edit:**
- `plan:internal/machinecli#setName`
- `plan:internal/machinecli#nameResult`
- `//internal/machinecli/doc.go`

**Uses:**
- `internal/boardengine#SetOwners`
- `internal/boardengine#OwnerRequest`
- `internal/fabricengine#Bolt.HasUnpushed`
- `internal/hubforge#CopyHub`
- `internal/hubforge#CloneSecond`

**Intent:** After the file write, `setName` runs the owner sweep (D6) over the hubs `sweptHubs` listed, so owners recorded under this machine's former names follow the rename.
The replace set is the written file's `names` other than the new name, empty entries excluded, and so holds the old name and the hostname rule's name.

`sweepHub` per hub, the others going on whatever one does:
1. reads `fabricengine.NewBolt(fabricengine.BoardDir(hub)).HasUnpushed()` before the write;
2. calls `boardengine.SetOwners(fabricengine.BoardDir(hub), boardengine.OwnerRequest{Machine: name, Replace: replace})`, with `Takeover` unset;
3. reports `failed` with the write's error and the way forward, a re-run of `lym machine name <name>`;
4. reports each `OwnerWrite.Lost` entry with the owner that stands and the same re-run as its way forward, adding that `lym adopt` takes back an entry whose standing owner is another name;
5. reports `swept` with the recorded slugs, `pushed` when nothing was recorded but the Bolt was ahead before the write and the write returned no error, or `nothing to sweep`.
Any failed hub or lost entry makes the verb exit non-zero after every hub is swept.
A run with the name already set runs the sweep again, so a re-run finishes a hub an earlier run failed.

`nameResult` gains the per-hub sweep reports and, once any hub swept or pushed, the recoveries `recoveries` lists, from D6's window bound:
- a run under an owner pause: the dobby's probe lifts it after the hub's sweep and runs a full pass; one it does not lift is lifted with `lym tender resume <slug>`; a hub that committed without pushing needs the re-run of `lym machine name <name>`;
- a Finalize halted by its owner check: `lym loom resume` in the run's pair;
- a dobby or orch that skipped or refused a worktree as owned elsewhere: `lym beam start <slug>`, or `lym beam start root` for the prime;
- lift any pause with `lym tender resume` only after that hub's sweep reports done.
`//internal/machinecli/doc.go` gains the sweep, its bounds (a hub neither registered nor holding the working directory is not swept; the way forward is running the verb from inside it) and its recovery output.

`TestNameSweep` is tier1 and copies its hubs through `hubforge.CopyHub`, covering as rows:
- after `osl46` on a machine named `osl-1046`, every open beam project and the prime owned by `osl-1046` read `osl46` on origin on each swept hub, an owner only origin held (pushed from a `hubforge.CloneSecond` clone) included, while a done entry and an owner under another name keep theirs;
- `osl46` over a hand-written file holding only `{"name":"osl46"}` records the hostname rule's name in `names` and sweeps owners recorded under it;
- with one hub's origin unreachable, that hub is reported failed with the re-run as its way forward, the other hub is swept, and the verb exits non-zero; with the origin restored, a re-run with the same name sweeps it;
- a hub with no former-name owner reports nothing to sweep;
- the output after a rename carries the recoveries.

**ImpactSummary:** Extends only the new machinecli package's set path and its doc; no other package changes.
