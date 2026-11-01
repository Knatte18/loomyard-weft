## Fixer-report, round 1

### Changes

- **F1** (fixed as proposed, extended to the other slug entry points), commits `faed9bbd` and the follow-up on the same branch:
  - `internal/boardengine/task.go`: `validateSlugNotReserved` refuses slug `RootSubject`; `NewTask` calls it beside `validateSlugLength`.
  - `internal/boardengine/rename.go`: `Rename` refuses the reserved target, since a rename sets a slug the same way.
  - `internal/boardengine/task.go` `ApplyPatch`: refuses a patch that moves an entry onto the reserved slug; an entry already stored as `root` stays patchable.
  - `internal/boardengine/render.go`: `rootProjectFile` is `projectFileName(RootSubject)`.
  - `internal/boardengine/doc.go`: the claim list says `project-root.md` is the root project's file and not a project's.
  - Tests: a `NewTask` refusal subtest, a `TargetIsRootSubject` row in `TestRename_Refusals`, and `ApplyPatch` subtests for the refusal and for patching an entry already named `root`.
  - `lym gate test ./internal/boardengine`: all passed.

### Self-check

- fix-code-correctness, MEDIUM: `ApplyPatch` accepts slug `root` — fixed with a change-only check, so a stored `root` entry stays patchable.
- fix-code-correctness, LOW: no `ApplyPatch` test row — fixed, rows added.
- fix-consistency, MEDIUM: same `ApplyPatch` gap — merged with the first, fixed.
- fix-consistency, LOW: no `ApplyPatch` case beside the length twin — merged, fixed.
- fix-consistency, LOW: render docs do not say `root` is reserved — not changed; the error text and `RootSubject` doc carry it, and the finding is optional.
- fix-proposal, LOW: a stored entry named `root` is not re-checked on load or render — deferred: refusing on load would break an existing board, and the review proposed no such check.
- fix-proposal, LOW: the reservation covers every kind — kept; `RootSubject` is also the `lym adopt` subject, so a task named `root` is ambiguous there too.
- fix-affected-set: no findings.

### Disputed

None.

### Deferred

- Load-time or render-time detection of an already stored entry named `root`: needs a decision on how a board holding one is migrated.
