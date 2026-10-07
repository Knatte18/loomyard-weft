# Prime-side fabric push and sync publish unpushed main commits

`prime-push-gate` · Note · enhancement, fabric

Found at run-pushes' PR-Gate (PR #422), 2026-10-07; a deliberate design choice there (its Decision 5), not a defect.

In the prime, `lyx fabric push` and `sync` now push the parent branch (`main`) as well as the records side.
That includes the `sync` that `lyx config` runs after a change, which pushes whether or not the call made a commit.
So when the hub holds local `main` commits not yet pushed, a `lyx config --set` publishes them without the full `go test ./...` the hub runs before every push to `main`.

## Direction

Either gate the prime-side push of the parent branch (refuse, or push only the records side, when the parent branch has unpushed commits the call did not make), or keep the hub from ever holding unpushed local `main` commits.
