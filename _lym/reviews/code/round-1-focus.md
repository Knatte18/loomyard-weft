---
round: 1
focus:
  - "internal/loomcli/rownotice.go `tell` and `verdictCount`: settle whether the visit key (verdicts in history, `1 + Repeats` each) is the same across a yield or pause and differs on every real re-entry of the same row, including a loop that re-reaches the row; check what `HistoryEntry.Repeats` counts. The mutation survivors at the `told == visit` line and the `1 +` constant name two checks the tests may lack."
  - "internal/loomcli/arm.go `newShed` and `watchRows`: settle whether every path that builds the shed for a run's steps goes through the wrap, whether an unknown `notify_parent_on` row refuses the steps with the recipe's rows named, and whether wrapping `shed.Producers[i].Producer` leaves each wrapped row's other behaviour (outcome, output pointer, error) unchanged."
  - "Part 1 coverage across the diff: the board entry asks to close every remaining path where a run agent addresses the operator. Settle whether the changed stencils (`parent-directive-operator-ban.md`, `bouncer-fragment-decision.md`, `burler-step-fix.md`) and `internal/gatecli/cardfixer.go` leave any agent or stencil that still tells the operator or is silent, and whether `SetNotifier` in `newStrandRig` sends the hold notice to the right parent with the right program name."
  - "Part 2 shape: the board entry leaves the verb-or-key choice to Discussion and names `notify_parent_on` as one option. Settle whether the key's wiring is complete: `Config`, `ConfigSpec` category, `template.yaml`, run_settings openness, and the docs the repository's rules name for a new config key (`docs/overview.md`, package `doc.go` files, `orch-role.md`)."
  - "Failure paths in `tell`: the visit is recorded before the mail goes out, so a failed post is never retried. Settle whether that, and the warn-only handling of a failed status or record read, matches what the `rowNoticeProducer` doc comments and `orch-role.md` promise."
  - "Test economy in `internal/loomcli/rownotice_test.go`: check each assertion against the behaviour its name states, and whether `shedrun.RowNoticeRecord` and the new `internal/shedrun` helpers have coverage the reach report cannot see."
---

Seed focus from the board entry (the decision record holds only `## Goal`, no amendments) and the diff stat.
