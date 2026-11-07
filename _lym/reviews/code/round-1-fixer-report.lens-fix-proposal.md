```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F2–F7), looking for a case where the proposed change is wrong.

**Finding 1 — BLOCKING — `confirmPasted` / `promptTurnStartedSince` (internal/shuttleengine/send.go, F2)**
- **Summary:** The F2 change makes `ErrCommandTakenAsPrompt` unreachable with the real Claude engine, so a command taken as a prompt is no longer detected.
- **Evidence:**
  - `claudeengine/signals.go` `typedInstruction` returns `""` for a prompt whose first word starts with `/` and holds no further `/`. `claudeengine/doc.go:174` says the same: "The Instruction is empty for a slash command".
  - `/clear`, `/compact the plan`, `/reload-plugins` and `/color green` all match that rule, so their turn start carries `Instruction == ""`. A command pasted behind whitespace and taken as a prompt is trimmed first, so it matches too.
  - `promptTurnStartedSince(..., text)` compares `signal.Instruction == strings.TrimSpace(text)`, which is `"" == "/clear"` and never true.
  - The failed paste then falls through to the box check. The box reads empty, so `confirmPasted` returns nil. The command never ran, yet the send reports success.
  - Before F2, any turn start failed the send, which was correct for this case. The review's premise, "the taken-as-prompt turn is exactly one whose instruction equals the trimmed pasted text", is false for slash commands on this engine.
- **Why the tests miss it:** the fake `slashPasteEngine` parser in `session_test.go` now emits `START:<text>` with `Instruction` set to the command text. The real parser never does that.
- **Alternative that works:** match on the signal itself instead of its instruction, because a command starts no turn. For example, fail on any turn start past `sentAt` whose `Instruction` is `""` or equals the text. Or, with the real parser, treat a turn start past `sentAt` as taken-as-prompt only when it has no `SessionStart` of the landing source. The F2 concern (a turn start from the operator or a hook) is a typed instruction, which is non-empty and different from the text, so it would no longer fail the send.

**F3, F4, F5, F6, F7:** no findings. The F3 row is correct, because `default` returns the error unchanged. The F4 rows exercise real branches. The doc and template edits are accurate.

Tool calls made (2):
- `grep Instruction` in claudeengine and engine.go: to see how the real engine fills `SessionSignal.Instruction`, which my context lacked.
- `sed` on `claudeengine/signals.go` lines 200–230: to read `typedInstruction`, because the grep only showed its first lines.
