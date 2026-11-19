```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already write, read or end what the record's decisions add or change. I found no BLOCKING finding.

**Findings**

1. **MEDIUM, D8 and D10: the server's read path does not run the load migrations, so revisions can disagree.**
   - Evidence: `migrateDraftStatus` and `migrateDiscussedStatus` run only inside `boardCriticalSection` (`internal/boardengine/board.go:116-117`), after the store loads. The record says D8 compares "the entry as loaded under the board's write lock". `GET /api/board` builds its projection by loading the store outside the write section, and the record never says it migrates.
   - Effect: on a board with a pending migration, the revision the page holds is computed from the unmigrated entry, while the write compares against the migrated one. Every edit of such an entry would be refused as stale, until some other write saves the migration.
   - Fix: D8 should state that the revision is computed from the store after the same load migrations, on both the read path and the write path. Or the server should build its projection only from what the write section saves.

2. **LOW, D8: the claim that the check is atomic "against every other writer" holds.**
   - Evidence: `SetStatusPushed` (`board.go:224`) bypasses `boardCriticalSection`, but it writes through `Bolt.PullThenCommitWritten`, which holds the same board write lock (`internal/fabricengine/bolt.go:209`, `acquireWriteLock` → `BoardWriteLockFile`). The sync's `commitDirty` also takes that lock.
   - So the landing status write and the sync both serialise with the server's write. That path survives, but it skips the migrations and the D8 check; the record does not need to say so.

3. **LOW, D3 and D1: the per-entry revision adds commit churn to `board-data.js`.**
   - Evidence: the revision hashes every stored field, including fields the page does not show, such as `run_settings` and profile. A run's `run_settings` write, or any scope-neutral field write, now also changes `board-data.js` in that write's sync commit.
   - That survives correctness-wise, but it is churn the record's D3 bound does not mention.

4. **LOW, D11: a hub takeover kills the server, which the record does not mention.**
   - Evidence: `pairteardown.EndSubject` (`internal/pairteardown/endsubject.go`) ends `runproc.HubOwner` processes when another machine takes over the root subject. A server registered under `HubOwner` is ended with them.
   - `tenderengine/processes.go:119` treats `HubOwner` as a known owner, so tender's reap leaves the server alone.
   - The page then shows the "server gone" line and `lym board open` restarts it, so this survives. The record could name takeover beside the idle and binary-replacement exits.

5. **LOW, D11: the watch loop survives the paths that rewrite `board.json`.**
   - Evidence: `Store.Save` writes through `state.WriteJSON` (temp and rename), and `fswatch.Watch` delivers create and rename events for watched names in the directory.
   - A local write, `SetStatusPushed`, a Bolt replay and a sync pull all rewrite `board.json` in place in the Bolt directory, so each wakes the long poll.
   - A Bolt directory removed and re-created would orphan the inotify watch silently. The always-running timer, which goes up to 8 s, still catches the change, so this survives at reduced latency.

6. **LOW, D9 and D11: the state file holding the token is safe from the sync only if the board-level `.lym` stays git-ignored.**
   - Evidence: the board sync "stages the whole Bolt tree" (D3). creel already keeps its journal and lock under the same board-level `.lym`, so it is ignored today and the state file survives.
   - The record relies on that silently. One line saying the token file must never be staged would pin the dependency, since a token pushed to a remote is a leak.

7. **LOW, D5: `xdg-open` on a `.html` launcher reaches whatever handles `text/html`, which is not always a browser.**
   - Evidence: the Sketch's D5 opened a file too, so this is no regression. The launcher's redirect works only if that handler runs JavaScript.
   - The record's bound covers `xdg-open` with no desktop, but not a non-browser handler.

**Paths traced, per decision**

- **D1:**
  - The render inside `boardCriticalSection` after `Save` survives.
  - The `followDecisionRecords` file rename before `RenderToDisk` survives.
  - The `SetStatusPushed` → `renderOwnerWrite` render survives.
  - The server's `GET` projection survives, except for finding 1.
- **D2:** the embedded page served by both the render and the server survives.
- **D3:**
  - The sync child's `commitDirty` under the write lock survives.
  - `SetStatusPushed` committing the rendered names survives.
  - The replay re-render survives.
  - Churn from the per-entry revision, finding 3.
- **D5:** the opener on a launcher is finding 7. `ResolveWithin` for `open` against `Resolve` for `serve` survives, since the server's working directory is the anchored directory.
- **D6:** unchanged from the Sketch, so not re-traced.
- **D8:**
  - The write-section load and migrate order breaks against the read path, finding 1.
  - `SetStatusPushed` and the sync lock survive (finding 2).
  - Sibling writes are covered by the record's bound and survive.
- **D9:** state file under the board-level `.lym` against the sync's whole-tree staging, finding 6.
- **D10:** writes through `UpsertTaskReporting`, `SetStatusReleasingProject` and `PromoteTo` all run inside `boardCriticalSection`, so the run locks and the task-creation guard apply. Survives.
- **D11:**
  - `fswatch` against `state.WriteJSON`'s rename survives.
  - Replay and pull rewriting `board.json` survive.
  - `EndSubject` with `HubOwner` is finding 4.
  - Tender's reap leaves the server alone, which survives.
- **D12:** the server inherits the spawner's environment but passes the board an empty strand name, as stated. Survives.

**Tool calls**

1. Grepped the load-migration callers, `Store.Save` and the `HubOwner` end paths. My context lacked where the migrations and the takeover ends run.
2. Read `Store.Save`, `fswatch.Watch` and `EndSubject`. My context lacked how the store is written and what the watch delivers.
3. Grepped `SetStatusPushed` and the lock sites. My context lacked whether that write takes the board lock.
4. Read the `SetStatusPushed` body and the sync's `commitDirty`. Same reason.
5. Grepped the lock in `Bolt.PullThenCommitWritten`. My context lacked which lock the commit-written path holds.
6. Read the save-to-render tail of the write section and `followDecisionRecords`. My context lacked whether anything mutates entries between `Save` and the render.
