```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff, one case per function or branch.

**List**

- `Board.SetStatusReleasingProject`: member `d1` fails validation (dangling `depends_on`). `fn` returns the error and `boardCriticalSection` returns before `Save`, so the in-memory partial release is dropped. The assigned `released` stays nil on error. Correct (`board.go:122-125`).
- `Store.setStatusReleasingProject`, unknown selector: returns "task not found". Correct.
- Same function, non-null status or a non-project entry: delegates to `s.SetStatus` with nil released, the old path. Correct.
- Same function, project with null status and run status `running · Webster`: `IsRunStatus` is true, so it returns the refusal and writes nothing. The `*project.Status` dereference is safe because run status and done both imply non-nil. Correct.
- Same function, project with null status and `done`: same refusal path. Correct.
- Same function, project with a hand-set non-empty status such as `paused`: plain `s.SetStatus(selector, nil)`, no member released. This matches the record, which names only the empty, run-status and done cases. Correct.
- Same function, project with `""` status and draft members `d1` and `d2`: the project is cleared, then each member through `Store.SetStatus`, each validated as its own release. It returns `["d1","d2"]` in store order. Correct.
- Same function, project with nil status and draft members: the project write is a no-op and the members are released. Correct, since nil is an empty status.
- Same function, numeric id or handle selector: `current.Slug` is used for the project write and for `draftMembers`, so the members match. Correct.
- Same function, after it returns, the post-`fn` steps run on the released members (draft to nil). `applyDraftRule` skips them because `after[i].Status == nil` hits `continue` at `draft.go:60`, so they are not re-drafted. `checkRunLocks` and `checkProjectRules` then judge the whole write, and a failure there also drops it unsaved. Correct.
- `draftMembers`: returns only entries with `Project == project`, `Kind == KindTask` and draft status. A nested project (kind project) and its members, outsiders, and released, running or done members are excluded. Empty store gives nil. Correct.
- `projectReleaseRefusal` with no draft members: says there is none to release, so nothing was written. Correct. With members: one `releaseCommand` per slug, joined by `"; "`. Correct.
- `releaseCommand`: `json.Marshal` of a map with a nil status gives `{"slug":"d1","status":null}`, the string the test pins. The discarded error cannot occur for this payload. Correct.
- `boardcli` `set-status`: a nil or empty `released` gives `outputSuccess`, unchanged for tasks and non-null statuses. A non-empty one gives a notice naming the members. The null and empty-string status mapping upstream of the call is unchanged. Correct.

**Findings:** none. No defect in the lens's scope.

Outside my lens, one thing for the handler: in `internal/boardcli/verbs_test.go`, the new `stepProjectRelease` and its doc comment sit between `stepProjects`' existing doc comment and `stepProjects` itself. `stepProjects`' comment ("A member's project change is refused while a worktree stands...") now floats above `stepProjectRelease`'s own comment, and `stepProjects` has lost its doc. That is a comment-placement defect, not a correctness one.

**Tool calls:** none made. The inherited context already held every definition I needed.
