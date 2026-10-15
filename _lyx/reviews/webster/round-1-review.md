---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/transcriptread_test.go:363
    summary: TestLocateSessions runs serially because of its t.Setenv HOME row but carries no comment saying why
    fix: Add a doc comment above func TestLocateSessions stating that the test is not parallel at the top because one step sets the process-global HOME, and that its table rows run in parallel on temporary roots.
---

### [LOW] TestLocateSessions is serial without saying why

**Location:** internal/shuttleengine/claudeengine/transcriptread_test.go:363
**Issue:** The repository's Go testing convention is that a test which is not parallel names the reason in a comment.
`TestLocateSessions` calls `t.Setenv("HOME", home)` and so cannot call `t.Parallel()`, but the function has no doc comment.
Card 2 states the reason in its own text, so the cause is known and only the comment is missing.
The neighbouring serial test `TestSessionUsage_SumsParentAndForks` does carry such a sentence.

## Rejected

- Residual Claude format in tokencount, settled as not a defect:
  a grep of `tools/tokencount` for the card 5 Verify pattern and for `CLAUDE`, the card 6 Verify, returns nothing, and no price table, projects root, JSON field name or instruction-file name remains.
  `readLinePrefix` (`^\d+\t`) in `cardPointers` strips a file-read result's line-number prefix.
  Card 5 lists `cardPointers` under `Uses` and keeps card pointer lines as a lyx-format rule, so the plan places it in tokencount.
- Shared parse, settled as not a defect:
  `SessionUsage` and `Claude.ReadSession` both call `parseTranscript` in `usage.go`, and `SessionUsage` sums `parsedTranscript.tokens()`.
  No second parse of the usage fields survives; `ContextTokens`, `CompactedSince` and `latestReading` keep their own backward scans of other fields, as before.
- D7 count paths, settled as not a defect:
  `locateSessions` and `readLocated` in `tools/tokencount/reads.go` fail naming the worktree or session and the engine, and keep the cause with `%w`.
  `CountRun` and `CountOrch` both use them, and `TestCountRunWithoutSessions` pins both paths including the `fs.ErrNotExist` chain.
  Absence yields no sessions in `claudeengine.locateSessions` and contributes nothing.
- Offline adapters, settled as not a defect:
  `RecordReaders` builds engines with `shuttleengine.Config{}` and `configengine.Dirs{}`, and `NewFromConfig` starts no process or tmux.
  Each capability is wrapped in its own adapter, `TestRecordReaders` asserts none is a `shuttleengine.Engine`, and an engine without a capability gets a nil field that `ReadSet.engines` reports in the header line.
- Per-card mechanical checks, settled as not a defect:
  every symbol in card 5's `Delete` group is absent from `tools/tokencount`, the re-signed helpers and functions match the card's signatures, and both Verify greps pass.
  The price table in `claudeengine/pricing.go` is identical to the one deleted from `cost.go`.
  `lyx gate test` passes for the touched packages at the untagged and `integration` tiers, and `cmd/lyx` passes with the raised budget row.
- Test economy, settled as not a defect:
  each new top-level test (`TestPriceMessage`, `TestReadSession`, `TestLocateSessions`, `TestRecordReaders`) is a table or covers one capability, and the added `TestSessionUsage_SumsParentAndForks` rows pin cases the card names.
  Pricing by model is covered by `TestPriceMessage`, and the calibration instruction-file lookup by the new subtest of `TestCalibrate`.
