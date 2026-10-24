# Support log — umbrella-groundwork

## Interview

Autonomous run: no operator answered; every pick below is the writer's own, recorded in the question ledger.

### Turn 1 — exploration

Read the board entry, the related entries (`umbrella-daemon`, `umbrella-anchor`, `loom-beam-darns`, `parallel-batch-parts`, `board-viewer`) and the code at the seams the entry names.
Found since the entry's notes: `hubgeom.ResolveParent` already names a pair's parent `ly:<slug>:orch` for a non-prime parent worktree; `lyx batten cancel` (2026-10-23) ends and removes a run in any state and tolerates an absent board entry; loom-tail-review removed Merge-In, Publish and PR-Gate from darn; both Describe tables require a decision record; `Owners.Effective` already resolves a parent owner through `Config.ParentOf` over origin records; Worktree-Create calls `Add` with the prime's location.

### Turn 2 — scope batch (auto-picked)

- Is `lyx beam` (create, promote) in scope? Recommended yes: the entry's Part 2 holds the create command and the convert verb, and `umbrella-daemon` points here for both; `start` and dobby stay there.
- Is a beam's landing in scope? Recommended yes, through batten and a two-row recipe; the workspace file is removed "when the beam lands", which needs the landing.
- Is the blocked-run cleanup of #724 still open? No: `lyx batten cancel` covers it; only `lyx batten cleanup`'s skip wording changes.
- Is the folded `fabric-add-adopt-local` note in scope? Yes, as an opt-in flag set only by the operator's verb.

### Turn 3 — board batch (auto-picked)

- Project as a kind or a flag on a task? Kind: promote changes a task into a project.
- Notes as members? Yes, the `workspaces` project holds notes.
- Both `project` and `order`? Yes, as the operator decided; `order` is checked against the members.
- Nested projects? Allowed, nearest beam up the chain wins.

### Turn 4 — orch and batten batch (auto-picked)

- Orch name: orchestrator, `ly:<slug>:orch`, matching the existing parent resolution.
- Where an orch may run: the prime and beams only.
- Notices with no beam orch: walk up to the nearest recorded orch, the prime last.
- Batten from a beam: state stays under the prime; never run inside the pair being created or destroyed.

### Turn 5 — lens review

Every lens returned a report. Facts checked before judging: shed writes a run failed when a row with no `on_reroute` returns a reroute (`internal/shedengine` doc); weft content takes no part in a landing merge (`internal/fabricengine` doc); `lyx orch stop` removes the strand but leaves the recorded `State.Strand`.

- existing-paths 1 (BLOCKING, D4 owner readers bypass `Config.ParentOf`): fixed; D4 now names one `ParentOf` in `hubgeom` handed to every owner reader.
- existing-paths 2 (D13/D8 prime-only orch readers): fixed; D8 and D13 name tender, overview, cleanup, loom's `primeOrchRecorded` and every `NotifyPrime` caller.
- existing-paths 3 + bound-breaker 2 (older binary refuses `kind: project`): fixed as an Open risk with its bound, deploy before the first project.
- existing-paths 4 (task-only owner paths): fixed in D1.
- existing-paths 5 (prime spawn writes the hub file too): fixed; D12 changes `RegenerateHubWorkspace` itself.
- existing-paths 6 (ready computation): fixed in D13's ready routing.
- existing-paths 7 + bound-breaker 10 (boards built without the slug seam): fixed; D14 states the bound.
- interactions 1 (BLOCKING, prune rewriting `order` on a held project): fixed; D2 gives the run lock the same prune exemption.
- interactions 2 + walkthrough 3 + bound-breaker 4 (promote with dependents): fixed; D1 forbids depending on a project and D10 refuses a promote with an open dependent.
- interactions 3 (ParentBeam returns a landed beam): fixed; `ParentBeam` skips done projects and the land refusal covers any depth.
- interactions 4 + walkthrough 7 (rules file at landing): fixed; D8 states the fact with its evidence, no mechanism needed.
- interactions 5 + walkthrough 6 + goal-scope (promote writes no workspace file): fixed; D10 step 4.
- interactions 6 (beam flag vs owner setter): fixed; D3 extends the owner setter.
- interactions 7 + bound-breaker 5 (members started mid-create): fixed; D9 writes the flag first so such a member stops and resumes, and states the already-spawned members' bound; D11 re-checks after the claim.
- interactions 8 (root pair's `ParentWorktree`): fixed in D6.
- interactions 9 (D14 contradiction): fixed; the exemption is for slug and kind unchanged.
- interactions 10 (driver watched rule fixed at launch): fixed as an Open risk.
- walkthrough 1 (BLOCKING, create from a beam forks from it): fixed; D5 and D9 pass the prime's location whenever the board picks the parent.
- walkthrough 2 (promoted beam's weft carries the run's records): fixed; D10 step 2 resets the weft.
- walkthrough 4 (ready notice input): fixed; D13 routes through `ParentBeam`.
- walkthrough 5 + goal-scope + leanness (Finalize with no reroute edge): fixed; D11 states the fact and builds Finalize never to reroute.
- walkthrough 8 (beam dirt refuses every member start): fixed; D5 no longer refuses a named parent's dirt.
- bound-breaker 1 (stopped orch keeps its recorded strand): fixed; D13 uses the liveness predicate.
- bound-breaker 6 (the LLM orch runs `lyx fabric add`): fixed; D6 adds `--adopt-local`.
- bound-breaker 7 (`P` type label): fixed; D1 names the way forward and Open risks names the effect.
- bound-breaker 8 (`ParentSlug` has no found signal): fixed; D4 falls through only when no record is found.
- bound-breaker 9 (another machine's hand-made child pair): fixed as a stated bound in D11.
- goal-scope (orch narrowed to beams): fixed; Scope Out states it with its reason.
- goal-scope (cross-machine ready-to-land handover): fixed; Scope Out states why it is not this task's.
- goal-scope (adopt and cleanup prime-only unexplained): fixed; they run from any worktree too.
- goal-scope (create needs a prior upsert): fixed; D9 gives the reason.
- goal-scope (orch sketch not recorded as rejected): fixed in Rejected alternatives.
- goal-scope ("sees only its own"): fixed; D8 names the filter.
- goal-scope (run size): rejected; the operator chose the bundle, and size is no design fault.
- board-fit 1 (`loom-beam-darns` needs a run-keeping conversion): fixed in Scope Out.
- board-fit 2 (`umbrella-daemon` later drops the ready notice): fixed in Scope Out.
- board-fit 3 + 5 + 8 (`run-profiles-ladder`, `batten-window-default`, `draft`): fixed in Constraints.
- board-fit 4 (`creel-agents`' hold notice via `NotifyPrime`): fixed in D13.
- board-fit 6 (anchor seams): rejected; informational, nothing to change.
- board-fit 7 (`landing-escalate-arbiter`): rejected; no conflict, and the stale Open risk is gone.
- leanness (D6 history rationale, #724 duplicate, `workspaces` data bullet, docs list, Notes bullets): fixed; trimmed or cut.
- leanness (Auto-mode assumptions restating Decisions): fixed; reduced to picks with no Decision of their own.
- leanness (support log Turn 1): rejected; the lens itself asked for no action.

## Rejected alternatives

- A `beam` flag on a task instead of a project kind: promote needs a kind change, and a project carries no run fields.
- Storing a beam's branch on the board: the operator decided the branch is derived from slug and `branch_prefix`.
- Fabric reading the board to know which pairs are beams: `boardengine` imports `fabricengine`, so the mark lives in the origin record instead.
- Starting an orch in `lyx beam create`: the orch became optional when beam and orch were split.
- A writer and a review in the beam landing: each member was already reviewed, and a describe from Go is enough.
- Batten state per beam: the prime is the one place every run is read from, and dobby will read it there.
- Letting `lyx orch` run in any task pair: a task pair has its driver, and orch parent resolution would then name task pairs.
- Lifting `ErrBranchExists` for every caller of `Add`, or for `lyx fabric add` without a flag: an unattended run, or the LLM orch running `lyx fabric add`, would adopt a stale branch under a reused slug.
- The entry's orch sketch, a run recording its starting thread as owner with notices to it: the beam chain already names each run's parent, and a recorded starter goes stale when that thread stops.
- Writing `beam: true` last in `lyx beam create`: a member started mid-create would fork from the prime for good.
- Refusing a beam worktree's tracked dirt at a member's start: the operator works in the beam, so it would refuse routinely.
- Keeping `lyx adopt` and `lyx cleanup` prime-only: a beam's orch needs both, and they act on the prime's location from anywhere as batten does.
- Recording a beam's landing as rerouting to a review: the members were reviewed, and a reroute with no edge fails the run.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Beam create, promote and land in scope; `start` and dobby are `umbrella-daemon`'s.
- Q2 (auto-pick): Project is a third kind with handle letter `P`.
- Q3 (auto-pick): Notes may be project members.
- Q4 (auto-pick): Member order lives in the project's `order`; unordered members render after by handle.
- Q5 (auto-pick): The `beam` flag is written only by the beam verbs.
- Q6 (auto-pick): Parent pair is the nearest beam up the project chain, else the prime.
- Q7 (auto-pick): The beam orch is the orchestrator, `ly:<slug>:orch`; the beam verbs start none.
- Q8 (auto-pick): `lyx orch` runs only in the prime and beams.
- Q9 (auto-pick): Batten state stays under the prime; any drivable worktree may run batten verbs.
- Q10 (auto-pick): Notices go to the nearest recorded orch up the chain, resolved per delivery.
- Q11 (auto-pick): Beam landing through batten with recipe `beam`: Go describe, darn Finalize.
- Q12 (auto-pick): Origin record carries a beam mark for workspace grouping.
- Q13 (auto-pick, revised at lens review): Local branch adoption only through `lyx fabric add --adopt-local`; the default branch records no parent.
- Q14 (auto-pick, revised at lens review): `lyx adopt` and `lyx cleanup` run from any drivable worktree; `lyx shed seed` stays prime-only.
- Q16 (auto-pick, lens review): `lyx beam create` writes the board flag first.
- Q17 (auto-pick, lens review): A named parent's dirt never refuses a pair.
- Q18 (auto-pick, lens review): `lyx beam promote` resets the pair's weft to a fresh fork's content.
- Q15 (resolved by research): #724's blocked-run cleanup is met by `lyx batten cancel`, commit `c14db65e0`.
