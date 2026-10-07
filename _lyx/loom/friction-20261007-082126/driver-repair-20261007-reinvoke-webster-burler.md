# Driver repair: re-invoke after an interrupted step

- Failure: `lyx shed step bugfix-sessions` (trace 71e9b399643cdbd1, producer Webster-Burler, round 2) exited with SIGQUIT and no envelope.
  The orchestrator sent the SIGQUIT to capture a stack dump of a stall in shuttle's wait (known bug #411).
- Trace lines acted on: `shed: step` and `burler: round starting round=2` for trace 71e9b399643cdbd1; no `shed: step done`.
- Status read: `current_producer` Webster-Burler and `history_length` 30, both unchanged, `interrupt_policy: reinvoke`.
- Action: re-invoked `lyx shed step bugfix-sessions`.
- Outcome: trace 5d6bfb9b1cbda7e4 handed round 2 back for judgment (`outcome: stuck`, next Webster-Bouncer); `history_length` 31.
