```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**

| Behaviour | Pinning test | Verdict |
|---|---|---|
| `AppendDecisions` writes the batch in input order with each PATTERN bullet | `TestAppendDecisions_WritesBatchWholeOrNothing/Batch` (`append_test.go`) | pinned |
| Empty batch is refused and nothing is written | same test, `Empty` row | pinned |
| An empty field in a later entry leaves the record untouched | same test, `LaterEntryEmptyField` row | pinned |
| A finding from the check restores the record | same test, `LaterEntryFinding` row. The one-entry `TestAppendDecision_FindingRestoresRecord` also covers the restore. | pinned |
| `AppendDecision` is the one-entry case of `AppendDecisions` | the existing `AppendDecision` tests (`append_test.go:246` and earlier), which run unchanged through the wrapper | pinned |
| `--from` beside a per-entry flag is refused | Refusals row "from beside a per-entry flag" | pinned |
| Unreadable file, non-array JSON, data after the array, stray `]`, unknown key, empty array, empty field, partial PATTERN form, repeated title, repeated PATTERN ID are each refused with a way forward and no append | one Refusals row each; every row sets `inputRefusal` and `noDarnReads` | pinned |
| Batch commits once, with one append, in file order, with the PATTERN change | `TestDecisionVerb_FromFileRecordsBatch` | pinned |
| Envelope has `headings` and `pattern_changes` for a batch and no `heading` or `pattern_change` | same test | pinned |
| `--amends-plan` with a batch appends every title in order | same test (`ReadAmendment` titles) | pinned |
| A failed title append mid-batch restores the amendment record | `TestDecisionVerb_AmendsPlan/BatchSecondTitleFails_RestoresTheAmendmentRecord` | pinned |
| A batch notice names every heading and the record, with no texts, for a non-Merriam agent | `FromFileRecordsBatch`, post loop with `inline=false` | pinned |
| Merriam's strand gets every entry's text inline | `FromFileRecordsBatch`, receiver `hub:task-a:webster` | pinned |
| A one-entry notice is unchanged (decision and rationale inline) | existing `TestDecisionVerb_NotifiesLiveAgents`, which asserts `input.decision` and `input.rationale` in every post | pinned |
| Darn batch: a committed record that lacks a later entry is refused | Refusals row "darn batch entry missing from the committed record after the commit" | pinned |
| `committedLacksEntry` returns the first lacking entry across a batch | the same darn row; the other lacking-entry cases are covered by the existing darn rows | pinned |
| Batch with no `--amends-plan` (plain `--from`) | `FromFileRecordsBatch` sets `amendsPlan: true`, so no test runs a `--from` batch without it. This is not a behaviour gap, because the append, commit and notice paths are shared. | no separate test needed |
| `--from` batch on a darn run succeeds | none. `--from` plus `darn` runs only on the failing-commit row. The darn pre-check is shared with the one-entry path (existing darn rows). | not a gap |
| Doc and help text for `--from`, and the orch stencil line | none. Static prose is exempt. | exempt |

**Test-economy findings:** none raised.

- `TestAppendDecisions_WritesBatchWholeOrNothing` adds only the batch case. It does not duplicate the `AppendDecision` tests; `LaterEntryFinding` shares the restore path with `TestAppendDecision_FindingRestoresRecord` but adds the multi-entry restore, so it is not redundant.
- The `unknownEntryKey` helper has no per-helper test. The "from entry with an unknown key" Refusals row covers it at the verb level.
- The 6 test-side `from` tweaks only add Refusals rows or `AmendsPlan` rows. Each pins a distinct refusal or restore path.

Findings: none.

Tool calls (1 of 6): read `internal/loomcli/decision_test.go` lines 735-885 to see the Refusals loop assertions and the existing `NotifiesLiveAgents` test. My context had only the diff hunks.
