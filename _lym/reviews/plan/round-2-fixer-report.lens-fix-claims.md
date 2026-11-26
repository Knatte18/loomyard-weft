```yaml
lens: fix-claims
outcome: reported
```

No findings.

The fix's factual claims hold against HEAD:
- **Worktree-Teardown runs before batten reads done.** `Worktree-Teardown` is the batten recipe's last row, after `Land` (`docs/overview.md:1830`, `contracts/recipes/batten-recipe.yaml:41-86` as cited in `docs/research/loom/lym-state-map-2026-11-15.md:317`). `runfsm` derives landed from batten `StateDone` (`internal/runfsm/state.go:71-73`).
- **A read past Land answers the parent's or the hub's value and succeeds.** `internal/hubgeom/tasksetting.go:20,29-41` leaves out a worktree with no directory and continues to the parent and the prime without error.
- **The level is kept "as the landing notice keeps land_notice".** `internal/dobbyengine/landnotice.go:75` reads only ahead of Land, or past Land while never read. That matches the new "not `PastLand`, or while its entry holds no `Level`".
- **The new `TestRunWatch` rows' expected outcomes follow from card 3's own rules.**
  - Under a stored `off`, nothing is owed.
  - Under a stored `stops`, the landed reading owes one line. That holds when no land_notice seam is set, because the copied reading is then `false` (`landnotice.go:78-80`) and is not skipped.

Calls made:
- grep of `internal/dobbyengine/passes.go` and `internal/battenshed` for teardown and `StateDone`: my context did not show where teardown sits in the row order.
- sed of `passes.go:70-90` and grep of the docs for `Worktree-Teardown`: to confirm that teardown is the row after `Land`.
