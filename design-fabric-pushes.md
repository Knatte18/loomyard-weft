# Fabric: pair create survives a refused push, no untested main from the prime, neutral export names

`fabric-pushes` · Task · bug, enhancement, fabric · running · Finalize

Fabric's pushes and its exported names: pair creation survives a refused push, the prime never publishes untested `main` commits by the way, and fabric's vocabulary stays inside fabric.

## Pair creation retries a refused weft push and rolls back its own branch

From issue #433, seen 2026-10-07 when the hub started three batten runs at once (`cache-ttl-per-role`, `batcher-v2`, `burler-cost`).

- `cache-ttl-per-role`'s Worktree-Create pushed its weft branch at the same moment as the other two pairs and GitHub answered `! [remote rejected] cache-ttl-per-role-weft -> cache-ttl-per-role-weft (failed)`, with no hook message; the other two pushes went through.
- `rollbackAdd` then could not delete the warp branch: "warp-branch deletion was refused by the destructive gate (check=ownership); the branch is left behind". The branch was already pushed to origin.
- Resuming the run blocked on "branch already exists, left behind by an earlier pair", so the hub had to delete the branch locally and on the remote by hand; the next resume went through.

Wanted:

- Retry a weft push that GitHub rejects with a bare `(failed)` a few times with backoff before giving up; a bare `(failed)` reads as a transient server-side refusal.
- `rollbackAdd` removes the warp branch it created itself in the same call, locally and on the remote, instead of the ownership check refusing it.

## The prime pushes no untested main commits

Found at run-pushes' PR-Gate (PR #422), 2026-10-07; a deliberate design choice there (its Decision 5), not a defect.

In the prime, `lyx fabric push` and `sync` now push the parent branch (`main`) as well as the records side.
That includes the `sync` that `lyx config` runs after a change, which pushes whether or not the call made a commit.
So when the hub holds local `main` commits not yet pushed, a `lyx config --set` publishes them without the full `go test ./...` the hub runs before every push to `main`.

Direction: either gate the prime-side push of the parent branch (refuse, or push only the records side, when the parent branch has unpushed commits the call did not make), or keep the hub from ever holding unpushed local `main` commits.

## Fabric exports without warp/weft in their names

Seen 2026-10-04 in `pattern-init`: extending the fabric-vocabulary scan to test files outside fabric flagged 83 files, and 38 of them only name fabric's own exports that carry the words, e.g. `hubforge.Hub.PairWarpWorktree` and `WEFT_SKIP_GIT`.
`pattern-init` exempts that closed set of identifiers rather than renaming them.
Rename those exports to neutral names so the vocabulary stays inside fabric, then shrink the exemption set to nothing.
