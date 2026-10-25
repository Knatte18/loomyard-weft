```yaml
lens: existing-paths
outcome: reported
```

**Paths traced, per decision**

- D1/D2 working-tree marker × `parentRecordsCommitter` → `lyxcwd.ResolveWorktree` → `CommitAnchoredPaths` at the parent's `AnchorRel` (landing child): **breaks** mid-move (F1).
- D5 `Add` anchor from the parent's Location × a parent with a standing move record (working-tree marker ≠ committed marker): **breaks** (F3).
- D7 `.weft/` move record × `restoreWeftLockDir` (creates a missing dir only, never clears): survives.
- D7 move record × `repairPairWiring`/`wireScratchLink`/`restoreLaunchers`: survives (D8 refuses the pair before them).
- D7 step 2 `EndSparing(HubOwner, push, sync)` for the prime dobby × other `HubOwner` registrants (only push, sync, dobby): survives.
- D7 step 2 `EndSparing(beam slug)` × beam's own pushes and child-pass descendants: survives (dobby-spawned pushes ended, as conceded).
- D7 dobby gate × `EnsureLive` callers (orch start, adopt, beam start): survives for dobby starts; adopt's own pass/Add/gate suite/loom resume **break** around it (F2).
- D7 move lock × existing weft writers and pulls (`weft.write.lock`, `fabric.push.lock`): **breaks** — none contend on the move lock (F4).
- D7 recovery "integration landed when HEAD ≠ record's tip" × weft commits/ff by other writers: **breaks** (F4).
- D7 prime refusal of a live creel daemon × creel geometry: **breaks** — state under `<hub>/_board/.lyx/creel`, not the prime anchor, and no stop verb (F5).
- D8 census × fetched refs only: survives (no network call).
- D10 pull stop × `pullWeftArm` → `PullWeft` (fetch+ff) before `settleUnmovedWeft`: the stop needs the ff split off — plan detail, survives as designed.

**Findings**

- **F1 — BLOCKING — design — D7 steps 1–5 (wait, then no hold of the parent lock).** Summary: a loom child run lands into the moving worktree mid-move, because D7 reads "the parent lock is free" once in step 1 and never takes or holds it. A loom child run's Go driver keeps running between batten passes (dobby only polls it with continue passes); its Finalize takes the parent lock after that read, then commits the parent's run records via `CommitAnchoredPaths` at the parent's working-tree `AnchorRel`, and merges into the parent's branch, while D7 has the weft content half-moved and the junctions removed. A beam's refusals do not cover the child's own strand. Evidence: `internal/loomcli/landingdeps.go:469-488` (`parentRecordsCommitter`, `ResolveWorktree`, `CommitAnchoredPaths`), `:375-382` (parent lock in Finalize), `internal/dobbyengine/doc.go:36` (continue passes; the child run outlives a pass). Fix: the verb acquires the parent lock of the worktree's branch in step 1 and holds it to step 8.

- **F2 — MEDIUM — design — D7 "The move record keeps `dobbyengine.EnsureLive` … from starting a dobby".** Summary: adopt's other steps bypass the gate with no stated bound. While the move record stands, `lyx adopt` still adds pairs through `Topology.Add` from the prime, resumes runs as detached first passes via `battencli.StartFirstPass` (outside dobby, so no pass gate), starts `lyx gate test --gate finalize` in the prime's anchor, and spawns `lyx loom resume` for owner-halted children; for the prime each writes or sets cwd under `A` mid-move. Evidence: `internal/adoptcli/doc.go` "Step 2", "Steps 4 and 5", "Step 6" and the owner-halt resume paragraph; `internal/battencli/cli.go:580` (`StartFirstPass`). Fix: adopt refuses while a move record stands in the worktree it acts on (for the prime, any).

- **F3 — MEDIUM — design — D5 × D7: `Add` from a worktree whose move record stands.** Summary: a fork from a moving worktree can get an anchor that differs from its content. D5 takes the new pair's anchor from the parent's Location, which `lyxcwd` reads from the working-tree marker; the fork starts from the weft branch's commit. A kill after step 4 wrote the marker `B` but before the commit leaves working tree `B` and HEAD `A`; a `lyx beam create`/`fabric add` from that parent then gets anchor `B` while its forked content and marker sit at `A`. Nothing refuses `Add` while the parent's record stands (D7 gates only `EnsureLive`, the dobby loop and reconcile). Evidence: `internal/loomcli/landingdeps.go:478` (same `ResolveWorktree` working-tree read pattern), D5/D7 step 4 in the record. Fix: `Add` refuses a fork point with a standing move record, naming `lyx fabric anchor <slug> --set <B>`.

- **F4 — MEDIUM — design — D7 move lock and recovery "an integration landed when `HEAD` differs from the tip the record holds".** Summary: the move lock excludes no existing weft writer, and the integration recovery test misreads a HEAD moved by another writer. Every weft writer contends on `.weft/weft.write.lock` and the coalescing push on `fabric.push.lock`; the record never says the verb takes either, and weft pulls are not refused while the record stands. Spared pushes, a pull (`PullWeft` fast-forwards whenever markers agree), `lyx fabric commit`, and the F1 parent-records commit can each commit or move the weft during the move; after a kill, such a moved HEAD reads as "integration landed", and the re-run renames `.lyx` and wires at `B` while content sits at `A`. Evidence: `internal/fabricengine/recordsgit.go:22-33` (lock files), `internal/fabricengine/pull.go:310-314` (`PullWeft` before settle). Fix: the verb holds the weft write lock across step 4, weft pulls and commits refuse while a move record stands, and recovery judges an integration by the committed marker at HEAD and HEAD descending from the recorded `U`.

- **F5 — MEDIUM — design — D7 prime refusal "a live hibernate or creel daemon of the hub".** Summary: the creel half rests on a false premise and has no way forward. The creel daemon's state is under `<hub>/_board/.lyx/creel`, not the prime's anchor, and it is started from the hub directory; no lyx verb stops it — it runs "until it is stopped or its binary is replaced" — so the refusal fires on every attempt (`PATTERN-refusal-way-forward`). Evidence: `internal/hubgeom/creelgeom.go:24`, `internal/fabricengine/junctionnames.go:167`, `internal/creelcli/cli.go:766-769`, `internal/creelcli/spawn.go:49`. Fix: drop the creel daemon from the prime refusals.

**Tool calls**

1. `battencli.StartFirstPass` and `parentRecordsCommitter` — to confirm how adopt starts its pass and how Finalize writes into the parent; context had only the doc lines.
2. `recordsgit.go` lock constants and `CommitAnchoredPaths` — to name the existing weft write locks the move lock does not cover.
3. `coalesce.go` push loop — to check whether a spared push commits (inconclusive; no finding rests on it).
