# Card 3 — card-verify-rerun

**Create:**
- `internal/websterengine/cardverify.go`
- `internal/websterengine/cardverify_test.go`

**Intent:**
Add the in-process evidence check D5 puts behind every policy-only warning: re-run the recorded batch's cards' own `**Verify:**` commands on the worktree as it stands.

`cardverify.go` declares `DefaultCardVerifyTimeout` (10 minutes) and `rerunCardVerifies(cards []planparser.Card, worktree string, timeout time.Duration) []string`.
It runs each card's `Verify` through `verifyrun.Run` in `worktree`, each under its own `context.WithTimeout`, output discarded, in card order.
It returns one failure line per failing card, `card NN-<slug> verify <command> <how>`, where `<how>` is `exited <code>`, `failed to start: <err>`, or `timed out after <timeout>`.
A card with no `Verify` (`HasVerify` false or an empty command) passes.
A zero `timeout` means `DefaultCardVerifyTimeout`.
It returns no error: a verify that cannot start is a failed verify under D5, and the cards are already in memory, so no I/O happens before the first command.

`cardverify_test.go` (`//go:build integration`, since it spawns shells) covers: all cards passing (`true`, and a card with no verify) → no failures; one card exiting non-zero → exactly that card's failure line with its exit code; a command that sleeps past a short timeout → a `timed out` line; and a command in a nonexistent working directory → a `failed to start` or non-zero line, never a panic.
