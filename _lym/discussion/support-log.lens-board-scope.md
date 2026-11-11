```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope. I checked the record's fit with the board entry, the touching entries, the peers and the parent decisions.

**Findings**

1. `decision` · MEDIUM · Open risks / D13. The parent needs to answer whether the `testmap-cross-package` run is held, and whether D13's changes go onto it before it writes code.
   - Evidence: the entry body says "The three build tasks stay held until this record is approved".
   - `lym board list` and `lym loom peers` show `testmap-cross-package` at "running · Darn-Write", with no changed paths yet.
   - From this worktree, `lym loom status testmap-cross-package` finds no seed, so its state can't be checked here.
   - The writer raised this under Open risks, and it is right that this is a question for the parent rather than something the writer can settle.

2. `consistency` · LOW · D13 (`coverage-test-select`). D13 leaves out that `coverage-test-select`'s D5 Bound names the age limit, which D7 retires.
   - Evidence: T14's stored D5 Bound reads "A dependency-drift miss lasts until testmap rebuilds the map by its present rules: … the age limit, a failed build at another HEAD, or a testmap call naming the package".
   - D7 retires `test_map_max_age_days`, and D5 removes the rebuild after a failed build at another HEAD.
   - D13 hands that entry only "the rules of D2", so that Bound would go stale.
   - Fix: add a line to D13 saying that Bound's rebuild list becomes D1's plan verdicts.

3. `design` · LOW · D13 (carrier). The record sends D13's changes through `lym loom decision add` on each entry's run, and it says that happens before Plan. `coverage-test-select` and `gate-scope-coverage` have no run yet.
   - Evidence: the board shows no status for T14 or T15.
   - `lym loom decision add` works on a run's record, and T17 (`decision-record-replace`) has no stored record yet, because `lym board get T17 --record` errors.
   - The record already names T17 as a possible carrier under Open risks, so this is only a timing gap and doesn't conflict with anything.

**Goal points of the entry**

- A linked-only change leaves a map fresh: met (D1).
- A failed build keeps the old map or writes a failed-build map: met (D5).
- testmap skips spawn-marked tests: met (D4).
- Another package's coverage is recorded by function or by block: met (D3).
- The changes to T4, T14, T15, E12 and E4: met (D13), apart from finding 2.
- The record does not edit its own entry, and it files its follow-up as a note, not a task.

**Entries, peers and parent commits checked**

- `testmap-cross-package` (T4): its Sketch and its parent's Added-after-Discussion entries agree with D1. They hand the staleness rule to this note and keep the present rule meanwhile. The only conflict is the running status in finding 1.
- `coverage-test-select` (T14): finding 2.
- `gate-scope-coverage` (T15): no conflict. Its D8 says the gate only reads maps, and the record leaves that unchanged.
- `testmap-backlog-drain` (E12): no conflict. D8 answers it, and that matches the note's own idea.
- `redundancy-sweep-existing` (E4): no conflict. The order in the project `test-suite-trim` (E12, then E4) is kept.
- `decision-record-replace` (T17): no conflict. It has no record yet.
- Peers `machine-name-verb`, `worktree-create-build-leftovers` and `load-turn-retry-cap`: none conflicts. They touch nothing of testmap, gateslot or dobbyengine's testmap step.
- Parent commits: `lym loom peers` lists none.

**Decision notes**

- `lym board list` shows no `Decision for testmap-upkeep-design:` note.
- T4, T14 and T15 carry the same four Added-after-Discussion parent decisions, and none contradicts the record.

**Tool calls**

- `lym board get T17 --record`: to read the record of the near peer `decision-record-replace`, which my context didn't hold. It has none.
- `lym loom status testmap-cross-package`: to check the T4 run's state beyond its board status. It returned no seed here.
