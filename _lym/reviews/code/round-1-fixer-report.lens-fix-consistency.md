```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F7 change set for stale siblings and twins left unchanged.

**Findings**

1. **MEDIUM**, `internal/runproc/doc.go:10-11`. The package doc still says `List` "drops every record whose pid is dead, a zombie, or now carries another start time, with its lock file". It also says a zombie "reads as gone". After F3, `List` leaves a zombie's record on disk and only omits it from the result. It deletes the record when the pid is gone or carries another start time. This contradicts the new `List` doc in `record.go:125` and `liveness`. The doc also says nothing about the end sequence keeping a zombie leader listed. Suggested rewrite: "drops, with its lock file, every record whose pid is gone or now carries another start time … a zombie's record is left out of the result but kept on disk until it is reaped".

2. **LOW**, `internal/proc/doc.go:12-15`. The text is consistent with the fix. No change is needed there. `Reap` is described generically, so the F1 call sites (tmux server, `code`) need no edit.

3. **LOW**, `PATTERN.md:79`. `PATTERN-reap-children` is now satisfied by the tmux server start and the vscode launches. No edit is needed. I did not search for other bare `cmd.Start()` sites outside the two the review named; that belongs to the affected-set lens.

4. **LOW**, `internal/runproc/record.go:109`. The `lister` doc reads "so a test reaches the Windows branch and a zombie on any platform". It is still accurate. The new `isAlive` field is documented on the struct, so no stale text.

**Calls**
- `Read internal/runproc/doc.go` (lines 1-30): my context lacked that file, and the earlier grep showed zombie wording in it.

The greps ran through one Bash call before this Read.
