```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the changed text still agrees with the sibling decisions, rows, criteria and cross-references it touches. Three findings, all LOW.

1. **LOW: D11 does not mention the new Finalize re-check.**
   - Changed text: D9's new bullet "Finalize re-runs `BeamLandRefusal` under the parent lock and halts naming the members…", and D8's Finalize row "after `BeamLandRefusal` passes again (D9)".
   - Problem: D11 still says "The land refusal leaves the start guard and the pawl run's own Worktree-Create; `lyx beam land` and Beam-Await (D9, D10) apply it."
   - Effect: that list of who applies the land refusal now leaves out Finalize, so D11 contradicts D8 and D9.

2. **LOW: the "landing was interrupted" notice fires in cases that are not a crash, and its stated cause does not match the new order.**
   - Changed text: D9's "A Beam-Await entered with no request standing and the beam's dobby not live, as a crash inside the accept above leaves it, posts the beam's orch one notice that its landing was interrupted".
   - Problem 1: the reordered accept now removes the request last. A crash before that removal leaves the request standing, and re-entry simply accepts it again. Only a crash between removing the request and Done matches this condition.
   - Problem 2: D4's `lyx beam stop` also stops the dobby and the orch, and leaves no request. A driver that batten re-spawns afterwards meets the same condition. It then posts an "interrupted landing" notice to an orch the operator stopped on purpose, when no landing was ever in progress.
   - Effect: the bullet disagrees with D4 and overstates its own cause.

3. **LOW: D4's bound and D1's new rule disagree about restarting a held beam's dobby.**
   - Changed text: D1's "None of them starts a dobby for a beam whose pawl run holds the landing hold (D9)", where "them" means the `EnsureLive` callers.
   - Problem: D4's bound still says "a hand `lyx batten run` there restarts the dobby, as it does today."
   - Evidence: batten restarts the parent worktree's dobby through `ensureParentDobby`, using `dobbyengine.SpawnDetached` rather than `EnsureLive` (`internal/battencli/cli.go:540-553`). D1's rule therefore does not cover it.
   - Effect: a hand run of a member in G while G holds the landing hold restarts G's dobby. That is the same churn F8 was fixed to bound, and D9 and D1 now say this cannot happen.

Tool calls:
- `grep` for the landing-hold, Beam-Await, `max_runs` and `lyx prime` lines across the decision record. My context held only the separate edit strings, not the file as a whole after the edits. The `git diff` in the same call printed nothing.
- `grep` for dobby spawn calls in `internal/battencli`, to learn how a hand batten run restarts a dobby, which my context did not show.
- `sed -n 530,560p internal/battencli/cli.go`, to read `ensureDobby` and `ensureParentDobby`'s spawn path.
