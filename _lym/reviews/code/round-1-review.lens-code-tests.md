```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test row that pins it, or none.

**Per behaviour**

1. **Anchor link at a subpath anchor.** Pinned by `TestWireJunctions_ScratchLink/AbsentRootLinked_Anchor_backend` (link, `dir_created`, `link_created`, Lstat unchanged on write). `TestAdd_FromParentPair` and `requireWiredOnlyAt` pin the link location through add and anchor change.
2. **`/<anchor>/.scratch` exclude line, no trailing slash.** Pinned by `TestScratchLink_SubpathAnchor/UnwireKeepsTheAnchorLink` and `RootLinkMovesToTheAnchor`. Anchor `.` is pinned by `RealDirectoryAdoptedIntoTheTarget`.
3. **Adopt, restore and leave-as-found at a subpath anchor.** Every row of `scratchShapeCases`, the adopt row and the `TestReconcile_ScratchLink` rows use `Shape{Anchor: "."}`. None at a subpath anchor. The change description claims the `cs` `dotnet/.scratch` real-directory adoption; no row reproduces it. The code paths are shared through `l.AnchorRel`, so only the anchor-joined paths go unpinned.
4. **Retire a fabric-owned root link at a subpath anchor.** Pinned by `RootLinkMovesToTheAnchor` (root absent, `link_removed` recorded, anchor link correct).
5. **Root exclude line kept while a sibling still links the root, then dropped.** Pinned by `RootLinkMovesToTheAnchor`. The prime keeps `/.scratch` after the pair is wired, and the line is gone after `reconcilePrimePair`. It also asserts a clean `git status`.
6. **Root entry that is not fabric's is left as found, its exclude line kept, and reported.** None. No test has a real directory, a regular file or a foreign link at the root of a subpath-anchored pair. This covers the `Lstat(root)` leave branch and the `retireRootScratchLink` combined outcome through `and`.
7. **Retire failure branches.** None. These are the `removeOwnedScratchLink` error, the root `Lstat` error and the failed exclude-line removal. The failed-exclude-write row uses anchor `.`, where the retire step is a no-op.
8. **Census fallbacks in `scratchLinkedInSibling`.** None. These are the `List(WorktreePath)` failure falling back to `l.WorktreePath()`, an unreadable list answering true, and an uninspectable sibling answering true.
9. **Anchor change drops the old anchor's exclude line, with the sibling census.** None. `requireWiredOnlyAt` asserts only the scratch link at the new anchor. No assertion in `anchorchange_integration_test.go` mentions the exclude line, so the new `unseedScratchExclude` call in `unwireAtSource` is unpinned. Deleting it breaks no test.
10. **`Remove` also unlinks a legacy root link at a subpath anchor.** None. `TestRemove_UnlinksScratchLinkBeforeWorktreeRemoval` runs only at anchor `.`. `TestRemove_SweepsAnchoredLinksOnSubpathHub` pins only the anchor link's `link_removed`. The `!anchorIsRoot` branch of `unlinkScratchLink` is uncovered.
11. **`withoutScratchLink` drops `.scratch` from the junction scans.** Pinned in `Unwire` by `UnwireKeepsTheAnchorLink`. Pinned in `Remove` by `LinksRemoved = 2` in `TestRemove_SweepsAnchoredLinksOnSubpathHub`. Pinned in Reconcile stale removal by `RootLinkMovesToTheAnchor`, whose reconcile would remove the anchor link if the filter were gone.
12. **Cleanup segment reads `.scratch` at the prime's anchor** (`segments.go:120`). None. `cleanup_integration_test.go:59` uses `Anchor: "."` and reads `<prime>/.scratch`, which is also the anchor, so reverting to the root passes. `scratch_test.go` calls `scratchSegment` with an explicit dir, so it never reaches the path join.
13. **Docs and comment changes** are static prose and exempt.

**Findings**

- **MEDIUM, design (coverage gap).** Cleanup scratch segment anchor path (`internal/cleanupcli/segments.go:120`). The change description claims it, and no test fails if it reverts to the root. Evidence: `cleanup_integration_test.go:59` and `:107` use anchor `.`.
- **MEDIUM, design (coverage gap).** Anchor change drops the old anchor's exclude line (`internal/fabricengine/anchorchange.go:475`). Evidence: `anchorchange_integration_test.go:117-134` asserts only the link, and the file asserts no exclude line.
- **MEDIUM, design (coverage gap).** `Remove` unlinks the legacy root link at a subpath anchor (`internal/fabricengine/scratchlink.go:286-289`). Evidence: `scratchlink_integration_test.go:712-803` runs only at anchor `.`, and `remove_junctions_integration_test.go:199-208` asserts only the anchor link.
- **MEDIUM, design (coverage gap).** A root `.scratch` that is not fabric's, at a subpath anchor, is never removed and keeps its exclude line (`scratchlink.go:215-221` and `:208-211`). This is the fasit's data-safety edge: a real root directory, or a link into another weft, is left as found. Evidence: the only subpath tests are `UnwireKeepsTheAnchorLink` and `RootLinkMovesToTheAnchor`, which use fabric-owned links (`scratchlink_integration_test.go:808-883`).
- **LOW, design (coverage gap).** No row adopts or leaves as found a real `.scratch` at a subpath anchor. This is the `cs` case named in the description. Evidence: every shape row uses `Shape{Anchor: "."}` (`scratchlink_integration_test.go:462`, `:1050`).
- **LOW, design (coverage gap).** The `scratchLinkedInSibling` failure fallbacks and the retire-step error branches have no row (`scratchlink.go:167-185`, `:208-225`). The conservative-answer contract is unpinned.

**Tool calls (1 of 6)**
- `grep` over `anchorchange_integration_test.go` and `cleanup_integration_test.go` for scratch and anchor use. My context held only the diff hunk of the first and only grep hits of the second.
