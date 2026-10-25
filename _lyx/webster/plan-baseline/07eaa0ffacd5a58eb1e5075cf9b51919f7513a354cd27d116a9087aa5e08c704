# Card 7 — Add takes the new pair's anchor from the fork point

**Edit:**
- `internal/fabricengine#Topology.add`
- `internal/fabricengine_test#TestAdd_FromParentPair`
- `internal/fabricengine_test#TestAdd_DropsParentRunRecords_SubpathAnchor`

**Create:**
- `internal/fabricengine#refuseEmptyOwnMarker`

**Uses:**
- `internal/hubforge#ReanchorPair`
- `internal/fabricengine#resolveAddParent`
- `internal/fabricengine#createWeftWorktreeDroppingRuns`
- `internal/fabricengine#writeLaunchers`
- `internal/fabricengine#WireJunctionsWith`
- `internal/fabricengine#WriteOrigin`
- `internal/fabricengine#CommitRecordsPaths`
- `internal/fabricengine#Topology.rollbackAdd`

**Intent:**
D5 for a forked pair, and D2's empty-marker refusal of a fork point.
`Topology.add` names its fork point `fork`: `parent.location` with `--parent`, else the acting `l`.
Before any mutation it calls `refuseEmptyOwnMarker` on the fork point's weft worktree (`RecordsWorktreePath(fork, fork.WorktreeName)`).
A forked (not adopted) pair takes `fork.AnchorRel` as its anchor, never a read of the weft it is writing: `fork` is the `l` of `createWeftWorktreeDroppingRuns`, so the run-records root is scoped under the parent's anchor, and of `writeLaunchers`, `WireJunctionsWith`, `WriteOrigin`, the origin commit's `anchorRel` and `rollbackAdd`, so the new pair's launchers, junctions, scratch link and origin record sit at the fork point's anchor, which its inherited marker names.
The other uses of `l` stay: the acting worktree is where the warp repo, remotes and the dirt check are read.
An adopted weft branch keeps today's path in this card; card 8 gives it its own anchor.

`refuseEmptyOwnMarker` returns nil when `<weftPath>/.lyx-anchor` is absent or holds a value, and otherwise an error naming the file and the way forward: `git -C <weftPath> restore .lyx-anchor` when the committed marker holds the anchor, else writing the anchor (or `.` for the repo root) into it and committing it there, then re-running the verb.
It touches nothing; cards 8 to 15 call it wherever D2 refuses an empty own marker.

Tests:
`TestAdd_FromParentPair` gains a subtest: the `beam` pair is re-anchored at `x` by `hubforge.ReanchorPair`, then a pair added with `--parent beam` and a pair added with the beam's own Location as `l` and no `--parent` each have `.lyx-anchor` `x` in their records worktree, junctions and scratch link at `<slug>/x`, their origin record at `x/_lyx/fabric/origin.json` in their records worktree, and their launcher under `_launchers/x/<slug>`.
Its refusal table gains `EmptyForkPointMarker`: the parent's weft root marker truncated to blanks refuses naming `.lyx-anchor` and `git -C`, with nothing created.
`TestAdd_DropsParentRunRecords_SubpathAnchor` gains a row whose parent is a pair re-anchored at another directory than the hub's: the row commits run records under that anchor in the parent pair's own records worktree (with `gitkit.CommitFile` there, since the test's existing helper commits through the prime's Location only), adds the child with `--parent` naming that pair, and asserts the child's working tree holds none of those run records under that anchor and its first records commit deletes them there.

**ImpactSummary:** one method's internal Location choice plus one new private refusal helper; a pair forked in a one-anchor hub gets exactly today's paths.
