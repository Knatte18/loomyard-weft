```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: for each guard, refusal, branch and test row the cards add, I traced whether a real entry point can reach the state it guards, and whether the guarded site recurs on a path the card leaves unguarded.

**Per guard: reachable or not**

- **Card 2 `checkExpected`, stale revision.** Reachable through all three writes card 4 makes:
  - `UpsertTaskReporting` and `PromoteTo` go through `writeReportingSiblings`, which calls `boardCriticalSection` (`internal/boardengine/siblings.go:114-116`).
  - `SetStatusReleasingProject` calls it directly (`projectrelease.go:19-25`).
  - Both load migrations are deterministic (`draft.go` `migrateDraftStatus`, `sketch.go` `migrateDiscussedStatus`, neither stamps a time), so the revision `Board.View` gives can match the write section's on an unmigrated board.
- **Card 2, removed entry (nil `Fresh`).** Reachable through edit: the check runs before `fn`, so `UpsertTaskReporting` never re-creates the slug.
- **Card 2, create of an existing slug.** Reachable only in a race from the server, because card 4's view pre-check answers first. Card 2's own test reaches it directly.
- **Card 4 transport refusals (Host, token, `t`, Content-Type, Origin, body cap).** Each is reachable through `ServeHTTP` with a crafted request, and none sits behind an earlier check.
- **Card 4 create pre-check (existing slug, 409).** Reachable.
- **Card 4 promote of a task, refused before the write.** Reachable, and needed: `store.promoteTo` returns a task unchanged with no error when `dependsOn` is nil (`store.go:828-832`).
- **Card 4 promote of a project.** Reachable; `promoteTo` refuses a project (`store.go:825-826`).
- **Card 4 release, draft and undraft, view-level stale or missing check.** Reachable. The write section's re-check is reachable only in a race, and card 2 covers it generically.
- **Card 4 non-task refusal for release, draft and undraft.** Reachable through the API, though the page never offers these actions on a note.
- **Card 4 run-status release refusal, and the draft and undraft status refusals (three undraft variants).** All reachable.
- **Card 4 unknown op and unknown field.** Reachable.
- **Card 4 edit of a running task's title, run lock.** Reachable: the title counts as scope (`runlock.go:146-147`, "a title, body or other scope change ... is refused").
- **Card 4 file route (unlisted name, `a/b.md`).** Reachable; `ServeHTTP` is driven directly, with no mux cleaning the path.
- **Card 5 `ErrRunning`.** Reachable when two `open`s race or `serve` is run by hand.
- **Card 5 `Live` over a missing state directory, and the watch-failure fallback to the timer.** Both reachable.
- **Card 6 `refuseRunAgent`.** Reachable for `serve` run by hand and for `open`. It runs before the `--board-path` refusal, and `PersistentPreRunE` refuses nothing first under `--board-path` (`cli.go:125-138`).
- **Card 6 `--board-path` refusal.** Reachable; `PersistentPreRunE` leaves `location` nil.
- **Card 6 lock held with a state whose pid is not running.** Reachable in the window between a new server taking the lock and removing the previous server's state file.
- **Card 6 wait bound, `Prime` error, spawn error, opener failure, opener cap.** All reachable through the `openDeps` fakes.
- **Card 8 mark refusal, and `BuildWorkspace`'s `folders` and `settings` refusals.** Reachable only through a test's hook, which is a programmer error, as the card states.
- **Card 8 degrade branch (beam anchor unreadable, logged at Warn, built with no members).** Reachable: a marked beam whose worktree is gone. No test row reaches it.
- **Card 9 `LymBinary` error falling back to `lym`.** Reachable through the swapped seam.

**Findings**

1. **MEDIUM, design.**
   - **Location:** `_lym/plan/04-board-server-routes.md:63`; the wider case is at lines 59 and 64.
   - **Summary:** The empty-revision "create" semantics are not guarded on the non-create ops. An `edit` request carrying an empty `revision` for an absent slug passes `Expecting(slug, "")` and lets `UpsertTaskReporting` create the entry. That bypasses create's field list (no `kind`) and its existing-slug pre-check, and the bound at line 83 ("the page reaches only these six writes") does not cover it. A `promote` carrying an empty revision on an existing note is refused with the create refusal's "already exists" message instead of a stale one.
   - **Evidence:**
     - `_lym/plan/02-entry-revision-check.md:31`: "An empty revision means the write expects no entry of that slug: a create."
     - `04-board-server-routes.md:63`: edit takes `Expecting(slug, revision)` "with no pre-check".
     - `04-board-server-routes.md:64`: promote likewise.
   - **Fix:** `applyWrite` refuses an empty `revision` for every op but `create`, and `TestServer`'s refusals row gains an edit with an empty revision of an absent slug that changes nothing.

2. **NIT, consistency.**
   - **Location:** `_lym/plan/00-overview.md:31`.
   - **Summary:** The Shared Decisions bullet says a promote carries its revision "straight to the write section". Card 4 gives promote a guard placed before the write, the refusal of a task, and that guard can only read the current `Board.View()`.
   - **Evidence:**
     - `04-board-server-routes.md:65`: "A task is refused before the write".
     - `store.go:828-832`: `promoteTo` returns an existing task unchanged, so the guard cannot sit in the write.
   - **Fix:** The bullet names promote's kind check against the view.

**Tool calls**

1. `writeReportingSiblings`, `migrateDraftStatus` and `migrateDiscussedStatus`: whether edit and promote reach `boardCriticalSection`, and whether the migrations are deterministic. The manifest held neither body.
2. `runlock.go` lines 1-80: which fields the run lock guards, for card 4's running-task row.
3. `runlock.go` lines 40-72: the `RunHolds` scope comment; the scope fields were still not named.
4. `runlock.go` lines 100-200, filtered: the `checkRunLocks` doc, to confirm the title is a scope field.
5. `store.go` `promoteTo`: whether it refuses a task or a project, for card 4's promote guards.
