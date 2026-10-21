# Card 4 — cleanup-follows-scratch-link

**Edit:**
- `internal/cleanupcli#TestCleanupVerb_RefusesOutsideThePrimeThenListsAndThenRemovesEverySeededLeftover`

**Prosa:**
- `internal/cleanupcli/doc.go`

**Uses:**
- `internal/cleanupcli#scratchSegment`
- `internal/cleanupcli#scratchDirName`

**Intent:**
`lyx cleanup`'s scratch segment keeps listing the prime's `.scratch` by that path; `os.ReadDir` follows the fabric link to its target, so the segment judges and removes the target's direct children and leaves the link (D5).
No production code changes: `scratchSegment` already reads `filepath.Join(location.WorktreePath(), scratchDirName)` and removes children by their path under it.

`internal/cleanupcli/doc.go`'s scratch-segment section states that the prime's `.scratch` is a fabric link into the prime pair's weft `.lyx`, that the segment works on the children of what it points at, and that it never removes the link itself.

Test: `TestCleanupVerb_RefusesOutsideThePrimeThenListsAndThenRemovesEverySeededLeftover` already seeds an aged, uncited child under the prime's `.scratch`, which since card 1 is a link on every hubforge hub; it gains an assertion that the prime's `.scratch` is a link before the run and is still a link, with its raw target unchanged, after the applied run that removed the child.

The run also appends to the `fabric-windows` board entry's body, through `lyx board` (read the body with `lyx board get`, write it back with `lyx board upsert`), a check that a directory junction's own modification time stays fixed when files are created under its target, naming the root `.scratch` link fabric wires into the weft `.lyx` as the case that relies on it (D6).
That entry is not this task's own, so the run's board lock allows the write.
The board write has no file target, so the card's `**Verify:**` checks the entry's body names `.scratch`, which it does not today.

**ImpactSummary:** One test gains assertions and one doc file changes; no production behaviour changes in `internal/cleanupcli`.

**Verify:** lyx board get '{"slug":"fabric-windows"}' --body | grep -q '\.scratch'
