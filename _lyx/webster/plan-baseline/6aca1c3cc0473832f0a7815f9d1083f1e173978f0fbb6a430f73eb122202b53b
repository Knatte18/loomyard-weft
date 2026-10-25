# Card 5 — board-draft-rule

**Edit:**
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine/doc.go`
- `internal/boardcli#storeVerbs`
- `plugins/ly/skills/board/SKILL.md`

**Create:**
- `internal/boardengine#StatusDraft`
- `internal/boardengine#IsEmptyStatus`
- `internal/boardengine#applyDraftRule`
- `internal/boardengine_test#TestDraftStatus`

**Intent:**
Make `draft` the status of every new task, by one rule at the store's commit point (D5, and the Added-after-Discussion entry placing it in `Board.boardCriticalSection`).
`IsEmptyStatus` reads a nil status and an empty string alike, the one meaning of an empty status in the start condition, the migration and the release paths.
`applyDraftRule` runs in `boardCriticalSection` beside `checkRunLocks` and `checkTaskCreation`, after the mutation and before the checks, over the entries as loaded (renamed already) and as the mutation left them, and sets `draft` on each entry of kind task whose status is empty, unless the entry was a task before the write and either the write left its status untouched (the same empty value as loaded) or the write set it to nil (a release).
So a create of any verb, a batch or merge create, a note or project upserted to a task and a note promoted give `draft` even when the payload sets `status` to null, an empty string written to a task whose loaded status differs gives `draft`, and an edit of a released task that leaves its status alone keeps it released.
The rule is not placed in `Store.Save`, which holds no pre-write entries, and the owner write in `ownerset.go` and the replay in `replay.go` run no draft rule: the first never makes a task and the second copies entries a critical-section write already normalised.
`draft` is not a run status, so `IsRunStatus` and the run lock are unchanged and a draft entry locks nothing.
The package doc's run-lock section names `draft` among the statuses that lock nothing, and a new paragraph states the rule and the release, `lyx board set-status '{"slug":"<slug>","status":null}'`, an upsert setting `status` to null on a task or a merge whose `set_status` sets null.
`set-status`'s and `merge`'s help in `storeVerbs` and the `ly:board` skill's `set-status` line say that a new task is `draft` and that a null status on a task releases it, rather than a plain clear.
`TestDraftStatus` (tier0, external, parallel), a table over a board built in a temp dir: `UpsertTask` of a new task, `UpsertTasksBatch` and `MergeTasks` creates, an upsert turning a note into a task with `status: null`, an upsert turning a project with no members into a task, `Promote` of a note, and `SetStatus` with an empty string on a task holding `running · X` each read `draft`; `SetStatus` nil on a draft task, an upsert with `status: null` on a task, and a merge whose `set_status` sets null each read nil; a later upsert of that released task that changes its brief alone still reads nil.

**ImpactSummary:** One rule inside the board's write critical section; store-level tests, which build a Store directly, are unaffected, and every verb gains draft through the one call site.
