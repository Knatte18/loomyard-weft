```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code (`RecipeWindowTitle`, `worktreeWindowTitle`, `statusWindowTitle`, and the added tests) by running concrete inputs through them.

Findings:

- **LOW** — `internal/boardengine/handles_test.go`, row `NoPrimeNameTitlesThePrimesSlugAsATask`. It passes even if the `primeName != ""` guard in `worktreeWindowTitle` is removed. With slug `main`, `slug == primeName` is false when `primeName` is `""`, so the guard changes nothing for this input. Only a slug of `""` would exercise the guard. The row still pins the result the doc comments promise, but it cannot fail on that guard.

No other findings. What I ran:

- **`RecipeWindowTitle`** (`primeName` is `""`, so the prime branch is skipped):
  - Task in project P3 gives `ly: P3: T7: pawl`.
  - Beam gives `ly: P2: pawl`.
  - Slug not on the board gives `ly: pawl`.
  - Unreadable board gives `ly: pawl`, taking `name` as the slug position.
- **`IDEWindowTitle` and `TerminalWindowTitle`** now pass `slug` as `name`, so their old outputs are unchanged. They are the only other callers of `worktreeWindowTitle`.
- **New `TestWindowTitles` rows:**
  - `after-closed`: the done beam is skipped by `ParentBeam` (`StatusDone` is `"done"`), so the code takes the `entry.Project` fallback and gives `ly: P8: T9: after-closed`. The row fails if the fallback is removed.
  - `orphan`: the project is absent, so no P# and the title is `ly: T10: orphan`.
  - Unreadable-board beam: gives `ly: bm`, which pins the corrected doc.
- **`statusWindowTitle`:** the shortname-and-project row gives `ly: P3: T4: tapestry`. The other `loomcli` rows and `TestStatusStrandSpec_TitlesTheWindowWithTheRunsHandleAndRecipe` have no shortname, so their results are unchanged. There are no other callers, and the imports in `sharedbootstrap.go` are still used.
- **`battencli`:** `TerminalWindowTitle(hub, "main", "run")` gives `ly: P3: T4: run` for the shortname row. The shortname row fails if the shortname is lost, and the project row fails if the project handle is lost.
- **`reedcli`:** the tier 1 log shows `TestTerminalWindowTitle` and its first subtest ran, so it was not skipped.

Tool calls:
1. One grep for `StatusDone` in `internal/boardengine`: my context lacked the constant's value. It decides whether the `after-closed` row really exercises the done-beam path.
2. Two greps of the tier 1 gate log for `TestTerminalWindowTitle`: my context lacked proof that the new test ran and was not skipped.
