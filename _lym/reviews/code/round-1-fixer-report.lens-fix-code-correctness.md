```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Task: run the code and test changes for F1–F4 through the correctness lens.

- **`checkLandingStartConfirmation` signature:** the new `sentText` parameter has two callers, `ClearSession` (`"/clear"`) and `CompactSession` (`"/compact the plan"`). Both are updated, and each value matches what `slashPasteEngine` pastes.
- **Row `{"",true},{"",false}`:**
  1. The first read after the paste is an empty, readable box, so `submitted` is set.
  2. Later reads are unreadable (`ok=false`), because the last answer in `inputBoxEngine.InputBoxText` sticks, so nothing is sent.
  3. The window closes and the function returns nil on the `submitted` branch.
  4. The window is 30s and backoff reaches 5s steps within about 11 reads. The read cap (`windowPollCap`) is 122 reads, so the window closes first and `wantWait` holds.
- **Row `{"",true},{sentText,true}`:**
  1. The second read holds exactly the sent text, which normalizes equal to `normalized`.
  2. With the `!submitted` guard in place the `if` is skipped, so no Enter is sent.
  3. With the guard narrowed to the foreign-text case, `guardedEnter` calls `SendKey("Enter")`, `sentCalls` is non-empty, and the row fails.
  4. `active=false`, so `operatorActivityHold` cannot suppress that Enter.
- **Unreadable row against the defect:** if the `submitted` return at window close were dropped, the function would return `ErrSubmissionNotLanded`, and the row's `err != nil` check would fail it.
- **Non-code edits:**
  - The help text in `cli.go`, the refusal string in `verbs.go` and the smoke-test doc comment are text only, with no behavior change.
  - The review says `verbs_test.go` asserts only substrings the new refusal text still contains.
  - Gate tier0 ran green on the three packages.

Further tool calls (2 of 6):
1. Read `readInputBox`, `windowPollCap` and the `InputBoxText` fake in `send.go` and `run_test.go`: my context lacked how an unreadable box and the sticky last answer behave.
2. Read the constants `idlePollInitial` and `confirmBackoffCap` in `send.go`: my context lacked them, and I needed them to check that the window closes before the read cap binds.
