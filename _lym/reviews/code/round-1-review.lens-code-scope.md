```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which traces each change in the diff to the board entry's goal points.

**Decisions** (the decision record holds only the board entry under `## Goal` and no `Added after Discussion` entries):
- G1: run `TestDiscussedPayload` repeatedly with the race detector.
- G2: find the unsynchronized map.
- G3: fix it.

**Per change:**
- C1: `internal/boardengine/store.go`. `Store.UpsertTask` (`fields = maps.Clone(fields)`), `Store.UpsertTasksBatch` (clones each element into `payloads`) and `Store.MergeTasks` (clone moved to the top) now clone the payload before the checks. This traces to G2 and G3. The unsynchronized map is the caller's payload map. The check helpers delete keys from it: `delete(fields, "discussed")` at `decision.go:184`, with the same in `front.go:31,37`, `owners.go:131` and `project.go:64`. `TestDiscussedPayload` hands one `tt.fields` map to parallel subtests (`decision_test.go:247-258`), so those deletes are concurrent writes.
- C2: the doc comments on the three store methods (`store.go:670-673`, `1069-1071`, `1259`). These are forced by C1: the "never changed" contract and the reworded owner_machine line ("dropped as a no-op" in place of "dropped from fields") restate what C1 makes true. They trace to G3.
- C3: `internal/boardengine/decision_test.go`. It adds the `maps` import, `wantFields := maps.Clone(tt.fields)` taken before each subtest, the unchanged-payload assertion, and a line in the doc comment. This is the regression check for C1's fix and traces to G3.

**Per decision:**
- G1: no change builds it, and none can. It is a step in the investigation, not something a diff carries. I raise no scope finding: the rubric's scope test is about changes the diff makes or omits, and the change description claims no race-detector run.
- G2: built by C1. The commit message and summary name the shared map and the deletes.
- G3: built by C1, C2 and C3.

**Unrequested scope:** none. No hunk falls outside G2 or G3 or the edits they force, and no `Pattern-Departure:` trailer is present or needed.

**Findings:** none.

Tool calls: none. Everything I needed was already in the inherited context.
