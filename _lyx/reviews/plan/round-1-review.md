---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/01-refuse-before-archive.md:46
    summary: The weft-dirty test asserts only the sentinel and the absent tags, not the rest of acceptance criterion 1 (no reconcile text, portal, launchers, both worktrees, both branches, empty mutation record)
  - id: F2
    severity: LOW
    location: _lyx/plan/00-overview.md:29
    summary: The verify section runs integration tests for three packages, where the decision record's open risk requires the full integration suite
  - id: F3
    severity: LOW
    location: _lyx/plan/01-refuse-before-archive.md:68
    summary: ImpactSummary names battencli but omits fabriccli, the other production caller, and the behaviour change callers actually see
  - id: F4
    severity: NIT
    location: _lyx/plan/01-refuse-before-archive.md:44
    summary: The planted same-tip tag must be lightweight, since archiveWeftTip compares the ref's objectname to the tip and an annotated tag fails that comparison
---

Evidence gathered by driving the substrate: I copied the tree into the session scratchpad, applied the D1 reorder to `remove.go` (dirtiness checks and probes ahead of `archiveWeftTip`, archive ahead of `removePortal`/`removeLaunchers`, `nameStrandedPortalTeardown` calls unwrapped), and ran `go test -tags integration ./...` across the whole repo.
Exactly four tests failed: `TestCrossProduct` (the five `removeCase` dirty cells at both anchors), `TestRemove_ResumedRemoveReusesArchiveTag`, `TestMutationRecord_RemoveDirtyWarpRefusalCarriesThePortalAndLauncherDeletions`, and `TestRemove_RefusalNamesStrandedPortalTeardown`.
Every one is a test Card 1 names; nothing in `battencli`, `fabriccli`, `ideengine` or elsewhere depends on the old order (`ideengine`'s `spawn_hub_integration_test.go` calls `Remove` with `force=true`).
The hermetic `go test` over the three packages passed.
I also wrote prototypes of the card's three new tests (same-tip reuse, weft-dirty refuse-then-resume, broken-gitfile warp probe failure) and the `removeCase` inversion with `PermittedRoots` dropped and `assertExists` effects; all passed against the reordered `Remove`, so every test the card specifies is feasible as described.

Focus settlements with no finding:

- Card 1's Intent order matches D1 step for step, the two status probes sit with their dirtiness checks ahead of `archiveWeftTip`, the archive precedes `removePortal`/`removeLaunchers`, and `force` skips only the two dirtiness checks (D4, D5).
- Scope: the `livestate_doc_test.go` anomaly-section deletion falls under Scope In's "comments and package doc that describe the old order", and `TestRemove_WarpStatusProbeFailurePushesNoTag` is licensed by the acceptance criterion on a failed warp probe.
  `cleanup.go`, `archive.go`, `battencli`'s `teardownRefusal` and old-tag pruning are all left alone.
- Other old-order dependants: `livestate_verbs_test.go`'s header (lines 24-26), `livestate_refusal_selftest_test.go`, `livestate_refusal_test.go`, `livestate_states_test.go` and `remove_siblingdirty_integration_test.go` cite `Remove`'s pre-flight by line number or by message text only; none pins teardown-before-refusal, and the empirical run confirms it.
  `removeCase`'s doc comment at `livestate_verbs_test.go:590` is covered by the card's "Rewrite `removeCase`'s doc comment".
- Granularity: the production reorder alone fails the four tests above, so a separate `remove.go` card would land a red commit; one card is justified.

### [MEDIUM] Weft-dirty refusal test under-asserts acceptance criterion 1

**Location:** `_lyx/plan/01-refuse-before-archive.md:46`
**Issue:** Acceptance criterion 1 is stated for the weft side: a no-force `Remove` with uncommitted weft changes returns `ErrPairSiblingDirty` without mentioning `lyx fabric reconcile`, and afterwards no archive tag exists on origin or locally, the portal, launchers, both worktrees and both branches exist, and the mutation record is empty.
`TestRemove_WeftDirtyRefusalPushesNoTagAndResumeArchivesOnce` asserts only the sentinel and the two empty tag lists.
The full intact-pair assertions live only in `TestRemove_DirtyRefusalLeavesPairIntact`, which drives the warp-dirty path (criterion 2).
The weft refusal comes from a different call (`refuseDirtyWeftWorktree`) at a different step, so an implementation that ordered the weft check after the portal teardown would pass every test the card specifies.
**Fix:** Make the weft-dirty test assert, after the refused call, that the error does not contain `lyx fabric reconcile`, `res.Mutated().Len()` is 0, and the portal link, launcher dir, both worktrees, the warp branch and the weft branch all exist.

### [LOW] Verify runs a narrower integration set than the open risk requires

**Location:** `_lyx/plan/00-overview.md:29`
**Issue:** The decision record's open risk says "the full integration suite must run to surface any beyond those D6 names".
The verify section runs `-tags integration` over `fabricengine`, `battencli` and `fabriccli` only.
My scratch run found no dependant outside those packages, but the verify guards the implementation that lands, not my prototype.
**Fix:** Replace the package-scoped integration line with `go test -tags integration ./...`.

### [LOW] ImpactSummary omits a production caller and the observable change

**Location:** `_lyx/plan/01-refuse-before-archive.md:68`
**Issue:** `Topology.Remove` has two production callers: `fabriccli` (`fabric.go:892`, `lyx fabric remove`) and `battencli` (`wire.go:437`).
The summary names only `battencli`, and "step order" does not state what either caller observes: a refused call now leaves the pair and the weft origin untouched instead of having pushed a tag and removed the portal and launchers.
**Fix:** State both callers and the observable change in one line.

### [NIT] The planted reuse tag must be lightweight

**Location:** `_lyx/plan/01-refuse-before-archive.md:44`
**Issue:** `archiveWeftTip` reads the existing tag with `for-each-ref --format=%(objectname)` and reuses it only when that equals the tip.
An annotated tag's objectname is the tag object, not the commit, so a test planting `git tag -a` would hit the "already exists at … not the weft tip" error instead of the reuse path.
The card says "creating `archive/<slug>/<tip[:12]>` at the tip" without saying lightweight.
**Fix:** Say the planted tag is lightweight (`git tag <name> <tip>`), matching what `archiveWeftTip` itself creates.
