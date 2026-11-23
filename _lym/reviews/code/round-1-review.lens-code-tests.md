```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviours and the tests that pin them**

1. **A one-entry reviewer list passes preflight** (`reviewerRepeat` returns `""` when `len <= 1`).
   Pinned by `TestCheckCampaign/ReviewerListOfOneEntryPasses` (`internal/hardenshed/campaign_test.go:106-108`).
   The row has no `wantSubs`, so any refusal fails it through the `wantSubs == nil` branch (`campaign_test.go:137-141`).
2. **An empty reviewer list still refuses.** `reviewerRepeat` now returns `""` for zero entries, but the earlier empty-list check catches the list first (`campaign.go:81-83`).
   Pinned by `TestCheckCampaign/ReviewerListEmpty` (`campaign_test.go:96-99`).
3. **Lists of two or more still refuse neighbouring equal models**, including the wrap from the last entry to the first.
   - Neighbouring entries: `ReviewerNeighboursAreOneModel` (`campaign_test.go:110-113`).
   - Last and first: `ReviewerLastAndFirstAreOneModel` (`campaign_test.go:115-118`).
   - A two-entry list of different models passes: `ReviewerListOfTwoModelsPasses` (`campaign_test.go:120-122`).
   - Changing the guard from `<= 1` to `<= 2` would fail the first two rows, because they use three-entry lists.
4. **The template defaults `reviewer` and `fixer` to `["sonnet[high]"]`.**
   Pinned by `TestLoadConfig/TemplateLoadsEveryDefault` (`internal/hardenshed/config_test.go:49`), which loads `ConfigTemplate()` and compares it with the `defaults` the diff updated (`config_test.go:38-41`).
5. **`run_settings` accepts `hardener.reviewer` and `hardener.fixer` as lists of model-specs.**
   Pinned by the new `TestPrepareRunSettings` row (`internal/configreg/runsettings_test.go:93-100`).
   `PrepareRunSettings` runs `checkLocalDocument`, which calls `ValidateLocal` and then the module's `CheckValues` (`runsettings.go:73`, `runsettings.go:144-151`). So the row would fail if either check refused list values or these model-specs.
   It asserts the exact YAML document, both lists in order.
   The redundancy report lists it as unique.
6. **The old one-entry refusal message is gone.** Removing a refusal deletes its assertion as well. The decision record's added entry licenses this, so it is no loosened check.
7. **Doc comments in `doc.go` and `campaign.go`, and the comments in `template.yaml`**: static prose, exempt.

**Findings:** none.

**Tool calls**

1. Read the `TestLoadConfig` rows in `internal/hardenshed/config_test.go:30-90` and `checkLocalDocument` in `internal/configreg/runsettings.go`. My context held only the diff hunk of `config_test.go`, not its rows, and lacked the body of `checkLocalDocument`. I needed both to confirm the template-default row exists and that the new `run_settings` row passes through `CheckValues`.
