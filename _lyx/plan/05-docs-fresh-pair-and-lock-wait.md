# Card 5 — docs-fresh-pair-and-lock-wait

**Prosa:**
- `docs/overview.md`
- `CONSTRAINTS.md`

**Uses:**
- `internal/fabricengine#Topology.Add`
- `internal/battenshed#PrimeLock`
- `internal/shedrun#RunsRootRel`

**Intent:**
Bring the two repo-level docs in line with cards 2 and 3, with semantic line breaks throughout.

`docs/overview.md`, batten bullet: replace the sentence beginning "Every child's weft branch forks from prime's `main-weft`" (the frozen, inert copies) with the new behaviour — a fresh pair's weft branch starts without its parent's `_lyx/shed/` run records, which fabric's `add` drops in the pair's first weft commit, so a child never sees prime's batten records or any other run's.
In the same bullet, state that `Worktree-Create` and `Worktree-Teardown` wait for a prime lock another run holds, polling every 2 s for up to 10 min, and halt `blocked` only once that wait runs out.
If the fabric `add` description elsewhere in the file states what a new pair's weft branch carries, update it to match.

`CONSTRAINTS.md`, Shed Run-Directory Invariant: add one bullet — a freshly forked weft branch never carries its parent's shed run records: `fabricengine.Add` drops the root `shedrun.RunsRootRel` names in the pair's first weft commit, and the adopt path drops nothing.
