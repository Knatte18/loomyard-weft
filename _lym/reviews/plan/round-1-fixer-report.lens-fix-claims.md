```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: checking that every claim the fix added or rewrote holds against the code at this worktree's HEAD.

One LOW finding:

- **LOW.** Card 13, the rewritten `TestHookReadExit` sentence: "`TestCreelCLI` covers the rest, a Stop that writes a block, a Stop with no mail, a tool-boundary call and a refused input."
  - **Summary:** No `TestCreelCLI` row reaches a parsed tool-boundary event that fails after the parse, for example a daemon refusal of the claim. The old `TestHookReadExit` row, "a tool-boundary call … 1 on an error", pinned that case and the fix cut it, so the claimed cover does not hold for it.
  - **Evidence:** The only existing hook-read refusal row is the unknown engine (`internal/creelcli/cli_test.go:617`). It fails before `HookInput` parses anything, so `hookReadExit` gets an empty event, not `HookEventToolBoundary`. The planned "input of another event" row also reaches it with an empty event. Card 13's new CLI rows include a non-strand caller only for hook-wait, not for hook-read.
  - **Fix options:** keep the tool-boundary-with-error row in `TestHookReadExit`, or add a `TestCreelCLI` hook-read row from a non-strand caller on a PostToolUse input that exits 1 with the refusal on stderr.

Every other claim held against the code:
- **Card 15, the `SetStrandsError` re-read row:** `Delivery.Strands` returns `strandsErr` (`creelfake` line 177). `post` re-reads through `readStrands` (`verbs.go:63`), and an interrupt function that sets the error after `strandTarget` makes only that re-read fail.
- **Card 15, the `SetPausedError` row:** `SetPausedError` makes `ClaimPause` fail (`creelfake` line 259). `interruptTarget` answers that with `unreadablePauseRefusal` (`verbs.go:548-550`).
- **Card 13:** `hookReadCmd` has a `RunE` (`hookread.go:38`).
- **Card 9:** `TestSmokeCreelClaudeStrand` asserts that a posted message's carried line is typed at idle (`smoke_llm_test.go` header), so "unchanged, still typed" holds.
- **Card 17:** every sandbox strand comes from `lym reed add`, the orch-role one included (`--role orch`, `SANDBOX-CREEL-SUITE.md:130`). Shuttle scenarios live in another suite (line 343).
- **Card 7:** `InterruptGuard` has exactly `RunningTurn` and `InterruptSettle`, and `readinessEngine` supplies the input box. The `guardedReadinessEngine` claims hold.
- **Card 10:** `Read` refuses EXTERNAL, so the EXTERNAL way forward through `post --wait-reply` is sound.

Tool calls:
1. Grepped `internal/testkit/creelfake` for `strandsErr` and `pausedErr`, to see which fake methods the scripted errors fail; not in context.
2. Grepped `interruptTarget` and `post` in `internal/creelengine` for the strand and pause reads, to place `SetStrandsError` and `SetPausedError` against the call order; not in context.
3. Grepped `hookReadCmd`'s `RunE` and the `TestSmokeCreelClaudeStrand` header, to check the card 13 name and the card 9 "unchanged" claim; not in context.
4. Grepped `SANDBOX-CREEL-SUITE.md` for how strands are added, to check card 17's claim that every strand comes from `lym reed add`; the orch-role line was not in context.
