```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on a concrete case and checked the end state against the decision's own purpose.

**Findings**

1. **BLOCKING — D6, and the Goal's claim that two machines work at once.** Every batten run record lives in the prime's records, not in the parent beam's worktree. So moving a project alone does not move its runs.
   - Evidence: `shedrun.RunDir` is the durable, fabric-synced `<anchor>/_lyx/shed/<slug>/` of the location it is given. `battencli.dobbyParentRuns` lists every batten run from `shedrun.List(prime)`, so each run is recorded under the prime whatever its parent is.
   - Case: G is owned by OSL-1046 and root by Neve. Neve's prime goes on writing the records of root's runs and of G's owner-paused pawl run. On OSL-1046, `lyx adopt G` resumes G's members, and their batten passes write the same prime records branch.
   - Result: two machines commit to one prime records branch. The purpose ("an adopt moves that project alone", each machine with only its own project) is not met. The record never says where a member's batten run record lives or how it moves with its project.

2. **BLOCKING — D8 Beam-Down and D4 `lyx beam stop` with `runproc`.** Ending the beam's dobby "through its `runproc` record" ends every process registered under the beam's slug.
   - The pawl run is registered under that same slug, and D11 relies on exactly that for cancel.
   - Case: Beam-Down runs in G's pawl driver and calls the stop.
     1. `runproc.End(G)` ends the pawl driver itself before the row returns Done.
     2. The next batten pass respawns the driver.
     3. Beam-Down runs again, and the loop repeats.
   - Case: an operator runs `lyx beam stop G` during the perch. That kills the review agents and the pawl driver.
   - The record needs the stop scoped to the dobby's role, or Beam-Down must not run in-process.

3. **MEDIUM — D9 and D11: a new member can start after Beam-Await ends.** The land check is a snapshot, and the guard that would catch a late member is removed.
   - Case:
     1. Beam-Await passes the land checks and is Done; the run goes on to Parent-Align and the perch, which take hours.
     2. G's dobby is stopped only at Beam-Down, so it still starts a member released meanwhile, with G as its parent.
     3. D11 removes the land refusal from Worktree-Create, so nothing refuses that member.
     4. Finalize lands G without the member, and teardown removes the member's parent pair.
   - Today Worktree-Create re-applies the refusal after its claim for exactly this case.

4. **MEDIUM — D5: runs in flight at the upgrade.** The record does not migrate a run whose recorded task owner differs from its project's owner.
   - Case: task T under root, root owned by A, and T's `owner_machine` is B, which drives T's run.
   - After the upgrade T reads as A's: B's pass owner-pauses it, and A's dobby is not driving it.
   - T then needs `lyx adopt root` on one side. The record never names this case or its way forward.

5. **LOW — D9: churn while the beam is owned elsewhere.** A yield at Beam-Await ends the child driver. Each batten pass then reads the driver as unconfirmed and respawns it, which yields again, until the owner pause lands.
   - This bounds the cost at one spawn per probe pass.
   - It also conflicts with D9's claim that one driver process per beam is the whole cost.

6. **LOW — D2: no fresh orch after a stop.** A fresh orch can only come from `lyx orch refresh` on a live orch.
   - `SessionID` survives the orch state reset (`orchengine/state.go`: "…SessionID survive"), so `lyx beam stop` then `lyx beam start` resumes the old session.
   - This is within the entry, but the record does not state it.

**Per decision: case used, purpose met or not**

| Decision | Case | Purpose |
|---|---|---|
| D1 | `lyx beam start G` from the prime | met |
| D2 | dead strand with a recorded id; a gone transcript | met (see finding 6) |
| D3 | `lyx beam start` in a task pair | met |
| D4 | `lyx s`; `lyx beam stop` | stop broken (finding 2) |
| D5 | T under root, plus a legacy task owner | partly met (finding 4) |
| D6 | the G/H example | broken (finding 1) |
| D7 | Neve's orch after OSL-1046 adopts root | met within its stated bound |
| D8 | a seed recording `beam` resumed at Beam-Describe | met; Beam-Down broken (finding 2) |
| D9 | land request accepted, then a member released | broken (finding 3) |
| D10 | create with `--anchor`; land refusals | met |
| D11 | a fresh pawl start through the guard | met, but loses the late-member guard (finding 3) |
| D12 | the last member lands | met |
| D13 | removal | met |

**Tool calls**

1. Read `lyxdirs/dirs.go` and `shedrun`: to find where batten run records live, which my context lacked.
2. Read `orchengine` state: to see whether `SessionID` survives a stop, which my context lacked.
3. Grepped for the batten dir and Worktree-Create on projects: for the record location, which my context lacked.
4. Read `DobbyRuns`: to see which directory dobby lists runs from, which my context lacked.
5. Read `shedrun/paths.go` `RunDir` and `dobbyParentRuns`: to confirm runs are listed from the prime's durable `_lyx/shed`, which my context lacked.
