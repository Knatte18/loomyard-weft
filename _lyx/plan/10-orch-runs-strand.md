# Card 10 — orch-runs-strand

**Create:**
- `plan:internal/orchcli#runsStrandName` -> `const runsStrandName = "runs"`
- `plan:internal/orchcli#runsStrandAddSpec` -> `func runsStrandAddSpec(cmd string) reedengine.AddSpec`
- `plan:internal/orchcli#orchCLI.ensureRunsStrand` -> `func (c *orchCLI) ensureRunsStrand()`
- `plan:internal/orchcli#reedStrandOps.AddStrand` -> `func (o reedStrandOps) AddStrand(spec reedengine.AddSpec) (reedengine.Strand, error)`
- `plan:internal/orchcli#reedStrandOps.ReplaceStrand` -> `func (o reedStrandOps) ReplaceStrand(guid string, spec reedengine.AddSpec) (reedengine.Strand, error)`
- `plan:internal/orchcli#fakeStrands.AddStrand` -> `func (f *fakeStrands) AddStrand(spec reedengine.AddSpec) (reedengine.Strand, error)`
- `plan:internal/orchcli#fakeStrands.ReplaceStrand` -> `func (f *fakeStrands) ReplaceStrand(guid string, spec reedengine.AddSpec) (reedengine.Strand, error)`
- `plan:internal/orchcli#TestStart_EnsuresTheRunsStrandBeforeTheOrch` -> `func TestStart_EnsuresTheRunsStrandBeforeTheOrch(t *testing.T)`

**Edit:**
- `internal/orchcli#strandOps`
- `internal/orchcli#reedStrandOps`
- `internal/orchcli#newReedStrandOps`
- `internal/orchcli#orchCLI`
- `internal/orchcli#Command`
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli#fakeStrands`
- `internal/orchcli#newStartHarness`
- `internal/orchcli#fakeStarter`
- `internal/orchcli#fakeStarter.StartSession`
- `internal/orchcli#TestStop`
- `internal/orchengine/doc.go`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Build D1: `lyx orch start` ensures the `runs` strand running `lyx batten overview` in the prime's reed session before it ensures the orch strand, so on a fresh session the overview lands directly above the orch.

`strandOps` gains `AddStrand(spec)` and `ReplaceStrand(guid, spec)`, which `reedStrandOps` delegates to `reedengine.Engine.AddStrand` and `ReplaceStrand` through two new function fields that `newReedStrandOps` fills.
`orchCLI` gains `executable func() (string, error)` and `currentBuild func() (statuspane.BuildIdentity, error)`, set to `os.Executable` and `statuspane.CurrentBuild` in `Command`'s pre-run, so the start harness substitutes both.

`runsStrandAddSpec(cmd)` is `NameOverride: runsStrandName`, the command, no segment and no color (`PATTERN-spawn-color`), and `Display{Anchor: render.AnchorTop, Size: render.Size{Kind: render.SizeContent, ShareNum: 1, ShareDen: 3, MinRows: 3}}` from named constants (D5: the operator's numbers).
The command is `<exe> batten overview`, composed through the shell seam like loom's status strand command.

`ensureRunsStrand` (`runs.go`) reads the strands, finds the one `agentname.Matches` addresses as `runs`, and reads the sidecar at `<orch dir>/runs-strand.json` (`statuspane.ReadSidecar`): with none it adds the strand; when `statuspane.StrandCurrent` holds it keeps it; otherwise, a dead strand or a live one launched by another lyx build, it replaces it with `ReplaceStrand`; after an add or a replace it records the new guid and the current build with `statuspane.WriteSidecar`.
Every failure, a failed read, add, replace or sidecar write or an unidentifiable build, is a `logger.Warn` and never fails `start`: the overview is the operator's view, never the orch's bookkeeping.
`startCmd` calls it right after `reedUp`, before it reads the orch strands.
Bound: it adds, replaces or keeps only the strand named `runs`.
`stop` stays as it is: it removes only the recorded orch strand, so the `runs` strand stays, and with the orch gone render gives the lone content strand every row of the stack above Selvage (card 1's sole-content-strand row).

`orchengine/doc.go`'s `start` and `stop` bullets name the `runs` strand; `docs/overview.md`'s orch bullet says start ensures it and stop leaves it.

Tests:
- `fakeStrands` records each add's and replace's spec and guid, answering a new strand guid, and gains an `addErr` it returns from `AddStrand`; it and `fakeStarter` append to a shared `calls` log the harness owns (`add`, `replace`, `start`), so a row reads the order of start's calls; `newStartHarness` wires that log and sets `executable` to a fixed path and `currentBuild` to a fixed identity on the harness, which a row can change.
- `TestStart_EnsuresTheRunsStrandBeforeTheOrch` (new), table-driven over the harness: no `runs` strand adds one named `runs` with the top content declaration, an empty `Segment` and the `batten overview` command, with `add` before `start` in the calls log, and writes the sidecar; a live `runs` strand recorded in the sidecar with the current build is kept; the same with another build in the sidecar is replaced; a dead `runs` strand is replaced; a failing add leaves the envelope a success with the orch launched.
- `TestStop` gains a row with a `runs` strand beside the orch: only the orch's guid is removed.
- `test-budget.yaml`: `internal/orchcli` 27 → 28.

**ImpactSummary:** orchcli only: `strandOps` widens by two methods, implemented by its one production adapter and its one fake; start gains a best-effort step.
