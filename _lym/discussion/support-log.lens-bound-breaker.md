```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked the record's claims, bounds and guarantees with concrete counterexamples.

**Findings**

1. **MEDIUM, D8 and D10 (and the D1/D10 acceptance criterion "GET /api/board returns the same projection the render writes"): the revision's hash basis is not named against the load migrations.**
   - `boardCriticalSection` runs `migrateDraftStatus` and `migrateDiscussedStatus` on the loaded store before `fn` (board.go lines 116-117).
   - `migrateDraftStatus` is one-shot through a store flag: it turns an empty-status open task into `draft` in memory until a write saves the result.
   - Counterexample: a board.json not yet migrated holds a task with status null. Suppose the server builds its GET projection from a plain `Store.Load`. The page then holds a revision hashed over status null, while the in-section check hashes status `draft`, so the first edit is refused as stale.
   - In the same case the GET projection differs from the render's, which runs after save on the migrated store.
   - Fix: state that the projection and the check both hash the entry after the section's load migrations, and that GET builds its store the same way.

2. **MEDIUM, D12 rationale and D9 bound: the claim that the refusal closes the agent's way around task creation does not hold.**
   - D12 says that without the refusal a run's agent "could start the server and create tasks through it".
   - Counterexample: the operator has already started the server. A run's agent, running as the same user, reads the state file (token, port) under the hub's `.lym` and POSTs a create. The server writes with an empty strand name, so `checkTaskCreation` passes.
   - D9's "exactly as it can run `lym board upsert` today" understates this. Today the agent must deliberately clear `LYM_STRAND_NAME`. Here it only needs to read a file.
   - Fix: state the bound honestly (the refusal only stops an agent from starting a server), or have the write route take the strand identity from somewhere other than possession of the token.

3. **MEDIUM, D5 and D11 liveness probe: a stale state file can hand the token to a foreign listener.**
   - D5 "asks the recorded server whether it is alive" without saying how liveness is proven.
   - Counterexample: the server dies and its state file stays. Another local user's process binds the freed port and answers any probe with 200. `open` judges it alive and hands the launcher to the browser. The foreign page's script reads `location.hash` and takes the token, which D9 says keeps other users out.
   - Fix: prove liveness by the server's lock. If `open` can take the lock with `TryAcquire`, no server runs and the state is stale. Or require an answer only the token holder can produce, not just 200.

4. **LOW, D11 and D12: the server inherits its starter's environment, skip flags included.**
   - `skipenv.go` reads `BOARD_SKIP_GIT` and `BOARD_SKIP_PUSH`, and the other writing verbs apply them.
   - A server started from a shell with either set leaves every page write uncommitted or unpushed, for every later opener of the hub, until it exits idle. The record is silent on this.
   - Fix: `serve` ignores or refuses those variables, or the bound says so.

5. **LOW, D9 launcher file: a sandboxed browser may not be able to read it.**
   - Snap-packaged browsers, Ubuntu's default Firefox and Chromium, read only non-hidden paths under `$HOME`.
   - With the hub outside `$HOME`, `xdg-open` succeeds, the browser cannot read the launcher, and lym reports success. No bound is stated.

6. **LOW, D9 CSP: allowing "its own inline script" means `'unsafe-inline'`, which gives no XSS defence.**
   - The asset's bytes are fixed (D2), so a hash source would allow exactly that script and nothing injected.

7. **LOW, D11: a transient fetch failure is taken as "server gone".**
   - "A page whose server is gone shows one line naming `lym board open`." A failure after a laptop resume shows that line while the server is alive.
   - The page should retry before declaring the server gone.

**Claims attacked**
- D8 "compare and write are one atomic step against every other writer": held. Every board.json writer takes `board.lock`: `boardCriticalSection`, `sync.go` (replay) and `fabricengine/bolt.go` for `SetStatusPushed` and the owner-set writes.
- D8 "stale refusal saves, renders and syncs nothing": held, since an error from `fn` returns before Save.
- D8 hash basis: broken (finding 1).
- D8 bound "a sibling write such as a project release is unchecked": held, and stated.
- D9 Host check against DNS rebinding: held.
- D9 token keeps out other users: broken through the stale-port path (finding 3).
- D9 "same as `lym board upsert` today": broken (finding 2).
- D9 no CORS plus JSON content type plus Origin check against cross-site writes: held.
- D9 launcher keeps the token off argv: held. Its readability is not guaranteed (finding 5).
- D9 CSP: weak (finding 6).
- D10 long poll bounded under a minute, outside the wakeups rule: held.
- D10 server writes spawn the sync child safely from a long-lived process: held. `spawn.go` uses Detach, Start, `proc.Reap` and a runproc `ChildRecord`.
- D11 single instance by lock, with a lost race exiting 0: held. A stale state read during the winner's start only delays the bounded wait.
- D11 replay and pull rewrites are seen as changes: held, since replay saves board.json under the lock in the watched directory.
- D11 idle exit kept off by an open page: held, with the tab-discard and suspend cases stated. Transient failure is finding 7.
- D11 config read once: held, and stated.
- D12 refusal closes the agent path: broken (finding 2).
- D12 server env: gap (finding 4).
- D3 "the server never reads these files": held.
- D5 `--board-path` refusal: held. The pre-run returns before resolution, so the verb itself must check.

**Tool calls**
1. Grepped `internal/boardengine/spawn.go` and `skipenv.go`. My context lacked whether the sync child is reaped when a long-lived process spawns it, and which environment variables the writing verbs inherit.
2. Grepped the lock sites and read the head of `boardCriticalSection`. My context lacked whether replay and sync share the write lock, and where the migrations run.
3. Grepped the migration functions and read `migrateDraftStatus`. My context lacked whether the migration runs only inside the section and is one-shot.
4. Grepped `SetStatusPushed`, owner-set and replay Save sites. My context lacked whether those writes bypass the lock.
5. Read `SetStatusPushed`'s body. My context lacked how its write is wrapped.
6. Grepped `BoardWriteLockFile` users. My context lacked confirmation that fabric's Bolt write path takes `board.lock`.
