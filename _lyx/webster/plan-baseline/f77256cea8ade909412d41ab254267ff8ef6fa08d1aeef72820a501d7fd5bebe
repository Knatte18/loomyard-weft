# Card 1 — scratch-link-wiring

**Create:**
- `internal/fabricengine#KindPathRenamed`
- `internal/fabricengine#scratchTargetName`
- `internal/fabricengine#scratchLinkTarget`
- `internal/fabricengine#scratchOutcome`
- `internal/fabricengine#scratchOutcome.detail`
- `internal/fabricengine#wireScratchLink`
- `internal/fabricengine#seedScratchLinkExclude`
- `internal/fabricengine#withoutRootScratchLink`
- `internal/fabricengine#scratchBeforeLinkHook`
- `internal/fabricengine#wireJunctionSet`
- `internal/fabricengine#SetScratchBeforeLinkHookForTest`
- `internal/fabricengine#scratchPathTracked`
- `internal/fabricengine#SetScratchPathTrackedForTest`
- `internal/fabricengine_test#scratchShapeCase`
- `internal/fabricengine_test#scratchShapeCases`
- `internal/fabricengine_test#TestWireJunctions_ScratchLink`
- `internal/fabricengine_test#TestWireJunctions_ScratchLinkRefusedAfterClassify`

**Edit:**
- `internal/fabricengine#WireJunctionsWith`
- `internal/fabricengine#scratchDirName`
- `internal/fabricengine#KindDirCreated`
- `internal/fabricengine#UnwireJunctions`
- `internal/fabricengine#Unwire`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine_test#manifestObservableKind`
- `internal/fabricengine_test#entryCovers`
- `internal/fabricengine_test#TestDotLyxJunction_LifecycleWiresSeedsBothExcludesAndUnwires`
- `internal/fabricengine_test#TestUnwire_RemovesOnDiskJunctionsIncludingStale`
- `internal/fabricengine_test#TestCodeWorktrees_ListOpenAndPairComplete`
- `cmd/lyx#uncontainedWriteAllowlist`

**Delete:**
- `internal/fabricengine#seedScratchExclude`

**Prosa:**
- `internal/fabricengine/doc.go`
- `internal/fabricengine/livestate_doc_test.go`

**Uses:**
- `internal/fabricengine#PathTracked`
- `internal/fabricengine#RecordsWorktreePath`
- `internal/fabricengine#hubRelativeTarget`
- `internal/fabricengine#scanOnDiskJunctionNames`
- `internal/fabricengine_test#CaptureManifest`
- `internal/fabricengine_test#DiffManifest`
- `internal/fabricengine_test#AssertRecordMatchesDiff`
- `internal/fabricengine_test#containsLine`
- `internal/fabricengine#Healthy`
- `internal/testkit/logcapture#Capture`

**Intent:**
Fabric wiring makes a warp worktree's root `.scratch` a directory link into the pair's weft-side `.lyx`, so the root listing a source-scan test opens holds a link whose own `Lstat` never moves when files under it change (D1, D2, D4).

New file `internal/fabricengine/scratchlink.go` holds the scratch step.
`scratchLinkTarget` returns `filepath.Join(RecordsWorktreePath(l, slug), l.AnchorRel, lyxdirs.DotLyxDirName, scratchTargetName)`, built like `WarpJunctions` builds every weft-side target and never by resolving the warp `.lyx` link.
The root entry is `filepath.Join(WorktreePath(l, slug), scratchDirName)`, at the worktree root on a subpath anchor too.
`scratchOutcome` carries two strings: `changed`, what the step changed (empty when nothing), and `left`, the leave or failure report with its way forward (empty when none); `detail` joins the non-empty ones with "; ".

`wireScratchLink` runs, in this order:

1. `seedScratchLinkExclude` on the warp worktree. It rewrites the warp `.git/info/exclude` through `mutateGitExclude`: it drops every line that trims to the legacy `/.scratch/` and appends `/.scratch`, no trailing slash, when absent; it runs no `git check-ignore` probe, and it records `KindFileWritten` at the exclude path only when the content changed.
   When it fails, the step stops: no classification, no link, no rename, and `left` is `scratch exclude not written at <exclude path>: <err>; clear what blocks writing the exclude file, then rerun ` + "`lyx fabric reconcile`".
2. `scratchPathTracked(warp worktree, ".scratch")`, before the root is classified and whether or not the root exists, since a link standing over tracked paths makes git refuse a later checkout that writes them back "beyond a symbolic link"; a tracked root, tracked content under it, or tracked paths whose root was deleted by hand is a leave (shape `tracked by git`, way forward: stop tracking `.scratch` on the warp branch, move the untracked content into the target <target> and remove the root `.scratch`, then rerun `lyx fabric reconcile`).
   A failed tracked check is D1 case 6: the root stays as found and `left` names the root, the target, the error and the case-6 way forward.
   It is a reported outcome, never a propagated error, so a row reaches it through `SetScratchPathTrackedForTest`, which swaps `scratchPathTracked` and restores `PathTracked` in `t.Cleanup`; its caller must not be parallel.
   `SetScratchPathTrackedForTest` lives in `internal/fabricengine/export_test.go` beside `SetScratchBeforeLinkHookForTest`, so production code cannot swap `scratchPathTracked`.
3. `os.Lstat` of the root entry, then classification:
   - absent: `os.MkdirAll` the target, recording `KindDirCreated` at the target when it was absent, then `scratchBeforeLinkHook` when non-nil, then `fslink.CreateDirLink(root, target)`, recording `KindLinkCreated` at the root. A failed `MkdirAll` or a refused link is D1 case 6: no link, the root as found, `left` names the root `.scratch`, the failing path, the error and the way forward (clear what the error names, moving any content under a root `.scratch` into the target <target> before removing it, then rerun `lyx fabric reconcile`).
     Every case-6 report names the root `.scratch` and the target, whichever path failed.
   - a link whose `fslink.RawTarget` equals the target after `filepath.Clean`, never resolved: `MkdirAll` the target when absent, recording `KindDirCreated`, with `changed` `scratch target restored at <target>`; a present target changes and records nothing.
   - a link to another target, a real directory while the target exists, a regular file or other non-directory: D1 case 4 leave, the entry and target untouched, `left` naming the root, the shape (`a link to another target`, `a real directory while the target <target> already exists`, `a regular file`) and the way forward (move the content into the target <target> and remove the root `.scratch`, then rerun `lyx fabric reconcile`).
   - a real directory, target absent (its `os.Lstat` failing with not-exist or not-a-directory): `MkdirAll` the target's parent, then `os.Rename(root, target)`, recording `KindPathRenamed` with Target the target and Detail `hubRelativeTarget(l.HubPath, root)`; then `scratchBeforeLinkHook`, then `CreateDirLink`, recording `KindLinkCreated`, with `changed` `moved .scratch into <target> and linked it`.
     A failed parent `MkdirAll` is D1 case 6, the root as found.
     A failed rename is a case-4 leave with shape `a real directory whose move to <target> failed: <err>`, whose way forward first says to close what holds a handle inside `.scratch` (stop reed; an open handle is the expected cause on Windows), as `mergeAdoptionTree`'s rename failure does.
     A refused link after the rename is D1 case 5: the recreated `.scratch` stays untouched and `left` names the target where the moved content lives and the way forward (move the recreated `.scratch`'s content into that target, remove the root `.scratch`, then rerun `lyx fabric reconcile`).
   A created link's `changed` is `scratch linked: <root> → <target>`.

Every left report whose way forward moves content into the target names the `scratchLinkTarget` path, written `<target>` above.
Immediately before its first `os.MkdirAll` or `os.Rename`, the step `os.Stat`s `RecordsWorktreePath(l, slug)` and returns an empty outcome, creating nothing, when it is absent (D3), so a weft torn down mid-repair before that check does not get its `.lyx/scratch` path recreated as a plain directory, which reconcile's next pass would take for an existing weft worktree.

The step removes nothing and never returns an error (D2); `os.Rename` is not a destructive primitive under `PATTERN-fabric-destruction-chokepoint`, and the step never reaches `destroy.go`.
`scratchBeforeLinkHook` exists only so a test can recreate `.scratch` between classification and link creation; `SetScratchBeforeLinkHookForTest` in `export_test.go` sets it and restores nil in `t.Cleanup`, and its caller must not be parallel.

`KindPathRenamed` joins `mutation.go`'s Kind block with a doc line saying what its Target and Detail are; it lands here with its only recording site.
`KindDirCreated`'s doc line widens to every directory-creation success site, `createExclusiveDir`'s mint among them, since the scratch step and `restoreWeftLockDir` record it too.

`scratchlink.go`'s `os.MkdirAll` calls are a banned token of `TestNoUncontainedWrite_FabricengineProductionSource`, so `uncontainedWriteAllowlist` gains a row for `internal/fabricengine/scratchlink.go` whose reason says the step's `MkdirAll` creates the scratch target and its parent inside the pair's weft worktree, built from `RecordsWorktreePath`, and its link goes through `fslink`, as the `junction.go` row reasons for `seedLyxJunction`.

`WireJunctionsWith` becomes `wireJunctionSet` (the three existing steps: `seedLyxJunction`, `seedWeftArtifactExcludes`, `seedGitExclude`) followed by `wireScratchLink`; when the outcome's `left` is non-empty it logs it through `logger.Warn` with the slug, since add, checkout and clone results carry no detail field.
`seedScratchExclude` is deleted; its one caller was `WireJunctionsWith`. `ExcludeAnchoredDir` stays for `internal/ideengine`.
`scratchDirName`'s comment says the root entry is a fabric-wired link agents write through.
`junction.go`'s file header names the scratch step among what wiring does, and `UnwireJunctions`'s doc comment says the scratch link and its `/.scratch` exclude stay, so it no longer claims to undo exactly what wiring seeded; `unwire.go`'s file header says an applied Unwire removes every fabric junction on disk but the root scratch link.
`Unwire`'s doc comment (`internal/fabricengine/unwire.go:53-54`) and `Topology.Remove`'s sweep comment (`internal/fabricengine/remove.go:242-247`) say the root scratch link is excepted from what they remove or sweep.

`withoutRootScratchLink` returns names without `scratchDirName` when `l.AnchorRel` is `.`, and names unchanged otherwise, since at a subpath anchor `scanOnDiskJunctionNames` reads `<worktree>/<AnchorRel>` and never sees the root link.
`Unwire` filters its scanned names through it before both the dry run and the applied run, so `lyx fabric unwire` neither reports nor removes the scratch link and never strips `/.scratch` (D3); without it, `unseedJunctionRecords` would fail on the link, since `WarpJunctions` derives a `<weft>/.scratch` target that does not exist.
`Topology.Remove` filters its scanned names the same way before `removeWarpJunction`, so the junction sweep never asks the gate to remove the scratch link against the wrong expected target, which would surface as a refusal and fail the removal; `LinksRemoved` keeps counting the anchored junctions only, and `git worktree remove` takes the link until card 3 unlinks it first.
Neither filter can let anything else through: it drops exactly one name, at the root anchor only.

`manifestObservableKind` gains `KindPathRenamed: true`, and `entryCovers` covers a change at or below a `path_renamed` entry's Detail as well as its Target, so the oracle's omission direction accounts for the renamed root subtree.

Tests, in a new `scratchlink_integration_test.go` under the `integration` build tag in package `fabricengine_test`:
`scratchShapeCase` holds a name, an arrange func shaping the prime pair of a `hubforge.CopyHub(t, hubforge.Shape{Anchor: "."})` hub (its root `.scratch`, its target and its exclude), whether the step leaves the entry, and the report phrase card 2 asserts.
`scratchShapeCases` lists the leave shapes: a link to another target; a real directory with a file while the target exists; a regular file; a file under `.scratch/` committed with `git add -f` while the target is absent, so a classification that renamed before the tracked check would move it; a tracked file under `.scratch/` whose root was then deleted by hand; a failed rename (root a real directory, target removed, the target's parent chmodded `0o555`); a failed exclude write (legacy `/.scratch/` line written, the arrange asserting the `info` directory's exclude lock file exists, then the common gitdir's `info` directory chmodded `0o555`); a failed target creation (root removed, a regular file at the target path).
A chmod row is skipped under `os.Geteuid() == 0`, and its arrange registers, after the hub is built, a `t.Cleanup` restoring the directory's mode to `0o755`, so the temp directory's removal can delete the hub.
`TestWireJunctions_ScratchLink` runs parallel rows calling `WireJunctionsWith(rec, l, slug, names)` with names from `RepoWiredNames`:
- absent root at anchor `.` and at anchor `backend`, the target removed too: the root is a link whose raw target is the target, the record holds `dir_created` at the target and `link_created` at the root, and the root's `os.Lstat` (mode, size, modification time) is unchanged after a file is written under `.scratch/`;
- a correct link: the record is empty;
- a correct link whose target was removed: the target exists again and the record holds `dir_created` at the target;
- a real directory with a file and an absent target, in a warp whose committed `.gitignore` holds `**/.scratch/` and whose exclude holds the legacy `/.scratch/`: the file is read through `.scratch/<name>`, the record holds `path_renamed` (Detail the root's hub-relative path), `link_created` and `file_written`, `AssertRecordMatchesDiff` passes over `DiffManifest(before, after, nil)`, the exclude lines hold `/.scratch` and no `/.scratch/`, and `gitkit.GitStatusPorcelain` of the worktree lists no `.scratch`;
- every `scratchShapeCases` row: `WireJunctionsWith` returns nil, and the root entry and target are as arranged (a failed exclude write also leaves the real directory unrenamed and no link; both tracked rows create no link and no rename).
`TestWireJunctions_ScratchLinkRefusedAfterClassify` is serial, since it sets package seams and captures the logger through `logcapture.Capture`.
Each row builds and arranges its hub before calling `SetScratchBeforeLinkHookForTest` or `SetScratchPathTrackedForTest`, so `hubforge.CopyHub`'s lazy template build, which wires through `WireJunctionsWith`, never fires a seam.
Its rows call `WireJunctionsWith` and assert it returns nil:
- a real directory with a file and an absent target, with a hook that recreates a real `.scratch`: the recreated directory stays untouched, the moved file sits at the target, and the captured warn names the target and `lyx fabric reconcile` (D1 case 5);
- an absent root, with a hook that writes a regular file at the root: no link, an empty target, and a captured warn naming the root and `lyx fabric reconcile` (D1 case 6);
- a real directory with a file, with `SetScratchPathTrackedForTest` returning an error: the directory unrenamed, no link, and a captured warn naming the root and `lyx fabric reconcile` (D1 case 6, the tracked check failing);
- an absent root and target, with `SetScratchPathTrackedForTest` returning false after removing the pair's weft worktree directory: neither the target nor a root link exists (D3, the weft-root check).
`TestDotLyxJunction_LifecycleWiresSeedsBothExcludesAndUnwires` asserts `/.scratch` where it asserted `/.scratch/`, in its body, doc comment and `//testtiming:keep` reason.
`TestUnwire_RemovesOnDiskJunctionsIncludingStale` keeps its exact `JunctionsRemoved` lists, which a `.scratch` entry would break, and asserts the root `.scratch` is still a link to its target and the exclude still holds `/.scratch` after the applied run; its `//testtiming:keep` reason gains that the root scratch link stays.
`TestCodeWorktrees_ListOpenAndPairComplete`, before it removes the `_lyx` junction, replaces `kept`'s root `.scratch` link with a real directory and asserts `PairCompleteAt(kept.Path)` complete, `Healthy` ok on `kept`'s resolved layout, and `h.Topology.Status` reporting `JunctionHealthy` for `kept`, then removes the directory and asserts all three again (D3: health stays scratch-blind; `Healthy` runs its own loop in `drift.go`, not `checkJunctionHealth`); its doc comment gains that it pins this.

Docs: `internal/fabricengine/doc.go` gains `path_renamed` in the mutation-record kind list and a `# The scratch link` section stating the link, its target, the exclude pattern and why it has no trailing slash, the step's order and outcomes, its weft-root check that creates nothing when the weft worktree is absent, that it never fails wiring and removes nothing, and that `lyx fabric unwire` keeps the link and its exclude;
the existing `Unwire` paragraph's claim that it removes every fabric junction present on disk gains the root scratch link as its exception.
`internal/fabricengine/livestate_doc_test.go` lists `path_renamed` among the manifest-observable kinds and states its Detail coverage.

**ImpactSummary:** `WireJunctionsWith`'s add, checkout and clone callers gain the scratch step with an unchanged signature; `Unwire` and `Remove` drop one scanned name at a root anchor; every hub built after this card carries the link, which the unwire and status tests named here pin; `cmd/lyx`'s uncontained-write guard gains one allowlist row.
