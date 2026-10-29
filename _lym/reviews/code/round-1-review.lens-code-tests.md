```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, listing for each behaviour the diff changes the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. Stop block JSON has `decision` `block`, the fixed pointer as `reason`, the mail as `systemMessage` and as `hookSpecificOutput.additionalContext` with `hookEventName` `Stop`, and the mail in no other field.
   Pinned by `TestHookContext` row `StopBlock` (`internal/shuttleengine/claudeengine/settings_test.go`). It compares the exact JSON string against `stopBlockReason`, so any extra or missing field, or the mail in the reason, fails it.
2. The tool-boundary block is unchanged, with `reason` and `systemMessage` both the text and no `additionalContext`.
   Pinned by `TestHookContext` row `ToolBoundaryBlock`, which is also an exact JSON match. It has the same literal as the old `BoundaryBlock` row.
3. `hook-read` passes the event its input carried.
   Pinned by `TestCreelCLI` in `internal/creelcli/cli_test.go`. The Stop subtests expect `hookBlock(..., HookEventStop, ...)`, and the PostToolUse subtests expect `HookEventToolBoundary`. The expected output is built from the real engine, so passing the wrong event fails them.
4. The seam signature takes the event, and every implementer and caller is updated.
   Pinned by compilation. Grep shows one implementer (`claudeengine`) and the callers `hookread.go`, `cli_test.go`, `settings_test.go` and `shuttleengines_test.go`. `TestToolBoundaryRenderer` pins that the registry's renderer is the claude engine, using the Stop event.
5. HookWake: the blocked-chain and Stop-cap rows settle each trigger on the one probe it causes.
   Pinned by the rows `MailArrivingInAWokenTurnsBlockedChainRingsNothingUntilANewTurnEnd` and `ARingAfterTheStopCapStartsAFreshCount`. Their determinism is a property of the test itself, so no separate test can pin it.
6. The other HookWake rows that post several messages in one trigger are also covered, **with a finding**: two rows still use the pattern the `settle` comment now says to avoid.
   - `deliver_test.go:1980-1983` posts two `bigBody` messages inside `settle`, then registers a waiter inside another `settle`.
   - `deliver_test.go:2064-2068` posts three messages to three different strands inside one `settle`.

**Findings**

- **[MEDIUM] consistency** — `internal/creelengine/deliver_test.go:1980` and `:2064`.
  - The `settle` comment now says a row that posts several messages at once posts them before the scheduler starts and settles the start. Two rows still post several messages inside one `settle` trigger, which is the pattern the commit says buffers a wake and leaves a trailing probe.
  - The row at 1980 posts two messages to alpha, so it can leave the same trailing probe that its following `settle` at 1985 would wait on.
  - The row at 2064 posts to three different strands. That may wake alpha's worker only once, so I could not confirm it is affected.
  - Fix: either move these posts before `e.start` as in the other two rows, or state in the comment why they are safe. The record's D4 only requires the blocked-chain and Stop-cap rows, so this is not a missed acceptance criterion, but the 1980 row is the same defect class.
  - Evidence: `deliver_test.go:1688-1696` (the comment and `settle`) and `:1980-1985`.

No test is missing for any other behaviour. The docs changes (`doc.go` in both packages) are prose and exempt.

**Tool calls (2 of 6)**
1. `awk` and `grep` over `internal/creelengine/deliver_test.go` for multi-post `settle` triggers, since the manifest only showed the two changed rows.
