```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out as proposed (fix-proposal lens).

1. **MEDIUM (arguably BLOCKING). `lymWrittenInPrime` and the `written` skip in `ignoredWork` (F1): a plain `--reset` now deletes operator content without naming it.**
   - `.vscode/settings.json` and `.claude/settings.local.json` are merged files, not files lym alone writes.
   - `internal/vscode/config.go:4` says "Every other settings key is the operator's and is kept". `internal/fabricengine/doc.go` also tells the operator that settings belong in the prime's `.vscode/settings.json`.
   - `internal/heddleengine/doc.go` says of `settings.local.json` that "a key the file holds that heddle never set stays". That includes the operator's Claude permission grants.
   - Concrete case: a hub with pushed records, where the operator added permissions or editor settings. `--reset` without `--discard-work` removes both files unnamed. Before this fix both were named. The board entry asks for refusal over ignored files that hold work.
   - `warp_exclude` is also operator-editable ("add it to the warp_exclude key"). Anything an operator lists there, such as a `.env`, is now skipped too.
   - The new texts say "what lym writes itself" and "ignored files lym did not write" (`doc.go`, `fabriccli/doc.go`, `fabric.go` help, `SANDBOX-FABRIC-SUITE.md`). They hide that operator keys inside those files are lost.

2. **MEDIUM. `RefuseHubWithoutBolt` (F8): when the hub is in use, the named clone cannot be carried out.**
   - The proposal's "Otherwise keep the plain form" covers `len(work.inUse) > 0`.
   - A used hub that lost its Bolt and holds a pair worktree is still told to "replace the hub with `... --reset`". `checkResetTarget` refuses that clone, since a pair refuses whatever the flags say.
   - This is the defect F8 describes ("can name a clone that is refused"), left in place for the used-hub case.
   - The sibling `hubWorkWayForward` handles it with "land or remove that work, or move it aside".

3. **MEDIUM. `RefuseHubWithoutBolt` discard branch (F8): the only way forward it offers destroys the work.**
   - The refusal says "replace the hub with `... --reset --discard-work`; that clone loses the work origin lacks: ...".
   - It offers no push, commit or move-aside, although `hubWorkWayForward` offers all of these before the destructive clone for the same state.
   - An operator or agent that follows the way forward of a `reconcile` refusal discards unpushed board commits.

4. **MEDIUM. `internal/fabricengine/doc.go`, "what lym writes itself: ... the board's lock files" (F1): the code does not hold this.**
   - Current code takes `board.lock` and `board.push.lock` through `BoltLockPath`, then `lock.TrackedLockPath`, under `_bolt/.lym/locks/`. The `.lym` skip already covers that.
   - The two root-level names the `written` list skips are leftovers of earlier code. The new test only passes because it adds exclude lines by hand.
   - The sibling leftovers on a real older hub (`.gitrepo-push.lock`, `board.json.swaplock`, seen in this machine's `_bolt`) are still named. So a plain `--reset` of that hub is still refused over lym's own empty files.

I built no breaking case for the F2, F3, F5, F6, F7 or F9 proposals.

Further tool calls:
1. Grepped for `settings.local.json` and `.heddle`, and read the `warp_exclude` paragraph of `doc.go`: my context did not show who writes those files.
2. Read `internal/heddleengine/doc.go` lines 14-32: my context did not show whether heddle keeps operator keys in `settings.local.json`.
