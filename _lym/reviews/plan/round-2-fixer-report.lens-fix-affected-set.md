```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each set-based finding's search and check the fix covers every member.

Unchecked findings: none. F14, F15 and F16 were each checked.

- **F14** (the tests that build a `battenCLI` literal, call `wire`, then run `CreateWorktree`): all of them are covered.
  - My search found `beamfork_integration_test.go:134`, `recordleak_integration_test.go:122`, `wire_test.go:54/426/462`, `lifecycle_integration_test.go:105/239/485` and the `wire_seams_test.go` sites.
  - The fix makes `battenCLI.wire` fill `pickAccount` and states that every test that builds a literal and calls `wire` reaches the real pick. That covers each member, including the ones the review did not name (`recordleak`, `wire_test.go`).
  - The production builders (`cancelcmd.go:413`, `arm.go:519`) also go through `wire`, so none of them is left with a nil field.
- **F15** (the callers of `relaunchUnderAccount`): all of them are covered.
  - The plan has four call sites: card 14's `Wait` site and load-turn site, and card 16's start-path site and `retryAPIError` site. Each now passes `startPath` with the right value.
  - Card 16's start path states the same return rule.
  - No other card names the symbol.
- **F16** (the `doc.go` sentences cards 14 and 16 falsify): no false sentences remain uncovered.
  - The three paragraphs the review named (lines 161, 190 and 570–574) are rewritten.
  - The other hits stay true under the new rules:
    - Line 111–112 describes api-error as the load hold reaching its cap; a counted hold still does.
    - Line 581 describes the cap; uncounted turn ends never reach it.
    - Line 953 says a not-ready teardown reports `ErrNotStarted` from `StartGated`. That matches the start-path died and timeout ends of the fix.

no findings

Tool calls:
- One grep of `.wire(` callers in battencli tests, plus a grep of `doc.go` for usage-limit, `APIErrorRetries`, load-hold and `ErrNotStarted` sentences. My context had only the `battenCLI{` literal list and the three named `doc.go` paragraphs, not the full set of `wire` callers or the other `doc.go` sentences.
