# Fixer report — plan review round 3

The one finding is fixed.
`lyx loom validate-plan` reports `ok: true` after the edits, with informational `create-new-unit` findings only.

## Changed

- **F1 (BLOCKING)**, a failing idle probe read as proof an injection landed:
  - `08-orch-cycle-machine.md`: in the confirmation rule for `handoff-requested` and `resuming`, only a turn end past the entry offset confirms an injection.
    A failing `SessionIdle` probe now means "keep waiting, flag unchanged", with the reason stated: the probe also fails on dialogs, drafts and unframable captures.
    A real turn in progress confirms through its own turn end.
    A passing probe with no turn end triggers the re-send.
  - `08-orch-cycle-machine.md`: a restarted watcher that loads a non-idle phase for the same strand first saves `PhaseInjected` false.
    The landed check then keeps running on later ticks instead of stopping after one probe that happened to fail.
  - `06-orch-state.md`: `PhaseInjected` names a turn end past `PhaseEventsOffset` as the landed evidence, and records the restart reset.
  - Card 8's test list gains four cases:
    - a resume `Send` error after a `clearing` timeout, with the probe failing and then passing: no re-send while it fails, then exactly one;
    - a resume `Send` error during a running turn that then ends: no re-send;
    - a handoff `Send` error behind a failing probe that later passes: exactly one re-send of the same `PendingHandoff`;
    - a restart in `resuming` with `PhaseInjected` persisted true, a failing probe, then a passing one: exactly one re-send.

## Deferred

None.
