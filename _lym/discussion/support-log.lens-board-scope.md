```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope — I checked whether the decision record fits the board.

**Goal points of the entry**
1. Bug 1, recreate or adopt the pair when its branches still exist: met by D1.
2. Bug 1, hand the run back with a stop naming the recovery instead of staying `running`: met by D2. It uses a `failed` stop, not a hand-back, and the record's auto-mode assumptions explain why.
3. Bug 1, a stop that names `lym fabric remove` leaves no record that dobby then drives: met by D3 and D2 together.
4. Bug 2, the archive step pushes the weft branch before the tag, or retries on "missing necessary objects": met by D4, which does both.
5. Bug 2, the tackler checks `git ls-remote`: met by D5.
6. Bug 3, a cancel's background work never removes a branch a later run owns: met by D6 and D7. Constraints records, with evidence, that the cancel's deletes run synchronously. The unpinned trigger is listed under open risks.
7. Common thread, decide whether one ordering rule covers all three cases: met. D7 sets a per-slug order and D6 adds leased deletes; the question ledger (Q1) gives the reasoning.

**Entries, peers and parent commits checked**
- `fabric-remote-async` (note E19): D4 adds one synchronous push to teardown, against the note's direction that every remote operation runs async. The note is not a task, and Constraints names the departure, so this is no finding.
- `landed-run-left-paused` (note B2): no conflict. The record excludes it from scope.
- Peers `shed-sub-run-block`, `decision-record-on-board`, `shuttle-autocompact-picker-stall`, `reconcile-strips-newer-keys` and `parent-align-fixed-reads-dirty`: none touches batten, the fabric teardown or cancel, and none conflicts.
  - `parent-align-fixed-reads-dirty` works in landingshed, and D3 edits the text of one Finalize stop in `internal/landingshed/finalize_owner.go`. That is a text-level overlap of the same package only, so no finding.
- Parent commits: `lym loom peers` lists none.

**Decision notes**
- I found no `Decision for batten-remote-order:` note on the board, and none was out of reach.

**Findings**
- LOW, D2 against goal point 2: the entry asks the run to be "hand[ed] back with a stop", and D2 writes `failed` instead. The record states and justifies this under auto-mode assumptions, and the effect the entry wants (dobby stops retrying, and the stop names the recovery) is kept. I raise it only so the review sees the wording gap; it needs no change.

No BLOCKING or MEDIUM findings, and no `decision`-class findings.

Further tool calls:
- One `lym board list` and `lym loom peers` grep, to check for a `Decision for` note added after the writer's read and for parent commits. My context held earlier copies, but a note could have been added since.
