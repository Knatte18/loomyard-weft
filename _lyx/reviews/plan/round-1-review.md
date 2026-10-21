---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/02-reconcile-converges-scratch.md (Intent, repairPairWiring paragraph)
    summary: D3 bounds the step on a weft root missing at the time of the step, but card 2 relies only on the repairWiring condition evaluated at iteration start, so a weft torn down mid-repair gets its .lyx/scratch path recreated as a plain directory
    fix: In card 1's Intent, have wireScratchLink os.Stat RecordsWorktreePath(l, slug) immediately before its first MkdirAll or Rename and return an empty outcome, creating nothing, when it is absent; in card 1's serial TestWireJunctions_ScratchLinkRefusedAfterClassify add a row whose SetScratchPathTrackedForTest fn removes the pair's weft worktree directory and returns false, asserting WireJunctionsWith returns nil and neither the target nor a root link exists; in card 2's Intent replace "no guard of its own is added" with a pointer to that guard.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/01-scratch-link-wiring.md (Intent, wireScratchLink steps 2 and 3)
    summary: The tracked leave, the case-4 shapes a link to another target and a regular file, and the case-6 reports tell the operator to move content into the target without naming the target path
    fix: In card 1's Intent, state that every left report whose way forward moves content into the target names the scratchLinkTarget path, and add the target path to the tracked-leave, the two case-4 shapes and every case-6 report.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/01-scratch-link-wiring.md and _lyx/plan/02-reconcile-converges-scratch.md (Intent)
    summary: The doc comments of Unwire, applyStaleRemoval and Topology.Remove's sweep still claim every on-disk fabric junction is removed or swept, and neither card names their rewrite
    fix: In card 1's Intent, add that Unwire's doc comment (internal/fabricengine/unwire.go:55-57) and Topology.Remove's sweep comment (internal/fabricengine/remove.go:240-245) say the root scratch link is excepted; in card 2's Intent, add the same for applyStaleRemoval's doc comment (internal/fabricengine/reconcile.go:802-804).
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/01-scratch-link-wiring.md (Tests, TestWireJunctions_ScratchLinkRefusedAfterClassify)
    summary: The card never says a row installs the hook or tracked seam only after hubforge.CopyHub has built its hub, so a lazily built template wired through CloneAndWire would fire the hook and be poisoned for every later CopyHub
    fix: In card 1's description of TestWireJunctions_ScratchLinkRefusedAfterClassify, and in card 2's Reconcile rows added to it, state that each row builds and arranges its hub before calling SetScratchBeforeLinkHookForTest or SetScratchPathTrackedForTest.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/01-scratch-link-wiring.md (Tests, scratchShapeCases failed-exclude-write row)
    summary: The failed-exclude-write row reaches the scratch step's exclude failure only when the exclude lock file already exists, else seedGitExclude fails first and WireJunctionsWith returns an error against the row's returns-nil assertion
    fix: In card 1's description of the failed-exclude-write row, state that its arrange asserts the info directory's exclude lock file exists before chmodding the directory 0o555.
  - id: F6
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/01-scratch-link-wiring.md (Intent, case 3 failed parent MkdirAll)
    summary: The prescribed case-3 parent MkdirAll failure has no named row, and it is reachable only through reconcile after a failed junction repair
    fix: In card 2's TestReconcile_ScratchLink, add a row whose prime pair's weft-side .lyx is replaced by a regular file, root .scratch a real directory with a file and target absent, asserting the pair's Detail names the root .scratch and lyx fabric reconcile and the root directory is still a real directory with its file.
  - id: F7
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/plan/01-scratch-link-wiring.md (Intent, scratchPathTracked paragraph)
    summary: The card places SetScratchBeforeLinkHookForTest in export_test.go but never says where SetScratchPathTrackedForTest lives, though it takes a *testing.T and must not ship in production source
    fix: In card 1's Intent, state that SetScratchPathTrackedForTest lives in internal/fabricengine/export_test.go beside SetScratchBeforeLinkHookForTest, so production code cannot swap scratchPathTracked.
  - id: F8
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/plan/03-remove-unlinks-scratch.md (Uses)
    summary: Card 3 lists plan:internal/fabricengine#scratchLinkTarget under Uses, but its Intent never uses it
    fix: Remove the plan:internal/fabricengine#scratchLinkTarget line from card 3's Uses list.
---

The plan carries every Decision and Constraint of the decision record.
D1 through D5 are in cards 1 to 4, and D6 is card 4's board write with its `**Verify:**`.
Every acceptance criterion has a row or an extended assertion, and no card edits this task's own board entry.
The kept findings are all LOW or NIT.

### [LOW] Step may recreate a weft torn down mid-repair

**Location:** `_lyx/plan/02-reconcile-converges-scratch.md`, Intent, `repairPairWiring` paragraph.
**Issue:** D3 says the step never runs "on a pair whose weft worktree root is missing at the time of the step; with no weft root, the step creates nothing".
Card 2 relies on `Reconcile`'s `repairWiring` condition, which is evaluated at iteration start (`internal/fabricengine/reconcile.go:211-236`).
Reconcile's own comments say a concurrent teardown can land mid-repair (`reconcile.go:245-250`, `markVanishedMidWalk` at `reconcile.go:611-623`).
The step's case-1 or case-2 `MkdirAll(target)` would then recreate `<weft>/.lyx/scratch` as a plain directory.
The next pass's `os.Stat(weftPath).IsDir()` takes that directory for an existing weft worktree.
The proposed guard is reached through the tracked-check seam, which runs before classification, so it needs no new target.

### [LOW] Ways forward name "the target" without its path

**Location:** `_lyx/plan/01-scratch-link-wiring.md`, Intent, `wireScratchLink` steps 2 and 3.
**Issue:** These reports name the target path: the case-4 shape "a real directory while the target <target> already exists", the failed-rename shape, and case 5.
The tracked leave, the shapes "a link to another target" and "a regular file", and the case-6 reports also say "move the content into the target", but they never give its path.

### [LOW] Three doc comments still claim every junction is swept

**Location:** cards 1 and 2, Intent.
**Issue:** Card 1 rewrites `UnwireJunctions`'s doc and `unwire.go`'s header for the root-link exception.
These comments make the same claim and stay unnamed:
- `Unwire`'s doc comment (`internal/fabricengine/unwire.go:55-57`);
- `applyStaleRemoval`'s doc comment (`internal/fabricengine/reconcile.go:802-804`);
- `Topology.Remove`'s sweep comment (`internal/fabricengine/remove.go:240-245`).

All three functions are already Edit targets of those cards, so the fix adds no target.

### [LOW] Seed order of hook against template build unstated

**Location:** `_lyx/plan/01-scratch-link-wiring.md`, Tests, `TestWireJunctions_ScratchLinkRefusedAfterClassify`.
**Issue:** `hubforge.CopyHub` builds its template lazily through `templateFor` (`internal/hubforge/template.go:149-165`).
That build wires the prime through `CloneAndWire`, which calls `WireJunctionsWith` (`internal/fabriccli/clone.go:95`).
A hook installed before the first `CopyHub` of the package would fire during the template build and poison the shared template.

### [LOW] Failed-exclude row depends on an existing lock file

**Location:** `_lyx/plan/01-scratch-link-wiring.md`, Tests, `scratchShapeCases` failed-exclude-write row.
**Issue:** `mutateGitExclude` acquires `info/<gitExcludeLockFileName>` before it reads (`internal/fabricengine/gitexclude.go:52-75`).
With `info` at `0o555` and no lock file present, `seedGitExclude` in `wireJunctionSet` fails first.
`WireJunctionsWith` then returns an error, against the row's assertion that it returns nil.

### [LOW] Case-3 parent MkdirAll failure has no row

**Location:** `_lyx/plan/01-scratch-link-wiring.md`, Intent, case 3.
**Issue:** Under `WireJunctionsWith`, `seedLyxJunction` has already materialised the weft-side `.lyx` before the step runs (`internal/fabricengine/junction.go:92-120`).
The branch is therefore reachable only under reconcile, after a failed junction repair, and no card names a row for it.

### [NIT] Seam setter location unstated

**Location:** `_lyx/plan/01-scratch-link-wiring.md`, Intent.
**Issue:** `SetScratchBeforeLinkHookForTest` is placed in `export_test.go`, while `SetScratchPathTrackedForTest`'s file is not named.
The bound on the new swap seam rests on its setter living in a test file.

### [NIT] Unused Uses entry on card 3

**Location:** `_lyx/plan/03-remove-unlinks-scratch.md`, Uses.
**Issue:** `scratchLinkTarget` is listed, but `unlinkScratchLink` uses `linkIsFabricOwned` and `ownedDriftedWiredJunction` and never builds the target.

## Rejected

- Focus item on card 1's Create granularity, covering `SetScratchBeforeLinkHookForTest`, `SetScratchPathTrackedForTest`, `scratchShapeCase` and `scratchOutcome.detail`: not a defect. The granularity rule governs cards, and all of these sit in the one card that owns the scratch step, with one glyph per created symbol as the target grammar requires.
- Card 3's use of `ownedDriftedWiredJunction` rather than `ownedWiredJunction` for a teardown: not a defect. `ownedWiredJunction` would refuse a claimed link that resolves inside the weft but elsewhere than the scratch target, and that refusal would fail `Remove` through `surfaceRefusal`. The drifted kind only unlinks and never follows the link, and `linkIsFabricOwned` has already bounded the link to the pair's weft.
- Card 1's edits to the `junction.go` and `unwire.go` file headers, which no target covers: not a defect. Webster's scope guard reports such deviations as informational only (`internal/websterengine/doc.go:64-71`), and adding the file self glyph beside its member glyphs would trip `redundant-file-target`.
- Test economy across cards 1 to 4: no redundant test found. Each new test function covers behaviour no existing test covers: wiring shapes, seam-driven cases 5 and 6, reconcile reporting, and the remove unlink at a root anchor. The subpath unlink and the health checks extend existing tests.
- Card order: cards 2 to 4 use only what card 1 creates or what already exists. Card 1 carries the `Unwire` and `Remove` filters that its own link makes necessary.
- Verify coverage: `## verify:` runs the integration tier over every targeted package: `internal/fabricengine/...`, `internal/cleanupcli/...` and `cmd/lyx`. Card 4's board write targets no package.
- Re-sign arrows: no card changes the parameters or results of an existing member. `wireJunctionSet` is a Create.
