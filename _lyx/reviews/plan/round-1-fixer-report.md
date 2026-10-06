# Round 1 fixer report — plan

All three findings are fixed; nothing is deferred.
Only files under `_lyx/plan/` and this round's two review files were written, and no git command was run.
`approved:` in `00-overview.md` is untouched.

## Fixed

- **F1 (MEDIUM, scope): vacuous add rows, unasserted D1 paths.**
  - `01-weft-lock-dir.md`: removed the `**Create:**` block and the Tests bullet for `internal/fabriccli/weftlockdir_integration_test.go`, since `add` already writes `.weft/` through the origin record on every path its rows covered.
    The `reedgeom_integration_test.go` row (b) now asserts the freshly added pair's `.weft/` first, which covers the `add` fork path with no new test, and a note names where the adopt-path and dormant-fork assertions live.
  - `04-add-adopts-live-pair.md`: the `origin_integration_test.go` row for an adopted weft that already carries a record now also asserts `.weft/` exists, the one `add` path that card 1's adopt-path `ensureWeftLockDirAt` alone creates.
  - `03-reconcile-adopts-origin-weft.md`: the no-origin raw-warp row now asserts the dormant weft worktree has `.weft/`, which pins `createWeftWorktree`.
- **F2 (LOW, scope): origin-created branch left behind by a failed weft switch.**
  - `02-weft-branch-resolver-checkout.md`: `switchOrForkWeft` returns `created=true` with the error when the switch fails after the resolver created the branch, and `Checkout` passes it to `rollbackSwitch` on the error path, so `deleteBranch` removes it.
    Added a `checkout_rollback_test.go` row for that case; its injection is an untracked file at the marker path, because `Checkout` refuses a tracked-dirty weft worktree before the switch.
- **F3 (LOW, consistency): rollback rows in the pre-flight file.**
  - `04-add-adopts-live-pair.md`: moved the two rollback rows (resolver-created branch deleted with origin unchanged, fast-forwarded branch not rewound) to `internal/fabricengine/add_rollback_adopt_test.go`, added that file to the card's Edit targets, and named its portal-blocker injection as the failure source.
    Removed the open "rollback hook or injected failure" wording from the `add_remoteleftover_integration_test.go` bullet.

## Deferred

None.
