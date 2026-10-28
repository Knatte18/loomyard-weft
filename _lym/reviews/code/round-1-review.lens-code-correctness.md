```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff, covering each function or branch, the case I ran and whether it is correct.

**Per function or branch (case run → result)**

- `ReadAgentActivity` and `noteWaiterlessRun` (`activity.go:77-141`):
  - A running run with a dead pid is read, logs the waiterless Warn once and has its API-error and usage-limit parts zeroed (`activity.go:111-114`). Correct.
  - A live run, a non-running run and a missing record each reach `Recover`. Correct.
  - A pid-less running run (PID 0) is read and zeroed but never logged, because the log needs `PID > 0`. Correct under D3, which only names runs with a gone waiter.
  - `isAlive` is called twice per run (`noteWaiterlessRun` and `runIsLive`), so a pid that dies between the calls gives a live-looking log with a zeroed reading. The only effect is a one-poll skew, so it is not a finding.
- `readRunActivity` stamp (`activity.go:169-171`):
  - A missing stamp gives `os.Stat` an error and leaves the time alone. A newer or older stamp wins or loses on `After`. Correct.
- `recordSentText` (`carryover.go:165`):
  - A failed open or write is logged and the send goes on, as required. The record happens at the top of `typeVerified`, before typing. Correct.
  - `sendVerified` also reaches `typeVerified` for the prompt and skill-load sends, so the prompt line is in the file too. This is harmless, because the first turn start is skipped anyway.
- `sentTexts` (`carryover.go:180`):
  - `scanner.Buffer(nil, 16<<20)` grows from the default size, so it works. A malformed line is skipped. Correct.
  - A line over 16MB would stop the scan silently. That is negligible.
- `operatorInstructions` (`carryover.go:201`):
  - An offset past the end or a negative one is clamped. A missing events file or an engine without a parser gives none. The first turn start is skipped. A sent text and an empty instruction are dropped. Correct.
  - A deferred-prompt run that died before its prompt was typed has no prompt turn start. Its first operator instruction is then skipped as the "prompt" and lost. This is a narrow edge case, covered by finding F3.
- `roleKey`, `unfinished` and `carriedInstructions` (`carryover.go:230-290`):
  - `stopOutcome` (`stop.go:16-22`) sets "stopped" only when output files are missing, so a finished run stopped late reads done and carries nothing. This supports D6's bound.
  - An empty key, an unreadable root, a record that fails to load, an engine lookup failure and a missing events file each return none or only the record's own carried instructions. Correct.
  - Repeats are dropped, and order is carried first, then operator. Correct.
  - Newest is chosen by `CreatedAt` string compare, then by run-directory name. See F4.
- `start` ordering (`run.go:549-562`):
  - The carry-over is read before the sweep. `spec.Prompt` is a by-value copy, appended once, so the start retry loop does not double-append. `launch` records the carried instructions on every attempt. The notice goes out once after success. Correct.
  - A failed start sends no notice, but its attempts' records keep the carried instructions, so the next start carries them again. Correct.
  - The block is unbounded: repeated re-spawns grow the prompt with no cap, while the notice truncates at 200 runes. It is not a defect today.
- `typedInstruction` (`signals.go:464`):
  - A missing or empty prompt gives an empty instruction. A `<task-notification>` prompt gives none. Any prompt starting with `/` gives none. See F1.
- `activityStampHook` and `buildSettings` (`settings.go:329-423`):
  - `touch '<dir>/activity-stamp'; true` always exits 0, so PreCompact is never blocked. The path comes from the POSIX events path and is quoted. In a hub the creel mail hook is appended after the stamp. A hub-less run carries the stamp alone. `PostToolUse` is no longer `omitempty`, which is intended. Correct.
- `noticeCarriedInstructions` (`carryover.go:306-324`):
  - A nil notifier still logs the Warn once. A notifier error is only logged. Whitespace is collapsed and the text truncated by rune. Correct.

**Findings**

- **F1 — LOW, design** — `internal/shuttleengine/claudeengine/signals.go:466`
  - The slash-command rule is `HasPrefix(trimmed, "/")`, so an operator instruction that starts with an absolute path (for example "/tmp/x.go fails to build, fix it") is treated as a command and is not carried over.
  - Evidence: `typedInstruction` has no check that the first token is a command name; `/compact` is the only case the test pins.
  - Suggested change: treat a prompt as a slash command only when its first whitespace-delimited token has no further `/`.
  - The decision record only says "slash command", so this is a refinement.
- **F2 — LOW, design** — `internal/shuttleengine/activity.go:99-115`
  - A running record whose lym process died and whose agent is also gone is now read forever, because only the outcome gates it. Before D3 such a record was skipped.
  - The callers `loomcli/looparm.go:168-181` (`len(readings) > 0`) and `loomcli/decision.go:855` (`liveRunStrands`) now count it as an active or live run.
  - D3 accepts the idle-notice effect, but these two callers treat presence as "agent running", not only age. The affected-set lens should confirm they can tolerate a stale waiterless record.
- **F3 — LOW, design** — `internal/shuttleengine/carryover.go:218-221`
  - `operatorInstructions` assumes the first turn start past the prompt offset is the run's prompt. If the prompt was never delivered (a deferred-prompt run that died first), the first operator instruction is dropped instead.
  - The loss is bounded to that one instruction, so I rate it LOW.
  - Suggested change: skip the first turn start only when it has no operator text, or when it matches the recorded prompt line.
- **F4 — LOW, design** — `internal/shuttleengine/carryover.go:270`
  - Predecessor choice compares the `CreatedAt` strings, which are RFC3339 at one-second resolution, then the run-directory name. Two runs of one role created in the same second (start-retry attempts, for example) are ordered by the directory name.
  - If run IDs are random, the newest unfinished predecessor can be picked wrongly.
  - Suggested change: break the tie by directory modification time or a monotonic ID. I did not read `createRunDir`, so I cannot say whether IDs are ordered.

**Tool calls**
1. `grep` for `runOutcomeStopped`, writes to `Outcome`, and `typeVerified`, `sendVerified` and `pasteVerified` call sites: the manifest lacked `stop.go`'s `stopOutcome` semantics and the set of entry points to the sent-text record.
