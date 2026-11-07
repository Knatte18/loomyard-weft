```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's changed code (F1, F2, F4) for correctness through the fix-code-correctness lens.

## Findings

**BLOCKING — `confirmPasted` (`internal/shuttleengine/send.go`, the `promptTurnStartedSince(..., text)` match) can never fire for the real Claude engine.**
- The change makes `ErrCommandTakenAsPrompt` depend on a turn start whose `Instruction` equals the trimmed pasted text.
- `typedInstruction` (`internal/shuttleengine/claudeengine/signals.go:209-220`) returns `""` for any prompt whose first word starts with `/` and has no further `/`. Its comment says "A task notification and a slash command are not instructions to the agent." `claudeengine/doc.go:174` says the same.
- Every command that goes through this path is such a prompt: `/clear`, `/compact <focus>`, `/reload-plugins`, `/color x`. A provider that takes `" /clear"` as a prompt records a turn start with `Instruction == ""`.
- `promptTurnStartedSince` therefore never matches, and the taken-as-prompt turn falls through. The box reads empty, and for a send with no `landingStart` (reload-plugins, color) `confirmPasted` returns nil. The command is reported as sent when it never ran. This is the failure the original change set out to catch.
- The new test rows hide this. `slashPasteEngine.ParseSessionSignals` is extended in `internal/shuttleengine/session_test.go` to give `START:<text>` an instruction equal to the command text, which the real parser never produces. The row "a turn the paste started with the command's text fails the command as taken for a prompt" passes against the fake only.
- F2's premise was "the taken-as-prompt turn is exactly one whose instruction equals the trimmed pasted text". That is false for slash commands.
- A sound alternative needs a signal the real engine emits for a slash-command turn start. For example, accept a turn start with an empty `Instruction`, or add a provider-neutral signal field. The new test must then use the real parser's shape.

**LOW — no `claudeengine`-level test covers the F2 match.** The only coverage is the fake engine, which has the shape the real parser does not produce (see above).

## Traced and found correct

- `clearBoxBeforeCommand`:
  - Box text → `*SessionBusyError` with `Draft: true`, no key, no paste. It maps to `ErrBusy` in `mapDeliveryError`, so creel requeues it.
  - Unreadable box → proceeds to the clear.
  - Operator active → busy error, no key.
  - `SendKey` failure → wrapped error, no paste.
- The new rows fail with their guard removed:
  - The draft-at-clear row fails because the paste goes out.
  - The `activeAtClear` row fails because a key goes out. It relies on exactly one `OperatorActive` call before the clear, and the row passes today.
  - The failed-key row fails because a nil error comes back.
  - The clearless row fails because the engine would not be selected.
- The F3 row passes `ErrCommandTakenAsPrompt` through `mapDeliveryError`'s default branch, which returns the error unchanged.

## Tool calls

1. `grep -rn "Instruction" internal/shuttleengine/claudeengine/*.go`: to see how the real engine fills `Instruction`; my context held only the fake parsers.
2. `sed -n 195,235p internal/shuttleengine/claudeengine/signals.go`: to read `typedInstruction`'s slash-command rule; the grep showed only its comment.
