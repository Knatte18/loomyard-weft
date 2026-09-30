# Batten run records no longer leak into fresh pairs

All five batches were already `done` when this session resumed, so no batch was forked here:

1. `shedrun-runs-root-rel` exports the anchor-relative run-records root from `shedrun`.
2. `fabric-add-drops-run-records` makes `Add` fork without the parent's run records and commit the drop with the origin record.
3. `battenshed-prime-lock-wait` makes `Worktree-Create` and `Worktree-Teardown` wait for a contended prime lock.
4. `batten-child-ignores-prime-records` adds batten integration tests reproducing the reported seed collision.
5. `docs-fresh-pair-and-lock-wait` updates `docs/overview.md` and `CONSTRAINTS.md` with the new behaviour.

The status listing also shows a batch numbered -1 (`01-shedrun-runs-root-rel`) as `stuck` and terminal. It has a digest, and the fork batch 1 of the same slug reports `done`.

The plan-level verify suite (`go build ./...`, `go test ./...`, and the integration-tagged tests for shedrun, fabricengine, battenshed and battencli) was run once by the integration fork.
It reported `status: OK` at HEAD `6b0ae73357d7797b8495546feb4c8a4c6a06a23b`, with no deviations.
