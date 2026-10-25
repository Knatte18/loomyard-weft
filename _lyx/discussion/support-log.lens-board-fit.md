```yaml
lens: board-fit
outcome: reported
```

Board-fit review of the umbrella-anchor decision record against the board, its peer runs and parent commits: one MEDIUM and one LOW finding, and nothing for the run's parent to decide.

**Findings**

1. **MEDIUM — D8, census of weft branches on origin** (class `consistency`). The step that removes the Bolt marker reads every weft branch on origin, synchronously, inside `lyx fabric reconcile`. This goes against the operator's decision in note E19 `fabric-remote-async`.
   - Evidence: E19's "Decisions (operator, 2026-10-18)" say "Every operation against a remote runs async, so no lyx command waits on the network". The one exception allowed is the check whether a pair's branches already exist on origin, which `fabric add` makes.
   - D8 needs to read origin's weft branch tips before it may remove the Bolt marker. Its "census that cannot read origin" wording means a live network read.
   - Fix: run the census over remote-tracking refs that are already fetched, with no network call, and keep the Bolt marker when a ref has not been fetched. Or run the census in the detached work that reconcile already spawns. Either way, state in D8 that the census does not wait on the network.

2. **LOW — D3 and the T4 task `dobby-overview-window`.** The overview will read each worktree's dobby files under `.lyx/beam/`, which sit at that worktree's own anchor. D3's scan and its `lyxcwd` constructor already cover this, so there is nothing to fix.
   - T4 depends on `dobby-skip-done-runs`, not on this task. If T4 lands first and builds a beam's dobby path from the prime's `AnchorRel`, D3's scan will catch it.
   - Optional fix: add one Open risks line saying the dobby log paths follow each worktree's own anchor.

**What I checked**

- This task's entry `umbrella-anchor`: no conflict.
  - The entry wants the parent's marker "kept out of the merge as raddle is". The record settles this as a fact from the code: the weft never merges. That premise is overtaken, not outdated.
  - The entry wants a beam to default to "the anchor of the worktree it is spawned from". D6 uses the parent beam's anchor through the fork, which matches how `beam create` picks its parent.
- Decision notes for this task: none. No `Decision for umbrella-anchor:` title appears in `lyx board list`.
- Peer run `dobby-skip-done-runs`: no conflict. Its decision record changes only `internal/dobbyengine`, which D3 does not touch (D3 touches `hubgeom` and `beamcli`). Its `PATTERN-cwd-resolution` line only says it adds no import, so it does not clash with this record's change to that entry.
- Parent commits: `lyx loom peers` lists none.
- Note E19 `fabric-remote-async`: conflict, finding 1. The Open risks line about local-only pairs agrees with E19, since no lyx flow creates them.
- Note F20 `loom-beam-darns`: no conflict. Darn runs inside a beam inherit the beam's anchor through D5.
- Task T4 `dobby-overview-window`: overlap only, finding 2.
- P1 `workspaces` (the project) and the other notes: only text overlap or none, judged from their titles and briefs.

**Tool calls**

- `lyx loom peers show dobby-skip-done-runs --record decision`: my context had the peer's changed paths but not its decisions.
- `lyx board get fabric-remote-async`: my context had only the brief; I needed the operator's decision text on remote operations.
- `lyx board get loom-beam-darns`: my context had only the brief, which names beams.
- `lyx board get dobby-overview-window`: my context had only the brief, which names per-beam dobby state paths.
