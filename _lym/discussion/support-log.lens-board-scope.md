```yaml
lens: board-scope
outcome: reported
```

Task: check the record's fit with the board (board-scope lens).

Findings:

1. MEDIUM — D6, D7 and Open risks (class `decision`). The peer `shed-sub-run-block` is now at Discussion-Write. Its entry says tapestry "stays unchanged while this is built", and its `cartoon` holds "tapestry's Discussion rows". D6 changes tapestry's Discussion-Bouncer row by adding `approve_seam: discussion`, so the two runs touch the same row at the same time. D7's guard settles the outcome mechanically, but which task owns the change to the shared Discussion row is for the parent to decide. Question for the parent: should this task change tapestry's Discussion-Bouncer row, or should `cartoon`, through task T1 or the follow-up, carry the store? The record already raises this under Open risks, so this is a confirmation, not a gap.
   Evidence: T1 body "Today's tapestry stays unchanged while this is built"; note `discussed-task-split`: "cartoon ... stores the record and flags the entry discussed, as the core above says".

2. LOW — Scope "Out: clearing the flag" and Auto-mode assumption "Nothing clears discussed". This conflicts with nothing on the board. Note `discussed-task-split` says "Changing one's mind loses nothing: the run is cancelled, the record stays", which matches keeping both. No finding beyond noting it agrees.

Goal points of the entry:
- Copy at approval and flag discussed: met (D6, D2).
- `decision add` updates the stored record the same way: met (D8, whole-record copy).
- lym writes record and flag, exempt from the claimed-entry lock: met (D2, D3).
- `lym board get` shows the flag and prints the record: met (D10).
- A cancelled run leaves the record and the flag: met (D4).
- Record in its own file beside `board.json`, carried by the board's commit and replay, never inside `board.json`, with the flag as a field lym sets like status: met (D1, D5, D2).
- Beam project record as precedent: followed in spirit. The caller rule is shared and selection is by recipe rows, and project records are left out, which the entry permits.
- Out items (Discussion skip, tapestry and darn unchanged, splitting): respected. Tapestry routing is unchanged; only the Bouncer config key is added (see finding 1).

Checked:
- Entry `decision-record-on-board`: no conflict.
- Note `discussed-task-split` (F14): no conflict. The record covers its core half; the split and `cartoon` are left out as F14 says.
- Task `shed-sub-run-block` (T1): see finding 1.
- Peer `shed-sub-run-block` at Discussion-Write, with no record or plan yet, so its design can't be compared.
- Parent commits: none listed.
- `Decision for decision-record-on-board:` notes: none on the board.

Tool calls:
- `lym loom peers` plus a grep of `lym board list` for decision notes: my context held only the earlier peers snapshot (Worktree-Create), and I needed the peer's current row and any new decision note.
