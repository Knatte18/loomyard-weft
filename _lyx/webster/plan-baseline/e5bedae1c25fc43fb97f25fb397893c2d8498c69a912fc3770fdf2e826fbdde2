# Card 3 — remove-unlinks-scratch

**Create:**
- `internal/fabricengine#unlinkScratchLink`
- `internal/fabricengine_test#TestRemove_UnlinksScratchLinkBeforeWorktreeRemoval`

**Edit:**
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine_test#TestRemove_SweepsAnchoredLinksOnSubpathHub`

**Prosa:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#linkIsFabricOwned`
- `internal/fabricengine#removeLink`
- `internal/fabricengine#ownedDriftedWiredJunction`
- `internal/fabricengine#scratchDirName`

**Intent:**
`lyx fabric remove` unlinks the worktree-root `.scratch` link itself before it removes the warp worktree directory, at a root and at a subpath anchor alike (D3).

`unlinkScratchLink`, in `scratchlink.go`, reads the root entry `filepath.Join(WorktreePath(l, slug), scratchDirName)` by its root path, independent of `scanOnDiskJunctionNames`, which reads `<worktree>/<AnchorRel>` and never sees the root link on a subpath-anchored hub.
It returns nil without touching anything when the entry is absent or not a link, or when `linkIsFabricOwned(l, slug, root)` does not claim it, the same ownership test the scan applies: a dangling link, or one resolving outside the pair's weft worktree, is left for `git worktree remove`, which unlinks it without following it.
A claimed link goes through `destroy.go`'s `removeLink` gate with `container` the warp worktree, `ownership` `ownedDriftedWiredJunction([]string{root})` and dirtiness N/A (a link holds no content; its target is untouched), which records `link_removed` at the root.
The ownership kind checks membership and link-ness only, because `linkIsFabricOwned` has already proven where the link resolves; it lets through exactly the one root path the call names, and only when it is a link.

`Topology.Remove` calls it inside its `pair.taskWorktree` branch, after the anchored junction sweep and before `removeWarpWorktreeDir`, passing its error through `surfaceRefusal` the way the sweep does, so an operational failure is left to the worktree removal.
A gate refusal there cannot occur from a real input: the call reaches the gate only for a root `linkIsFabricOwned` just claimed as a link, which passes the ownership kind's membership and link checks, its container is the warp worktree the root lies directly under, and dirtiness is N/A.
It is a fail-loud wiring guard in `refusal-spec.md`'s sense, which takes no reaching test; should it fire, its error names the root and says to remove the root `.scratch` link by hand and rerun `lyx fabric remove`, which then finds no link and proceeds.
At a root anchor the sweep already dropped the scratch name (card 1), so the link is unlinked once.
The link's target lives in the pair's weft `.lyx` and goes with the weft worktree teardown; nothing in this card touches it.
`LinksRemoved` keeps counting the anchored junctions only, so `TestRemove_SweepsAnchoredLinksOnSubpathHub`'s exact count stays.

Tests: `TestRemove_UnlinksScratchLinkBeforeWorktreeRemoval`, in `scratchlink_integration_test.go`, runs parallel rows on a `hubforge.CopyHub(t, hubforge.Shape{Anchor: "."})` hub, each with a pair added through `hubforge.AddPairWith` and `SkipPush`, then calls `Remove(l, slug, true, false)` and asserts it returns nil:
- the wired link, with a file written under the pair's `.scratch/`: the record holds exactly one `link_removed` at `<slug>/.scratch`, ahead of the `worktree_removed` entry for the warp worktree, and no entry at or below the target's hub-relative path ahead of the `worktree_removed` entry for the weft worktree, so the target's content is untouched until the weft teardown;
- the root `.scratch` replaced by a real directory holding a file: no `link_removed` at `<slug>/.scratch`;
- the root `.scratch` replaced by a link to a directory outside the pair's weft worktree, which `linkIsFabricOwned` does not claim: no `link_removed` at `<slug>/.scratch`, and that directory and its content survive.
`TestRemove_SweepsAnchoredLinksOnSubpathHub`, the subpath-anchored Add and Remove this card needs, gains, beside its unchanged `LinksRemoved` count of 2: the result's record holds exactly one `link_removed` at `<slug>/.scratch`, ahead of the warp worktree's `worktree_removed`.

Docs: the `# The scratch link` section of `internal/fabricengine/doc.go` states that `lyx fabric remove` unlinks the root link through the gate before removing the warp worktree, and that its target goes with the weft worktree.

**ImpactSummary:** One new call inside `Topology.Remove`'s task-worktree branch; the result's fields and every refusal are unchanged, and the record gains one `link_removed` entry per removed pair that carries a scratch link.
