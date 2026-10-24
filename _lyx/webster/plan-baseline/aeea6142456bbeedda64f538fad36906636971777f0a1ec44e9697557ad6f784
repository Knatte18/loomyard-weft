# Card 35 — beam-promote

**Create:**
- `plan:internal/beamcli#promoteDeps` -> `type promoteDeps struct`
- `plan:internal/beamcli#promote` -> `func promote(ctx context.Context, deps promoteDeps, slug string) (promoteResult, error)`
- `plan:internal/beamcli#promoteResult` -> `type promoteResult struct`
- `plan:internal/beamcli#hubPromoteDeps` -> `func hubPromoteDeps(prime *lyxcwd.Location) (promoteDeps, error)`
- `plan:internal/beamcli#TestPromote` -> `func TestPromote(t *testing.T)`
- `plan:internal/battencli#EndRunForPromote` -> `func EndRunForPromote(ctx context.Context, prime *lyxcwd.Location, slug string) error`
- `plan:internal/battencli#ClearRunForPromote` -> `func ClearRunForPromote(ctx context.Context, prime *lyxcwd.Location, slug string) error`
- `plan:internal/battencli#endRunPhase` -> `func endRunPhase(ctx context.Context, slug string, remove bool, rerun string, deps cancelDeps) (cancelReport, error)`
- `plan:internal/battencli#clearRunPhase` -> `func clearRunPhase(slug string, remove bool, rerun string, report *cancelReport, deps cancelDeps) error`

**Edit:**
- `plan:internal/beamcli#Command`
- `internal/battencli#cancelRun`
- `internal/battencli#startGuard`
- `internal/battencli#hubStartGuard`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#TestStartGuard_Refuse`
- `internal/battencli#TestCancelRun`

**Extends:**
- `internal/beamcli/doc.go`

**Uses:**
- `internal/battencli#hubCancelDeps`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#MarkBeam`
- `internal/fabricengine#ResetRecordsToParent`
- `internal/fabricengine#SpawnPairPush`
- `internal/fabricengine#RegenerateHubWorkspace`
- `internal/boardengine#SetOwners`
- `plan:internal/boardengine#ReadTasks`

**Intent:**
D10: `lyx beam promote <slug>` turns a spawned task into a beam, keeping its pair and the commits on its branch.
It refuses, each with its way forward: a task never started, holding no owner and no run status, naming a new project entry through `lyx board upsert` and `lyx beam create` on it; a started task with no pair on this machine, and a task another machine owns, naming `lyx adopt <slug>`; a task whose run has passed Finalize's parent-side merge (batten's `ChildReachedParent` read), naming `lyx batten run <slug>` to finish the landing; a task an open task depends on, naming each dependent and `lyx board set-deps`; and, before step 1, a task whose pair's origin record has an empty `ParentBranch`, naming the pair a root pair, made by `lyx fabric add --adopt-local` over the default branch, that cannot become a beam, and naming as its way forward the never-started refusal's route, a project entry through `lyx board upsert` under a slug of its own and `lyx beam create` on it, so no root pair reaches step 4's root-pair refusal after its run has been ended, marked and unclaimed.
On a project whose `beam` is false it runs `lyx beam create`'s steps, and on a project whose `beam` is true and whose pair carries the mark it finishes step 6.
In order, each step skipped when done, so a re-run after any step finishes the promote:
1. end the task's run processes and strands: `battencli.EndRunForPromote`, the first part of `cancelRun`'s phase without `--remove` (the cancel's own refusals, marker and `EndForCancel`), stopping before records and claim; `cancelRun` splits at that point into `endRunPhase` and `clearRunPhase`, two halves over `cancelDeps` it calls in turn with its own re-run text, so the cancel verb itself is unchanged, and the promote entry points pass `lyx beam promote <slug>` as the re-run every inherited refusal names;
   `endRunPhase` removes the cancel marker when it ends, so no marker stands between steps 1 and 3;
   an unreachable origin makes the phase end-only, as it does the cancel's, and promote then stops after step 1 with the run's processes ended, naming a re-run of `lyx beam promote <slug>` once origin answers, and writes no mark;
2. write the beam mark into the pair's origin record (`MarkBeam`) in one records commit, and spawn the pair's push;
3. clear the task's claim and batten's records: `battencli.ClearRunForPromote`, the rest of that cancel phase (`RemoveRecords`, `ClearClaim`);
4. reset the pair's records content to a fresh fork of its parent's (`ResetRecordsToParent`, card 8), whose refusal of uncommitted content outside the record pathspec names the paths and a re-run of `lyx beam promote <slug>`; spawn the pair's push; `promote` takes no force flag;
5. through the owner setter, record this machine as owner with `Beam` and `ToProject` in one Bolt write, turning the entry into a beam project and clearing its `depends_on`, `recipe`, `profile` and `run_settings`;
6. regenerate the workspace files.
Batten's start guard gains a `Promoted` seam reading the slug's local pair's origin record: a task whose pair carries the beam mark is refused naming `lyx beam promote <slug>` to finish the promote, so an entry left an open unclaimed task by a promote stopped after step 2 or 3 is never started; a project entry passes this check (card 36 judges it).
The `Promoted` check runs whether or not the run's status file is found, unlike the left-pair check, which runs only with no status file: after step 2 the run is still claimed and its status file still stands, and a resume must not restart the ended run inside a marked pair.
Bound: promote ends a running run for good; nothing of the loom run survives in the pair's checkouts but its code commits, and its discussion, plan and run records stay reachable in the records history behind step 4's commit, on the beam's records branch and every member's forked from it.
A promote stopped before step 2 leaves the run claimed with its processes ended, as an interrupted cancel does: a resume would restart it, and a re-run of promote ends it again.
Bound: batten's start guard is the guard; a hand `lyx loom resume` inside the pair between steps 1 and 4, while its loom seed still stands, restarts the ended run, and a re-run of promote ends it again, the same bound as a stop before step 2.
`TestPromote` over fake deps reaches each refusal, the root-pair one over a fake origin record with an empty parent branch, with no step run and its text asserted to carry both `lyx board upsert` and `lyx beam create`, not only the words root pair, runs the six steps in order for a running task, finishes from a stop after each of steps 1 through 5, finishes step 6 alone for a project whose `beam` is true and whose pair carries the mark, stops after step 1 naming a re-run of `lyx beam promote` when the first half reports end-only, and runs create's steps for a grouping project; `TestStartGuard_Refuse` gains a marked-pair task refused naming `lyx beam promote`, both with no status file and with one found, the stop after step 2; `TestCancelRun` gains rows driving `endRunPhase` then `clearRunPhase` over its fake `cancelDeps`, which together do what a cancel without `--remove` does, minus its notice, and a refusal of `endRunPhase` naming the re-run text it was given.

**ImpactSummary:** Adds the promote verb and two batten entry points over the existing cancel phase, plus one start-guard refusal for a marked pair.
