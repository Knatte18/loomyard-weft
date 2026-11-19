```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked every pair of the record's decisions that share state, then each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D8 (revision in the projection, hashed after load migrations): held.
- D1 × D3 (`board-data.js` carries the render time; D3 says a change to an unshown field changes it): held. The render time already changes the file on every write, so D3's line is true but adds nothing.
- D1 × D11 (config read once against the projection's label lists): held; D11's bound states it, and the D1 criterion fixes "same config".
- D2 × D9 (the page served over `http` keeps D2's `&lt;script src="board-data.js"&gt;`, under a hash-only CSP, with no route serving that file): broken in a minor way. See F3.
- D2 × D4 (the `file://` read-only mode and the missing-data line): held.
- D3 × D11 (skip variables read once against "every write syncs"): held; D11's bound states it.
- D3 × D11 (a takeover ending the server through `runproc`, against the render-and-sync promise of D3): held as a bound. A SIGTERM falls outside D11's "on either exit … waits for those in flight", but a killed CLI write has the same exposure, and the next write syncs.
- D4 × D9/D10 (links over `http` use the file route, which needs the `t` token): held. The page sets the `href` through the DOM.
- D5 × D11 (liveness read by the lock, against the order `serve` starts in): broken. See F1.
- D5 × D11 (a held lock read as live, against the drain on exit): broken. See F2.
- D5 × D11 (`open` resolves its cwd with `ResolveWithin`; `serve` uses `Resolve` from "the opener's anchored directory"): held. `AnchorPath()` is exactly the directory `Resolve` gates on.
- D5 × D12 × D11 (`open` spawns `serve`, which inherits the strand name; the run-agent refusal; the empty strand name on writes): held.
- D5 × D9 (`open`'s probe against the Host check): held, given D9 pins `127.0.0.1` for every address lym builds.
- D6 × D5 (the `board open` task's cwd is the workspace folder, the anchor): held.
- D8 × D10 (release, promote, draft and undraft carry a revision that covers status; a running task's status moves with each step): held. A stale release is refused with the fresh entry, as designed.
- D8 × D10 (promote changes kind and handle, and the check is by slug): held.
- D8 bound × D10 (a project's release of a sibling is unchecked): held.
- D8 × D10 response (the fresh projection's revision after a successful write, against the revision the next check computes): not stated. See F4.
- D10 long poll × D11 idle exit (a re-ask under a minute counts as an `/api` request): held.
- D10 long poll × D11 drain (in-flight long polls hold the exit up to the poll bound): feeds F2.
- D10 create × D8 create (refused on an existing slug, where CLI upsert would update): held. The D10 criterion is read over new slugs.
- D10 release × D10 undraft (the same write on a draft task): held, redundant but consistent.
- D12 × D9 (the token in the state file against the refusal): held. Both bounds state it.
- D13 × D8/D11 (new exported `boardengine` methods carrying a revision; state and watch loop in `boardserve`): held.
- Criteria × D9 (the file route's `t` token): broken. See F5.
- Criteria × D6 (the two programmer-error refusals): broken. See F6.
- Criteria × D10 (the page's forms and the stale display): broken. See F7.
- Criteria × D5 (the envelope prints the address without the token): uncovered. NIT, folded into F5.
- Constraint `PATTERN-daemon-wakeups` × D11 and D10: held.
- Constraint `PATTERN-child-processes` × D5's foreground opener: held as a bound.
- Constraint `PATTERN-tolerant-records` × D5's "leftover state ignored": held, apart from F1.
- Constraint test-speed × D9's Host check (`httptest` binds `127.0.0.1`): held.

**Findings**

**F1 — MEDIUM, design. D5 × D11, `decision-record.md` D5 bullet 2 and D11 bullet 2.**
D5's claim that "a held lock proves a live server … so the recorded port is the server's" is broken while a server is starting.
- D11 orders `serve` as: take the lock, build the board, register, bind the port, then write the state. Between taking the lock and writing the state, the lock is held while the state file on disk is still the leftover from an earlier server, with its old port and token.
- An `open` that reads in that window treats the leftover as live: either the `open` that spawned the server, or a second `open` pressed meanwhile. It sends a bearer-token request to the leftover port, where another user's process may now listen.
- That is the stale-state token leak that support-log Turn 4 ("interactions 6, bound-breaker 3") records as fixed.
- Fix: `serve` removes the leftover state right after it takes the lock, before anything else. Alternatively, `open` accepts only state whose pid is a live holder, or whose pid is the server it started. Add a criterion for the window.
- Evidence: `decision-record.md` lines 128–131 (D5) and 276–280 (D11); `support-log.md` line 45.

**F2 — LOW, design. D5 × D11 exits, `decision-record.md` D11 bullet 6.**
A server draining on binary replacement or idle exit still holds its lock.
- It "stops taking requests and waits for those in flight", and in-flight long polls last up to the bound under a minute (D10).
- During that window, `open`'s liveness read reports it live and hands out a launcher to a server that refuses the page. Typically this is the first press after an upgrade.
- The page then retries and shows the `lym board open` line, and a second press works. No bound states this.
- Fix: state the window as a bound, or have the drain wake the long polls at once and release the lock before the drain wait.
- Evidence: `decision-record.md` lines 128–130 and 288–291; D10 line 244.

**F3 — LOW, consistency. D2 × D9/D10, `decision-record.md` D2 paragraph 2 and D9 bullet 6.**
- The one embedded page keeps `&lt;script src="board-data.js"&gt;` (the D2 criterion requires the tag). Over `http`, D10 lists no route for that file, and D9's CSP allows inline script only by hash.
- So in `http` mode the tag is a 404 and a CSP violation.
- The record does not say how the page tells the modes apart, so that the missing `window.BOARD` over `http` does not trigger D4's `lym board rerender` line.
- Fix: state that the page decides its mode by protocol before reading `window.BOARD`, or that the server answers the script path.
- Evidence: `decision-record.md` lines 61–63, 118, 226, 242–246 and 407.

**F4 — MEDIUM, design. D8 × D10 write response, `decision-record.md` D8 bullets 2–3 and D10 "Each answers with the entry's fresh projection".**
- D8 says the revision is computed over the entry as loaded and migrated. D10's success answer carries a revision computed over the in-memory entry after the write.
- The record does not say this equals what the next load-and-migrate yields.
- `Task` holds `[]string` fields without `omitempty` and a custom encoding of `Unknown`. Any difference between the in-memory and the reloaded encoding makes the operator's immediate second edit stale.
- No criterion covers editing twice in a row.
- Fix: state that the response revision is computed by the same function over the saved entry as a reload reads it. Add the criterion "an edit carrying the revision a successful write returned is saved".
- Evidence: `decision-record.md` lines 195–204 and 254; `internal/boardengine/task.go` lines 27–31 and 77–79.

**F5 — LOW, consistency. Acceptance criteria against D9, `decision-record.md` D9 bullet 4.**
- The file route's token in the `t` query parameter is a trust check with no criterion. D9's criterion names only `/api` requests, and D10's file-route criterion tests only names.
- The D5 envelope's "address without the token" is likewise unpinned (NIT).
- Fix: add "`GET /file/&lt;name&gt;` without `t` or with a wrong one is refused", and add the envelope assertion to the D5 criterion.
- Evidence: `decision-record.md` lines 221, 429 and 436.

**F6 — LOW, consistency. Acceptance criteria against D6, `decision-record.md` D6 bullet 3.**
- Two refusals have no criterion: the regeneration refusing a member named as the beam mark, and the builder refusing `folders` or `settings`. Both are refusals that "write nothing".
- Fix: add a criterion for each refusal, with no file written.
- Evidence: `decision-record.md` lines 164–168 and 420–424.

**F7 — LOW, consistency. Acceptance criteria against D10 page behaviour, `decision-record.md` D10 bullet 6.**
- Three D10 behaviours sit in no criterion: the create and edit forms, the actions offered per kind and status, and the stale refusal showing the fresh entry beside the kept, unsent edit.
- The D4 `Review:` criterion covers only refusal text and the absence of edit controls at `file://`.
- Fix: extend the D4 `Review:` criterion, or add a D10 `Review:` criterion.
- Evidence: `decision-record.md` lines 257–259 and 410–415.

I made no tool calls; every finding comes from the record and the code already in context.
