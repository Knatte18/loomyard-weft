```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the round-2 changes to `decision-record.md` and `support-log.md` agree with the sibling text they touch.

**Findings**

1. **MEDIUM.** Changed text: Decision 5's fd reopen "once `placeDobby` returns its Geometry and before anything else writes", read against Decision 3's last bullet, the Open risks bullet and the new criterion "a refusal with no resolved worktree leaves its envelope in `dobby.out` and nothing in `dobby.log`".
   Summary: a dobby whose fds an older binary pointed at `dobby.log` gets no reopen before a `placeDobby` refusal. Its envelope therefore lands in `dobby.log`, not `dobby.out`.
   Evidence: Decision 5 puts the reopen after `placeDobby` returns, so a refusal, which returns no Geometry, keeps the inherited fd 1 and 2. That contradicts Decision 5's own "no raw process output reaches the file". It also contradicts the `dobby.out` bound that Decision 3 and Open risks state for a refusal at `filepath.Abs` or `lymcwd.ResolveWorktree`, and the new criterion. Either the bound must name this inherited-fd case, or the reopen must move before `placeDobby`, which needs a path it does not yet have.

2. **LOW.** Changed text: Decision 1, "at a path the told Geometry gains beside `LogPath`" (the prime's `dobby.log`).
   Summary: Decision 1 added a new told Geometry path, but the Constraints bullet for PATTERN-cwd-resolution still names only `dobby.out` as the new path `dobbyengine` takes from its told Geometry.
   Evidence: the Constraints line reads "`dobbyengine` takes the new `dobby.out` path from its told Geometry and imports no `lymcwd`". The prime log path is the twin that was not updated.

3. **LOW.** Changed text: Decision 3's post list, which now adds a `placeDobby` refusal after a re-exec and a fatal `Run` error after a re-exec.
   Summary: the support log's question-ledger row was left as it was.
   Evidence: `support-log.md` still reads "Orch notices for dobby exits — auto-pick: failed exec, refused re-exec start and a wiring failure after a re-exec only; resolved (Decision 3)".

4. **LOW.** Changed text: Decision 2's second read, "Only a refused re-exec whose second read still holds its own pid posts the refused-re-exec notice".
   Summary: the post list in Decision 3 still names "a refused re-exec start" without the second-read condition.
   Evidence: Decision 3 reads "A failed exec, a refused re-exec start, … post one notice". A reader of Decision 3 alone would post on every refused re-exec. A clause pointing to Decision 2's second read would keep the two in step.

5. **LOW.** Changed text: Decision 1, the worktree-gone line "appended without rotating since this process does not hold the prime's lock".
   Summary: this is a third lockless writer, and Decision 5's list of writers that append without rotating does not cover it.
   Evidence: the beam dobby does hold a lock, its own, so it is neither "the lock holder" of the prime's file nor "a process holding no lock". Decision 5 reads "Only the lock holder rotates; a process holding no lock, such as Decision 3's refused start or wiring failure, appends its line without rotating". It should say that only the holder of that file's own lock rotates it.

I made no further tool calls: everything I checked is shown in its changed form in my context.
