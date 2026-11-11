```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope. I checked the record's fit with the board entry, the touching notes, the peers and the parent commits.

**Findings**

1. `MEDIUM` (class `decision`): D2, and Scope "Out".
   - **Summary:** D2 changes what `lym gate testmap` builds and when a map counts as stale. That is the cost and upkeep question note `testmap-upkeep` owns.
   - **Evidence:**
     - D2 builds every package's coverage binary with coverpkg over every in-module package the binary links, which raises the cost of each build.
     - D2 also marks every existing map stale through its format marker.
     - `testmap-upkeep` says "The operator finds keeping the maps fresh this way too large a job to run again and again". It proposes partial updates driven by recorded blocks, and the shape of those blocks depends on D2's format.
     - The project's `order` puts `testmap-upkeep` before `gate-scope-by-coverage-map`.
     - The record names the hit-rate dependence under Open risks, but not that D2 raises the upkeep cost the note complains of.
   - **Question for the parent:** may this task change the map format and testmap's build cost now, ahead of the `testmap-upkeep` policy? Or should D2's cross-package recording wait for that note, or be settled there?

2. `MEDIUM` (class `decision`): Scope "Out" (this hub's `gate.yaml`) and Open risks bullet 2.
   - **Summary:** the entry's goal, "Scope a verify gate by the coverage map", is about this hub's Darn-Write gate on OSL-1046. As written, the record lands a template default that this hub's tuned `gate.yaml` overrides. The hub's own gate would stay on `diff` until the operator acts.
   - **Evidence:**
     - The entry's "Seen" section names `testmap-verb` and `arras-recipe-perch` on this hub.
     - The hub's `gate.yaml` writes `darn-write: {tiers: [0, 1, 2], scope: diff}`.
   - **Question for the parent:** is the switch an operator step after landing, as the record assumes? Or should the run carry it, for example through a `run_settings` or config decision the parent records?

3. `LOW` (class `scope`): Open risks bullet 3.
   - **Summary:** the format marker makes every map need a rebuild. That pushes back the full map set that `redundancy-sweep-existing`, which has high priority, waits on.
   - **Evidence:**
     - The sweep note says "It needs a full set of coverage maps, one per package".
     - The maps are drained at `test_map_batch` per refresh (note `testmap-backlog-drain`).
     - Today no maps exist, so the cost only bites if maps are built before this lands.
     - The record names the backlog note, but not the effect on the sweep.
   - **Suggested fix:** one clause in the risk line.

**Goal points of the entry**

- Run the tests whose recorded blocks cover the changed code: met (D2, D3, D4).
- Plus the tests of the changed packages: met (D3).
- Everything else only in the landing gate: met (D1 bound; `finalize` stays `all`).
- Read how the computed scope forms today and what the map holds per test: met (support log, Turn 1).
- Decide what a map miss does, widening and never narrowing: met (D5). One part is a deliberate bound: drift in a package no test covered at build time.
- Waits on `testmap-verb`: landed as `143307f6` (`lym gate testmap`), so no conflict.

**Entries, peers and parent commits checked**

- Project `test-suite-trim`: order considered; findings 1 and 3.
- Note `testmap-upkeep`: finding 1. Its item 2 ("runs the whole package when its map is missing or stale") agrees with D5.
- Note `testmap-backlog-drain`: no conflict; the record defers to it.
- Note `redundancy-sweep-existing`: no judgment conflict, since D2 keeps the own-package judgment; finding 3.
- Note `finalize-flaky-rerun`: none. It concerns Finalize's fixer, and the record leaves `finalize` unchanged.
- Peer `codex-bubblewrap-sandbox`: none (no changed paths).
- Peer `adopt-owner-handover`: none (no changed paths). The `adopt` gate key is left unchanged.
- Parent commits: `lym loom peers` lists none. Landed `1b6e3fca` (stop-on-fail, failed-test rerun, budget) is honoured by D6; no conflict.

**Decision notes**

- No board note titled `Decision for gate-scope-by-coverage-map:` exists in `lym board list`.
- The record has no `Added after Discussion` entries.
- No notes were unreachable.

**Tool calls**

None. The board list, the task entry, `lym loom peers` and every touching note were already in my context from the writer's Step 1 reads.
