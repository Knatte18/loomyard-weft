---
round: 1
focus:
  - "internal/driftengine/capture.go, steprun.go, spawn.go: does a capture that exits 0 with no file under `.lym/drift/baseline` fail non-zero with a stated cause, and does that cause reach the Drift-Await halt reason rather than only the log path?"
  - "internal/driftcli/cli.go: does the CLI path surface the capture's stated cause and exit non-zero on a missing baseline, on every route that can report a capture as done?"
  - "internal/driftengine/dirbase.go and the matching change in compare.go: is the worktree-root base applied the same way for capture and compare, and does a nonexistent dir fail by name before spawn?"
  - "internal/hubreconcile/reinterpret.go and walk.go: does reconcile rewrite or flag, with a way forward, a relative Drift `dir` on an anchored hub, and only that case? Check an already-migrated file, an absolute dir, a hub without an anchor, and a second reconcile run for idempotence and for no repeated notice."
  - "internal/configengine/config.go and internal/configreg: is the reinterpretation hook generic enough to carry a future value change, and does it leave existing registrations unchanged?"
  - "internal/orchcli/reconcilenotifier.go and pushnotifier.go: is the hub's orch told when a reconcile changes or flags a file, and is it silent when nothing changed? Check failure handling when the notice cannot be delivered."
  - "Scope against the Goal: is there a change the Goal does not license, e.g. in cmd/lym/main.go or the PATTERN.md line? Is the Goal's third Part 1 item, finding why the capture wrote no file and why the first re-capture reproduced it, answered by the diff or by the change description?"
  - "Test economy: capture_test.go, dirbase_test.go, reinterpret_integration_test.go and ensure_integration_test.go for duplicated coverage, per-helper tests the public surface already covers, and behavior the diff adds without a test."
  - "Commit messages and `_lym/landing/summary.md`: any claim the diff does not bear out; any `Pattern-Departure:` trailer and whether it names a real entry that forbids the way taken."
---

Seed focus from the board entry (the decision record holds only `## Goal`, so it is the sole measuring stick) and the diff stat of the range.
