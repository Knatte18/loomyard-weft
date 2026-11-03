---
round: 1
focus:
  - "internal/shuttleengine/wait.go handleGatedBoundary and run.go noBoxReprompts: the field comment says it is cleared wherever unsentReprompt is. Grep every site that sets unsentReprompt to false or true and settle whether each one resets noBoxReprompts, so no stale count survives a new arrival, a new boundary or a writer turn that starts meanwhile."
  - "wait.go bound path: a held send that is not NoInputBox resets the count, so settle whether a mix of no-input-box holds and other holds can defeat the bound the board entry asks for, and whether finalize(OutcomeDone) is the right escalation next to the other terminal paths in the same function, given the entry says the re-prompt escalates."
  - "send.go dismissPicker and busyReading: settle whether the Escape goes out only behind the operator-activity and running-turn guards, whether picker.tried is set before the key so one send never plays it twice, and whether the recursive busyReading after the key can loop or skip the stray-clear path."
  - "send.go and conditionedsend.go: NoInputBox is derived from reason == readinessReasonNoInputBox. Settle that the constant is the reason judgeReadiness returns for a pane with no input box and nothing else, and that the dismissPicker activity-hold reason (which appends the picker name) correctly leaves NoInputBox unset or set."
  - "claudeengine/session.go DismissiblePicker: settle whether matching on normalizeCapture needles with no input box is correct against the pane fixture and the board entry's wording ('We recommend compacting earlier', 'esc not now'), and whether the fixture testdata/pane-autocompact-picker.txt matches what the board entry reports from Claude Code."
  - "Scope against the board entry: the entry asks for a dismissal when readiness holds a pane with no input box, and a bounded re-prompt. Settle whether dismissing in every verified send's idle wait, not only the gate re-prompt, is licensed, and whether the new SessionBusyError.NoInputBox field and DismissiblePickerReader interface go beyond it."
  - "Docs: the diff edits shuttleengine/doc.go, claudeengine/doc.go and wait.go package comments. Settle whether the repository's constraints document names other docs a shuttle behavior change must update, and whether those land in this diff."
  - "Test economy: TestGate_RepromptOverPicker cases and TestDismissiblePicker rows against existing readiness, stray-clear and rewind-picker tests. Name any case an existing test already covers, and any added behavior left uncovered (the dismissal key-send failure path, the running-turn guard)."
  - "Commit message and landing summary, where present: check each claim (once per send, guards, bound count, attempts spent) against the diff."
---

Seeded from the board entry in the decision record's `## Goal` and the diff range in the prompt.
The record's other sections are empty, so the entry is the only measuring stick.
