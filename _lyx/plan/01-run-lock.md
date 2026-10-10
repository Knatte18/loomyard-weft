# Card 1 — run-lock

**Create:**
- `internal/boardengine#RunStatus`
- `internal/boardengine#IsRunStatus`
- `internal/boardengine#ErrRunLocked`
- `internal/boardengine#RunLockedError`
- `internal/boardengine#RunLockedError.Error`
- `internal/boardengine#RunLockedError.Is`
- `internal/boardengine#checkRunLocks`
- `internal/boardengine/runlock_test.go`

**Edit:**
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine/doc.go`
- `internal/boardcli/doc.go`
- `internal/boardcli/intake_test.go`
- `plugins/ly/skills/board/SKILL.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
A board entry held by a run refuses every write that would change its scope, enforced once in the board's write critical section.

The new symbols live in a new file `internal/boardengine/runlock.go`.

- `RunStatus(state, producer)` composes the run-status form, `state + " · " + producer`, through one unexported separator constant.
  `IsRunStatus` is its predicate: true when the status is non-nil and splits on that separator into two non-empty halves.
  Nothing else locks: nil, empty, `done`, `abandoned` and a hand-set word such as `active` are not run statuses.
- `checkRunLocks(loaded, after)` judges each entry of `loaded` whose status `IsRunStatus` by its net change from `loaded` to `after`, so a status clear earlier in the same write (a batch listing one slug twice, a merge whose `set_status` and upsert hit the same entry) never unlocks a later edit.
  It refuses when the entry is absent from `after`, or when any field but `Status` differs: `Title`, `Kind`, `Labels`, `Issues`, `Recipe`, `DependsOn`, `Isolated`, `Brief`, `Body`, `ShortName`.
  Slice fields compare element-wise with nil equal to empty, so an upsert repeating the stored values is no change.
  Prune exception: before comparing `DependsOn`, drop from the loaded list every slug the same write removes as a `done` entry (present in `loaded` with status `done`, absent from `after`); every other `DependsOn` change on a locked entry is still refused.
  It returns the first refusal in `loaded` order as a `*RunLockedError`.
- `RunLockedError` carries the entry's `Slug` and `Status`; `Is` matches `ErrRunLocked`, so callers use `errors.Is`.
  `Error()` names the slug, the current status, and both ways forward, with only board and batten verbs (`PATTERN-no-denied-recovery`):
  record the new finding as a note of its own with `lyx board upsert` and `"kind":"note"`;
  to unlock, run `lyx batten status <slug>` from the hub's prime worktree and clear the status with `lyx board set-status` only when no run holds the entry, which is when that check refuses because the slug has no batten seed, reports state `done`, or reports a halted state (`paused`, `blocked`, `failed`, `awaiting`) whose run the operator has decided to abandon; a `running` state means a run holds it.
- `boardCriticalSection` snapshots the loaded store's entries right after `Load`, cloning each entry's slice fields so no mutator can alias them, and runs `checkRunLocks(snapshot, store.Tasks())` after `fn` returns and before `Save`, returning its error unsaved.
  A `noWrite` result skips the check, as nothing is written.
  Every mutating Board method goes through it, so every caller gets the lock and a refused write mutates nothing on disk.

What this lets through, bounded: a status-only net change on a locked entry (`set-status`, a merge's `set_status`, an upsert differing only in `status`) passes, and `set-status` itself is unguarded, so clearing a live run's status is possible; the refusal names the `lyx batten status` check to run first.
`intake import` with `into` a locked entry is refused by the board write, which already precedes the GitHub comment and close, so the issue stays open and uncommented.

Tests, in the new `runlock_test.go` as one table-driven `TestRunLock` over a temp-dir Board with `SkipGit`, seeded with a locked entry (each of `running · Webster`, `blocked · X`, `paused · X`, `failed · X`, `awaiting · X`), assert per row the error (`errors.Is(err, boardengine.ErrRunLocked)` and the message naming slug, status, `lyx batten status <slug>`, the prime worktree and the note way forward) or success, and that a refused row leaves `board.json` byte-identical.
Refused rows: upsert changing each of body, brief, labels, issues, title, kind, recipe, depends_on; upsert changing status and body together; `SetDeps`; `Promote` of a locked note; `RemoveTask`; merge removing it; merge upserting into it with a change; the same merge with `set_status` clearing it; `UpsertTasksBatch` with one changing item (no item lands); `UpsertTasksBatch` listing it twice, first clearing status, then changing body.
Passing rows: `SetStatus` to another value and to nil; merge whose `set_status` targets it; an identical upsert; a status-only upsert; a batch whose only item on it is status-only; prune on a board where it depends on a `done` entry (the done entry is removed and dropped from its `depends_on`); writes to other entries, including adding a dependency on the locked entry; and one body-changing upsert on an entry whose status is `active`, which shows the check consults the predicate.
`IsRunStatus` and `RunStatus` get their rows in the same test: `IsRunStatus` is false for nil, empty, `done`, `abandoned` and `active` and true for each seeded run status, so writes on an unlocked entry need no rows of their own, as the existing board tests already pin them.
Add a row to `TestIntakeImport_RefusedBeforeAnyWrite` in `internal/boardcli/intake_test.go`: the seed also holds a locked entry, and `{"issue":N,"into":"<locked>"}` exits 1 with the run-lock message, makes no GitHub mutation and leaves `board.json` unchanged.
Raise `internal/boardengine`'s row in `test-budget.yaml` by one.

Docs: `internal/boardengine/doc.go` gains a `# Run lock` section (the predicate, what is locked, the status-only pass, the prune exception, net-change judging, the refusal and its sentinel);
`internal/boardcli/doc.go` names the verbs that refuse on a locked entry (`upsert`, `upsert-batch`, `remove`, `merge`, `set-deps`, `promote`, `intake import` with `into`), and that `set-status` and `prune` never do;
`plugins/ly/skills/board/SKILL.md` says a claimed entry takes no scope edits and a finding goes to a note of its own.

**ImpactSummary:** every mutating Board method gains one in-memory check before save; writes to run-held entries that changed scope now fail, all other writes are unchanged.

**Commit:** 1: board: lock a run-held entry's scope in the write critical section
