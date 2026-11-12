```yaml
lens: board-scope
outcome: reported
```

The record answers every goal point of the entry. The one real departure is that it drops the extra weft branch the operator described, which needs the parent's call. One draft task's inputs and the meaning it reads from a map's `commit` field are changed without the record saying so.

**Findings**

1. **BLOCKING — class `decision` (D1, Open risks):** Should the extra weft branch and junction be dropped?
   - Question for the parent: the entry says the design "settles whether that is one branch per code branch or per pair, how it is created and forked with a pair, and how it merges". D1 replaces that shape with the existing weft branch. Does the parent accept that?
   - Evidence: the entry's "Its shape (operator, 2026-11-12): one extra branch and one extra junction". The writer cannot settle this alone. The record already lists it in Open risks, so the raise is correct and should not be dropped.

2. **MEDIUM — class `decision` (Open risks, second bullet):** Does the raddle split into several tasks still hold?
   - Question for the parent: the entry's Idea says raddle splits the work because new weft branches are added. D1 adds none, so does that plan still apply?
   - Evidence: the entry's "## Idea": "The work is several tasks, since it adds new branches to the weft repository … so raddle splits it". The record already raises it; keep it.

3. **MEDIUM — class `design` (D9 against `testmap-upkeep-cost-and-guide`):** D9 deletes data that task needs.
   - D9 removes `<bolt>/.lym/testcover/`, including `testmap-exit.json`. That is a pass record task T2 reads to report upkeep cost "since `36621f0e` landed".
   - If this run lands first, T2 loses that input. The record does not mention T2's dependency.
   - Fix: keep or archive the exit record instead of deleting it with the directory, or record the ordering against T2.

4. **MEDIUM — class `design` (D6 against `testmap-upkeep-cost-and-guide`):** D6 changes what a map's `commit` field means.
   - T2's step 2 checks "which [maps] are at HEAD". Under D6 a current map keeps its old `commit`, so `commit != HEAD` no longer means the map is stale.
   - T2's guide would also describe the old place (`.lym`) and the old default (on). D1 and D8 change both.
   - Fix: state in D6 what tells a current map from a stale one, such as the exit record's `planned_head` or the plan's verdict. Also add a note on T2 that its guide must follow this task.

5. **LOW — class `consistency` (D7 against `redundancy-sweep-existing`):** The record does not say where a sweep's hand rebuild goes.
   - That note (Sketch `testmap-upkeep-design` D9/D13) has the sweep run a hand `lym gate testmap <package>` before judging a package.
   - Under D3 and D7, that call in a cut task's worktree rebuilds and commits to that task's weft only, so the parent never gets the rebuilt map. That fits the direction, but one line in D7 would stop the plan writer from guessing.

**Goal points of the entry**
- Not in the warp, in the weft: met (D1).
- Extra branch and junction, per code branch or per pair, created and forked, merge: not followed. D1 departs from it and raises it in Open risks (finding 1).
- A task builds and keeps its own maps for its own gate: met (D1, D7).
- At landing, the parent's upkeep carries its maps forward: met (D3).
- A new weft starts from the parent's maps, and the design checks inheritance and git cost per weft: met (D2, D6).
- Size, churn and format: met (D6).
- Machines and the stamp: met (D5).
- Who commits, and two machines' passes: met (D4, D5).
- `test_map_upkeep` default: met (D8).
- Idea of several tasks via raddle: questioned in Open risks (finding 2).

**Entries, peers and parent commits checked**
- `testmap-upkeep-cost-and-guide` (T2): conflict (findings 3 and 4).
- `test-suite-trim` (P6): none.
- `redundancy-sweep-existing` (E4): no conflict, one clarity gap (finding 5).
- `selector-non-body-miss` (E3): none. The record leaves the selector's miss rules out of scope.
- `lym loom peers`: no peers and no parent commits listed.
- Commits `36621f0e` and `79f0eb4a` (in the record's context): D8 reverses the default `36621f0e` set. The entry asks for exactly that decision, so no conflict.

**Decision notes**
- No board note titled `Decision for testmap-checked-in:` is on the board list. There were none to read and none I could not reach.

**Tool calls**
- `lym board get` for `redundancy-sweep-existing` and `selector-non-body-miss`: both touch the maps, and my context held only their titles from the board list.
