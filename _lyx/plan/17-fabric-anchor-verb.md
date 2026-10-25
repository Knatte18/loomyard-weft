# Card 17 — `lyx fabric anchor [<slug>] [--set <dir>]`

**Extends:**
- `internal/fabriccli#Command`
- `internal/fabriccli/doc.go`

**Create:**
- `plan:internal/fabriccli#anchorRequest` -> `type anchorRequest struct`
- `plan:internal/fabriccli#anchorResult` -> `type anchorResult struct`
- `plan:internal/fabriccli#anchorDeps` -> `type anchorDeps struct`
- `plan:internal/fabriccli#changeAnchor` -> `func changeAnchor(ctx context.Context, deps anchorDeps, req anchorRequest) (anchorResult, error)`
- `plan:internal/fabriccli#hubAnchorDeps` -> `func hubAnchorDeps(acting *lyxcwd.Location) (anchorDeps, error)`
- `plan:internal/fabriccli#runAnchor` -> `func runAnchor(ctx context.Context, out io.Writer, args []string, set string) int`
- `plan:internal/fabriccli#TestChangeAnchor` -> `func TestChangeAnchor(t *testing.T)`
- `plan:internal/fabriccli_test#TestRunCLI_Anchor` -> `func TestRunCLI_Anchor(t *testing.T)`

**Uses:**
- `plan:internal/fabricengine#Topology.BeginAnchorChange`
- `plan:internal/fabricengine#AnchorChange.Apply`
- `plan:internal/fabricengine#AnchorChange.Finish`
- `plan:internal/fabricengine#AnchorChange.Close`
- `plan:internal/fabricengine#MoveInProgress`
- `plan:internal/fabricengine#ErrAnchorChange`
- `plan:internal/fabricengine#BranchAnchorStopped`
- `plan:internal/lyxcwd#NamedWorktree`
- `plan:internal/hubforge#ReanchorPair`
- `internal/hubgeom#DobbyGeometry`
- `internal/hubgeom#DobbyStart`
- `internal/dobbyengine#EnsureLive`
- `internal/fabricengine#ReconcileOrigin`
- `internal/hubforge#CloneSecond`

**Intent:**
D7's verb, the CLI layer above card 15's engine: session, run, dobby and lock steps run here, git work in `fabricengine` (`PATTERN-fabric-git`).
`Command` gains `anchor [<slug>]` with `--set <dir>`, audience operator, whose help states what it does, what it refuses and its cost: a `--set` commits the new marker on the worktree's weft branch, shared by every machine that checks it out, whose pulls and pushes then stop until `lyx fabric anchor <slug> --set <dir>` integrates there.
`runAnchor` resolves the acting worktree with the cwd gate as every fabric verb does, builds `hubAnchorDeps` and writes `changeAnchor`'s result as the envelope.

`changeAnchor`:
without `--set` it reports the anchor of `<slug>` (the acting worktree when omitted) from `lyxcwd.NamedWorktree` and changes nothing;
`<slug>` names the prime by its worktree name or a pair present on this machine, else it refuses naming `lyx fabric pairs`.
With `--set B`, before touching anything, it refuses, each naming its way forward:
the board names another machine as the worktree's effective owner while `B` differs from the marker at the weft's fetched upstream tip or the weft has no upstream (`lyx adopt <slug>` or running it on the owner); a worktree whose weft has a standing move record is exempt, so a re-run recovering a killed move here always runs, bounded to the record's own `B` (a different `B` is still refused by `BeginAnchorChange`), since `lyx adopt` refuses while the record stands;
the worktree's reed state holds a strand, live or stored (`lyx reed down` in that worktree);
the worktree's run driver lock is held, a registered batten run of it is live, or its run status names an unfinished run (wait for it, or `lyx batten cancel <slug>`, which card 11 lets through an entry stopped only by this change);
on Windows, where `runproc.End` signals nothing, a live dobby it would end, naming its pid (`taskkill /PID <pid> /T /F`, then the `--set` again); the OS comes from a `GOOS` seam of `anchorDeps`, as `pairteardown` takes `goos`;
for the prime only: any live reed session or stored strand in the hub (`lyx reed down` in each worktree it names, the orch's included), any live batten run (wait for each run it names to halt or finish, which `lyx batten status` in that run's worktree reports, or pause it with `lyx batten pause`), a held gate slot (wait for its holder, named from the slot), or a live hibernate daemon of the hub (`lyx hibernate cancel`), each then the `--set` again; the creel daemon is not refused.
Then `BeginAnchorChange` adds the engine refusals and returns a no-op for an equal anchor, reported as such.
Then it waits, printing what it waits on, until no dobby it will end has a pass in flight: the worktree's own dobby, and for the prime every dobby of this machine; a pass is in flight while its driver lock is held or while its dobby has a live child process in the run-process table other than a spared push or board sync.
Then it takes the parent lock of the worktree's branch through `internal/parentlock` (`Store.Acquire`) as its own owner, this process's pid and start time, queuing behind a holder, and holds it to the end.
It ends the worktree's dobby when it is a beam, and for the prime every dobby of this machine, through `runproc.EndSparing` sparing the push and sync roles.
It closes its log sink under `A/.lyx`, opening it in the weft's lock directory while the record stands, calls `Apply`, and reopens the sink at the anchor `Apply` left.
An `Apply` error restarts every dobby it ended (`dobbyengine.EnsureLive` for the prime, the beam's own start otherwise), releases the parent lock and returns the error.
Then `Finish`, then it starts the worktree's dobby when it is a beam this machine owns and its lock reads free, or runs `dobbyengine.EnsureLive` for the prime, releases the parent lock and calls `Close`.
A dobby start that fails after `Finish` fails nothing: the move has landed, so the result reports the start's failure with the way forward `lyx beam start` and the verb exits 0.
A kill leaves the record for a re-run of the same `--set` from any worktree of the hub; a kill after `Finish` leaves owed only the push, which the next weft push carries, and the dobby start, which `lyx beam start` makes.
The verb holds no lock against a session, a hand pass or a hidden `lyx beam dobby` started meanwhile; the help says the operator starts none in the worktree, and for the prime none in the hub, while it runs.
`anchorDeps` holds one seam per external read or action above (owners, reed state, run locks and status, batten liveness, gate slots, hibernate daemon, pass-in-flight probe, parent lock, dobby end and start, the engine calls and the log sink), filled by `hubAnchorDeps` over the hub.
`internal/fabriccli/doc.go` documents the verb.

Tests:
`TestChangeAnchor` (tier0, fake `anchorDeps`) covers: the print path; each refusal above, each with no engine call, no dobby ended and no lock taken, each naming its way forward; the prime's hub-wide refusals; the Windows live-dobby refusal through the `GOOS` seam set to `windows`; an owned-elsewhere worktree whose `B` equals the fetched upstream tip's marker, and one with a standing move record and a different upstream marker, each not refused; the order wait, parent lock, log sink closed and opened in the lock directory, dobby end, `Apply`, sink reopened at `B`, `Finish`, dobby start, parent lock release; the dobby end called with the push and sync roles spared; a pass in flight, including a spawned pass not yet holding its driver lock, keeps it waiting until the fake reports none; an `Apply` error restarts the ended dobbies and releases the parent lock; a beam this machine does not own, and one whose lock reads held, gets no dobby start after `Finish`; a failed dobby start after `Finish` is reported and the result is no error; a no-op ends no dobby.
`TestRunCLI_Anchor` (tier1, hubforge) first commits `sub/.keep` in the prime's warp, so `sub` is a directory git tracks, and covers: `lyx fabric anchor` prints `.`; `lyx fabric anchor <slug> --set sub` moves the pair (marker, content, junctions and launchers at `sub`) and a second print reports `sub`; a child forked before its parent's change keeps its own anchor and marker, resolves at `.` and commits records there, and its warp commit merged into the parent with `lyx fabric merge` run in the parent leaves the parent's marker and anchor unchanged, since the weft is no merge participant; after the prime's change on one hub, a second clone of it (`hubforge.CloneSecond`) stops its prime weft pull and replaying push with both sides unchanged, its hub-scope `ReconcileOrigin` reports the prime weft `anchor_stopped` with its `.lyx` unchanged, and its own `lyx fabric anchor <prime> --set sub` integrates, carrying its unpushed records commits to `sub`, with the dobby start the test binary refuses reported and the verb exiting 0.

**ImpactSummary:** `Command` gains one subcommand and the package doc one section; the rest is new code in `fabriccli`, so no existing verb changes.
