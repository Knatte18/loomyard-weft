```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's changed code through the correctness lens, running a concrete input through each changed branch and asking whether each added test fails with the defect in place.

I made no further tool calls and found no wrong result or broken caller. These are the test findings:

1. **LOW, `attach_test.go` row `{"terminal_api_error", "api-error", true, false}` (`TestAttach_OutcomeDisposition`, F12).**
   - **Summary:** The row probably passes with the `isTerminalOutcome` api-error case deleted, so it may not pin F12.
   - **Evidence:**
     - The fixture's strand is tracked and live, and the existing `unrecognized_outcome` row (`"some-future-value"`) already expects `found=false` with an identical setup.
     - An unrecognized value is not terminal, so the table only separates terminal from unrecognized values for an untracked or dead strand.
     - `TestAttach_UntrackedTerminalRecord_RespawnEligibleRegardlessOfAge` (`attach_test.go:456`) is the likelier host: with a fresh dir mtime, a terminal outcome is respawn-eligible and an unrecognized one is `verdictError`.

2. **LOW, `apierrorretry_test.go` row "earlier API errors in the events file count against the re-sends" (F11).**
   - **Summary:** The row does not pin the attach resume-from-prompt-offset behavior.
   - **Evidence:** The row runs with `PromptOffset` 0, so a cursor that started at 0 passes it as well. It only shows that errors already in the file count. A row with a non-zero `PromptOffset` after an earlier API error would pin the cursor start.

3. **LOW, `apierrorretry_test.go` row "a send that fails is tried again on a later tick" (F11, F8).**
   - **Summary:** The row does not show the retry happening on a later tick, and it asserts nothing about the new `failures.Fail`/`Recover` logging.
   - **Evidence:**
     - `reactingReed.SendText` returns an error before it records anything.
     - The row asserts only one delivered send within the gap bound.
     - If `sendVerified` retries the failed `SendText` inside one `sendWithin` call, the row passes without ever reaching the `err != nil` branch.
     - Nothing checks the single-warn-per-episode behavior F8 asked for.

Verified fine:
- **`retryAPIError`:** the hoisted `outcome` keeps the old semantics.
- **`endOnAPIError`:** the `cmp.Or` fallback applies to the `Result` and the log.
- **Rows that fail with their defect in place:** the unknown-text row, the later-start-and-end clear row, the covered row (the run ends before the lift step if the `run.covered` check is removed), the `unfinished` carry-over row, `TestLoopAgreesWithState_APIError`, and the tackler `wantQuestion` assertion.
- **Tripwire:** dropping the `retryAPIError [OutcomeAPIError]` audit entry matches the code, since the return no longer holds the literal `OutcomeAPIError`.

Calls made: none. My context already held every changed file and the review.
