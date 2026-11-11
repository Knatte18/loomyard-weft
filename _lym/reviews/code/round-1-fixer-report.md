## Changes

All findings fixed as proposed. Commits `aea1dd58` (F1–F7) and a follow-up (F1 hardening after the self-check).
The findings share two files, so F1–F7 went in one commit instead of one per finding.

- F1: `renderEveryWorktree` (`internal/heddlecli/verbs.go`) also skips an entry under a worktree staging directory (`fabricengine.InWorktreeStaging`) and one whose parent is not the hub.
  The parent comparison resolves symlinks and folds case on Windows (`samePath`).
  The `renderEveryWorktree` doc comment and `doc.go` name the new skips.
  `WriteRendersEveryLiveWorktree` creates a staging worktree and an out-of-hub worktree before the first write, so the existing `rendered` and `failed` assertions pin both skips.
- F2: `OverrideVerbs` asserts `rendered` names the prime after `set --layer lym` and `remove --layer lym`, and no `rendered` after both with `--no-render`.
  `WriteRendersEveryLiveWorktree` adds a `remove --no-render` step leaving the pair's file untouched.
  `renderedNames` and `requireNoRendered` became package-level helpers.
- F3: `WriteRendersEveryLiveWorktree` writes an unusable recorded anchor for the pair and a malformed override file, then asserts the pair in `failed` with the render way forward while the prime renders, and `render_warnings` holds one warning opening with the prime's name and naming the override file.
- F4: `withRenderWayForward` appends the `lym heddle render <name>` way forward to both the error and the unwritten case, unless the cause already names `lym heddle render`.
  The remove step asserts exactly one way forward.
- F5: `doc.go` and the `set` and `remove` help name `render_warnings`.
- F6: `WriteHoldsTheBoltLockAndCommitsNothing` records whether the prime's `CLAUDE.local.md` exists when the sync starts and asserts it did.
- F7: the prime status message and a comment now say no tracked file changed because the rendered files are git-excluded.

Gate repair: a `doc.go` comment line wrapped mid-sentence (the prunable, staging and out-of-hub skips).
It is now one sentence per line.
No other change.

## Self-check

- fix-code-correctness, LOW, no test for the staging and out-of-hub skips: fixed with my own test (the two extra worktrees above).
- fix-code-correctness, LOW, plain string match of the hub path: fixed, merged with the fix-proposal MEDIUM below.
- fix-proposal, MEDIUM, same plain string match can skip every worktree silently: fixed with `samePath` (symlinks resolved, Windows case folded).
- fix-proposal, LOW, the `EventFor` and `Render` error paths of the failure branch are untested: disputed, `premise false`.
- fix-proposal, LOW, the F4 count assertion passes on the engine's own way forward: disputed, `every fix worse`.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

- Self-check fix-proposal LOW (error paths untested), ground `premise false`: the `err != nil` case is one switch branch, reached through `NamedWorktree` failing in the new test, and it handles `EventFor` and `Render` errors identically through `withRenderWayForward`.
  A held Bolt lock, the alternative, fails the prime too and cannot show the prime still rendering.
- Self-check fix-proposal LOW (F4 way forward lacks the worktree name when the engine text names the verb), ground `every fix worse`: the review's F4 fix requires no second way forward when the cause already names `lym heddle render`.
  Appending one anyway prints two, which is the defect F4 reports.

## Deferred

None.

## Rulings

None.
