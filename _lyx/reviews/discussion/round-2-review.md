---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 (second paragraph) and Acceptance criteria
    summary: D3's claim that lyx fabric remove still unlinks the root .scratch link through the shared scan is false on a subpath-anchored hub, and no acceptance criterion pins the remove-side unlink
    fix: In D3, replace the sentence starting "The shared scan itself keeps returning the link" with a statement that `lyx fabric remove` unlinks the worktree-root `.scratch` link itself, by its root path and independent of `scanOnDiskJunctionNames` (which reads `<worktree>/<AnchorRel>`), through the destroy.go link-removal gate recorded as `link_removed`, before it removes the warp worktree directory; state that the stale-removal and unwire skip of the scratch name applies to that same root entry. Add an acceptance criterion - `lyx fabric remove` on a pair with a wired `.scratch` link, at a root anchor and at a subpath anchor, records `link_removed` for the link before the worktree removal and leaves the target's content untouched until the weft teardown.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1 cases 1-2, D2 Bound, Acceptance criteria
    summary: A failed target MkdirAll (cases 1 and 2) and a refused link creation in case 1 have no stated outcome, report or bound, and the case-1 race leaves an empty target that turns every later pass into a permanent case-4 leave
    fix: In D1, add a case after case 5 - the target MkdirAll fails, or case 1's link creation is refused - the step leaves the root `.scratch` as it found it and reports the path, the error and the way forward (clear what blocks the target or the root `.scratch`, then rerun `lyx fabric reconcile`). In D2's Bound, add that this case leaves the worktree's cache misses and, after a case-1 refusal, an empty target that makes the next pass a case-4 leave until the operator clears it as the report says. Add an acceptance criterion - a refused case-1 link creation and a failed target creation each succeed, create no link and report the path and the way forward.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1 case 3 and D2 Bound
    summary: Case 3 renames a real .scratch out of the worktree with no bound on tracked content, so a warp branch that tracks files under .scratch/ gets them deleted from git's view, leaving a dirty tree that blocks a non-forced remove
    fix: In D1, make case 3 apply only when no tracked path lies under the root `.scratch` (asked of git through `internal/fabricengine`'s own git path), and add a real `.scratch` holding tracked content to case 4's leave list with the same report. In D2's Bound, state that the rename never moves tracked content. Add an acceptance criterion - wiring a worktree whose branch tracks a file under `.scratch/` leaves the directory and its files in place, creates no link and reports the leave case.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 first paragraph and D4
    summary: D1 runs the scratch step after the existing junction and exclude steps without saying that it replaces the seedScratchExclude call, which still writes the trailing-slash /.scratch/ behind a check-ignore probe that D4 rejects
    fix: In D1's first paragraph, after "after the existing junction and exclude steps", add that the scratch step replaces `seedScratchExclude`'s call in `WireJunctionsWith`, `seedScratchExclude` is deleted, and `ExcludeAnchoredDir` stays for its other callers.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2 (path_renamed sentence)
    summary: D2 says path_renamed lands with its cmd/lyx/destructiveguard_test.go guard entry, but that guard file holds no per-Kind table, so the plan writer is sent to a slot that does not exist
    fix: In D2, replace "it lands with its recording site and its `cmd/lyx/destructiveguard_test.go` guard entry, as the `Kind` doc requires" with "it lands in the same commit as its recording site".
---

### [MEDIUM] Remove's unlink of the scratch link rests on a scan that cannot see it on a subpath anchor

**Location:** `_lyx/discussion/decision-record.md` D3, second paragraph, and `## Acceptance criteria`.

**Issue:** D3 says the shared scan keeps returning the link, "so `lyx fabric remove` still unlinks it through `removeWarpJunction`".
`scanOnDiskJunctionNames` (`internal/fabricengine/reconcile.go:725`) reads `filepath.Join(WorktreePath(l, slug), l.AnchorRel)`, not the worktree root, and `Remove` (`internal/fabricengine/remove.go:250`) unlinks only what that scan returns.
On a subpath-anchored hub the root `.scratch` link is never returned, so remove hands a live link into the weft to `removeWarpWorktreeDir` and its gated `removePath` fallback.
The unlink-before-removal order is the existing guard against a recursive delete walking into weft content through a junction, which is the Windows case `fabric-windows` exists for.
No acceptance criterion pins the remove-side unlink at either anchor, so the focus question "do the acceptance criteria detect every mutation the Decisions state" fails for this `link_removed`.

### [MEDIUM] Target creation and case-1 link refusal have no outcome

**Location:** `_lyx/discussion/decision-record.md` D1 cases 1 and 2, D2 Bound, `## Acceptance criteria`.

**Issue:** D1 lists five outcomes and D2 bounds three of them, but cases 1 and 2 each start with `MkdirAll` of the target and case 1 then creates the link.
Neither a failed `MkdirAll` nor a refused link creation in case 1 is named, reported or bounded; case 5 covers a refusal only after a successful rename.
D2's "no outcome fails wiring" means these must succeed somehow, but the record leaves the report and the resulting worktree shape for the plan writer to invent.
The case-1 race leaves a worse residue than the record admits: the target now exists empty while something recreated a real `.scratch`, so every later pass classifies it as case 4 ("a real directory while the target already exists") and never converges, with no report naming why.

### [MEDIUM] Case 3 moves tracked content

**Location:** `_lyx/discussion/decision-record.md` D1 case 3 and D2 Bound.

**Issue:** Fabric wires arbitrary warp repos, and case 3 renames any real root `.scratch` whose target is absent.
A branch that tracks files under `.scratch/` checks them out as a real directory on `lyx fabric add` or `checkout`, the step renames them into the weft, and git then reports every one as deleted.
The `/.scratch` exclude does not hide tracked paths, so the worktree is dirty, a non-forced `lyx fabric remove` refuses, and a later checkout that touches those paths meets a link where a directory was.
`ExcludeAnchoredDir`'s own doc (`internal/fabricengine/gitexclude.go:93`) already marks tracked content under such a directory as a case the caller must not reach.
The rename is a new mutation edge, and D2's bound ("nothing worse" than cache misses) does not hold for it.

### [LOW] The record never retires `seedScratchExclude`

**Location:** `_lyx/discussion/decision-record.md` D1 first paragraph and D4.

**Issue:** `WireJunctionsWith` ends with `seedScratchExclude` (`internal/fabricengine/junction.go:119`), which probes `git check-ignore .scratch/` and appends `/.scratch/`.
D1 places the scratch step "after the existing junction and exclude steps", which reads as keeping that call.
Kept, it writes the trailing-slash line D4 rejects on a repo without a tracked rule, and the scratch step then replaces it in the same wire, recording two `file_written` entries for one convergence.
D4's "the scratch step owns the exclude" implies the replacement, but the record never says it.

### [LOW] D2 cites a guard entry that has no slot

**Location:** `_lyx/discussion/decision-record.md` D2, the `path_renamed` sentence.

**Issue:** `cmd/lyx/destructiveguard_test.go` holds tables of banned tokens, recording executors in `destroy.go` and mutating result types; it has no per-Kind table, and its header says a new Kind is caught by nothing there.
The `Kind` doc's sentence the record copies is stale, so a plan writer following D2 looks for an entry to add and finds none.

## Focus verdicts

- Stale-junction removal: not a defect at a root anchor.
  `linkIsFabricOwned` claims the link because its target lies inside the weft worktree, D3 drops the scratch name in `applyStaleRemoval` and `Unwire`, and the second-reconcile acceptance criterion pins the link, its exclude line and the absent stale report.
  The subpath-anchor gap in the shared scan is F1.
- Exclude under a tracked `**/.scratch/`: not a defect.
  D4 writes `/.scratch` with no probe, its rationale names the false-positive probe correctly, and an acceptance criterion pins the fixture; the leftover `seedScratchExclude` call is F4.
- Step ordering and failure bounds: the exclude-failure and case-5 bounds match their acceptance criteria; the unbounded cases are F2 and F3.
- Reconcile entry paths: not a defect.
  `repairPairWiring` is reached on the existing-weft branch and on the `weft_recreated` branch with no error (`internal/fabricengine/reconcile.go:235`), never on a failed recreate or a raw adoption, and D3 places the step once there after the junction repair on both branches, with the re-wire form skipping it.
- Mutation kinds and other board entries: D2 names `link_created`, `path_renamed` and `file_written`, and D6 names `fabric-windows`, an existing note, and the content it appends; neither touches this task's own entry.
  The guard-entry slip is F5 and the missing remove criterion is in F1.
