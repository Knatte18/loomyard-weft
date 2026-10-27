```yaml
lens: board-fit
outcome: reported
```

Lens: board-fit. I checked the record against the board, the loom peers and the listed parent commit.

**List**

- Board entry `dobby-and-adopt-hardening` (T2): the body is already in context. The board list holds no note titled `Decision for dobby-and-adopt-hardening:`, so there was no decision note to read and none I failed to reach. The entry's own "Decision for part 4" section is reflected in Decisions 10 and 11.
- Entry and peer `hub-reader-creates-lym` (T4): they overlap but do not conflict.
  - The peer drops the `MkdirAll` from `state.ReadJSON` and skips the lock when its directory is absent.
  - Decision 2's state read before the lock, and the `check.json` and history writes in Decisions 7 and 10, all go through `internal/state` writers, which create their own directories, so they still hold.
  - Both touch `shedengine/run.go` and `battencli`, but at different places, so this is text-only.
- Entry and peer `test-logs-leak-into-bolt` (T3): they overlap but do not conflict.
  - Its D2 keeps `_board/.lym/` out of Bolt, so the new `.lym` files (`dobby.out`, `dobby.log.1`, `check.json`) stay untracked.
  - The record's Scope Out bullet misstates what that entry covers (F1 below).
- Peer `readme-draft-not-ready` (T5): no overlap. It touches only board README rendering.
- Entry `workspaces` (P1, the parent project): no conflict.
- Note `docs-link-skip-bench-data` (T6): no overlap.
- Parent commit `b793cc8` (Codex bench on P2): no overlap or conflict. It adds docs and research data only, with no code, contract or PATTERN change.

**Findings**

- **F1**
  - Severity: LOW
  - Class: consistency
  - Location: `decision-record.md`, Scope, the first Out bullet.
  - Summary: the bullet says the trace sink's placement under the hub's `.lym/logs` is the subject of `test-logs-leak-into-bolt`. That entry's decision record is about something else: a `fabriccli` tier1 test that redirects the trace sink onto the live hub (its D1), and `Bolt.Commit` seeding the `.lym/` exclude (its D2). Its own Scope puts the logger's override semantics out of scope, and nothing in it settles where a dobby's trace file belongs. The bullet's justification therefore points at the wrong entry, and Decision 5 alone is what justifies the exclusion. The fix drops the cross-reference and keeps the Decision 5 reason.
  - Evidence: `lym loom peers show test-logs-leak-into-bolt --record decision`, its Scope and D1/D2. The record's text is at `_lym/discussion/decision-record.md`, Scope, Out, first bullet.

**Tool calls**

1. `lym board list`: the procedure requires the board list, including any decision-note titles, and my context lacked it.
2. `lym loom peers`: the procedure requires the peers and the parent commits, and my context lacked them.
3. `lym loom peers show hub-reader-creates-lym --record decision`: this is the closest peer, since it changes `internal/state` and `shedengine`, which the record writes through.
4. `git show --stat b793cc8`: this is the parent commit `lym loom peers` lists.
5. `lym loom peers show test-logs-leak-into-bolt --record decision`: the record's Scope cites this entry, and I needed its record to check that citation.
