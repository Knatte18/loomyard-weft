```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked the record's bounds, guards and claims about existing code.

**Claims attacked**
- D8, the compare and write are atomic against every other writer: held for writers that use `boardCriticalSection`, and Bolt's replay commit takes the same board write lock. I did not check whether the owner-set, hold and registry writers take it.
- D8, the projection's revision and the write section's revision are taken over the same bytes after migrations: held. `migrateDraftStatus` is deterministic on the store flag.
- D8, a create of an existing slug is refused carrying the existing entry: broken at an edge (F5).
- D8, the check covers only the entries the write names: held, and it is stated.
- D9, the token keeps out other users: broken on Windows (F1).
- D9, the Host check, no CORS, and the `Origin` and `Content-Type` checks keep out web pages: held. The `Authorization` header forces a preflight that the server never answers.
- D9, `/` holds no board data: held.
- D9 bound, the server lets through nothing the CLI's operator writes do not: held in principle, but the writes depend on the starting worktree (F3).
- D5, a held lock proves a live server whose recorded port is its own: broken during the server's startup and drain windows (F2).
- D11, a second `serve` exits and the opener's wait finds the winner: held, except for the startup window in F2.
- D11, a server whose starting worktree is removed keeps serving the hub until it exits: broken for writes, and the exit never comes while a page is open (F3).
- D11, the idle exit is kept alive by an open page: held. A frozen tab degrades to the retry line, as D11 states.
- D11, `.lym` is never staged: held. Creel has the same precedent, and fabric's third bucket refuses never-committed paths.
- D11, a takeover ends the server through `runproc`: held only if the server's new role stays out of `subjectPassSpared` (F6).
- D11, file watch and timer fallback, and replays count as changes: held.
- D3 bound, mixed-version exposure heals on the next up-to-date write: broken for an older machine replaying a newer machine's commits (F4).
- D10, release acts on a running task as `set-status` does: the bound is stated, but its consequence is not (F7).
- D10, release and undraft set status to none without re-drafting: held, provided the none status is nil. `applyDraftRule` drafts an emptied status and spares only nil.
- D10, the file route serves only names the projection lists: held.
- D4, link sanitising (entity tricks, upper-case scheme, C0 trimming, `/\host`): held.
- D1 JSON escaping and D2 identical-bytes rewrites: held.
- D6, no extra member without a hook, and the bare `lym` fallback: held.
- D6, every copy of `board open` in a multi-root window gives the same result: broken, because the copy that runs decides the server's starting worktree (part of F3).

**Findings**

**F1 — BLOCKING, design, D9 (and D11's state and launcher files)**
- Mode 0600 does not keep other users from reading the token on Windows, which is in scope.
- Counterexample: a hub under `C:\Code\…` inherits an ACL that lets other authenticated users read files. Another user reads the state file or launcher and writes the board as the operator.
- The repo already pins this fact: `internal/githubclient/doc.go:31` says Windows "ignores Go's permission bits", and `internal/githubclient/cache_windows.go:17-20` installs an owner-only DACL.
- Fix: state an owner-only DACL on Windows for the state file and launcher, citing that pin.

**F2 — MEDIUM, design, D5 liveness read and D11 ordering**
- "A held lock proves a live server … so the recorded port is the server's" fails in two windows.
- Startup: D11 has `serve` take the lock first, then build the board, bind and write its state. A second `open` (a quick second key press) in that gap reads the lock as held and hands the opener the leftover state file of an earlier server, with a dead port and token.
- Drain: D11 says an exiting server stops taking requests and drains while still holding the lock. An `open` during that window, such as right after an upgrade, hands over a server that refuses new requests.
- Only the start path checks that the state answers its token; the live path does not.
- Evidence: D5 bullets 2–3, D11 bullets 2 and 5.

**F3 — MEDIUM, design, D11 bound "keeps serving the hub until it exits" (with D5 and D6)**
- The server builds its board once, from the starting worktree's layout. The closures that config builds keep reading that worktree: `ValidateSlug` reloads the fabric config through `ConfigDirs(layout)` on every create, and `ParentOf` and `PairExists` also use that layout (`internal/boardcli/cli.go:162-170`).
- Counterexample: a multi-root window runs the first matching `board open`, which can be a task pair's folder (D6), and `ResolveWithin` makes that pair the server's anchor (D5). The pair lands and is torn down. Every page create now fails loading the slug's fabric config.
- An open page polls at least once a minute, so the idle exit never comes and the broken server lives indefinitely.
- D6's claim that every copy gives the same result is false for the same reason.

**F4 — MEDIUM, design, D3 bound (mixed versions)**
- The bound covers a downgrade's removal commit, but not an older binary replaying a newer machine's commits.
- The older binary does not claim `board-data.js`. Since the projection carries the render time, every sync commit from an upgraded machine touches `board-data.js`.
- By D3's own rule ("a commit touching an unclaimed path is replayed as a plain cherry-pick"), every such commit replays on the older machine as a plain cherry-pick and loses the by-entry merge of `board.json`.
- Any concurrent local write then conflicts, and that machine's board writes are skipped as a diverged Bolt until it upgrades. The bound does not state this.
- Evidence: D3 bullet 3 and bound, D1 top-level render time, `internal/boardengine/replay.go:50`.

**F5 — LOW, design, D8 create refusal**
- "Refused when the slug already exists, with the existing entry's projection" has no entry to carry when the slug is taken by something other than a stored entry.
- `checkNewSlugs` also judges the raddle store (`internal/boardengine/board.go:120,172`), so a slug colliding with a raddle store is refused with nothing to project.
- The page's stale-or-exists display path is unspecified for that refusal.

**F6 — LOW, design, D11 takeover claim**
- "A hub takeover … ends it … through runproc" holds only if the server's role is outside `subjectPassSpared`. That list spares the sync, push and Slack relay as hub-wide roles (`internal/runproc/end.go:96-100`).
- PATTERN-child-processes needs a role the package names; D11 and D13 name neither the role nor the change to `runproc`.

**F7 — MEDIUM, design, D10 release bound**
- "Release acts on any task … a running task included" states what is let through but not what follows.
- Release sets the status to nil, and `applyDraftRule` treats nil as the release, which leaves the task startable in the write that makes it (`internal/boardengine/draft.go:42-47`, `board.go:135-139`).
- One page click on a running task's Release clears the run's state and hands it to the parent's dobby as startable while the original run still holds its pair.
- The record names no guard that bounds this; I did not check the dobby's start guard.

**Tool calls**
1. `boardengine/spawn.go`, `store.Save` and `skipenv.go`: the sync child's working directory and registration, and the skip variables, for the D11 bounds. These were not in context.
2. Every board writer that takes the write lock, plus `migrateDraftStatus` and `applyDraftRule`: for D8's atomicity claim and D10's release and draft semantics.
3. The lock use of the writers outside the critical section (replay, hold, registry, owner-set, `SetStatusPushed`): for D8's "every other writer". Partial result.
4. The lock calls in replay and `bolt.go`: to confirm which writers hold the board write lock.
5. `fabricengine/bolt.go`'s `acquireWriteLock` and `Commit`'s staging: to confirm the replay lock and D3/D11's staging claim.
