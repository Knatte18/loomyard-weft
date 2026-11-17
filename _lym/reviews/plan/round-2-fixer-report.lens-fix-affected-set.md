```yaml
lens: fix-affected-set
outcome: reported
```

Checked the sets in findings F1–F6: every member is covered except one gap, below.

**Unchecked findings:** none. I reran the search for each set-based finding. F5 and F6 were checked from the changed text and my greps.

- **F1, stale allowlist entries:** the only `internal/seatengine.Prompts` entry in `testOnlyCodeAllowed` is the one card 8 now deletes. No other entry in the list is reached by a plan symbol. Covered.
- **F2, tables built with nil values:** the only nil-values caller in the catalog was card 6's `SeatTables`. Every production `fixTable` and `reviewTable` call passes real values (`fix.go:179,254,293`, `review.go:146,157`). Covered.
- **F4, the verb failing on an unmet requirement:** card 8's Intent bullets, its `TestManifestCmd` row and overview line 55 are all changed. The only other "non-zero" mentions in `_lym/plan/lens-review*.md` are reports from an earlier round, not cards. Covered.
- **F5, the fork-deny gate:** card 6 and overview line 51 are changed. Card 2 line 47 ("gates it on the fork requirement") still agrees. Covered.
- **F6, the copy warnings:** both ways forward are widened. Covered.

**Finding, LOW:** card 6 line 77 (`PrepareEach` first sets each spec's `Skills` with `cfg.Agents.Skills(...)`) adds behaviour that `TestCompare`'s `PrepareEach` rows never cover.
- `PrepareEach` now expands the skills, but those rows (card 6 lines 115–116) still cover only a failing `Prepare` and an absent record.
- No row asserts that a spec with a `ConfigRole` reaches `Prepare` with the role's skills.
- The text does not say what happens when `cfg.Agents.Skills` returns an error. `Runner.start` (`internal/shuttleengine/run.go:576-579`) returns that error wrapped.

Tool calls:
1. Read `cmd/lym/testonlycode_linux_test.go` lines 47–75, to rerun F1's set against the whole allowlist; my context held only the `Prompts` line.
2. Grepped `_lym/plan/*.md` for "non-zero", `Unmet` and `RequirementForks`, to rerun F4's and F5's sets across every card; my context held only the cards I had read.
3. Grepped `internal/burlerengine` and `internal/landingshed` for `fixTable(` and `reviewTable(` calls, to rerun F2's set of nil-values builders; my context lacked the `reviewTable` callers.
4. Grepped `_lym/plan/*.md` for `Agents.Skills`, `LanguageSkills`, `language.yaml` and `addSkills`, to rerun F3's set of skill-source sites in every card.
