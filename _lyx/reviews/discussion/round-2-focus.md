---
round: 2
focus:
  - "D7 adds a PATTERN rule that an interrupt key reaches an agent pane only through shuttle's guarded interrupt. `Runner.Inject` in `internal/shuttleengine/run.go` deliberately skips the ready-pane guard and can deliver any key, Escape included. Settle whether the decision record states what becomes of `Runner.Inject` for the interrupt key, what bounds it, and whether an Acceptance criterion pins that bound."
  - "D4 settles the pause engine only between the first Esc and the next read, while D5 defines `still_busy` as a turn still on screen after the second Esc and D1 reads the lagging session state. Settle whether one named reading, taken after a settle following the second Esc, decides `still_busy` versus `interrupted` consistently across D1, D4, D5 and Acceptance criterion 4."
  - "D4's orch-watcher retry runs through the watcher's existing check that settles an entry already owning the orch, and the orch is recorded before its first Esc. Settle whether the record orders the remembered first-Esc time ahead of that check, and states the bound for a watcher restarted during the settle."
  - "D6 and the pre-change-entry Acceptance criterion quote the guard's no-input-box reason. Settle whether the quoted text matches `readinessReasonNoInputBox` in `internal/shuttleengine/readiness.go`, or avoids quoting a literal."
---

Judged against the rubric's `Do not flag` list and the writer-stencil symmetry rule: each entry is a bound or consistency question, not a missing-enumeration or missing-alternatives complaint.
The LOW findings on orch refusal visibility and D1's drift claim stay in the ledger and are not promoted here.
