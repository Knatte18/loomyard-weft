```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the code paths that already write or read what each decision changes.

**Findings**

1. **BLOCKING — D4 (owner resolution).** Batten, adopt and the other owner readers do not go through the board's `Config.ParentOf`, so D4's fallback reaches only `boardcli`.
   - Evidence: `internal/battencli/owner.go` `hubOwnerSeams` builds `parentOf` from `fabricengine.ParentSlug` alone and passes it to `owners.Effective` and `SetOwners`.
   - `adoptcli/deps.go`, `loomcli/ownercheck.go`, `fabriccli/remove.go` and `pairteardown/originseams.go` each wire their own origin-only `ParentOf`.
   - Effect: batten's start check reads an unspawned beam member as owned by the prime's owner, not the beam's, which is the opposite of what D4 and the PATTERN-owner-machine constraint claim.
   - Fix: D4 must say the fallback lives in one resolver that every `ParentOf` provider uses, not only `Config.ParentOf`.

2. **MEDIUM — D13 (notice routing).** Three readers still look only at the prime's orch, and the record does not mention them.
   - `loomcli/drivewiring.go` `primeOrchRecorded` decides whether the run's parent receives notices. For a run in a beam with its own orch, it still answers from the prime's orch state, so the driver's watched or relay decision disagrees with batten's walked parent.
   - `tendercli/deps.go` `OrchStrand` and `WatcherLive` read only the prime's orch. Hibernate, pause and revive never see a beam orch's strand or watcher, so a beam orch is neither held nor revived.
   - `battencli/overview.go` (`orchPaths: PrimePaths(prime)`) and `cleanupcli/segments.go` (`CleanupHandoffs(PrimePaths)`) also read the prime only. A beam orch's handoffs are never cleaned, and the overview shows only the prime's orch.
   - Fix: D8 and D13 need to say which of these become per-orch and which stay prime-only.

3. **MEDIUM — D1 (cross-version board).** An older binary that replays a board holding a project entry will fail.
   - Evidence: `boardengine/replay.go` calls `normalizeTask` on incoming entries at lines 283 and 365. `normalizeTask` refuses any kind other than task and note.
   - Effect: an older binary on another machine fails its detached sync replay once `kind: "project"` reaches Bolt. The handle assigner there would also treat a project as a note and give it a letter from its type label, so handles flip between binaries.
   - The PATTERN-tolerant-records constraint covers unknown keys only, not a new value of a known key. The record needs a stated bound, such as every machine deploying first, or should name this risk.

4. **MEDIUM — D1 and D9 (project owner).** Several existing owner paths are task-only, so D9 step 3 would be silently skipped.
   - Evidence: `boardengine/ownerset.go:164` skips an entry whose kind is not task. `owners.go:59` sets `IsTask` by kind. `store.go:665` fills the list's effective owner only for tasks.
   - Effect: `SetOwners` would report the beam's owner as not recorded.
   - D1 says only that a project "may carry `owner_machine`". It should state that the setter, `Effective` and the list view treat a project with `beam: true` as a task does.

5. **MEDIUM — D12 (workspace writers).** A second writer of the hub workspace file is not mentioned.
   - Evidence: `ideengine/spawn.go:51` calls `RegenerateHubWorkspace` on every prime spawn, and its doc says the file lists "prime, Bolt, every task pair".
   - D12 talks only about "the regeneration that runs after every fabric verb". Whether a prime spawn also writes the beam files and the one-level hub file depends on D12 changing `RegenerateHubWorkspace` itself. The record should say so, and the `ideengine` doc changes with it.

6. **LOW — D13 (ready notice).** The ready computation does not know about projects. `battencli/land.go:159` takes ready to mean kind task, no status and no `depends_on`. D13's routing of each ready task to its own parent's orch has to change this reader. It already counts only tasks, so projects are correctly excluded. This is just a note for the plan.

7. **LOW — D14 (slug check).** The slug seam is filled only by `boardcli`. Other board writers, such as batten's `claimBoardEntry`, adopt's `SetOwners` and the replay, build a `Board` without it. Those write only status or owner fields, so they make no slug and nothing breaks. The record should still bound the check to the `boardcli` and `boardengine` write paths.

**Paths traced**

| Decision | Path | Result |
|---|---|---|
| D4 | `boardcli` `Config.ParentOf` | survives |
| D4 | battencli owner seams, adoptcli, loomcli ownercheck, fabriccli remove, pairteardown originseams | breaks: origin only |
| D1 | `replay.go` `normalizeTask` on an older binary | breaks |
| D1 | older binary's handle assigner | breaks: letter churn |
| D1 | `layer.go` (task-only layers) | survives |
| D1, D9 | `ownerset.go:164` | breaks: task-only |
| D1, D9 | `owners.go` `IsTask` | breaks: task-only |
| D1, D9 | `store.go` list owner | breaks: task-only |
| D8 | batten paths via `BattenDir` and `PrimeRunLock` on the prime location | survives |
| D8 | orch `orchPaths` per worktree | survives |
| D8 | `cleanupcli` orch handoffs | breaks: prime only |
| D8 | `battencli/overview.go` orch view | partial: prime only |
| D8 | `tendercli` revive and hibernate | breaks: prime only |
| D13 | battencli `noticesReachDriversParent` | survives: the record changes it |
| D13 | `loomcli` `primeOrchRecorded` | breaks |
| D13 | `land.go` ready computation | survives with routing work |
| D12 | fabric verbs' `regenerateAfterVerb` | survives |
| D12 | `ideengine` prime spawn | unstated |
| D12 | `portals.go` (retired file path only) | survives |
| D5, D12 | `WriteOrigin`, whose only writer is `Add` | survives: the beam mark must go through it or a sibling |
| D10 | cancel's `clearRunClaim` before the kind change | survives |
| D14 | `boardcli` writes | survives |
| D14 | status and owner writers outside `boardcli` | survive: they make no slug |

**Tool calls**

1. Grepped the `normalizeTask` callers, the `ideengine` and `portals` workspace references, the `PrimePaths` users and the `WriteOrigin` writers. My context lacked who else writes or reads these.
2. Read the `PrimePaths` call sites in `loomcli`, `tendercli`, `cleanupcli` and `overview`, plus the kind checks in `store.go`. I needed what each reader does.
3. Read `loomcli` `primeOrchRecorded`, tender's `OrchStrand`, and the board kind checks on load and list. I needed to confirm these misread the state under D13 and D1.
4. Read battencli `hubOwnerSeams` and grepped the task-only kind checks in owners, ownerset, layer and land. I needed to confirm batten bypasses `Config.ParentOf` and which owner paths are task-only.
