```yaml
lens: board-scope
outcome: reported
```

Task: board-scope lens review of the move-task-by-handback decision record and support log.

**Findings**

1. MEDIUM, class `design`, D2 and D4: the record leaves the move note uncommitted, so the archive tag does not keep it, which contradicts the entry.
   - The entry's step 2 says: "The archive tag keeps the run records, the hand-back and the commits, so the new run can read them."
   - D2 says: "The note is never committed by the writer; the entry's body is its durable copy". Its bound adds that `--remove` keeps the note only under a local snapshot tag on this machine.
   - So the pushed records archive tag never holds the note. The record departs from this goal point and never says so. Rejected alternatives explains why the note is not committed, but the decision record does not acknowledge that it departs from the entry.
   - Fix options: commit the note before the cancel so the records archive tag carries it, or state the departure in D2 with its reason.

2. LOW, class `consistency`, D8: the entry's own term "hand-back file" is renamed to "move note". This is justified by the collision with the driver's hand-back and is allowed under the glossary rule, so it is no contradiction. It is noted only because the entry and its title keep the old word.

**Goal points of the entry**

- Writer writes what it learned to a file in its run records (done, decisions, left, commits): met in D2, but the note is not kept by the archive tag (finding 1).
- The run is cancelled with `lym batten cancel`, setting the entry to `draft` and freeing it: met in D4 and Constraints, with `--remove`.
- The archive tag keeps records, the hand-back and the commits: partly met. Commits and records are kept; the hand-back is not (finding 1).
- The orch writes the hand-back into the body or into a new task under the new parent, then releases it: met. D4 and D5 pick the body of the same entry, which the entry's "or" allows.
- Settle whether the new run re-uses or only reads the old commits: met, D6 (read only).
- Settle verb or orch guidance, never a recipe step: met, D1.
- Prerequisite adopt work has landed: no conflict.

**Entries, peers and parent commits checked**

- `stale-records-ref-blocks-rerun` (peer, running): no conflict. Its record fixes the re-run block that D5 depends on, and the record names it under Open risks.
- `per-task-settings-one-lookup` (peer): no conflict.
- `orch-autonomy-mode`: text overlap in `contracts/orchrole/` only. Its "parent orch settles halts" does not contradict D4's release step. No conflict.
- `seed-killed-way-forward-loops` and the remaining notes and projects: unrelated, no conflict.
- Parent commits: `lym loom peers` listed none.

**Decision notes**

- No `Decision for move-task-by-handback:` note exists on the board, so none was read and none was out of reach.

**Tool calls**

- Read the peer `stale-records-ref-blocks-rerun`'s decision record, to check D5's re-run against its design. My context held only its board entry. Its record holds only the entry body so far.
