# Merge-review: parent lock, Merge-In row and batten lock handling

All five batches recorded `done`, each on its first fork with no recovery.

## Batches

- **01-parentlock-store** (head `2a4193fea`):
  the `internal/parentlock` package (lock record, ticket queue, release and hand-off, force releases, queue wait, wait record, idle rule), shedengine's `on_reroute` edge with its cap and halting Stuck, shedbuild's `on_reroute` key, the loop hooks in shedverbs, and shuttleengine's agent-wait reading.
  Deviations: `internal/shedengine/goto_test.go`, `internal/shedverbs/loop_test.go`.
- **07-fabric-parent-reads** (head `d0bee07d8`):
  parent tips, changed paths, the holds-work test and the pair fast-forward in fabricengine; the verify failure record naming its row; `parent_lock_idle_release_min` in landing.yaml; the merge record in landingshed; the lock and parent-tip seams and the shared parent guard.
  Deviation: `internal/landingshed/approval.go` (byte-level atomic write split out for reuse).
- **12-landing-merge-in-producer** (head `428f4dbac`):
  the Merge-In producer, a merge segment colour, BLOCKING-aware auto-accept with the merge review note, loom.yaml's merge keys, the recipe seams and the `loom-rubric-merge-review` stencil.
  Deviations: `publish.go`, `stuck.go` (shared Stuck helpers), `reedengine/config_test.go`, `shedrecipe/entries_bouncer_test.go`, `shedrecipe/entries_burlerhalves_test.go`, `loomcli/wiring_test.go`, `test-budget.yaml`.
  The fork reported that card 15's stated zero-value behaviour for absent keys contradicts `LoadConfig`, which fills them from the template; it kept the code's behaviour.
- **18-merge-in-row** (head `4c20273ed`):
  Merge-In and the Merge-Review perch in the recipes, Publish no longer merges in, the `skip_merge_review` switch, and Publish, PR-Gate and Finalize holding the lock and rerouting on a moved parent.
  Deviations include `landingshed/stuck.go` (reroute helpers), several test files, `testkit/envkit/envkit.go`, `loomcli/start.go`, `loomcli/doc.go`, `darnengine/template.yaml`, `README.md` and `docs/overview.md`.
  The fork wrote a friction note at `_lyx/loom/friction/18-merge-in-row.md` about a card gate line naming a non-directory package path.
- **21-loop-lock-hooks** (head `22174be3d`):
  the loop's lock release and hand-off hooks, the lock wait read as waiting in `lyx loom status`, batten's retry following the failing row, batten's PR-Gate lock backstop and resume on a moved base, `lyx batten lock list` and `lock release`, and the orch stencil's PR-Gate fork.
  Deviations: `loomcli/driverprompt.go`, `loomcli/cli_test.go`, `battenshed/notice.go`.
  A follow-up commit fixed a spurious Warn from the wake hook that broke `lyx loom status` JSON output.

The plan-level verify gate runs after this turn ends.
