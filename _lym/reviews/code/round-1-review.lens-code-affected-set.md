```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the diff, I found what calls, reads or asserts it and checked whether the diff leaves any of it wrong.

**List, per change:**

1. **`reviewTargetSnapshot` root moved from `env.WorktreeRoot` to `env.AnchorPath`** (`internal/shedrecipe/entries_burler.go:59`). Set found:
   - Callers: the `burlerReviewEntry` snapshot (`entries_burler.go:182`) and the Bouncer snapshot (`entries_bouncer.go:312`). Covered: both now share the anchor root.
   - Root the reviewers and fixer resolve target paths against: `hubgeom.burlerGeometry` sets `WorktreeRoot: l.AnchorPath()` (`internal/hubgeom/hubgeom.go:169`). `loomcli` builds the recipe Env with `AnchorPath: location.AnchorPath()` (`internal/loomcli/wiring.go:913`) and uses `hubgeom.BurlerGeometry` (`wiring.go:718`). Covered: the roots agree.
   - `standalonegeom.BurlerGeometry` (`WorktreeRoot: target`, `AnchorPath: stateDir`) never builds a `shedrecipe.Env`. Its only `shedrecipe.Env` sites are `loomcli/wiring.go`, `loomcli/landingdeps.go` and `battencli/wire.go`. Not affected.
   - `battencli`'s Env sets no `AnchorPath`, but no batten row is a BurlerReview or Bouncer row. Not affected.
   - `shuttleparity/seats.go:109-110` builds a `seatengine.Geometry`, not a snapshot. Not affected.
   - Other readers of the round's target paths: the Bouncer's `artifact_paths` already resolve under `env.AnchorPath` (`entries_bouncer.go:259`). `recordPostFixTargetID` reuses the configured snapshot's root (`bouncerstalemate.go:31-32`). `recordRoundChanges` and `ChangedSince` use the same snapshot (`roundcontext.go:312`). Covered: they all use one root.
   - The Bouncer's `"worktree"` prompt key (`bouncer.go:1738`, `bouncerverify.go:283`) is a prompt value, not where target paths resolve. Not affected.
   - Test: `TestBurlerReviewEntry_ProfileMapping/OverlayTargetRecordsTheRoundSnapshot` was updated (`entries_burlerhalves_test.go:242-255`). The fixture's separate `anchor` and `worktree` dirs (`fixture_test.go:273-275`) make it discriminate. Covered.
   - Docs, specs, stencils and PATTERN: no spec, stencil, `PATTERN.md` or `pattern/` text describes the snapshot root. `lymdirs.ReviewSnapshotDirName`'s doc (`internal/lymdirs/dirs.go:108`) concerns the cache dir only, which was already anchor-rooted. Nothing stale.
   - `TargetSnapshot`'s field docs (`internal/shedadapters/targetsnapshot.go:30-33`): left inaccurate, see finding 1.

2. **`overlayID` returns an error when no target file exists** (`internal/shedadapters/targetsnapshot.go:114-116`, doc at `:45`). Set found:
   - `Take` (`:59-64`) propagates the error and skips `copyOverlay`.
   - `recordRoundStart` (`burlerreview.go:229-231`) logs at Warn and records an empty `TargetID`.
   - `recordPostFixTargetID` (`bouncerstalemate.go:33-37`) logs at Warn and records an empty `PostFixTargetID`.
   - `roundUnchanged` (`bouncerstalemate.go:55`) and `fixLeftTargetUnchanged` (`bouncerverify.go:185`) treat an empty `TargetID` as changed.
   - `recordRoundChanges` (`roundcontext.go:298`) writes no changes file when the previous round's id is empty.
   - The `roundsnapshot.go:23` doc ("TargetID is empty when … its id could not be read") already covers this. Covered.
   - Test: `TestTargetSnapshot/an overlay target with no file is an error` (`targetsnapshot_test.go:87-97`). Covered.
   - No caller, test or doc asserts the new error message text.

3. **Commit messages and departures:** `4a23f3a41` and `f635cccab` each carry a `Root-Cause:` trailer, and each trailer names the defect its diff touches. No `Pattern-Departure:` or `Loosened-Check:` trailer appears, and none is needed: no test assertion was loosened, since the one changed row moves its fixture and keeps the exact `TargetID` equality. Covered.

4. **Declared PATTERN changes:** none are declared, and the diff touches no `PATTERN.md` or `pattern/` file. **Stencil wording:** no stencil changed. **Repository rules:** the changed code adds no hard-coded path or branch, no new I/O in a leaf package, and no git call. Covered.

**Findings:**

1. **LOW, consistency.** At `internal/shedadapters/targetsnapshot.go:30-33`, `TargetSnapshot.Paths` is documented as "the worktree-relative target paths". After the diff, its one production builder (`internal/shedrecipe/entries_burler.go:59`) roots it at the anchor path, so the paths are anchor-relative on a subpath-anchored hub. The same doc's `WorktreeRoot` field name reads as the worktree root it no longer holds. The field doc should say "root-relative" or "relative to WorktreeRoot". Evidence: `targetsnapshot.go:30-33` against `entries_burler.go:59` and `hubgeom.go:169`.

No other member of the affected set is left wrong.

Tool calls:
- Grep of docs, specs, stencils and patterns for snapshot wording: my context lacked any doc-side search.
- Second, narrower grep of specs, contracts and `PATTERN.md`, plus a `ReviewSnapshotDirName` search: the first grep was dominated by research tables.
- Grep of the Bouncer's readers of `WorktreeRoot`, `ArtifactPaths` and `TargetPaths`: the context lacked the other readers of the round's target paths.
- Read of `entries_bouncer.go:250-266`, `bouncerverify.go:275-290` and `bouncer.go:1730-1742`: these confirm the artifact-path root and the use of the `"worktree"` prompt key.
