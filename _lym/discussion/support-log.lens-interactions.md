```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of the record's decisions that act on the same state, file, key or ordering.

**Pairs checked**
- D1 entry revision × D8 revision function: **broken** (finding 1).
- D1 projection × D10 `GET /api/board` (one builder, file and server): held.
- D1 projection inputs × D10 board revision as the hash of `board.json` bytes: held for entries and the prime owner, since `primeOwner` is stored in `board.json` (`internal/boardengine/board.go`, which passes `store.primeOwner`). **Broken** for decision-record existence (finding 4).
- D2 two page modes × D4 links over `file://` and over `http`: held.
- D2 "loads nothing beyond its own origin" × D9 launcher redirect: held. The launcher is a separate file, not the page.
- D3 "the server never reads these files" × D10 file route reading Bolt files: held. D3 names only `board.html` and `board-data.js`.
- D3 mixed versions × D8 revisions: held. An older binary's `board-data.js` has no revisions, but the `file://` page is read-only.
- D5 `ResolveWithin` × D11 `serve` resolving strictly from the opener's anchored directory: held, as long as `open` passes the anchored directory and not its own cwd.
- D5 `--board-path` refusal × D11/D12 `serve`: **gap** (finding 5).
- D5 alive probe × D11 stale state after a crash: **gap** (finding 6).
- D6 `board open` task (reveal, no focus) × D5 now returning quickly: held.
- D8 check before the mutation × the section's load-time migrations and `applyDraftRule`: held for writes. A refusal returns before `Save`, so nothing is rendered or synced.
- D8 create of an existing slug × D10 create: held.
- D8 sibling bound (project release) × D10 release: held.
- D9 Host `127.0.0.1:<port>` × D5 printed address and the launcher's address: held, as long as the launcher never uses `localhost`. Not stated (finding 7).
- D9 token in the `t` query × D4 links set only as an `href` through the DOM: held.
- D9 "file only the user can read" × Constraint `PATTERN-tolerant-records` (write through `internal/state`): held by accident (finding 8).
- D10 draft and undraft × the release bound: **contradiction** (finding 2).
- D10 undraft (status none) × the board's `applyDraftRule`: held. A nil status is never re-drafted.
- D10 long poll under a minute × D11 idle exit at 30 minutes: held.
- D10 waiting requests × D11 exit on binary replacement: **gap** (finding 3).
- D11 watch on `board.json` × `Store.Save`'s temp-file-and-rename write: held. The rename lands on the watched name.
- D11 board built once × D8 check per write: held. The section reloads the store on every write.
- D11 spawn environment × D12 strand rule: held. `open` refuses in an agent session, an orch's strand is allowed, and the server passes an empty strand.
- D12 × D6 VS Code task terminal: held. No strand name is set there.

**Findings**

1. **MEDIUM** — D8 × D1/D10: a revision taken from an entry before migration never matches the check.
   - D8 compares against "the entry as loaded under the board's write lock, before the mutation runs". The write section runs `migrateDraftStatus` and `migrateDiscussedStatus` on the loaded store before the mutation.
   - The server builds the projection itself (D3, D10) from `board.json` read outside the section. If that read skips the same migrations, an entry the migrations would change gets a revision that never matches. Every edit of it is then refused as stale, until some other write saves the migrated store.
   - The record should say that the revision is computed over the store as the section loads it, migrations included, in both the projection and the check.

2. **MEDIUM** — D10 contradicts itself on draft.
   - The operations list says "draft a task whose status is none" and "undraft a task whose status is `draft`".
   - The bound says "Release and draft act on any task the CLI's `set-status` acts on, a running task included".
   - Under the first reading, draft never touches a running task. Under the second, it can overwrite a run status with `draft`, which would drop the run's status the way release does but leave the task drafted.
   - Pick one. The operations list is the safe one, so the bound should name release alone.

3. **LOW** — D11 binary-replacement exit × D10 writes.
   - The server can exit while `POST /api/write` is inside the board's critical section, or between saving and spawning the sync.
   - The record does not say the exit waits for in-flight requests.
   - creel's exit does not need this: it holds no board write lock.

4. **LOW** — D10 board revision × D1 projection.
   - The projection also depends on files outside `board.json`. `withMissingRecordsUndiscussed` reads whether each decision-record file exists, and the design and decision names come from the Bolt.
   - A replay that adds only a decision file changes the projection without changing the board revision, so a live page misses it until the next `board.json` change.

5. **LOW** — D5 × D11/D12: `serve` under `--board-path`.
   - D5 refuses `open --board-path`, but nothing says `serve` refuses it.
   - Under that flag, pre-run builds a `Board` with no outputs, and such a board refuses writes. `serve` would come up, then fail every write.

6. **LOW** — D5 alive probe × D11 stale state.
   - After a crash, the state file names a port that another process may now hold.
   - D5's probe must check the server's token, not only that something answers, before it reuses the recorded server.
   - The record says only "asks the recorded server whether it is alive".

7. **LOW** — D9 × D5 launcher address.
   - The Host check accepts only `127.0.0.1:<port>`, but the record never pins the launcher's address to that form.
   - An implementer who writes `localhost` makes every request fail the Host check.

8. **LOW** — D9 file mode × Constraint `PATTERN-tolerant-records`.
   - The "readable by the user alone" claim holds only because `internal/state.WriteJSON` goes through `fsx.AtomicWriteBytes`. That function writes with `os.CreateTemp`, which creates files with mode 0600, and the rename keeps that mode. Nothing pins this.
   - The launcher is not a JSON record, so its 0600 mode has no stated mechanism at all.
   - D9's acceptance criterion does test both files, which covers the risk.

**Acceptance criteria against decisions**
- The D10 criterion that each operation "leaves `board.json` as the matching CLI verb leaves it" has to allow for the `created` and `revised` stamps.
- The D8 criterion holds with D10.
- The D11 timer criterion holds with the fallback the record states.
- The other criteria agree with the decisions they name.

**Tool calls**
- `grep` over `internal/state`, `Store.Save` and render's prime owner: the context lacked the state file's mode, how `board.json` is written, and where `primeOwner` comes from.
- `sed` of `internal/state/state.go` and a `grep` of `board.go` and `render.go`: the context lacked the `WriteJSON` path and the `withMissingRecordsUndiscussed` input.
- `grep` of `fsx.AtomicWriteBytes`: the context lacked the file mode of the atomic write.
- `sed` of `internal/fsx/fsx.go`: confirmed that `os.CreateTemp` (mode 0600) and a rename set the mode.
