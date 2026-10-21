---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1, D2, D3
    summary: reconcile's existing stale-junction sweep classifies the new .scratch link as a fabric-owned stale junction and removes it, with an exclude strip, on every reconcile, undoing D1 and contradicting D2; the record never mentions the sweep
    fix: In D3, state that the stale-junction sweep (applyStaleRemoval via scanOnDiskJunctionNames) never claims the root .scratch link, for example by adding the scratch link name to the names that sweep skips, and add an acceptance criterion that a second lyx fabric reconcile on a converged pair keeps the .scratch link and its /.scratch exclude line and reports no stale removal.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4
    summary: D4 leaves open whether the step keeps ExcludeAnchoredDir's trailing-slash check-ignore short-circuit; in this repo the tracked .gitignore line **/.scratch/ answers ignored for that probe, so nothing is written and the link stays untracked dirt
    fix: In D4, state that the scratch step decides whether to write /.scratch by probing the slash-less path .scratch (which git matches as a non-directory for a link), or writes /.scratch unconditionally, and never uses the trailing-slash .scratch/ probe; extend the exclude acceptance criterion so its fixture carries a tracked .gitignore holding **/.scratch/ and still requires /.scratch in the exclude and a clean git status.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D2 and D1
    summary: D2 makes an exclude-write failure non-fatal, a downgrade of today's failing seedScratchExclude, with no bound; a link created before a failed exclude write is untracked dirt that trips warpclean's dirty check and git worktree remove, worse than cache misses
    fix: In D1, order the scratch step so the exclude write runs first and the link is created only after the write succeeded or the pattern is already present; in D2, add that a failed exclude write creates no link and is reported like a leave case, and widen the Bound sentence to say a failed exclude leaves the worktree as it was before the step.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D1 case 4, D2 Bound, Acceptance criteria bullet 5
    summary: after a successful rename followed by a refused link, the old content lives only at the target and its .scratch/<name> paths no longer resolve; D2's nothing-worse bound, criterion 5's target-untouched claim and the reported way forward are all wrong for that case
    fix: Split the post-rename refused link out of D1 case 4 into its own case whose report names the target path where the moved content now lives and gives the way forward as moving the recreated .scratch's content into that target, removing the root .scratch and rerunning lyx fabric reconcile; in D2's Bound, state that this case leaves the moved content reachable only at the target until the operator clears it; reword criterion 5 to exclude that case and add a criterion for it.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D3 paragraph 2 against D1 paragraph 2
    summary: on the re-wired branch repairPairWiring already reaches the step through WireJunctionsWith, which returns only an error, so D3 reads either as a second explicit run or as an inner run with no pair detail, and a re-point failure earlier in WireJunctionsWith skips the step silently
    fix: In D3, state that repairPairWiring runs the scratch step once itself on both branches, after the junction repair whether or not that repair failed, and that the step returns its outcome to repairPairWiring for the pair detail; in D1, state that WireJunctionsWith runs the step only for add, checkout and clone, or that reconcile calls a WireJunctionsWith form that skips it.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md Acceptance criteria bullet 7
    summary: criterion 7 says reconcile creates nothing for a pair whose weft worktree is missing, but reconcile recreates a missing weft and then repairs wiring, so the step legitimately creates the link; D3's qualifier at the time of the step was dropped
    fix: Reword criterion 7 to say reconcile creates no scratch link or target for an unmanaged worktree or for a pair whose weft worktree is still missing when the scratch step would run, and add that a pair whose weft reconcile recreates in the same pass gets its link.
  - id: F7
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D1 case 2 and Notes for the plan writer bullet 1
    summary: classifying a dangling .scratch link as a link to the target needs its literal target, but fslink.PointsTo and seedLyxJunction's verify branch, which the notes name as the model, resolve through EvalSymlinks and fail on a dangling link
    fix: In D1 case 2, state that the step recognises a link to the target by comparing fslink.RawTarget against the target path, so a dangling link is still classified as case 2 and not as a link to another target.
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D1 Rationale
    summary: the claim that scratch keeps today's lifetime is false when reconcile recreates a lost weft worktree, since scratch content now goes with the weft where a real warp .scratch survived it
    fix: In D1's Rationale, replace the lifetime sentence with one saying scratch now lives and dies with the pair's weft worktree, including a weft that reconcile recreates.
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: no criterion detects D2's mutation recording of link, rename and exclude writes, or D3's rule that unwire leaves the scratch link and its exclude in place
    fix: Add one criterion that a conversion records the link, the rename and the exclude write on the verb's mutations record, and one that lyx fabric unwire leaves the .scratch link and the /.scratch exclude line in place.
  - id: F10
    severity: LOW
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D2 paragraph 2
    summary: D2 requires the rename to be recorded on the mutations record, but no rename Kind exists among the fabricengine mutation kinds, and D2 does not say which kind records it
    fix: In D2, state the kind a rename is recorded under, either a new rename kind on the mutations record or the existing dir_created kind at the target path.
  - id: F11
    severity: LOW
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Open risks
    summary: the Windows junction mtime risk is handed to fabric-windows, a low-priority note taken only if Windows becomes relevant whose body never mentions scratch, and no decision adds the risk to it
    fix: Add a Decision that the run appends the scratch junction modification-time check to the fabric-windows board entry's body through lyx board, and reword the Open risks bullet to say that entry carries it.
---

### [BLOCKING] Reconcile's stale-junction sweep removes the new link

**Location:** `_lyx/discussion/decision-record.md` D1, D2, D3; evidence `internal/fabricengine/reconcile.go` `scanOnDiskJunctionNames`, `linkIsFabricOwned`, `applyStaleRemoval`.

**Issue:** `repairPairWiring` ends with `applyStaleRemoval`, which lists every link in the anchor directory, claims each one that resolves inside the slug's weft worktree, and removes those absent from `RepoWiredNames` through `removeWarpJunction`, then strips their exclude entry.
The `.scratch` link resolves to `<weft>/.lyx/scratch`, is not a wired name and is not in `HubReservedNames`, so every `lyx fabric reconcile` removes the link D3 relies on reconcile to converge.
D2's claim that nothing removes the link and D3's convergence both fail, and the record never mentions the sweep.

### [MEDIUM] D4 leaves the check-ignore probe shape open, and the existing probe writes nothing here

**Location:** `_lyx/discussion/decision-record.md` D4; evidence `internal/fabricengine/gitexclude.go` `ExcludeAnchoredDir`, `.gitignore:23`.

**Issue:** `ExcludeAnchoredDir` returns early when `git check-ignore -q -- .scratch/` succeeds.
This repo's tracked `.gitignore` holds `**/.scratch/`, so that probe exits 0, while `git check-ignore .scratch` exits 1: git does not match a non-directory against a trailing-slash pattern.
The common `info/exclude` holds only `/_lyx` and `/.lyx` today.
A plan that reuses the existing helper writes nothing, and the link in the prime and in every pair shows as untracked.
The current acceptance criterion would pass in a fixture without that `.gitignore`.

### [MEDIUM] A failed exclude write is an unbounded guard downgrade

**Location:** `_lyx/discussion/decision-record.md` D1, D2; evidence `internal/fabricengine/junction.go:119`, `internal/fabricengine/warpclean.go`.

**Issue:** Today a `seedScratchExclude` error fails `WireJunctionsWith`.
D2 makes every outcome non-fatal, and D1 does not order the link against the exclude write.
A link left without its exclude is `?? .scratch` in `git status --porcelain`, which `warpclean` reads for `preflight`'s dirty check and `git worktree remove` refuses on.
That outcome is no leave case, and D2's bound of cache misses and nothing worse does not cover it.

### [MEDIUM] Post-rename refused link breaks D2's bound and criterion 5

**Location:** `_lyx/discussion/decision-record.md` D1 case 4, D2 Bound, Acceptance criteria bullet 5.

**Issue:** When the rename succeeds and `CreateDirLink` then refuses because `.scratch` was recreated, the old content sits only at the target.
Its `.scratch/<name>` paths, which board entries cite for the prime, stop resolving, and cleanup's scratch segment no longer sees it.
The reported way forward does not say the content has already moved or where it went.
Criterion 5 asserts that the target is untouched, but the rename created it.

### [MEDIUM] D3's re-wired branch reads two ways

**Location:** `_lyx/discussion/decision-record.md` D3 paragraph 2 against D1 paragraph 2; evidence `internal/fabricengine/reconcile.go` `repairPairWiring`.

**Issue:** On the re-wired branch, `repairPairWiring` already calls `WireJunctionsWith`, which by D1 runs the scratch step and returns only an error.
D3 says reconcile runs the step from `repairPairWiring` on both branches and adds a detail.
That reads either as a second, explicit run or as the inner run, which has no path into `pr.Detail`.
When the junction re-point fails before the step, the step does not run that pass, and D3 never says so.

### [MEDIUM] Criterion 7 contradicts the recreated-weft path

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria bullet 7; evidence `internal/fabricengine/reconcile.go` `Reconcile` loop, `repairWiring`.

**Issue:** `reconcileMissingWeft` recreates a missing weft, and `repairWiring` then falls through to `repairPairWiring`, where by D3 the step runs with the weft present and creates the link.
The criterion drops D3's qualifier "at the time of the step", and a test built to it would fail or suppress the step.

### [LOW] Dangling-link classification uses a resolver that fails on dangling links

**Location:** `_lyx/discussion/decision-record.md` D1 case 2, Notes bullet 1; evidence `internal/fslink/fslink_linux.go` `PointsTo`, `RawTarget`.

**Issue:** `PointsTo` errors when the target is missing.
Under classify-first order, a dangling link can be told apart from a link to another target only by its literal target, which `fslink.RawTarget` supplies.
D1 does not say this, and the model the notes name resolves the target and errors.

### [LOW] Lifetime claim false on weft re-creation

**Location:** `_lyx/discussion/decision-record.md` D1 Rationale.

**Issue:** A real warp `.scratch` survived a lost and recreated weft.
Under D1 its content is lost with the weft, and case 2 only heals the dangling link.

### [LOW] Criteria miss D2's recording and D3's unwire rule

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria.

**Issue:** Criterion 3 covers only the no-op that records nothing.
No criterion detects a missing mutation entry on conversion, or a scratch link that unwire removes, though `dotlyxjunction_integration_test.go` pins today's unwire behaviour for the exclude line.

### [LOW] Rename has no mutation kind

**Location:** `_lyx/discussion/decision-record.md` D2 paragraph 2; evidence `internal/fabricengine/mutation.go` `Kind` constants.

**Issue:** No existing kind names a rename, so the plan writer has to pick one D2 never names.

### [LOW] Windows risk hand-off has no owner

**Location:** `_lyx/discussion/decision-record.md` Open risks; evidence board entry `fabric-windows`.

**Issue:** `fabric-windows` is a low-priority note, taken only if Windows becomes relevant, and its body does not mention scratch or junction modification time.
Nothing in the record puts the measurement there.
Writing another entry's body is outside the rubric's own-entry bullet.

## Focus verdicts

- Goal and D1 premise: on Linux the premise holds.
  `hashOpen` hashes `f.Info()`, an lstat of each entry, and a symlink's own modification time is fixed at creation; `scankit`'s `WalkDir` neither stats nor opens the link, and no test uses `filepath.Walk`.
  The Windows case stays an open risk, handled in F11.
- D1 cases 3 and 4: F4.
- D3 reach rules: the managed and weft-present conditions match the reconcile loop, since raw-adopted and unmanaged pairs never reach `repairPairWiring`; the branch ambiguity is F5, and F1 undoes the result.
- D4 with D2: the rewrite touches the common gitdir's exclude, shared by the prime and every sibling.
  `/.scratch` matches everything `/.scratch/` matched, and unwire's strip set covers only wired names, so no worktree loses coverage.
  The probe and failure gaps are F2 and F3.
- Criteria against Decisions: F6 and F9; criterion 5 also in F4.

## Rejected

- Anchor geometry of the `.scratch` link for subpath-anchored hubs: the existing exclude seeding and cleanup's scratch segment already treat `.scratch` as the worktree root, so the record follows established geometry and introduces no new gap.
