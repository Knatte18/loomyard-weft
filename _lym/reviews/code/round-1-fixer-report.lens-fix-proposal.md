```yaml
lens: fix-proposal
outcome: reported
```

**Task:** check, through the fix-proposal lens, the proposals the fixer carried out for F1 to F5, and challenge each proposal itself.

**Findings**

1. **LOW** — `_lym/landing/summary.md`, "No part of lym reads a title back" (line 54): the sentence "reed's window ordering reads tmux window names, which this change leaves alone" is now false.
   - F1's proposal renames the status strand's window title, so the change no longer leaves tmux window names alone.
   - `internal/reedengine/windoworder.go:24-29` returns `d.Title`, and falls back to the name `overview` only when the title is empty. Reed's window ordering therefore reads the display title of the strand.
   - The status strand's `Display.Title` is now `ly: P3: T4: tapestry` where it was `T4: tapestry`. Ordering is by anchor and order, so the order is unlikely to change, but the sentence no longer holds.
   - The fixer rewrote only the bullet at lines 41-45 and left the stale sentence at line 54.
   - Fix: reword the sentence to say the status window's title now carries the shortname, and that the window ordering reads window names that include it.

2. **LOW** — `internal/loomcli/sharedbootstrap.go`, the `statusWindowTitle` comment "A run never starts in the prime, so no prime is looked up".
   - The claim is unverified and no caller or test pins it.
   - If a run does start in the prime, its status window reads `ly: tapestry`, with no `orch` label, because `RecipeWindowTitle` passes `primeName` as `""`.
   - That case is harmless, but the comment states a guarantee the code does not hold. I found no evidence either way, so this is a stated premise and not a confirmed bug.

3. **LOW** — `internal/boardengine/handles_test.go`, the `TestWindowTitles` row `ATaskUnderADoneBeamTakesTheBeamsHandle`.
   - This is a challenge to F4's proposal, and the proposal holds.
   - The row expects `ly: P8: T9: after-closed`, which is the fallback to `entry.Project` that the review asked to be pinned.
   - The behaviour is arguably odd: a closed beam's handle is shown as the `<P#>`. The test pins current behaviour and is consistent with the review's `fix` key. No change is needed.

F2 (own change: a tier 1 integration test in place of the `export_test.go` the review proposed), F3 (doc correction) and F5 (test rename and extra row) held up. I found no case where the proposal is wrong or breaks a sibling.

**Calls made**
- One `grep` for readers of `Display.Title` and `.Title` in `reedengine`, `render` and `statuspane`, plus a grep of the summary for the "window ordering" sentence. My context lacked both.
