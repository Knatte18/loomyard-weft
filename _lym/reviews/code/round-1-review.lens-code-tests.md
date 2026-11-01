```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff changes, I name the test that pins it, or none.

**Behaviour → pinning test**
1. The refuse-raw steer keeps the full gate rule. `TestCheckTexts/ShippedRefuseRawRow` (`internal/shellguard/rows_test.go:~45`) requires the steer to contain "Gate rule", `lym gate test`, `--tier`, `--card`, the foreground phrase and "no gate slot is free". Dropping any of these fails the test.
2. The refuse-raw notice names only the refusal and the tier and card scopes. The same test compares it against an exact string (`wantNotice`, `rows_test.go:~50`). Any change to the wording fails it, including re-adding the rule. The failure message names the notice.
3. The gate rule is stated once across the notices. The `gateRuleNotices != 1` count (`rows_test.go:~58`) covers this. It runs over all rows with a non-empty refuse list, so it fails if a second row's notice gains the "Gate rule" marker. It fails the same way if the raw-go notice loses it.
4. The raw-go notice is unchanged and still carries the full rule. `TestBuildDenyNotice_SentenceContent` (`internal/shuttleengine/claudeengine/settings_test.go:~972`) requires "Gate rule: run lym gate test in the foreground with the Bash tool timeout…" and "never cd into…" in the assembled notice. It uses an empty `RefuseRaw`, so it pins the raw-go notice on its own.
5. The refuse-raw notice reaches the appended prompt only when the refuse list is non-empty. The `append-system-prompt.txt` check (`settings_test.go:~896`) asserts `strings.Contains(prompt, refuseRow.Notice)`. It derives the text from `Row.Notice`, so it follows the change without edits and still pins presence.
6. Both notices are emitted together and the rule appears once in the assembled prompt. No test pins this. `TestBuildDenyNotice_MatchesInstalledDenies` joins the installed notices and compares them to themselves, with no count of "Gate rule". The once-in-the-prompt guarantee rests only on the `shellguard` row-level count.
7. `checkTexts` passes the shortened notice. The loop in `ShippedRefuseRawRow` and the package `init` cover this.
8. The `doc.go` paragraph and the `Rows` comment are prose and exempt.

**Findings**
- **LOW, design.** `internal/shuttleengine/claudeengine/settings_test.go:896` and `:974`. Behaviour 6 has no test. Neither the `RefuseRaw` prompt test nor `TestBuildDenyNotice_SentenceContent` counts "Gate rule" in the assembled notice with a non-empty refuse list. A change in `buildDenyNotice` or `appendPromptText` that duplicated or dropped a row would not trip the row-level count. Suggested fix: in the `RefuseRaw` prompt test, assert `strings.Count(prompt, "Gate rule") == 1` when `wantRefused`, and when it is false as well, since the raw-go row is always installed.
- **NIT, design.** `rows_test.go:~50`. The exact-string `wantNotice` repeats the production literal, and the notice is static prose. Its useful content is "no 'Gate rule' phrase, but the refused prefixes, `--tier` and `--card`". Asserting those would be less brittle. This is advisory, because the exact string does fail on any regression. It does not duplicate the count test, which covers a different property.
- The count test counts the substring "Gate rule". A rephrased re-statement of the rule in another notice would pass it. The exact-string check on the refuse-raw notice closes this for that row. A future row would not be covered.

**Tool calls:** none. Every file I needed was already in context.
