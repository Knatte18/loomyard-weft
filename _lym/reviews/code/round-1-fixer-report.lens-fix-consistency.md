```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set (new `TestRunCheckMarks` row; new `TestRunCLI_Down_IsNotRefusedInBolt` and `fabricRefusalText` const) for agreement with its surroundings.

1. **LOW**, `internal/reedcli/cli_test.go`, the new `fabricRefusalText` const.
   - The new const and its twin literal now disagree on style.
   - `TestRunCLI_AttachAndUp_RefuseFabricCheckouts` still spells out `"reed: lym reed " + tt.verb + " boots a session only in a worktree of the hub"` inline.
   - The const holds the shared tail of that same string.
   - A wording change to the refusal at `internal/reedcli/cli.go:168` now has to be made in two places in one file, and nothing links them.
   - Evidence: the refusal test's `want` list, against the const declared at the bottom of the new test block.

2. **LOW**, `internal/reedcli/cli_test.go`, the file header comment (lines 1–7).
   - It lists "the refusal of attach and up in fabric's own checkouts" but does not say the file now also pins that `down` is not refused in Bolt.
   - A reader choosing by header would miss the new test.

3. **LOW (reporting)**, review finding F2 versus the change set.
   - F2's `fix` asked for rows proving attach and up still pass the guard in a task worktree, plus a row for `down`.
   - Only the `down` row landed. The attach and up rows were removed because they start a real tmux server, which tier0 rejects, and tier2 left servers unregistered.
   - Nothing in the changed files says the accepted-path half is untested, and `cli_test.go`'s header still reads as if the guard is fully pinned.
   - The fixer-report must record that part as deferred or fixed differently, with the tmux evidence. Otherwise the review premise looks resolved when it is not.

4. **No finding**, the F1 row.
   - It matches its siblings' naming and mark shape. The tier-end mark sequence follows the existing `FailedEndsWithItsFirstFailedTierAlone` pattern.
   - The text matches the `checkMarkOf` branch at `check.go:334` exactly.

Tool calls made: none; the context already held every changed file.
