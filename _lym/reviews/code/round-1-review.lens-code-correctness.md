```yaml
lens: code-correctness
outcome: reported
```

Task: run the merge-parent code through concrete cases under the code-correctness lens.

**Per-function list**

- `mergeParent`, beam `b` forked from main, hub budget off (`Soft` 0): rounds = max(0, 1) = 1, hard = max(Hard, 2). Correct.
- `mergeParent`, hub `Soft` 3 and `Hard` 5: both kept. Correct.
- `mergeParent`, `ReadOwners` or `Tasks` or `CodeReviewCaps` fails: each wraps the error with the verb, and nothing is filed. Correct.
- `mergeParent`, task or grouping project or done beam: `judgeLandEntry` refuses before any read, with a way forward. Correct.
- `mergeParent`, `origin.ParentBranch == ""`: refuses naming `lym fabric reconcile`. Correct. `judgeLandEntry` has already refused a missing or non-beam origin record.
- `mergeParent`, label lookup finds no entry named `slug`: `labels` is nil. Unreachable, because the owners check passed for the same board. Correct.
- `mergeParent`, `FileTask` fails: wraps the error and names a retry. Correct for `UpsertTask` failing. See finding 1 for the case where `SetStatus` fails.
- `openMergeTask`, a merge task with `Status` nil (released) or `"draft"`: counts as open. A `"done"` one is skipped. Correct.
- `openMergeTask`, a hand-written task in the project ending `-merge-in`: counts as an open merge task. The match is by project and suffix only. Acceptable but loose; see finding 3.
- `freeMergeTaskSlug`, no collision: `b-merge-in`. With `b-merge-in` taken: `b-merge-in-2`. Correct.
- `freeMergeTaskSlug`, 30-character beam: the head is cut to 32 − 9 = 23 characters, so the slug is 32 long. Correct. For the `-N` tails, `room` is smaller and the head is cut further. Correct, and the loop always ends.
- `mergeParentBody`: a pure format string. `%[1]s` and `%[2]s` are used consistently. Correct.
- `hubLandDeps.FileTask`: `UpsertTask` then `SetStatus(slug, nil)`, the same two-step release `fronttask` uses (`internal/fronttask/hub.go:45`). Correct on success.
- `hubLandDeps.CodeReviewCaps`: `loomengine.LoadConfig(fabricengine.ConfigDirs(prime), "loom")`, then `cfg.Caps(PerchCode)`, which resolves through `reviewbudget.Resolve` (`internal/loomengine/caps.go:35`). A config load failure is returned and wrapped by the caller. Correct.
- `mergeParentCmd` and `onHubLandDeps`: they follow `landCmd`'s path (abort check, `finishVerb`). Correct.

**Findings**

1. LOW, `design`, `internal/beamcli/deps.go` (the `FileTask` closure at lines ~187-199).
   - `UpsertTask` can succeed and `SetStatus(task.Slug, nil)` then fail. That leaves a draft merge task on the board, and `mergeParent` reports "file the merge task … failed; run again".
   - The rerun finds the draft task through `openMergeTask` (`mergeparent.go`), since its status is not done. It refuses with "stands open", and the way forward is `lym batten cancel --remove`.
   - That cancel is aimed at a task that has no run. This is a narrow failure path, and `fronttask` has the same shape. Evidence: `deps.go:187-199` and `mergeparent.go:~78`.

2. LOW, `design`, `internal/beamcli/mergeparent.go` (the `Tasks` read, `openMergeTask`, `freeMergeTaskSlug`, `FileTask` sequence).
   - The check and the write are not atomic. Two concurrent calls, such as a button double-click, can both read "no open merge task" and pick the same free slug.
   - `UpsertTask` is an upsert by slug, so the second call overwrites the first task's fields instead of failing. It also re-releases the task, even if the first has already started.
   - It is a narrow race. A confirm prompt precedes the button, which lowers the odds.

3. NIT, `design`, `internal/beamcli/mergeparent.go` (`mergeTaskSlugPattern`).
   - The pattern is unanchored at the front, so any non-done task in the beam's project whose slug ends in `-merge-in` or `-merge-in-N` counts as the beam's open merge task, including one written by hand.
   - The refusal then names a task the verb did not file. The summary acknowledges that the board has no marker field.

**Tool calls**
1. Bash, grep of `LoadConfig`, `ConfigDirs`, `Caps` and `run_settings`: my context lacked the config-load definitions and the board's `run_settings` validation.
2. Bash, read of `board.go` `UpsertTask` and `SetStatus`, `draft` in `store.go`, and the `SetStatus(…, nil)` callers: my context lacked the board's release semantics.
