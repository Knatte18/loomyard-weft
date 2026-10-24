```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end against the code in context.

**Decisions walked**
- D1, promote turning a task into a project that a done task's `depends_on` names: purpose met.
- D2, prune of a member while its project is run-held by its landing: purpose met.
- D3, a hand-set `beam: true` with no branch on origin, followed by `lyx beam create`: purpose met.
- D4, a member pair whose beam pair is not on this machine, read through `ParentOf`: **gap** (F3).
- D5, a beam member started while the prime has tracked dirt: **broken** (F2).
- D6, `--adopt-local main` from a prime that is on a beam branch: purpose met. The Land prime check stops it (`internal/battenshed/land.go:190`).
- D7, a task of beam `b` started from a beam worktree: Add gets the prime location and `Parent=b`, and the origin records `b`. Met except for the dirt in F2.
- D8, a run started from a beam worktree: met. Guard holes are left to bound-breaker.
- D9, a create re-run after a stop before step 2: met.
- D10, a promote and then `lyx beam land`: **broken** (F1).
- D11, `lyx beam land` of a beam made by `create`, which starts `lyx batten run <slug>` with no batten status file and the pair present: **broken** (F1).
- D12, regeneration after a create and after a promote: purpose met.
- D13, the ready notice for a task whose `ParentBeam` worktree is absent on this machine: **gap** (F4).
- D14, an upsert of `x-weft`: purpose met.
- D15, an unordered member: purpose met.
- D16: purpose met.

**Findings**

F1 — BLOCKING, design, D11 (and D10's later land).
- **Issue:** batten's start guard refuses the beam landing before Worktree-Create runs, so `lyx beam land` cannot start.
- **Walk:** `lyx beam land` starts `lyx batten run <slug>` for a beam whose pair exists and which has no batten status file. Create never writes one, and promote step 3 clears it.
- **Evidence:** `startGuard.refuse` (`internal/battencli/startguard.go:78-85`) refuses `!found` with any left pair part, naming `lyx batten cancel <slug> --remove`. That way forward would destroy the beam.
- **Gap:** D11 says "Worktree-Create finds the pair present and skips", but the guard runs earlier, "before the pre-run writes anything" (startguard.go:1). D11 adds land refusals to the start guard but does not exempt a beam project from the left-pair refusal.

F2 — MEDIUM, design, D5/D7.
- **Issue:** a beam member's create is refused for the prime's tracked dirt.
- **Walk:** Worktree-Create calls `Add(prime, slug, {Parent: beam})`. `add` runs `SourceDirtyPaths(l)` on the acting location, the prime, and returns `ErrSourceDirty` (`internal/fabricengine/add.go:188-193`).
- **Gap:** D5 drops the dirt refusal only for "a named parent's worktree". D7 says the start guard "checks none for one forked from a beam". Neither says that Add's acting-worktree dirt check gives way when `Parent` is named. So a prime with tracked dirt would block every beam member's create, which contradicts D5's purpose that members start whatever the dirt state.

F3 — LOW, design, D4.
- **Issue:** "origin found with an empty parent reads as the prime" does not say which origin field `ParentOf` derives the parent slug from.
- **Evidence:** the existing `fabricengine.ParentSlug` returns "" both for a parent on the prime's branch and when "no live pair holds the parent branch" (`internal/fabricengine/origin.go:136-157`). `ResolveParent` reads `ParentWorktree` (`internal/hubgeom/parent.go:83`).
- **Effect:** built on `ParentSlug`, a member whose beam pair is absent locally reads as a prime member. Its owner would then read as the prime's owner and its `parent_beam` as empty, so the prime's orch's stencil filter picks it up.

F4 — LOW, design, D13.
- **Issue:** D13 starts the ready-notice walk "from the worktree of that task's `ParentBeam`" and names the prime only for an empty `ParentBeam`.
- **Gap:** when the beam's worktree is not on this machine there is no origin record to walk from, and D13 does not say what happens.
- **Evidence:** `ReadOriginFor` answers not-found for an absent weft (`internal/fabricengine/origin.go:121-126`).

**Tool calls:** none; everything needed was already in context.
