```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, and apply the rubric's test economy. I made no tool calls and worked from the inherited diff.

**Per behaviour: the test that pins it**

| Behaviour | Pinning test | Result |
|---|---|---|
| `AppendDecisions` writes entries in input order with each PATTERN bullet beside its entry | `TestAppendDecisions_WritesBatchWholeOrNothing/Batch` (`append_test.go`) | pinned |
| Empty batch is an error and writes nothing | same test, `Empty` | pinned |
| An empty field in a later entry leaves the record unchanged | same test, `LaterEntryEmptyField` | pinned |
| A finding over a batch restores the prior bytes | same test, `LaterEntryFinding` | pinned |
| `AppendDecision` is the one-entry case | the existing `TestAppendDecision_*` tests now run through `AppendDecisions` | pinned |
| `--from` beside a per-entry flag is refused | refusals row "from beside a per-entry flag" | pinned for `--title` only |
| `--from` file unreadable is refused | row "from file unreadable" | pinned |
| `--from` file not an array is refused | row "from file not an array" | pinned |
| Data after the array is refused | row "from file with data after the array" | pinned |
| Empty array is refused | row "from file with no entry" | pinned |
| Entry with an empty field is refused | row "from entry with an empty field" | pinned |
| Partial PATTERN-change form is refused | row "from entry with a partial PATTERN-change form" | pinned |
| Repeated title is refused, trimmed | row "from entries sharing a title" | pinned |
| Repeated PATTERN entry ID is refused | row "from entries sharing a PATTERN entry" | pinned |
| Unknown key in the `--from` file is refused | **none** | see finding 1 |
| One append, one commit, `headings`, `pattern_changes`, no `heading` or `pattern_change` for a batch | `TestDecisionVerb_FromFileRecordsBatch` | pinned |
| `--amends-plan` with a batch appends every title in order | same test (`ReadAmendment` titles) | pinned |
| Batch notice: one per receiver, short with every heading, Merriam inline | same test (`f.posts`) | pinned |
| Darn check confirms every batch entry is in the committed record | refusals row "darn batch entry missing from the committed record after the commit" | pinned, with the first entry present and the second missing |
| One-entry call's notice, envelope and behaviour unchanged | existing `TestDecisionVerb_AppendsAndCommits`, `TestDecisionVerb_NotifiesLiveAgents` | pinned by existing tests, not re-added |
| Restore rules after a failure with a batch and `--amends-plan` | no batch-specific test; the existing single-title restore tests exercise the same code | see finding 2 |

**Findings**

1. MEDIUM, `design`, `internal/loomcli/decision_test.go` (refusals table) and `internal/loomcli/decision.go:148`.
   - The decision record and the acceptance criteria require an unknown key in the `--from` file to be refused with a way forward and nothing appended.
   - No row in the diff covers it, and a search of `decision.go` and `decision_test.go` for `DisallowUnknownFields` and "unknown" finds nothing.
   - `decisionEntries` uses a bare `json.NewDecoder(...).Decode`, which silently drops a misspelt key.
   - A misspelt `pattern_entry`, for example, would record the decision with no PATTERN bullet. A test row would fail today, so this is a required behaviour with neither code nor coverage.

2. LOW, `design`, `internal/loomcli/decision.go:403-407` (`beginAmendment` title loop).
   - A failure partway through the titles loop restores the amendment record. This is new behaviour: the loop is new, and a one-entry call can never fail on its second title.
   - No test makes the second `appendAmendment` fail, so a regression that skips the restore after a partial batch would pass the suite.
   - Fix: add one row to the existing amend-failure tests.

3. LOW, `internal/loomcli/decision_test.go`, `TestDecisionVerb_FromFileRecordsBatch`.
   - The spec says `headings` is added "for every call", including a one-entry call.
   - Only the batch test asserts `headings`. The inherited view of `TestDecisionVerb_AppendsAndCommits` does not show a `headings` assertion for a one-entry call.
   - If that test omits it, extend that test with the key rather than adding a new one.

**Test economy**

I found no duplicate tests, no per-helper tests that the public surface already covers, and no static-prose assertions. Each refusals row reaches a distinct branch of `decisionEntries`. The `append_test.go` batch test overlaps `TestAppendDecision_FindingRestoresRecord` only on the restore assertion, and it adds the multi-entry case, so it stays.

**Tool calls**

None.
