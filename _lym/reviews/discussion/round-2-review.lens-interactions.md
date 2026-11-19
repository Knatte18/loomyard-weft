```yaml
lens: interactions
outcome: reported
```

Task: review the decision record through the interactions lens, checking each pair of decisions that touch the same state.

**Pairs checked**
- D1 ↔ D8 (when the revision is taken compared with the render's undiscussed copy): held.
- D1 ↔ D3 (render time in `board-data.js` means every sync commit touches it): held, stated in D3's bound.
- D1 ↔ D10 (projection and board revision returned by `GET /api/board`): broken. See F-c.
- D2 ↔ D9 (one page in two modes, CSP sent only as a header, script tag blocked over http): held.
- D2 ↔ D3 (fixed bytes rewritten on every render): held.
- D4 ↔ D9 ↔ D10 (the design and decision links over http, the file route, the `t` token): broken. See F-d.
- D5 ↔ D11, start (lock read, lock wait, leftover state removed, state written after bind, racing serves): held.
- D5 ↔ D11, exit and drain (lock released before the drain; an `open` during the drain starts a fresh server): held.
- D5 ↔ D11, server log (pid line and refusal message): broken. See F-e.
- D5 ↔ D12 (orch allowed, keep-warm cap on the opener wait): held.
- D5/D11 ↔ the boardcli pre-run (`Resolve` gate in the prime's anchored directory, `--board-path` refused): held.
- D8 ↔ D10 release, promote, draft and undraft (status is in the hash; sibling side effects bounded): held.
- D8 ↔ `boardCriticalSection` migrations (both sides hash after the migrations): held.
- D8 ↔ D3 replay (another machine's write makes the page's revision stale): held.
- D9 ↔ D5 (Host check against the probe, token sent through the launcher): held.
- D9 Origin check ↔ the D2 page over http: held.
- D10 long poll ↔ D11 exit (waiters woken with "unchanged", page retries): held.
- D11 state directory under the board-level `.lym` ↔ D3 sync staging: held, since `.lym` is excluded (`recordsgit.go:418-435`).
- D11 skip and env variables ↔ D12 empty strand name: held, stated in the bounds.
- D6 tasks ↔ D5 cwd (the task folder is the anchor; multi-root runs the first copy): held.
- D6 workspace seam ↔ the Constraint on `PATTERN-fabric-boundary` and the Added entry on the seam: held.
- D6 keys ↔ the `alt+shift+1` to `alt+shift+6` constraint: held.
- Added entries of 2026-11-19 ↔ Open risks and support-log Q8, Q9, the Rejected alternative and Turn 4: broken. See F-a.
- Acceptance criteria ↔ their decisions: held, apart from the D4 Review criterion. See F-d.

**Findings**

- **F-a: MEDIUM, consistency.** Location: `_lym/discussion/decision-record.md:452`.
  - Summary: two Added after Discussion entries (2026-11-19) settle the questions in the Open risks bullet and direct its removal, but it still asks the parent to confirm the dropped refusal and the kept render outputs.
  - The support log still says "raised to the parent in Open risks" in Q8 and Q9 (`support-log.md:106-107`). The Rejected alternative still says "left to the parent in Open risks" (`support-log.md:75`). Turn 4 still says "already stand in Open risks" (`support-log.md:62`).
  - Evidence: `decision-record.md:404-414` (the "stay committed render outputs" and "open's refusal … is dropped" entries) against `decision-record.md:452-457`.
  - Fix: drop the bullet so Open risks states that no risk is open. Make Q8, Q9 and the Rejected alternative cite the two Added entries. Make the Turn 4 line say the parent answered.

- **F-c: LOW, design.** Location: `_lym/discussion/decision-record.md:282`.
  - Summary: D10 returns the projection, which D1 and D8 build through the load-and-migrate step, together with the board revision, a SHA-256 of `board.json`'s bytes. Nothing ties the two to one read of the file.
  - A write that lands between the two reads pairs the old projection with the new revision. The page's next `after=` poll then waits on the new revision and shows the stale view until some later write.
  - Evidence: `decision-record.md:230-231` and `decision-record.md:282-284`.
  - Fix: state that the board revision and the projection come from one read of `board.json`'s bytes.

- **F-d: LOW, consistency.** Location: `_lym/discussion/decision-record.md:112`.
  - Summary: over http, D4 links the design and decision files to "the server's file route (D10)". D9 refuses that route unless the token is in a `t` query parameter, and neither D4 nor its Review criterion says the link carries `t`. A link built as D4 describes it is therefore refused.
  - Evidence: `decision-record.md:112`, `decision-record.md:257`, `decision-record.md:474` and `decision-record.md:501`.
  - Fix: in D4's link bullet and its criterion, state that the http link carries the tab's token as `t`.

- **F-e: LOW, consistency.** Location: `_lym/discussion/decision-record.md:150`.
  - Summary: D5's refusal relies on "the log opens with the server's pid", but D11 sends every started server's stderr, a losing racer's included, to the same log file in the state directory.
  - If each spawn appends, the log opens with the first server's pid, not the live one's. If each spawn truncates, a racing second `open` erases the winner's pid line.
  - Evidence: `decision-record.md:150`, `decision-record.md:319` and `decision-record.md:324-325`.
  - Fix: either name the log's last pid line as the live server's and have every spawn append, or log the pid to a per-start file the state names.

Tool calls: none. The inherited context held every definition this lens needed.
