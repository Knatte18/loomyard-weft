---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: internal/shortcutcli/shortcuts.yaml:11
    summary: the p and r shortcuts land on an auto-pick that the board reserves for the operator and that the decision record leaves for a parent review this run never had
    fix: the operator confirms or withdraws Decision 12; on a confirm the run passes PR-Gate as it stands, and on a withdraw the two lines in internal/shortcutcli/shortcuts.yaml, their pins in cmd/lyx/shortcuts_test.go and internal/shortcutcli/shortcut_test.go, and the p and r mentions in internal/shortcutcli/doc.go and docs/overview.md are removed
---

### [BLOCKING] The p and r shortcuts need the operator's call before landing

**Location:** `internal/shortcutcli/shortcuts.yaml:11` (commit `e581d895c`), Decision 12 of `_lyx/discussion/decision-record.md`.

**Issue:** The board entry's "Pause and resume shortcuts" section says "Whether to add them at all, and which names, is the operator's call", and names `p` and `r` only as proposals for the operator to confirm.
Decision 12 records the pair as "an auto-pick of the board's proposed names, for parent review to confirm", and lists it again under Auto-mode assumptions.
No confirmation exists: the run's history in `_lyx/shed/hibernate-trigger/status.json` holds no Parent-Review row, and the one parent-review alignment attempt closed on a prompt-composition error (`unfilled top-level marker(s): pr_gate_measure`) with no verdict.
An approve here lands the task with no orch turn, so the shortcuts would ship on a choice the board gives to the operator and nobody has made.
The rest of the change matches the decision record.

## Checked against the goal and the decisions

- Decisions 1 to 9: `internal/hibernateengine` and `internal/hibernatecli` carry the `request`, `cancel`, `status` and hidden `watch` verbs. The three triggers, the caller and platform refusals, the locked request file under `lyxdirs.HibernateDirName`, the daemon identity test, the ten-second grace, the 120 five-second re-reads and the five-minute wake window are all there. So are confirmation over `idle-done`, `idle-stalled`, `idle-fresh`, `asking` and `dead` with the land lock, `still_busy` and refused checks, and the orch notice.
- Decision 7 and the carried-over risk `lift-without-resume-lines`: `tenderengine.Lift` shares `liftEntries` with `Resume`, so a lift by identity gives the resume lines, which closes that risk.
- Decision 10: the `systemctl` sleep deny is in the standing deny table, with its notice and the `PATTERN-no-denied-recovery` closed list updated. The board does not name it, but the record takes it in scope and it serves the board's direction that no LLM session runs the hibernate.
- Decision 11: the orch role stencil routes a hibernate request through `lyx hibernate request`, and the verbs carry the operator audience annotation for the command index.
- Decision 13: `hibernate` is excluded from the sandbox suite with its reason.
- Docs: both packages' `doc.go`, `docs/overview.md`, the refusal spec, and `PATTERN-daemon-wakeups` for the new unbounded wait.
- The remaining open risks (second hub, operator typing, replaced binary, Claude-only deny, polkit, cold boot after a failed resume) are recorded bounds and do not contradict a decision.

## Overlap

- `overview-seats` (row 12, own row 22): both change `docs/overview.md`, at different places (module table lines and module list entries). This is text-only bookkeeping and not a design conflict. By row, this run lands first and `overview-seats` takes its changes at Merge-In.
- `perch-budget-arbiter` (row 9): no changed paths, no overlap.
- Parent commits: none listed.
- Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml` gains rows for `internal/hibernatecli` and `internal/hibernateengine` and raises `internal/shuttleengine/claudeengine` to 112.
