```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the fix-code-correctness lens over the F1–F6 change set (`add.go`, `destroy.go`, `addadoptonly.go` and the new tests).

I traced `ignoredTaskPathDeletable`, the deferred delete in `Add`, and each added test row. I found no wrong result and no broken caller.

**What I checked**
- `ignoredTaskPathDeletable` returns false, with a nil error, for each failure of `Lstat`, `List`, the walk and `check-ignore`. `Add` then returns `*ErrTaskPathTaken`, as the review asked.
- The delete now runs just before `createGitWorktree`, after the `defer regenerateAfterVerb`. A refusal anywhere in the pre-flight leaves the directory in place.
- A failed delete returns before any worktree exists, so no rollback is needed.
- `removeIgnoredTaskPath` is called only from `Add`, so the signature change breaks nothing else.
- `TestAdd_TaskPathLeftoverKeptWhenPreflightRefuses` ends at the `ErrNotLive{Records:true}` refusal, after the path is judged. It fails if the delete moves back before the pre-flight.
- `TestAdd_TaskPathLeftoverDeleteFails` uses an error that is not a late-writer error, so the seam stops after one attempt and no sleep runs. It fails if the `ErrStrayPathKept` wrap is dropped.
- The `TrackedFileKeepsIt` row fails if the tracked commit is removed, because the row would then delete the directory.

**Findings**

1. LOW — `ErrTaskPathTaken` comment, `addadoptonly.go`:
   - It says "A directory holding only git-ignored files is deleted by Add and never refused."
   - The code refuses such a directory when `Lstat`, `List`, the walk or `check-ignore` fails, which the review's F5 asked for.
   - Evidence: `ignoredTaskPathDeletable` returns false through `keep(...)` for each of those failures.
   - Suggested wording: "…is deleted by Add unless git cannot judge it."
2. LOW — `keep` warning in `ignoredTaskPathDeletable`, `destroy.go`:
   - The message "git could not judge the files…" is also logged when `Lstat` fails (path gone or unreadable) and when `List` fails. Neither is a git judgement of files.
   - The wording is slightly misleading but the behaviour is correct.

Tool calls made: none. My context already held every changed file and the test log.
