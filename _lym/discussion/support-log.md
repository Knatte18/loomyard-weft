# Support log — board-viewer-build

## Interview

This run was autonomous: no operator answered, and every pick below is the writer's own, recorded in the question ledger.

### Turn 1 — the task and its starting point

The board entry is a discussed Sketch task.
Its stored decision record (D1 to D6 and five Added after Discussion entries) designs a read-only viewer:
`board-data.js` and `board.html` as render outputs, `lym board open` opening the file, and VS Code tasks and keys.
The entry body then gained "Wanted next (operator, 2026-11-19)": the viewer edits the board, refreshes live,
writes through `boardengine`, and carries the edited entry's revision so a stale write is refused and shows the fresh entry.
No `Decision for board-viewer-build:` note is on the board.
No viewer code exists yet; `boardView` and `layerSection` are the README renderer's own types.

### Turn 2 — facts from the code

- Every board write runs through one critical section under the Bolt write lock: load, mutate, guards, save, render, then a detached `board sync`.
- No per-entry revision exists. `Task.Revised` is whole seconds and is not bumped by a status change, so it cannot detect a stale status write.
- The run-agent task-creation guard keys on the strand-name environment variable; an empty strand name is the operator.
- Draft is the status value `draft`; release and undraft set the status to none.
- No production HTTP server exists. creel's daemon is the on-demand, single-instance, detached pattern: lock in the hub's board-level `.lym`, `proc.Detach`, `proc.Reap`, exit on binary replacement.
- `internal/fswatch` is the one file-event source; the PATTERN-daemon-wakeups guard lists every unbounded wait.
- A page at `file://` cannot `fetch` and cannot write; editing needs a process that accepts writes.

### Turn 3 — design picks

- Editing and live refresh need a server, so the design adds a per-hub loopback server started on demand by `lym board open` (D9 to D11).
- The revision is the hash of the stored entry, checked under the write lock (D8).
- The server serves the embedded page and computes the projection itself, so the committed files stay a `file://` read-only view (D2, D3).
- `open` and `serve` refuse in a run agent's session, so the server cannot become a way around the task-creation guard (D12).
- The Sketch's D1 to D6 are kept, with D1 gaining the revision and the area-label list, D2 the two page modes, D4 the `http` links, and D5 rewritten around the server.

### Turn 4 — lens review

Six forks returned reports, saved beside this log as `support-log.lens-<lens>.md`; none was missing.
Findings, merged across lenses:

- interactions 1, walkthrough 1, existing-paths 1, bound-breaker 1 (revision hashed before or after the load migrations): fixed; D8 hashes after the migrations on both the projection and the check, with a criterion.
- interactions 2 (D10 draft contradicts its bound): fixed; draft and undraft act only from none and `draft`, the bound names release alone, with a criterion.
- interactions 3 (exit during a write): fixed; D11's exits drain in-flight requests.
- interactions 4, walkthrough 4 (a decision-file-only change does not wake the page): fixed as a stated bound in D10.
- interactions 5 (`serve` under `--board-path`): fixed; `serve` refuses it, with a criterion.
- interactions 6, bound-breaker 3 (stale state hands the token to a foreign listener): fixed; liveness is read by the server's lock, which proves the recorded port, and `serve` waits briefly for the lock so the read never turns it away.
- interactions 7 (`localhost` against the Host check): fixed; D9 pins `127.0.0.1` for every address lym builds.
- interactions 8 (file modes held by accident): fixed; D9 states mode 0600 for the launcher, and the criterion tests both files.
- interactions criteria note (date stamps): fixed; the D10 criterion sets the stamps aside.
- walkthrough 2 (promote releases a Sketch into a raddle run): fixed as a stated bound in D10.
- walkthrough 3 (projection identity criterion): fixed; the D1 criterion compares the same function over the same store and config, render time aside.
- existing-paths 2 (`SetStatusPushed` serialises on the same lock): no change; it confirms D8's atomicity claim.
- existing-paths 3 (`board-data.js` churn from the revision): fixed as a line in D3's bound.
- existing-paths 4 (takeover ends the server): fixed; D11 names it.
- existing-paths 5 (watch survives rewrites): no change; it confirms D11.
- existing-paths 6 (token file and the sync's staging): fixed; D11 states the `.lym` is never staged.
- existing-paths 7, bound-breaker 5 (non-browser handler, confined browser): fixed as a bound in D5, and the envelope prints the launcher path.
- bound-breaker 2 (an agent can use the token of a running server): fixed; D9's and D12's bounds state it, and D12's rationale no longer claims the refusal closes that path.
- bound-breaker 4 (skip variables inherited): fixed; `serve` applies them as the CLI does, and D11's bound states the effect.
- bound-breaker 6 (`'unsafe-inline'` CSP): fixed; D9 uses hash sources, with a criterion.
- bound-breaker 7 (transient failure read as server gone): fixed; the page retries before the line, in D11 and the D4 review criterion.
- board-scope 1 and 2 (dropped refusal, kept render outputs): no change; both are `decision` class and already stand in Open risks for the parent.
- leanness 1 (rationale arguing against alternatives): fixed for D8, D9 and D12.
- leanness 2 (D13 repeats D6): fixed; bullet cut.
- leanness 3 (D3's server sentence): fixed; moved into D2.
- leanness 4 and 5 (Notes repeat the record): fixed; D7 note shortened, creel note cut, `selfexec.New` moved into D11.
- leanness 6 (auto-mode lines restate decisions): fixed; the lines point to the ledger.
- leanness 7 (Open risks names its resolution): fixed; the clause is cut.

## Rejected alternatives

- **A `file://` page that polls `board-data.js` and writes through a VS Code task or a custom URL scheme.**
  A `file://` page can re-insert a script tag to refresh, but it has no way to send a write; a URL-scheme handler needs per-platform registration and still needs a process to run the write.
- **Drop `board-data.js` and `board.html` as render outputs.**
  The server makes them unnecessary for `lym board open`, and they add churn to every sync commit, but the entry's Delivers line names them; left to the parent in Open risks.
- **The server serves the Bolt's `board.html` and `board-data.js` files.**
  A stale file from an older binary would then talk to a newer server's API; the embedded asset and a live projection cannot drift from the server.
- **`Task.Revised` as the revision.**
  It holds whole seconds and does not change on a status write, so two writes in one second and every status change would pass the check.
- **A stored revision counter in `board.json`.**
  It needs a schema change, a migration of every entry and every writer to bump it; a derived hash needs none.
- **The server in the foreground of the VS Code task terminal.**
  It dies with the window, blocks a terminal, and a second press meets VS Code's running-task prompt; the on-demand daemon reuses one server.
- **Server-sent events or a websocket for live refresh.**
  `EventSource` cannot send the token header, and a websocket adds a protocol the page and server must both carry; a bounded long poll uses plain `fetch`.
- **The token in the opened URL.**
  The opener's command line is readable by other users on the machine; the launcher file keeps the token in a file only the user can read.
- **Shell out from the server to `lym board upsert` and friends.**
  It would keep one code path, but the revision check must sit inside the write lock, which only an in-process `boardengine` call can do.
- **Let the server set or clear the strand name itself.**
  Clearing it for a server an agent started would let the agent create tasks through the page; refusing the start is simpler and closes that path.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): how does a page edit the board? Answer: a per-hub loopback server, `lym board serve`, started on demand by `lym board open`. Rationale: a `file://` page cannot write; see Rejected alternatives.
- Q2 (auto-pick): what is an entry's revision? Answer: the hex SHA-256 of the entry's stored JSON, derived on read and checked under the write lock. Rationale: covers status changes and needs no schema change.
- Q3 (auto-pick): how does the page learn of other writers? Answer: a bounded long poll on `GET /api/board`, woken by an `fswatch` event on `board.json` or the fallback timer. Rationale: plain `fetch` with the token header, no extra protocol.
- Q4 (auto-pick): how is the server protected? Answer: loopback bind, per-start token passed through a user-only launcher file, Host check, no CORS, JSON content type and Origin check on writes. Rationale: the standard local-tool defences, and the token stays off process listings.
- Q5 (auto-pick): which fields does the page edit? Answer: title, labels, brief, body, project and a task's dependencies; slug, kind, run settings, profile and records stay CLI-only. Rationale: the entry asks for create and edit; the rest have their own verbs with their own rules.
- Q6 (auto-pick): how long does the server live? Answer: until 30 minutes pass with no `/api` request, or its binary is replaced. Rationale: an open page keeps it alive; no stop verb is needed.
- Q7 (auto-pick): may an agent open the viewer? Answer: no; `open` and `serve` refuse in a run agent's session, and the orch may run them. Rationale: closes a way around the task-creation guard.
- Q8 (auto-pick): keep `board-data.js` and `board.html` as committed render outputs? Answer: kept, as the entry's Delivers line names them; raised to the parent in Open risks. Rationale: dropping a named deliverable is the parent's call.
- Q9 (auto-pick): keep D5's refusal naming `lym board rerender`? Answer: dropped, since the server serves the embedded page; raised to the parent in Open risks. Rationale: the refusal would guard a file the verb no longer reads.
- Q10 (auto-pick): where does the server code live? Answer: a new `internal/boardserve`, with the verbs in `internal/boardcli` and the projection, page and revision in `internal/boardengine`. Rationale: keeps transport and lifecycle out of the engine and the CLI thin.
