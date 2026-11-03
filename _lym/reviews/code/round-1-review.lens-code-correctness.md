```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff. For each changed function or branch, one case run through the code.

**Per-branch list**

- `removePair` refill pass (`remove.go:327-335`).
  - A refill after the sibling's teardown, with `pair.taskWorktree` true and the path unregistered, is deleted. `KindPathRemoved` is recorded and `StrayPath` stays empty. Correct.
  - A delete that fails after three attempts sets `refilledKept`, logs a warning and still succeeds, with `StrayPath = target`. Correct.
  - A path that is registered again (same slug re-added) is skipped. That cannot happen under the exclusive teardown lock, and the check covers it anyway. Correct.
  - A pair whose task worktree was absent or already a stray at entry (`pair.taskWorktree` false) is never deleted. Correct.
  - A refill arriving before the `pathPresent` check at line 307 cannot occur, because `removeWarpWorktreeDir` would have returned an error first. The `RemoveStepTaskWorktree` step is therefore consistent. Correct.
- `StrayPath` merge (`remove.go:357-360`). `pair.strayPath || refilledKept` gives the right value in both stray cases. Correct.
- `PairWorktreeStands` (`remove.go:1001-1004`).
  - A registered worktree on disk gives true.
  - A plain directory gives false.
  - An absent path gives false.
  - `List` failing gives false, which is wrong in the direction this seam needs. See F1.
- Prune third pass (`prune.go:183-202`).
  - A hub directory with a valid slug that is not live is reported `Unowned` and `Removed` stays false, in dry run, apply and force alike. Correct.
  - `&lt;slug&gt;-weft` is skipped by the weft suffix, hidden directories by the dot prefix, reserved hub names by `validateWorktreeSlug`, and symlinks because `IsDir` is false. Correct.
  - The prime (the Main entry) is in `liveWarpSlugs`, so it is not reported. Correct.
  - Any other non-reserved, slug-shaped hub directory, such as an operator's own folder, is reported on every run. This is by design (doc.go and summary).
  - `Error` is set on an `Unowned` entry, and prune's `Error` is a designed refusal (carve-out noted at `fabriccli/fabric.go:1001`). Nothing in the CLI turns it into a failure. Correct.
- `createRefusal` `ErrTaskPathTaken` branch (`wire.go:364-371`). A non-worktree directory gives a `PreconditionError` naming the path and way forward. `ErrBranchExists` is checked first, which is right. A registered worktree that fails `ResolveWorktree` is the weak case. See F2.
- `removePairAndBranch` warning (`cancel.go:670-672`). A `StrayPath` from the removal adds a warning and `PairRemoved` stays true. Correct. When nothing but a stray directory remains, `Remove` returns `ErrPairNotFound` with an empty `RemoveResult`, so no warning is emitted. That follows the existing nothing-left path, and Worktree-Create's halt then names the path. Not a defect.
- `boardcli` `PairExists` wiring and `ErrTaskPathTaken` text change. Correct.

**Findings**

- F1 — LOW, design, `internal/fabricengine/remove.go:1001-1004` (via `isRegisteredLinkedWorktreeIn`, `remove.go:1015-1021`).
  - `PairWorktreeStands` fails open. A failed `git worktree list` returns false for a live pair, so the rename and project-change refusal is skipped.
  - The old stat check could not fail this way. The helper's own doc says "false is the conservative direction", which holds for deletes but not for this guard.
  - Evidence: `remove.go:1015-1021` returns false on `List` error, and `PairWorktreeStands` ANDs it directly.
- F2 — LOW, design, `internal/battencli/wire.go:365-370`.
  - `ErrTaskPathTaken` is raised by Add for any existing path (`add.go:255`). That includes a registered worktree that `taskWorktreePresent` read as absent because `ResolveWorktree` failed.
  - The text "is not a worktree … delete it" is then wrong. It tells the operator to delete a registered worktree that may hold work.
  - Evidence: `add.go:255-256` and `wire.go:274-276`. The comment at `wire.go:336` asserts the stray-only case without checking registration.

**Tool calls**

1. grep of `internal/fabriccli` for `Unowned` and `.Error != ""`: to see whether the prune CLI treats an entry's `Error` as a failure.
2. Read of `fabriccli/fabric.go` 900-930: to read the lines the first grep matched.
3. Read of `fabriccli/fabric.go` 975-1020: to confirm those matches are in the cleanup path, not prune.
