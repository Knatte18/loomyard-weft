```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: traced the existing code paths that write, reset, reapply or read each state the record introduces or changes.

**Paths traced, per decision**
- D1, kind `project` vs `normalizeTask` (`internal/boardengine/task.go:258`): the new binary changes it, so it holds. An older binary breaks, and Open risks already covers that.
- D1, owner setter `SetOwners` skips non-task entries (`internal/boardengine/ownerset.go:164`): D1 already states the change, so it holds.
- D1, the list view's owner fill is task-only (`internal/boardcli/cli.go:720`): D1 states the change, so it holds.
- D1/D15, layering (`internal/boardengine/layer.go:24,143`), which reads every non-note entry as a task: **breaks**. A project with no open `depends_on` gets layer `Ready` in `lyx board list` and `get`.
- D13, the Land row's `ReadyTasks` (`internal/battencli/land.go:159`): it filters `KindTask`, so it holds.
- D2, the run lock's prune exemption (`internal/boardengine/runlock.go:66`): D2 extends it, so it holds.
- D4, `Owners.Effective` (`internal/boardengine/owners.go:104`) reads entries in memory and calls `parentOf`: it holds, with ParentOf swapped.
- D4, `boardcli`'s `cfg.ParentOf = fabricengine.ParentSlug` (`internal/boardcli/cli.go:141`): D4 replaces it, so it holds.
- D5/D7, `add`'s `SourceDirtyPaths(l)` on the acting location (`internal/fabricengine/add.go:188-193`): **breaks**. With the prime acting and `Parent` naming a beam, Add still refuses the prime's dirt, which D7 says a beam member never checks.
- D11, the start guard's left-pair refusal (`internal/battencli/startguard.go:78-86`): **breaks**. A fresh `lyx batten run` of a beam project has no status file and a present pair, so it is refused before Worktree-Create.
- D10, the start guard after a promote: batten's records are cleared, so the left-pair refusal fires before D10's marked-pair refusal. The later `land` hits the same left-pair break as D11.
- D11, Finalize's large-merge reroute (`internal/landingshed/doc.go:36`, `finalize.go:880-883`): D11 states the change, so it holds.
- D11, the Land prime check on an empty parent branch (`internal/battenshed/land.go:185-190`): it holds for D6's root pair.
- D12, `WriteOrigin` through `state.WriteJSON` keepUnknown (`internal/state/state.go:29,68-75`): a new binary declares `beam`, so it holds. An older binary forking from a beam's weft keeps `beam: true` on the new pair, which falls under the Open risk's deploy-first bound.
- D12, `RegenerateHubWorkspace` callers, the fabric verbs and `ideengine/spawn.go:51`: D12 changes the function itself, so it holds.
- D8, `RefuseUnlessPrime`, also used by `lyx shed seed`: Scope keeps seed prime-only, so it holds.
- D13, `noticesReachDriversParent` and `orchStrandRecorded` (`internal/battencli/wire.go:541-560`), and `primeOrchRecorded` (`internal/loomcli/drivewiring.go:355`): D13 names them, so they hold.

**Findings**

1. **BLOCKING**, class `design`, at D11 (also D10's later land): `lyx beam land` starts `lyx batten run <slug>` over a beam whose pair exists and which has no batten status file. Batten's start guard refuses that start ("left a pair from an earlier run … run `lyx batten cancel <slug> --remove`"), so the landing never reaches Worktree-Create's skip. The record exempts only Worktree-Create, never this refusal. Evidence: `internal/battencli/startguard.go:78-86`, and D11 "Worktree-Create finds the pair present and skips".

2. **BLOCKING**, class `design`, at D1/D15/D8: board layering puts every non-note entry with no open dependency in layer `Ready`, so a project lists as Ready in `lyx board list`. An orch starting a Ready entry from its `parent_beam` filter runs `lyx batten run` on a beam project. By D11 that is the beam's landing, which merges and tears down the beam. The record is silent on a project's layer outside the README. Evidence: `internal/boardengine/layer.go:24,143`, and D1 "Only a task is claimable, bar the landing run of a project whose `beam` is true".

3. **MEDIUM**, class `design`, at D5/D7: `add` refuses tracked dirt in the acting worktree before anything else. D5 and D7 pass the prime as the acting location even when `Parent` names a beam, so a beam member is still refused on the prime's dirt. That contradicts D7's "checks none for one forked from a beam", and the record never says the acting location's dirt check gives way when a parent is named. Evidence: `internal/fabricengine/add.go:188-193`.

Tool calls:
1. grep of boardengine for `Ready` and `KindNote`: to find the ready computation, which the manifest lacked.
2. `layer.go` 180-215 and grep for `KindTask` uses: to see how layering and the owner and claim paths treat a non-task kind.
3. grep of `readyLayer` and claim in boardengine: to locate the layer assignment.
4. `layer.go` 100-150: to confirm a dependency-free non-note entry gets layer Ready.
