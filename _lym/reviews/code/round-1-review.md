---
verdict: APPROVED
---

The commit fixes the race the board entry names.
The four payload checks delete a key that equals the stored value: `discussed` (decision.go:184), `front` and `on_land` (front.go:31, front.go:37), `owner_machine` (owners.go:131) and `beam` (project.go:64).
They deleted that key from the caller's own map, and `TestDiscussedPayload` hands one row's map to its parallel Upsert, Batch and Merge subtests.
`Store.UpsertTask`, `Store.UpsertTasksBatch` and `Store.MergeTasks` now clone each payload before the checks, so the subtests only read the shared map.

Handler checks:

- Against the fasit: G2 and G3, find the map and fix it, are built.
  G1 is an investigation step, and the handler drove it here: `lym gate test ./internal/boardengine -- -run TestDiscussedPayload -race -count=20` passed 280 of 280 runs, the test's subtests twenty times over, with exit 0.
- MergeTasks: the final `s.UpsertTask(upsert)` gets the same stripped map plus `issues`, `created` and `project` that the base built, so the merge result is unchanged.
- Shallow clone: no path writes into a nested value, since `NewTask` and `ApplyPatch` only marshal the values.
  A nil payload and a nil batch element behave as on the base.
- Callers and Board wrappers only read the map, and none relied on a dropped or added key.
- The test's expected value is cloned from the literal in the loop body, before the paused parallel subtests run.
  `StoredValuePasses` fails the assertion on the base code on every path.
- The `Root-Cause:` trailer names the in-place deletion that the diff removes.
  The summary's claims, including the failure on the old code, match the diff.
  The two-way door holds: no record format, config meaning or outward push changes.
- No PATTERN changes are declared, and the test keeps its tier and `t.Parallel`.

## Rejected

- code-tests LOW (decision_test.go:244, only the `discussed` delete is driven): each method clones once at its top, ahead of every check, so the `StoredValuePasses` row on each path already pins the behaviour the diff adds. The regression it describes, a check moved ahead of the clone, is speculative, and a row per helper would duplicate that coverage.
