# Card 34 — beam-create

**Create:**
- `internal/beamcli/doc.go`
- `plan:internal/beamcli#Command` -> `func Command() *cobra.Command`
- `plan:internal/beamcli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/beamcli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/beamcli#createDeps` -> `type createDeps struct`
- `plan:internal/beamcli#createResult` -> `type createResult struct`
- `plan:internal/beamcli#create` -> `func create(deps createDeps, slug string) (createResult, error)`
- `plan:internal/beamcli#hubCreateDeps` -> `func hubCreateDeps(prime *lyxcwd.Location) (createDeps, error)`
- `plan:internal/beamcli#TestCreate` -> `func TestCreate(t *testing.T)`

**Edit:**
- `cmd/lyx#newRoot`
- `pattern/PATTERN-cli-cobra.md`

**Extends:**
- `docs/overview.md`

**Uses:**
- `internal/battencli#ResolveBattenPrime`
- `internal/boardengine#ReadTasks`
- `internal/boardengine#ParentBeam`
- `internal/boardengine#ReadOwners`
- `internal/boardengine#SetOwners`
- `internal/fabricengine#AddOptions`
- `internal/fabricengine#SpawnPairPush`
- `internal/fabricengine#RegenerateHubWorkspace`
- `internal/hubgeom#ParentOf`
- `internal/machinename#Name`
- `internal/fabricengine#MarkBeam`
- `internal/fabricengine#Topology.TaskBranchOnOrigin`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`

**Intent:**
A new module `lyx beam`, a cobra subtree mounted in `cmd/lyx/main.go`'s `newRoot` (PATTERN-cli-cobra), with `Command`, `RunCLI` and `RunCLIIn`, and this card's verb `create <slug>` (D9); `promote` and `land` follow on cards 35 and 36.
Like `adoptcli`, `beamcli` has no engine package and calls the owning modules' functions in-process, which PATTERN-cli-cobra's background lists as a deviation; the package doc records the verbs, their steps and bounds.
Every verb resolves its location through `battencli.ResolveBattenPrime`, so it runs from the prime and from any beam and acts on the prime's location.
`beamcli` is a hub-only CLI like `adoptcli` and `battencli`, so `PATTERN-cliwire-sole-wiring`'s standalone wiring does not apply: its location comes from `battencli.ResolveBattenPrime` (card 23) and it declares no `cliwire.Module`.

`create` takes an existing project entry that is not done and whose `beam` is false, or whose `beam` is true and whose effective owner (`ReadOwners` with `hubgeom.ParentOf`) is this machine or none; the project's title, brief and labels are board writing, so the verb makes no entry.
It refuses, each with its way forward: an absent entry or a note, naming `lyx board upsert` of a project entry under a slug of its own; a task, naming `lyx beam promote <slug>`; a done project, naming a new project entry; a beam another machine owns, naming the owner and `lyx beam create <slug>` run there, or `lyx adopt <slug>` to take the beam over and then `lyx beam create <slug>` here.
In order, each step skipped when its result is already there, so a re-run finishes a stopped create:
1. record this machine as the project's owner with `beam: true` through `SetOwners` (`OwnerRequest.Beam`), skipped when both already stand;
2. create the pair with `Add` from the prime's location, `Parent` from `ParentBeam` over the project's own `project` chain, and `Beam: true`, so the mark lands in the same origin commit; when the pair exists it is not added again, and a pair whose origin record lacks the mark, such as one made by `lyx fabric add --adopt-local` after `Add` refused a leftover branch, gets it through `MarkBeam` in one records commit;
   before the add, a parent beam whose pair is absent on this machine is refused naming the parent beam and `lyx beam create <parent>` when its branch is not on origin (`Topology.TaskBranchOnOrigin`) or `lyx adopt <parent>` when it is, as Worktree-Create's precondition words it (card 28), never fabric's own "not a pair of this hub";
3. spawn the pair's push (`SpawnPairPush`);
4. regenerate the workspace files (`RegenerateHubWorkspace`, card 7).
Writing the flag first makes a member started mid-create stop at Worktree-Create (card 28) and resume into the beam rather than fork from the prime for good; members already spawned under the grouping project keep the prime as their parent.
The envelope names the beam's worktree and `lyx orch start` there as the way to give it an orch; the verb starts none.
`TestCreate` drives `create` over fake deps: each refusal above with its text, a fresh create running all four steps in order with the parent beam passed to `Add`, and a re-run after a stop following each step finishing the rest only, also for a project whose `beam` is already true;
an existing unmarked pair marked through `MarkBeam` and not added; a nested beam whose parent beam's pair is absent refused naming `lyx beam create <parent>`, and with the parent's branch on origin naming `lyx adopt <parent>`;
and the envelope naming the beam's worktree and `lyx orch start` there.
A hub-level row runs `lyx beam create` through `RunCLIIn` from a marked beam's anchor and reaches the verb's own refusal of an absent entry, not the guard's.
`docs/overview.md` gains the `beam` module row and its execution-stack note.

**ImpactSummary:** Adds one CLI module and its mount in newRoot; no existing module changes behaviour.
