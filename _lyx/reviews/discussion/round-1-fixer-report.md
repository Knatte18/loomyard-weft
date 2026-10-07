# Round 1 fixer report

Every finding is fixed in `decision-record.md`, and the F10 narrative moved to `support-log.md`.

## Changes

- F1: D4's Bound states the deadline is the caller's own `Spec.Timeout` from its own config (hours for loom and webster), that each Attach starts a fresh deadline, and that a no-notifier run, including a standalone `lyx shuttle run`, holds silently to it.
  The Auto-mode assumption and the Open risk are restated against those deadlines, not 30 minutes.
- F2: D2 counts only a list value as authoritative (a non-list value falls back to the transcript) and its Bound states the consequence of an under-reporting payload.
  The Open risk now matches D2, and two acceptance criteria cover the non-list and non-`running` cases.
- F3: D5 states the fixed notice prefix, the 200-character cut, newline collapse, and that the orch reads the quote as a report, not an instruction.
- F4: D6 decides the no-`AskingOffset` case (attach from the prompt offset, notify once) and the with-offset case (attach after it, no notice).
  D5 persists the notified offset before calling the notifier, so a crash loses one notice and never repeats it.
  Acceptance criteria follow.
- F5: D5's notice carries the outstanding tasks with kind and label, D9 refers to that, and a criterion covers the expired-shell hold.
- F6: D8 trims trailing whitespace before the ends-with check, and the `lyx shuttle send` criterion covers it.
- F7: Scope gains bullets for the trace line, the legacy asking attach and the AskUserQuestion steer text.
- F8: Acceptance criteria added for the D8 stencil and steer statements, the D9 webster note and warning, and the live scenarios' `background_tasks` check.
- F9: Constraints gain "lyx never kills an agent's background shell, and a fork never expires like a shell" and the no-Notification-hook limit; D3 and D9 state the same in their Decisions.
- F10: the SIGQUIT, hibernation, `continued-in` and goroutine-dump narrative moved from D1 and D3 into the support log's Interview entry 1.

## Deferred

None.
