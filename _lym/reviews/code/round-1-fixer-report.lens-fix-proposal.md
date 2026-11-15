```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1, F3–F10), looking for a case where a proposal is wrong or breaks something.

**No findings.**

Cases I built and the evidence that cleared them:

- **F1, the all-paths set.** `CodeWorktreeChanges` lists every uncommitted path, tracked ones too.
  - An arbiter that commits a path that was already dirty at spawn also changes its `Dirty` hash, so it is flagged either way.
  - An arbiter that commits a new file leaves `Changes` equal to the spawn set, so it is not flagged.
  - The comparison is order-insensitive (`samePaths`), and a row pins the case of a file that was already untracked at spawn and is left alone.
- **F1, false positives from the run's own files.**
  - The attempt directory is `shedrun.ArbiterDir`, which is under `RunDir` and holds the `_lym` records (`internal/shedrun/paths.go:349`).
  - `.lym/` is in `.gitignore`.
  - `CodeWorktreeChanges` never lists the records checkout.
  - So the arbiter's own `prompt.md`, verdict and envelope files cannot make the set differ. An arbiter that leaves a build artifact in the code checkout is handed on, which is the intended behaviour.
- **F4, passing the full `edits` into the resume hand-on.** The question now carries the "Check the worktree first" note plus the commits, which is what F4 wanted. The commit list is not duplicated, because `appendTo` writes it once. The gate passed.
- **F6, driving `Judge` a second time after `Settle`.** The second call takes the `settleExisting` → `conclude` path with the worktree unchanged and the attempt not applied, so it spawns nothing. It returns the same envelope's commits.
- **F9, the "only `git add` and `git commit`" bullet.** It restricts git commands that change something. It does not forbid read-only `git status`, `git log` or `git diff`, or `lym` verbs. It agrees with the template's retained "Push, amend, rebase, reset or switch a branch" line.
- **F7, the `commits` test.** `slices.Equal(got, slices.Clone(nil))` with a non-nil check on `got` pins both the list and the empty-list rendering.
- **Old attempt records.** An `attempt.json` written before `Changes` existed reads as an empty set. Only a worktree that was already dirty could then false-positive, and this branch is unreleased, so no such record exists.

Tool calls:
1. `grep` for `func ArbiterDir` in `internal/shedrun`, to learn where the attempt directory sits relative to the code checkout. My context did not hold it.
