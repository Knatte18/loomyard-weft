```yaml
lens: board-scope
outcome: reported
```

Task: review the decision record's fit with the board (the board-scope lens).

**Findings**

1. **MEDIUM, class `decision`, D5 and Open risks.** D5 drops something the entry's Delivers line names: `lym board open` "with its refusal naming lym board rerender".
   - The record already raises this under Open risks, so it is consistent.
   - Only the run's parent can settle it, because a named deliverable is being removed.
   - Question for the parent: confirm that the `lym board rerender` refusal goes, now that the server serves the embedded page.
   - Evidence: the entry body's Delivers line; D5 has no refusal for an absent `board.html`.
2. **LOW, class `decision`, D1 to D3 and Open risks.** The committed `board.html` and `board-data.js` now serve only a page someone opens by hand at `file://`.
   - The record keeps them, as Delivers requires, and asks the parent in Open risks. Nothing contradicts the board, so this needs no fix.
   - Question for the parent: should these two render outputs stay committed, now that editing needs the server?
   - Evidence: D2 and D3, plus the record's Open risks bullet.

Nothing else contradicts the board.

**Each goal point of the entry, met or not**
- `board-data.js` and `board.html` as render outputs, in the manifest, carried by the sync and the owner-set write, and claimed by FabricReplay: met (D1 to D3).
- The viewer page with filters, search, README grouping, expandable entries, escaping and link rules, and colour scheme: met (D4).
- `lym board open`:
  - Its ResolveWithin cwd: met.
  - Its refusal naming `lym board rerender`: not met; it is dropped and raised (finding 1).
  - Its per-platform opener, logged and waited on: met (D5).
- The `board open` task in `tasks.json`, `beam start` through fabric's workspace-content hook, and keys `alt+shift+0` and `alt+shift+9`: met (D6).
- Exports:
  - boardengine's constants and embedded viewer, boardcli's open verb and opener seam: met (D2, D5, D13).
  - fabricengine's hook type and its Install function, internal/vscode's task labels and the tasks member, the ide hook installed in `cmd/lym/main.go`: met (D6).
- Wanted next:
  - Create and edit tasks and notes, release, promote, draft and undraft: met (D10).
  - Live refresh when an agent, the orch or the dobby writes the board: met for writes that land in this machine's Bolt (D11). The record states that bound.
  - Writes go through boardengine: met (D10, D13).
  - A write carries its revision; a stale write is refused and the page shows the fresh entry: met (D8, D10).
- The Added after Discussion entries (D7 moot, no launch.json, phone goal met, the open chain and LY title stay out, the D6 seam placement) are carried verbatim, and no decision contradicts them.
- No decision edits this task's own board entry, files a task or promotes a note.

**Entries, peers and parent commits checked**
- `single-repo-records-branches` (T2, a peer in Discussion-Write): no conflict. Its record keeps `_bolt` as the board's worktree, so D11 can still watch `board.json` in the Bolt.
- `two-repo-merge-probe-regression` (T4): no conflict; its changes stay inside fabricengine's layout probe.
- `loop-ends-without-envelope` (T8): no conflict. It also touches `internal/proc/proc_linux.go`, but that is the same file only, and D11 only calls `proc.Detach` and `proc.Reap`.
- `empty-landing-not-flagged` (T7) and `parent-review-code-and-drift` (T13): no conflict; their changes are in landing, drift and loom.
- `sandbox-closed-area-build` (F9, a note with the `ide` label): no conflict; it is only a note and not active.
- Parent commits: `lym loom peers` lists none.

**Decision notes**
- No note titled `Decision for board-viewer-build:` is on the board. I checked the board list already in my context, and there was none I could not reach.

**Tool calls**
- `lym loom peers`: my context had no peer or parent-commit list.
- Searched the `single-repo-records-branches` decision record for board and Bolt terms: my context did not have that peer's record, and it is the nearest peer touching where the board lives.
