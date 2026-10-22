# Card 16 — adopt-module

**Create:**
- `internal/adoptcli/doc.go`
- `internal/adoptcli/testmain_test.go`
- `plan:internal/adoptcli#Command` -> `func Command() *cobra.Command`
- `plan:internal/adoptcli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/adoptcli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/adoptcli#request` -> `type request struct`
- `plan:internal/adoptcli#slugOutcome` -> `type slugOutcome struct`
- `plan:internal/adoptcli#outcome` -> `type outcome struct`
- `plan:internal/adoptcli#adoptDeps` -> `type adoptDeps struct`
- `plan:internal/adoptcli#adopt` -> `func adopt(deps adoptDeps, req request) (outcome, error)`
- `plan:internal/adoptcli#hubDeps` -> `func hubDeps(prime *lyxcwd.Location) (adoptDeps, error)`
- `plan:internal/adoptcli#hubOnlyPaused` -> `func hubOnlyPaused(recordPath string) func(slug string) (bool, error)`
- `plan:internal/battencli#ResumeInWindow` -> `func ResumeInWindow(location *lyxcwd.Location, slug string) (map[string]any, error)`
- `plan:internal/fabricengine#DropLocalResult` -> `type DropLocalResult struct`
- `plan:internal/fabricengine#DropLocal` -> `func DropLocal(rec *Mutations, l *lyxcwd.Location, slug string) (DropLocalResult, error)`
- `plan:internal/hubforge#CloneSecond` -> `func CloneSecond(tb testing.TB, h *Hub) *Hub`
- `plan:internal/adoptcli#TestAdopt` -> `func TestAdopt(t *testing.T)`
- `plan:internal/adoptcli#TestHubOnlyPaused` -> `func TestHubOnlyPaused(t *testing.T)`
- `plan:internal/adoptcli_test#TestAdoptIntegration_SecondCloneTakesOver` -> `func TestAdoptIntegration_SecondCloneTakesOver(t *testing.T)`
- `plan:internal/fabricengine_test#TestDropLocal` -> `func TestDropLocal(t *testing.T)`

**Edit:**
- `cmd/lyx#newRoot`
- `cmd/lyx#excludedModules`
- `cmd/lyx#destructiveGuardRecordingExecutors`
- `cmd/lyx#destructiveGuardMutatingResultTypes`
- `internal/fabricengine/destroy.go`
- `docs/overview.md`

**Uses:**
- `internal/battencli#RefuseUnlessPrime`
- `internal/battencli#battenCLI.runInWindow`
- `internal/fabricengine#ReconcileOrigin`
- `internal/fabricengine#Topology.Add`
- `plan:internal/fabricengine#EntryOwnedElsewhere`
- `plan:internal/fabricengine#ParentSlug`
- `internal/pairteardown#HubSeamOptions`
- `internal/pairteardown#Teardown.OriginSeams`
- `plan:internal/boardengine#SetOwners`
- `plan:internal/boardengine#ReadOwners`
- `internal/tenderengine#Lift`
- `internal/tendercli#HubDeps`
- `plan:internal/pauserecord#OwnerCaller`
- `plan:internal/landingshed#OwnerHaltPrefix`
- `plan:internal/machinename#Name`
- `internal/hubforge#CopyHub`

**Intent:**
D5: `lyx adopt <slug>...`, `lyx adopt --prime [<slug>...]` and `lyx adopt --all`, with `--keep-local` and `--drop-local`, a new top-level module with no engine of its own that calls each owning module's function in-process, as `lyx cleanup` does (`PATTERN-cli-cobra`); it runs from the hub's prime only, by `battencli.RefuseUnlessPrime`.
`request` holds the slugs, `Prime`, `All`, `KeepLocal` and `DropLocal`; `adoptDeps` holds every step's seam (`Machine`, `ReadOwners`, `Reconcile`, `LivePair`, `AddPair`, `DropLocal`, `SetOwners`, `PauseRecord`, `Lift`, `RunRecord`, `ResumeRun`, `ChildStatus`, `ResumeChild`), which `hubDeps` fills over the hub and `TestAdopt` fakes.
Before anything runs, `adopt` reads the local Bolt and refuses an unknown slug (naming `lyx board list`), a note (naming `lyx board promote`), a done task (naming that it already landed, and `lyx resume <slug>` to let a run that landed there go on to Land under its owner pause), `--keep-local` with `--drop-local`, and no slug without `--prime` or `--all`; `--all` names the prime and every task that is not done and has an owner or a run status, passing over a done task.
Step 1 runs a moving hub-scope origin reconcile whose `pairteardown` seams take a `HubPaused` from `hubOnlyPaused`, which covers a slug only by a live hub entry, so a pair whose strands are alive under any slug-scope pause, an owner pause included, reads live; an unreachable origin refuses naming a re-run once origin answers, a reconcile error refuses naming it and the re-run, and a stop on Bolt or the prime's records branch refuses with the report and that branch's way forward; nothing below runs then.
A named slug the report reads owned_elsewhere stops before its owner write when a branch of it was judged `needs_replay`, `ahead`, `rewound`, `rewritten` or `diverged`, or when adopt's own `LivePair` call, made for every such slug with a local pair, reads it live; the slug's envelope entry names each branch and its state and the two ways forward, `lyx adopt <slug> --keep-local` and `lyx adopt <slug> --drop-local`, and the other slugs go on.
`--keep-local` records the owner over those local commits and over a live pair; `--drop-local` refuses a live pair naming `--keep-local`, and otherwise calls `fabricengine.DropLocal`.
`DropLocal`, in `destroy.go` behind the shared check pipeline (`PATTERN-fabric-destruction-chokepoint`) and filling its caller's `*Mutations` record (`PATTERN-mutation-record`), keeps each of the pair's local branch tips origin lacks under a local `archive/<slug>/<tip>` tag, then moves the branch to origin's tip with `reset --keep` in its worktree, refusing as the reconcile's dirty state does, naming the paths, when a tracked change sits there; it pushes nothing; `DropLocalResult` embeds `MutationRecord` and lists the tags it kept.
`destroy.go`'s header names the new primitive, and `destructiveGuardRecordingExecutors` and `destructiveGuardMutatingResultTypes` register `DropLocal` and `DropLocalResult`.
Step 2 adds, through `Topology.Add` from the prime, the pair of each named task whose branch origin holds and this machine does not; a refusal stops that slug and the other slugs go on, and the leftover-branch refusal is reworded as batten's create refusal is, naming deleting the leftover local branch and re-running `lyx adopt <slug>`, never `lyx fabric checkout` from the prime.
Step 3 records this machine as the owner of every named entry steps 1 and 2 did not stop, with `Takeover`, and of the prime when it is named or when the board records no prime owner (D2's first act), in one `boardengine.SetOwners` call; a refused push refuses the verb, resumes nothing and names its class's way forward and then a re-run of `lyx adopt` with the same arguments, and a slug the re-read shows owned by another stops naming that owner and that a re-run of `lyx adopt <slug>` takes it over again, and starts no run.
Step 4 lifts, through `tenderengine.Lift`, each live entry carrying `pauserecord.OwnerCaller` whose scope is an adopted slug, by its scope, caller and pause time; a slug still covered by another entry gets an envelope line naming it and `lyx resume <slug>` for an entry of its own scope or `lyx resume` for a hub entry.
Step 5 resumes each adopted task whose batten run record is not done through `battencli.ResumeInWindow`, which runs `lyx batten run <slug> --window`'s body (`runInWindow`) for that slug, so the run's own driver lock refuses a second driver; a resume failure is the slug's error and the other slugs go on; a task with no run record gets its owner and no run.
A child halted at Finalize's owner halt, read as a blocked child status whose error opens with `landingshed.OwnerHaltPrefix`, is resumed with `lyx loom resume` spawned in its task worktree (the spawn logged through `logger`), both when its own slug is adopted and when `--prime` or a parent slug adopts its parent while this machine owns its run; a child halted for any other reason stays halted.
The envelope carries `reconcile`, `mutations` and `partial`, and per slug what happened: a stop with its branches, pair added, owner recorded, pause lifted or standing, run resumed or none, the error, and a warning when this machine does not own the slug's parent, since that run halts at Finalize here.
Bound: adopt writes owners without the old owner's help and stops no driver live elsewhere, which pauses at its next owner check; under a hub pause `LivePair` reads not live, so `--drop-local` can move a branch under interrupted strands, as any hub-paused pair's reconcile does today.
`newRoot` registers `adoptcli.Command()` and names `adopt` in the root's module list; `excludedModules` gains `adopt`, "takes over runs from another machine, which needs a second clone of the hub that no sandbox scenario builds", for `PATTERN-sandbox-coverage`.
`docs/overview.md` gains `internal/adoptcli/` in the module tree and an `adopt` module entry beside `cleanup`, and `doc.go` holds the verb's design as above.
`hubforge.CloneSecond` clones a second hub from `h`'s own bares through `fabriccli.CloneAndWire`, as a second machine working on the same remote.
`TestAdopt` is a table over fake deps: each refusal before anything runs, the done task's naming `lyx resume`; an unreachable origin, a reconcile error, a Bolt stop and a stop on the prime's records branch refusing with nothing after; an owned-elsewhere slug with a diverged branch, and one with a `needs_replay` records branch, stopping with both ways forward while another slug is adopted; `--keep-local` recording it; `--drop-local` dropping then recording it; `--drop-local` over a live pair refusing naming `--keep-local`; a refused add stopping its slug while another is adopted; a refused push resuming nothing and naming the re-run; a lost owner write starting no run; a hub with no prime owner recording this machine as the prime owner beside the named slug; an owner pause lifted while an unlabelled pause stands named with `lyx resume <slug>` and a hub pause named with `lyx resume`; an adopted task with a run record resumed once through `ResumeRun`, and a failing resume reported on its slug while the next slug is resumed; a task with no run record getting no run; an owner-halted child resumed when its own slug is adopted, when its parent is adopted with `--prime` and when its pair parent's slug is adopted, and a child halted otherwise left alone; a slug whose parent this machine does not own carrying the Finalize warning; `--all` choosing the prime and the tasks with an owner or a run status and passing over a done task and an unowned idle task; `--prime` writing the prime's owner only.
`TestHubOnlyPaused` covers a hub entry covering, a slug entry not covering, and an unreadable record returned.
`TestAdoptIntegration_SecondCloneTakesOver` (integration over two `hubforge` clones): the second clone adopts a slug whose pair only the first holds, adds the pair, records and pushes the owner, lifts its owner pause; and with the first clone's pair carrying local code commits origin lacks while its run lock is held under a slug-scope owner pause, which `LivePair` reads live through `hubOnlyPaused`, an adopt back stops, `--drop-local` refuses naming `--keep-local`, and `--keep-local` records the owner.
`TestDropLocal` (integration) covers a records branch and a code branch ahead of origin tagged and moved to origin's tip with the mutations recorded, and a dirty tracked path refusing with nothing moved.

**ImpactSummary:** a new top-level module and its registration; battencli, fabricengine and hubforge each gain one exported function, and the destruction guard's tables gain one executor and one result type.

**Commit:** 16: adopt: take over a task or the prime from another machine
