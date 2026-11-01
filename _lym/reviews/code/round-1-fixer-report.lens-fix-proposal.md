```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal (reserve slug `root` in `NewTask`, build `rootProjectFile` from `RootSubject`, add a test row, fix the `doc.go` claim) and the rename check the fixer added.

Findings:

1. **LOW** — `internal/boardengine/task.go` `validateSlugNotReserved`, called from `NewTask` and `Rename` only.
   - Summary: an entry with slug `root` that is already on a board still renders into `project-root.md` and overwrites the root project's file, because nothing re-checks stored entries.
   - Evidence: `ApplyPatch` (`task.go:435`) calls `validateSlugLength(result.Slug)` but not the new check, so a patch that carries `"slug":"root"` is not refused there. I did not confirm that `store.go:530` lets a patch change the slug.
   - A board that already holds such an entry gets no refusal on load or render. `Render` still returns no error for it.
   - The `fix` key named only `NewTask`, so this is a limit of the proposal rather than a fault in the change.

2. **LOW** — `internal/boardengine/task.go` `NewTask` check.
   - Summary: the reservation covers every kind, not only projects.
   - Evidence: the review's premise is a project colliding on `project-root.md`, but the check refuses a task or note with slug `root` as well. The error text says "reserved for the root project", which reads correctly for all kinds.
   - Nothing in the repo uses slug `root` for a non-project: a grep for `"slug":"root"` and `Slug: "root"` found no match.
   - The broad reservation is consistent with `RootSubject` being the `lym adopt` subject name, so a task named `root` would also be ambiguous there. No breakage found.

No finding on the other changes. The `RootSubject` constant is defined in the same package, so the `rootProjectFile` initialisation order is fine. The new `NewTask` test row uses a project with only a slug and kind and still reaches the reserved check first. The `doc.go` claim now matches the file set `Render` writes.

Tool calls (2 of 6):
- `grep` for a literal slug `root` in `.go`, `.json`, `.md` and `.yaml` files: my context did not show whether any fixture or caller already uses it.
- `grep` for `NewTask(` and `ApplyPatch(` call sites outside tests: my context did not show which store paths reach the new check.
