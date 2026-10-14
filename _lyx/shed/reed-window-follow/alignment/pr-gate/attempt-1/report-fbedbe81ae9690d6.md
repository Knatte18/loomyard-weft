---
verdict: APPROVED
---

## Alignment at pr-gate: reed-window-follow

The diff of pull request 616 at `100f0f30b` against `main` is measured against board entry 501.
The decision record has no `Added after Discussion` entries, so the board body is the measure unamended.

### Against the board entry

The entry's Scope (operator, 2026-10-14) asks that reed, when a new agent strand spawns, move each attached client to the new strand's window, but only a client on the previous newest agent window or on the Driver window, and leave a client the operator moved elsewhere by hand.

- `newAgentWindowSelected` in `internal/reedengine/ownwindow.go` now also passes the viewer on from `relevantWindow` (the highest-index live agent window as listed before the new one opens) and from any window holding a live own-window pane whose strand role matches `agentname.RoleDriver`.
- A viewer on an older live agent window or the shell still stays, which covers the hand-moved client.
- Each attached client is covered by the single session-wide `select-window`, since every client of a tmux session shares its current window; `doc.go` states this.
- The unit table `TestNewAgentWindowSelected` and the tmux-tier subtests in `windoworder_integration_test.go` pin each case, and the focus-rule paragraph in `internal/reedengine/doc.go` is updated in the same commit.

The change leaves attach's own window selection, removal and reconcile switches, and the Driver itself (`orch-split`) alone, as the entry's Direction leaves them.
Nothing in the diff goes beyond the entry, and no point of it is left out.

### Open risk carried over

The record's one open risk is that replacing the viewed agent now lands the viewer on the replacement's window when the removal fell back to the newest live agent; the tmux replace subtest's expectation changes to match.
This follows from the entry's own rule: the viewer was following the newest agent window.
It names no parent correction, so it does not escalate.

### Overlaps

`lyx loom peers` lists no landed parent commits.
This run's `own_row` is 22.

- `hibernate-trigger` (row 14): both change `cmd/lyx/testdata/test-budget.yaml`, at different rows; a shared bookkeeping file only, not an overlap.
  This run lands first, and `hibernate-trigger` takes the change at Merge-In.
- `run-window-early` (row 17): it opens the run window as soon as the worktree exists, through `lyx reed attach`; it changes no file in `internal/reedengine`.
  The two are complementary rather than conflicting: an early attach lands the viewer on the overview, which already passes the viewer on, and this run then keeps it following later agents.
  No shared file and no conflicting decision; were it counted, this run lands first.
- `overview-seats` (row 9): its record concerns the loom checklist, seat names and the producer layer; it changes no reed focus rule.
  No overlap.
- `perch-budget-arbiter`, `twb-f3-ir`, `twb-f3-wr`, `twb-f3-pr`: no changed paths yet and no reed scope; no overlap.

### Superseded

No board entry is made obsolete; `orch-split` still owns dropping the LLM Driver.
