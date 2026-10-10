# board: refuse scope edits to a board entry a run holds

A board entry whose `status` is a run status (`<state> · <producer>`, as loom writes on every transition) now refuses any write that changes its scope.
Before, the orch could fold new issues into a running task's entry, so the entry stopped matching the plan its run was executing.

The refusal names the entry, its status and two ways forward: record the finding as a note of its own, or, when no run holds the entry, check `lyx batten status <slug>` from the prime worktree and clear the status with `lyx board set-status`.
It is a `boardengine` error matchable with `errors.Is` against `ErrRunLocked`, printed through the usual JSON error envelope.

## What changes

- `internal/boardengine/runlock.go` declares the run-status form once: `RunStatus` composes it and `IsRunStatus` recognizes it.
  Loom's `boardStatus` and batten's claim both compose through `RunStatus`, so the writers and the lock cannot disagree on the form.
- The lock is checked inside the board's write critical section, on each entry's net change from the entry as loaded under the file lock.
  Every mutating path is covered: `upsert`, `upsert-batch`, `remove`, `merge` (removed entries and the upserted entry), `set-deps`, `promote`, `prune` and `intake import` with `into`.
  A refused write mutates nothing, and a refused `intake import --into` happens before the GitHub comment and close.
- `lyx batten run` claims the entry as `running · Worktree-Create` in its Worktree-Create row, before the pair is created, so no edit slips in between the claim and loom's first transition.
  Absent and `done` entries are skipped.
  A failing board write only warns, and the run proceeds.
  Batten writes no status after that row.
- `boardengine.OpenHub` is the one opener for the hub's board, shared by batten's claim and loom's landing and status writes; `boardkit` gains the matching test helper.
- Docs: `internal/boardengine/doc.go`, `internal/boardcli/doc.go`, and `plugins/ly/skills/board/SKILL.md` (a claimed entry takes no scope edits; findings go to a note).

## Design decisions

- Only a run status locks.
  No status, an empty one, `done`, `abandoned` and hand-set words such as `active` leave the entry open, and landing's `done` releases the lock.
- Every field but `status` is locked, and so is the entry's existence: title, kind, labels, issues, recipe, `depends_on`, isolated, brief, body, short name.
  A write that changes nothing passes.
- `status` stays writable in a status-only write, so loom's per-transition write, landing's `done`, `set-status` and a `merge`'s `set_status` keep working.
  A write changing `status` together with another field is refused, and a batch or `merge` is judged on the combined change, so an earlier status clear in the same write never unlocks a later edit.
- `prune` may drop a `done` slug that it removes from a locked entry's `depends_on`; any other `depends_on` change is refused.
- `merge` is refused when it removes a locked entry, or changes an existing locked entry it upserts into (carried issues count as a change).
- If `Topology.Add` fails after the claim, the claim stays; the stopped run resumes with `lyx batten run <slug>`, and abandoning it takes the manual unlock above.
- No new `PATTERN.md` entry: the lock is one module's rule, enforced at one site and documented in `doc.go`.

## Risks to check

- `set-status` is unguarded: an agent that skips the refusal's `lyx batten status` check can clear a live run's status and edit the entry.
  Loom's next status change writes the status back, so the edit shows only in the board's history.
- A batten process that died without recording a halt still reads `running`, so its entry stays locked until the operator clears the status.
- A failed claim write leaves the lock window open until loom's first transition, as before this change.
- Any flow that edits a running entry's body mid-run now fails and must use a note; the board skill text says so.
- The planned `git-robustness` task ("batten refuses a done slug") touches the same Worktree-Create row; the claim already skips a `done` entry, so the two should compose without a behavior conflict.
- Out of scope: `lyx fabric remove` of an unlanded pair does not clear the status, so that entry needs the manual unlock.

## Not run by hand

- The refusal on a real board through the CLI, for example `lyx board upsert` changing the body of an entry marked `running · Webster`.
- `lyx batten run` against a live hub, to see the entry read `running · Worktree-Create` before the pair appears.
- `go test ./...` and `go test -tags integration ./...` should be green before the push to `main`; confirm.
