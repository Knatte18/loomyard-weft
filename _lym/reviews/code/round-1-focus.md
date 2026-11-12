---
round: 1
focus:
  - "internal/dobbyengine/testmap.go: does the start rule match D8 on every branch (no exit record, moved HEAD, queued work), and do the holds (below two slots for queued work alone, live pass or unregistered child, standing anchor move) apply exactly where D8 says?"
  - "internal/beamcli startTestMap and internal/dobbyengine: does the dobby log a failed `--upkeep` registration exit once per episode and start a pass again on a later tick, and does an unregistered live child block a second start?"
  - "internal/configengine/layers.go, internal/dobbyengine/config.go, internal/configreg: is `test_map_every: 0` read as `test_map_upkeep: false` in every layer before merge and DropRetired, does Migrate drop other values, do `test_map_batch` and `test_map_every` load as retired keys, and is the in-memory pass silent?"
  - "internal/fabriccli/anchordeps.go: does dobbyPassInFlight stop counting a RoleTestMap process, and does anything else still wait on a pass?"
  - "internal/dobbyengine/state.go and loop.go: is the retired landing count still readable from an old state record, and is nothing left that writes or depends on it?"
  - "Tests in dobbyengine, beamcli, configengine and configreg: does each acceptance criterion have coverage, and does any new test duplicate another (see the redundancy report's unique and not-judged entries)?"
  - "docs and pattern diffs (docs/overview.md, doc.go files, pattern/PATTERN-test-speed.md, internal/runproc/record.go): is each change licensed by the decision record, and does any doc still name the retired keys or landing count as live?"
---

Seed focus from the decision record's D8, D11 and acceptance criteria against the diff's file list.
