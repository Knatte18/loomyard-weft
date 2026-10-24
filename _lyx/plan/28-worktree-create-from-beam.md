# Card 28 — worktree-create-from-beam

**Create:**
- `plan:internal/battencli#beamForkPrecondition` -> `func beamForkPrecondition(slug, beam string, beamEntry boardengine.Task, beamFound, pairPresent bool, branchOnOrigin func() (bool, error)) error`
- `plan:internal/battencli#TestBeamForkPrecondition` -> `func TestBeamForkPrecondition(t *testing.T)`
- `plan:internal/battencli#TestWorktreeCreate_ForksFromTheBeam` -> `func TestWorktreeCreate_ForksFromTheBeam(t *testing.T)`

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli#startGuard`
- `internal/battencli#hubStartGuard`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#TestStartGuard_Refuse`

**Extends:**
- `docs/overview.md`

**Uses:**
- `internal/boardengine#ParentBeam`
- `internal/boardengine#ReadTasks`
- `internal/fabricengine#Topology.TaskBranchOnOrigin`
- `internal/fabricengine#AddOptions`
- `internal/battenshed#PreconditionError`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`

**Intent:**
D7: Worktree-Create forks a task's pair from its beam.
After its claim, `CreateWorktree` reads the board and resolves the task's parent with `ParentBeam`; an empty answer calls `Add` from the prime's location as today, and a beam calls `Add` with `AddOptions{Parent: <beam>}` and the prime's location as the acting location (D5), so the member forks from the beam whichever worktree started the run.
`beamForkPrecondition` stops the row with a `battenshed.PreconditionError` before the add:
- the beam's pair absent on this machine names the beam and, by `Topology.TaskBranchOnOrigin` of the beam, `lyx beam create <beam>` when its branch is not on origin (which also covers a create stopped before its pair, and a hand-set `beam: true`, D3) or `lyx adopt <beam>` when it is;
- a beam whose board entry is run-held by its own landing names the landing and `lyx batten status <beam>`, and, since that landing in turn stops on this open member (card 36), `lyx batten cancel <slug>` to end this member's run so the landing can go on.
The precondition runs only after `taskWorktreeComplete` finds no pair for the slug, so a run whose own pair is already present, a beam's landing or a sub-beam's included, reaches the existing skip and never meets its parent beam's state.
A nil `ParentBeam` seam on a hand-built `startGuard` reads every task as forked from the prime, so the dirt check keeps applying.
A member started mid-create stops here and resumes into the beam once the create finishes, with `lyx batten run <slug>`.
The start guard gains a `ParentBeam` seam over the local board: its tracked-dirt refusal of the source applies only to a task forked from the prime, and checks nothing for a member of a beam, whose pair forks from the beam's last commit.
`TestBeamForkPrecondition` reaches each precondition, the pass and a `branchOnOrigin` error, which is returned; `TestStartGuard_Refuse` gains rows where a dirty prime passes a beam member's start and still refuses a prime task's;
`TestWorktreeCreate_ForksFromTheBeam` over a hubforge hub with a beam pair `b` and a board holding beam project `b` and member task `m`, the prime holding tracked dirt: the row's create makes `m`'s pair with an origin record naming `b`, a member of a grouping project gets the prime, a run armed from `b`'s location (through card 23's guard) still forks `m` from `b`, and a task whose local branch is left from a removed pair is refused with `ErrBranchExists`, since batten never sets `AdoptLocal`.
`docs/overview.md`'s batten row says Worktree-Create forks from the task's beam.

**ImpactSummary:** Worktree-Create and the start guard learn the task's beam; tasks outside any beam keep today's path unchanged.
