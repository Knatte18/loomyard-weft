```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision traced end to end with one concrete case, checking the end state against the decision's own purpose.

**Findings**

1. **MEDIUM — D8 and D1: the two sides of the revision compare may hash different bytes.**
   - D8 hashes "the entry's JSON encoding as `board.json` stores it" and compares it inside `boardCriticalSection`.
   - That section runs `migrateDraftStatus` and `migrateDiscussedStatus` on the loaded store before `fn`. The record says the compare runs "before the mutation", which means after these migrations.
   - The page's revision comes from the projection. That is either `board-data.js`, rendered after save, or the server's own `GET /api/board` build from `board.json`. The record does not say whether that build loads through the same migrations.
   - Case: an older binary writes `board.json` with a pre-migration entry. The server reads it raw and hands the page revision R1. The edit's compare hashes the migrated in-memory entry and gets R2 ≠ R1. Every edit of that entry is then refused as stale until some other write saves the migrated form.
   - Store-level `unknown` keys show that re-encoding is not byte-identical to the file.
   - Purpose missed: "a stale one is refused" becomes "a fresh one is refused" whenever the two encodings differ.
   - Fix: state that one `boardengine` function computes the revision from the entry as loaded and migrated, and that both the projection and the check use it.
   - Evidence: `internal/boardengine/board.go`, the `boardCriticalSection` order (Load, the migrations, `fn`). `internal/boardengine/store.go`, `Store.Save` carrying `Unknown`.

2. **LOW — D10's promote can start a run from the page, and its bound does not say so.**
   - Case: the operator promotes a discussed Sketch note in the page.
   - `PromoteTo` with an empty recipe gives it the raddle recipe and releases the task in the same write. Its parent's dobby then starts a raddle run.
   - This matches the CLI. D10's bound says only that "release and draft act on any task…".
   - Fix: name in D10's bound that promote, like the CLI's, can release a Sketch into a raddle run.
   - Evidence: `PromoteTo`'s doc comment and the `promoted` path in `boardCriticalSection`, which adds the slug to `released` for `RecipeRaddle`.

3. **LOW — D1's criterion that `GET /api/board` returns "the same projection the render writes" cannot hold byte for byte.**
   - The projection carries the render time, and D11 reads the server's `boardengine` config only once, at start.
   - After a `board.yaml` label change, the render's projection and the server's split labels differently until the server restarts.
   - Fix: phrase the criterion as "the same projection function over the same store and config, render time aside".
   - Evidence: D1's top-level fields and D11's bound.

4. **LOW — D10 and D11: the page cannot learn of a change that leaves `board.json` alone.**
   - A `lym board upsert --record-file` that rewrites only `decision-<slug>.md` for an already-discussed entry does not change `board.json`'s bytes. The long poll then never wakes, and a page showing that file stays stale until reload.
   - This is acceptable, since the board entries are what refresh live, but no bound states it.
   - Evidence: `decision.go` keeps the record beside `board.json` as a separate file. D11 watches `board.json` alone.

**Decisions walked**

- **D1** (board with a sub-project and a `</script>` body; projection to `board-data.js` and to the API): met, apart from findings 1 and 3.
- **D2** (`board.html` loaded at `file://` reads `board-data.js`; the same bytes served at `/` switch to API mode): met.
- **D3** (a write renders both files, the sync commits them, a replay re-renders): met. The server ignores the files, so a stale file never reaches it.
- **D4** (a body link of `javascript:alert(1)`, a title holding `<b>`, a refusal message holding a tag): met. All are rendered as text, and the file link takes the token-bearing route over `http`.
- **D5** (open from a subdirectory with no server):
  1. `ResolveWithin` finds the anchored dir.
  2. The stale state file's probe fails.
  3. `open` spawns `serve` with the anchored dir as cwd, and `serve` runs `Resolve` and gets the lock.
  4. `serve` writes its state, and `open`'s bounded wait then probes it successfully.
  5. `open` writes the launcher and runs the opener on its path.
  6. The launcher redirects to `http://127.0.0.1:port/#t=…`. The Host check passes and the page reads the token.

  Met.
- **D6** (seed the prime, then regenerate the workspace): unchanged from the Sketch. Met.
- **D8** (two tabs edit T5; tab B saves after tab A):
  1. Tab A's save changes T5, so its hash moves.
  2. Tab B's revision no longer matches, so its write is refused with the fresh entry, and nothing is saved.

  A status-only race is caught, because the hash covers status. Met, apart from finding 1.
- **D9** (a hostile web page POSTs to `127.0.0.1:port`): it lacks the token, and its content type and Origin fail the checks. A DNS-rebound host fails the Host check. Met.
- **D10** (release a running task):
  1. `SetStatusReleasingProject(nil)` runs with the revision checked.
  2. Its status becomes none, and a project release follows when this was the project's last member.
  3. The response carries the entry, and the long poll returns at once because `board.json` changed, so the page also sees the dependents' new layers.

  Met. Finding 2 covers promote.
- **D11** (an agent's `lym board upsert` while a page long-polls):
  1. `state.WriteJSON` replaces `board.json`.
  2. fswatch delivers an event for the name `board.json`.
  3. The server recomputes the board's revision and wakes the waiter, and the page refetches.

  With fswatch unavailable, the timer catches the change within 8 s. A second `serve` loses the lock and exits 0. Met.
- **D12** (the orch runs `open`; a run's agent runs `open`): the orch proceeds, and the server's writes carry an empty strand name. The agent is refused. Met.
- **D13** (where the server and the revision check live): this is placement, with no mechanism to run. Consistent with D8 to D11.

**Tool calls**

- Read `internal/boardengine/board.go`, `boardCriticalSection` and `PromoteTo`: the context had only a summary of the order of steps and of promote's raddle release, and finding 1 and finding 2 needed the code itself.
- Read `internal/boardengine/store.go` `Save`, grepped `internal/boardengine/decision.go` and read part of `internal/fswatch/fswatch.go`: to confirm the atomic `board.json` write that fswatch sees, where the decision files live for the file route, and whether fswatch filters events by name.
