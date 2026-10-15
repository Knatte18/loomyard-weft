---
verdict: APPROVED
---

## Alignment at PR-Gate: refusal-route-config-empty

Judged: pull request 637 at `9e97026d5462d4233b84da511155c47ecfd6d12a` against `main`, the decision record, the plan (cards 01 to 06) and board entry `refusal-route-config-empty` (issues #625 and #632).

### Goal coverage

- Part 1, the label: begin-batch's blocking-findings refusal now returns `*PlanConflictError` wrapping `ErrPlanConflict`, and its envelope carries `{"plan_conflict": "<cause>"}` with `plan_defect` or `tree_moved` from the begun set (D1).
  `record-batch` and `recover-batch` keep `plan_drifted`, as the record's Out list says.
- Part 1, the route: begin-batch writes `.lyx/webster/plan-conflict.md` under the mutate lock after the fingerprint persist, removes it as the verb's first step after the abort check, and `websterengine.Run` removes it after taking run.lock (D2).
  Webster declares `on_reroute: Plan-Burler-Review`, and the adapter's stuck arm uses `ClaimConflictRoute` to reroute a new finding set and halt a repeated one, both reasons naming the hand-edit way forward (D3).
  `loomRerouteCapWayForward` is row-neutral, and the Plan-Burler-Review instructions name the record.
- Part 1, the third point ("consider" running the check at Plan-Bouncer or Batchifier): the record rules it out because the plan gates already run it, and the change adds no such check.
- The master stencil, its claims, the refusal-spec rows (one changed and three added) and the recipe header comment match D4 and D5.
- Part 2: `dispatch` and `menu` refuse without a terminal before any layer dispatch, hub-write guard, scaffold or editor, naming `--print` and `--set` (D6).
  `EditPath` treats an empty or whitespace-only result as `ErrAborted`, removing a scaffolded file or restoring the entry bytes (D7).
- The `doc.go` files of `websterengine`, `shedadapters`, `configcli` and `configengine` and the loom tail paragraph of `docs/overview.md` change in the same commits.

No change goes beyond the goal, and none breaks a recorded decision.

### Carried-over risks

The open risks in the decision record (a `plan_defect` label on an operator commit that lands before the first batch, the rerouted round's inherited ledger and round number, the routed file written before shed's cap decision, a record that survives a failed removal, a record left by a halted Plan-Review, and the pane numbering shared with `run-pane-timing`) are accepted trade-offs that the record states.
None is a scope deviation.
The Plan-Review carry-over `card6-menu-hub-write-guard-subtest-unspecified` is a test-detail point.
`menu` refuses before it reads anything, so no hub-write guard can run before the refusal.

### Overlaps

- `run-pane-timing` (row 12, Webster): changes the same files at other places: `internal/shedadapters/webster.go` (copying the Merriam gate spans into the history entry), `internal/shedadapters/doc.go`, `internal/shedadapters/webster_test.go`, `internal/shedadapters/burlerreview_test.go`, `internal/websterengine/runlevel.go`, `internal/websterengine/runlevel_test.go`, `internal/websterengine/doc.go`, `internal/loomrecipe/recipe_test.go` and `docs/overview.md`.
  The two designs change different behaviour.
  Its spans go on the Webster history entry, and this run's reroute is a verdict on the stuck arm.
  It reads `shedengine/reroute.go` only to add spans to reroute entries, and this run leaves the shed engine untouched.
  Neither decision record contradicts or duplicates the other.
  The pane rendering of Webster's backward reroute is named in this run's open risks.
  This is a text-only overlap.
  Landing order: this run (own row 22) is further along than `run-pane-timing` (row 12), so this run lands first and `run-pane-timing` takes these changes at Merge-In.
- `glyph-audit-tool` (row 7), `role-language-skills` (row 4) and `orch-context-overview` (row 5): no changed paths yet, and their decision records touch neither the begin-batch refusal, the Webster reroute nor the config editor path.
  No overlap.
- Parent commits: `lyx loom peers` lists none.

Shared bookkeeping: `docs/overview.md` gains one line in the loom recipe paragraph.

### Superseded entries

None.
`kickstart-per-card` depends on this run but is not made obsolete by it.
