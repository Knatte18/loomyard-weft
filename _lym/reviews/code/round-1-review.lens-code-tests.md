```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that would fail if it broke.

**Behaviour list**

1. **`Store.UpsertTask` leaves the caller's map unchanged** (store.go:675, `fields = maps.Clone(fields)`).
   Pinned by `TestDiscussedPayload` `Upsert/StoredValuePasses` (decision_test.go:244 and 258-260).
   On the base code, `checkDiscussedPayload` deletes `discussed` from the caller's map (decision.go:184), so `reflect.DeepEqual(tt.fields, wantFields)` fails.
   The refusal rows (`FalseOnFlaggedEntryRefused`, `TrueOnNewEntryRefused`, `NonBoolRefused`) return before any delete, so they pass on either code. They only pin the no-side-effect-on-reject case.
2. **`Store.UpsertTasksBatch` leaves the caller's maps unchanged** (store.go:1073-1077).
   Pinned by `Batch/StoredValuePasses`, through the same `checkDiscussedPayload` delete at store.go:1091 on the base code.
3. **`Store.MergeTasks` leaves the caller's map unchanged, clone moved above the checks** (store.go:1260).
   Pinned by `Merge/StoredValuePasses`. On the base code, `checkDiscussedPayload(s, upsert)` deleted `discussed` from the caller's map before the clone that the diff removed at the old store.go:1314.
   The merge result itself is unchanged: `issues`, `created` and `project` are still written to the clone that feeds `s.UpsertTask(upsert)`.
   `TestMergeTasks` pins the project carry (store_test.go:920, :937) and `TestMergeTasksIssueCarry` (store_test.go:2297) pins the issue carry, so a broken move would fail those.
4. **Expected value taken independently of the writes.**
   `wantFields := maps.Clone(tt.fields)` runs in the loop body before `t.Run` (decision_test.go:249). The parallel subtests pause until the parent's loop returns, so every snapshot is taken before any write.
   The expected value is a copy of the literal input, not output of the code under test, so the test is not tautological.
5. **Nil payload and nil batch element.**
   `maps.Clone(nil)` returns nil, so behaviour matches the base code. Nothing changed here, so no test is required.
6. **Changed doc comments on the three methods.** Prose, exempt.

**Findings**

- **LOW, design**, `internal/boardengine/decision_test.go:244`: the unchanged-payload assertion only reaches the `discussed` delete.
  - The other deleting checks are never driven with a value equal to the stored one: `owner_machine` (owners.go:131), `beam` (project.go:64), `front` and `on_land` (front.go:31, :37).
  - One clone per method covers all of them, so a single row per path does pin each clone.
  - A regression that re-introduced in-place deletion in one of those helpers, say by moving it before the clone, would not fail any row.
  - This is borderline. The clone sits at the top of each method, so the current rows suffice for the diff as written. Raise it only if the handler judges per-helper coverage necessary.

No other gap: every behaviour the diff adds has a row that fails on the base code.

Tool calls:
1. Grepped `internal/boardengine/*_test.go` for `func Test.*Merge`, to find the tests that pin merge results. The manifest held only `TestDiscussedPayload`.
2. Grepped `store_test.go` in those tests' ranges for project and created assertions, to confirm the moved clone's carry behaviour is pinned.
