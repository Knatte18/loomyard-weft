```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope. I checked the record against the board entry, the touching entries, the run peers and the decision notes.

**Findings:** none at BLOCKING or MEDIUM.

- LOW, D5/D6 against the entry's part 3 ("outside every gate … no run waits on it"). `lym gate testmap` holds one gate slot for the whole rebuild. A gate run can therefore wait for that slot when the pool is full, about four minutes for `internal/fabricengine`. D5 states the bound ("can delay other gate runs by one slot's worth at most"), so the record already says where it departs from the entry's wording. It is not a contradiction; it is listed only so the parent sees the trade.

**Goal points of the entry:**
- Mechanical way to find redundant tests, at a cost the gates can carry: met by D7 (capped by `redundancy_max_tests`, advisory) and D2.
- Part 1, package-scoped tool, one package at a time, module-wide view weighed: met by D2. The module-wide view is kept as a column from the same run, and `-pkg` narrowing stays.
- Part 2, a map per package with its commit, stale on a package change, other maps still valid: met by D4 and D5. Staleness is judged per package by file hash.
- Part 3, one job rebuilds stale maps outside gates, and no run waits; open where it runs and where maps live: met and settled by D5, D6 and D4 (machine-local `.lym`, the prime parent's dobby plus a by-hand verb).
- Part 4, gate step on the tests the diff adds or changes, a finding for Code-Review naming the covering tests, never failing, `//testtiming:keep` not flagged, a stale map said so: met by D7 and D8.
- Constraint, per-test serial coverage: met (Constraints).
- Constraint, no per-module redundancy test in the suite: met (Scope Out, Rejected alternatives).
- Constraint, block coverage is evidence and nothing refuses or deletes: met (Goal, Constraints, D7, D8).
- "Weigh a smaller first stage": weighed and rejected with a reason (support log Turn 2, Rejected alternatives). The entry invites this, so it is no contradiction.

**Entries, peers and parent commits checked:**
- `fabric-test-cuts` (peer, Darn-Write): no conflict. Its record has the cuts confirmed against the module-wide `cmd/testtiming -redundancy` check. D2 keeps the module-wide verdict as a column, and the peer works on its own copy of the tool. The record's Open risks names this.
- `gate-shard-slow-package` (note E3): no conflict. Sharding a tier run by `-run` does not touch the map, the verb or the report step, and the record leaves the entry alone.
- `fabric-test-speed-work` (project P3): no conflict. It groups the fabric speed tasks, and this task is not a member and duplicates none of them.
- `sampler-drift-trial` (peer, Code-Burler-Fix): no conflict. Its changed paths (driftcli, driftengine, loomrecipe, shedrecipe doc, sampler recipe) do not meet the gate, testtiming, dobby or code-review stencil work. Any overlap is bookkeeping at most.
- Parent commits: `lym loom peers` listed none.

**Decision notes:**
- No `Decision for redundant-test-finder:` note appears in `lym board list`, so none was read and none was unreachable.
- The record has no `Added after Discussion` entries.

**Tool calls:**
1. Grepped `fabric-test-cuts/_lym/discussion/decision-record.md` for its testtiming and keep instructions. My context had the peer's board entry but not its decision record, which the peers listing said exists.
