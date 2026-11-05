---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/scratchlink.go:181
    summary: >-
      The sibling census counts only a link at a sibling's root `.scratch`, so a sibling's real root `.scratch` directory left as found no longer keeps the root `/.scratch` exclude line and shows as untracked.
    fix: >-
      In `scratchLinkedInSibling`, replace the `fslink.IsLink` test with an `os.Lstat` of `filepath.Join(entryPath, anchorRel, scratchDirName)` that answers true for any entry that exists, whether link, directory or file, and true for any error other than `fs.ErrNotExist`.
      Add a row to `TestScratchLink_SubpathAnchor` where the prime holds a real root `.scratch` directory, a second pair is wired and retired, and the `/.scratch` line is still in the exclude file afterwards.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/doc.go:575
    summary: >-
      The `lym fabric remove` paragraph still says Remove unlinks the root link at a root and a subpath anchor alike and records one `link_removed` at the root, which contradicts the code and the paragraph the diff added at line 565.
    fix: >-
      Rewrite the three lines at `doc.go:575-577` to say that `lym fabric remove` unlinks the `.scratch` link at the anchor through the destructive gate, and at a subpath anchor also a root `.scratch` link an earlier binary wired, ahead of the warp worktree removal, recording a `link_removed` at each;
      that it claims only a link resolving into the pair's weft worktree and leaves any other entry to the worktree removal;
      and that the target goes with the weft worktree teardown.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/cleanupcli/segments.go:120
    summary: >-
      The cleanup scratch segment now reads `.scratch` at the prime's anchor, and no test fails if that reverts to the worktree root, because the only integration test uses anchor `.`.
    fix: >-
      Add a subpath-anchor row to `internal/cleanupcli/cleanup_integration_test.go`.
      Build the hub with `hubforge.Shape{Anchor: "backend", Pairs: ...}`, age a child under `filepath.Join(h.PrimeWorktree(), "backend", scratchDirName)` and assert that `--apply` removes it and keeps the link.
      Build the path from `h.Location.AnchorRel` rather than the worktree root.
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/anchorchange.go:475
    summary: >-
      The anchor change now drops the old anchor's `.scratch` exclude line under the sibling census, and no assertion in the anchor-change tests mentions the exclude file, so deleting the call breaks no test.
    fix: >-
      In `internal/fabricengine/anchorchange_integration_test.go`, extend `requireWiredOnlyAt` to read `gitkit.ExcludeLines` for the pair's warp worktree and require `/<anchor>/.scratch` while the other anchors' scratch lines are absent once no sibling still links them,
      or add one row to the existing anchor-change table that asserts the old anchor's line is gone and the new one is present.
  - id: F5
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/scratchlink.go:286
    summary: >-
      `Remove` at a subpath anchor also unlinks a legacy root `.scratch` link through the destructive gate, and no test runs Remove against a pair with a root link at a subpath anchor.
    fix: >-
      In `TestRemove_UnlinksScratchLinkBeforeWorktreeRemoval`, add a row on `hubforge.Shape{Anchor: "backend"}` that uses `wireLegacyRootScratchLink` on an added pair and asserts one `link_removed` at the root `.scratch` and one at the anchor link, both ahead of the warp `worktree_removed`,
      and the weft target untouched ahead of the weft `worktree_removed`.
  - id: F6
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/scratchlink.go:215
    summary: >-
      At a subpath anchor a root `.scratch` that is not fabric's, such as a real directory, a regular file or a link outside the pair's weft worktree, must be left as found with its exclude line kept and a way forward reported, and no test covers it.
    fix: >-
      Add rows to `TestScratchLink_SubpathAnchor` for a real root directory and a root link to another target on `Shape{Anchor: "backend"}`.
      Assert that wiring through `wireWithRecord` keeps the entry and its content, keeps `/.scratch` in the exclude file, still creates the anchor link, and that `Reconcile`'s Detail names the root path and `lym fabric reconcile`.
  - id: F7
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/scratchlink_integration_test.go:462
    summary: >-
      Every adopt and leave-as-found row, including the real-directory adoption the description cites for `cs`, runs at anchor `.`, so no row exercises the anchor-joined paths at a subpath anchor.
    fix: >-
      Make `scratchShapeCases` run for both anchors, `.` and `backend`, in `TestWireJunctions_ScratchLink` and `TestReconcile_ScratchLink`,
      by looping over the anchor in the `hubforge.Shape` of the `Leaves_` subtests and in the `RealDirectoryAdoptedIntoTheTarget` row, with each row's expected exclude line built from the anchor.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/loom-plan-spec.md:128
    summary: >-
      The plan spec still names the repository root's git-ignored `.scratch/` as the transient draft location, although in an anchored hub nothing is wired or excluded at the root any more.
    fix: >-
      In `contracts/specs/loom-plan-spec.md:128`, change the phrase about the repository root's git-ignored `.scratch/` to the worktree anchor's git-ignored `.scratch/`, and check Webster's `create-not-done` skip for a path rule that assumes the root.
---

### [MEDIUM] Sibling census ignores a sibling's real root `.scratch`

**Location:** `internal/fabricengine/scratchlink.go:181`
**Issue:** `unseedScratchExclude` keeps the root `/.scratch` line only while a sibling worktree has a root link.
A sibling whose root `.scratch` is a real directory left as found counts as no link, so the next pair to retire its root link drops the shared line and the sibling's `.scratch` shows as untracked.
The retiring pair's own left-as-found root keeps the line (`scratchlink.go:215`), and the description's design note says a root entry fabric does not own keeps its exclude line, so the census holds the rule for one worktree only.

### [MEDIUM] `doc.go` remove paragraph contradicts the diff

**Location:** `internal/fabricengine/doc.go:575`
**Issue:** The paragraph says `Remove` unlinks the root link at a root and at a subpath anchor alike and records one `link_removed` at the root.
At a subpath anchor the primary link is now the anchor link, the root link is a second, legacy unlink, and the diff's own paragraph at `doc.go:565` says so.
`TestRemove_SweepsAnchoredLinksOnSubpathHub` asserts the `link_removed` at the anchor.

### [MEDIUM] Cleanup anchored path is uncovered

**Location:** `internal/cleanupcli/segments.go:120`
**Issue:** The focus directive says the diff changes only comments in `cleanupcli`; it does not.
`segments.go:120` now joins `location.AnchorRel` onto `location.WorktreePath()`, so the summary's claim that the segment reads `.scratch` at the prime's anchor holds.
No test reaches it at a subpath anchor: `cleanup_integration_test.go:59` builds the hub at anchor `.` and `:107` reads `<prime>/.scratch`, and `scratch_test.go` passes `scratchSegment` an explicit directory.

### [MEDIUM] Anchor-change exclude drop is unpinned

**Location:** `internal/fabricengine/anchorchange.go:475`
**Issue:** `unwireAtSource` now ends with `unseedScratchExclude(rec, source, slug, source.AnchorRel)`.
`requireWiredOnlyAt` asserts only the scratch link at the new anchor and no test in the anchor-change file reads the exclude lines, so removing the call fails nothing.

### [MEDIUM] `Remove` of a legacy root link at a subpath anchor is unpinned

**Location:** `internal/fabricengine/scratchlink.go:286`
**Issue:** The `!anchorIsRoot` branch of `unlinkScratchLink` is a destructive step through the gate, and no test removes a pair with a root link at a subpath anchor.
`TestRemove_UnlinksScratchLinkBeforeWorktreeRemoval` runs at anchor `.` and `TestRemove_SweepsAnchoredLinksOnSubpathHub` asserts only the anchor link.

### [MEDIUM] Root entries fabric does not own are untested at a subpath anchor

**Location:** `internal/fabricengine/scratchlink.go:215`
**Issue:** The decision record's goal moves the prime and the worktrees over without losing content, so a real root directory, a regular file or a foreign link at the root must survive wiring with its exclude line and a reported way forward.
The only subpath tests, `UnwireKeepsTheAnchorLink` and `RootLinkMovesToTheAnchor`, use fabric-owned links, so the `Lstat(root)` branch and the `and` combination of retire and wire outcomes are unpinned.

### [LOW] Adopt and leave-as-found rows run only at anchor `.`

**Location:** `internal/fabricengine/scratchlink_integration_test.go:462`
**Issue:** The `cs` case in the description, a real `dotnet/.scratch` moved into the target and linked, has no row.
Every `scratchShapeCases` row and the adopt row use `Shape{Anchor: "."}`.

### [LOW] Plan spec names the repository root's `.scratch/`

**Location:** `contracts/specs/loom-plan-spec.md:128`
**Issue:** The spec places a `Create` target's transient draft under the repository root's git-ignored `.scratch/`, which in an anchored hub no longer exists or is excluded.

## Rejected

- Retire-before-wire leaves no scratch link at either place when the anchor step then leaves or fails (correctness lens, LOW): the outcome reports the way forward, the weft target is untouched and the next `lym fabric reconcile` links the anchor, and the summary states the step never fails wiring.
- A dangling legacy root link is reported as not this pair's and kept (correctness lens, LOW): the anchor step recreates the target in the same pass, so the next reconcile retires the link; the report is accurate that the link did not resolve into the weft worktree.
- The census keeps the root line for any worktree of the repository with a root link, including a user's own (correctness lens, LOW): this is the stated design, a dead line costs less than a dirtied live link, and the summary records it.
- `cleanupcli/doc.go` still says "the prime's `.scratch/`" in its last scratch-segment sentence (affected-set lens, LOW): the sentence stays true, since the directory is the prime's `.scratch/` at its anchor, and the lines above it already state the anchor.
- Census fallbacks and retire-step error branches have no row (tests lens, LOW): the fallbacks answer conservatively through one-line returns and a test for each would pin a defensive path the rubric's trust rule does not ask for.
