```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff to `internal/shellguard/rows.go`, `rows_test.go` and `doc.go`.

**List (function or branch, case run, result):**
- `refuseRawRow`, refuse list `["go test","dotnet test"]`:
  - `refused` is "a raw go test or dotnet test run is refused here".
  - `steer` is `refused` plus ". Gate rule: …", byte-identical to the old steer. Correct.
  - `Notice` is `refused` plus "; run it through lym gate test, with --tier to name the tiers or --card to name a card.". It has no quote, backslash or newline, so `checkTexts` passes. Correct.
- `refuseRawRow`, one-element list `["go test"]`, which is also the `init` input:
  - `refused` is "a raw go test run is refused here", and the join with " or " adds no separator.
  - `init` runs `Rows("", ["go test"])` through `checkTexts`, and the text holds no forbidden character. Correct.
- `Rows`, empty or nil `refuseRaw`: the `len > 0` guard skips `refuseRawRow`, so no refuse-raw row and no notice. The raw go notice still carries the full rule, so it is stated once. Correct.
- `Rows`, raw go row:
  - The code is unchanged, and `rawGoSteer + "."` is still the Notice, carrying the whole rule.
  - It is `SwitchNone`, so it is installed for every run. The consumer `buildDenyNotice` (`claudeengine/settings.go:327`) emits every installed deny whose notice is non-empty.
  - The refuse-raw row is `SwitchNone` too, so its notice never appears without the raw go notice, and the rule is never absent. Correct.
- Empty `foreground`: the steer and notice text only concatenate it, and the check passes. Correct.
- The refuse-raw notice drops the `--tier` and `--card` example and the "foreground" and "no free slot" parts, and keeps the `--tier` and `--card` scopes.
  - The raw go notice names only `--tier`. The refuse-raw notice adds `--card`, and nothing contradicts anything else.
  - The old `Row.Steer` is unchanged, so the denied-call path is unchanged. Correct.
- Existing consumer assertions in `claudeengine/settings_test.go`:
  - Line 896 (`refuseRow.Notice` is in the prompt) still holds, because the test reads the notice from the row.
  - Line 974 ("Gate rule: … 3600000", from the raw go notice) still holds.
  - Line 448 (`deny.notice == row.Notice`) still holds. Correct.
- `TestCheckTexts` `ShippedRefuseRawRow`:
  - It counts notices containing "Gate rule". Python, computed-rm, machine-sleep and publish contain none, raw go contains it, and the refuse-raw notice no longer does, so the count is 1. Correct.
  - It would fail at 0 if the raw go notice lost the rule, and at 2 if the refuse-raw notice regained it. Correct.
  - The exact `wantNotice` string matches the code output for the sample list. Correct.
- Doc comments in `rows.go` and `doc.go`: they match the behaviour above. The `Rows` doc sentence is clumsy ("the gate rule in the raw go row's steer and notice and the refuse-raw row's steer carries") but accurate.

**Findings:** none.

**Tool calls:** none. The manifest already held the diff, `rows.go`, the `claudeengine` notice builder and the consumer tests.
