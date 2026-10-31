```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: does the change set agree with its sibling cards, criteria and cross-references?

1. **MEDIUM** · card 3, TestSessionSpec (F3 cut). D8 and D9 now have no test.
   - The cut removed the only assertions behind the acceptance criteria in `_lym/discussion/decision-record.md:214-215`. The rendered stencil "names `sampler-done`, `decision add --by operator` and notes-only board writes". It also "names the no-dev-build-in-this-hub rule and the own `tmux -L` server rule".
   - Card 3's Intent still opens "D5, D8 and D9", but no card asserts the D8 and D9 wording.
   - The stencil-wide scans the review cites only check absences: no unbounded ask-operator and no skill-load instructions. They do not check that these phrases are present.
   - The earlier plan round disputed this same cut on these grounds, and `_lym/plan/lens-review.md:14` records it as "disputed, every fix worse … dropping them leaves two criteria untested". The fix now contradicts that recorded disposition without addressing it.

2. **LOW** · card 5, `**Commit:** 5: shedrecipe: register the SamplerSession build engine`.
   - The card now also creates and embeds `contracts/recipes/sampler-recipe.yaml` and extends `RecipeEngines`.
   - Its ImpactSummary and the `00-overview.md` index line were updated, but the commit subject was not.

3. **LOW** · card 6, Uses `contracts/recipes/darn-recipe.yaml`.
   - That entry was there because card 6 copied the Code-Review rows from darn's recipe. That copying moved to card 5, which now lists the same Uses entry.
   - Card 6's remaining Intent reads darn's rows only through the parsed recipe (`TestDarnVerifyBudget`, `verifyGatedRows`), so its Uses entry is now stale.

Tool calls:
- Listed `contracts/recipes/`, grepped for D8/D9 and TestSessionSpec references, and grepped tests mentioning `step_idle_timeout_min`. My context had none of the decision record or the earlier lens-review dispositions.
- Grepped `decision-record.md` for the acceptance lines and `loomengine/config_test.go` for template-comment pins. My context lacked whether a test pins the template comment; none does.
- Listed `_lym/discussion/` and grepped it for the D8/D9 acceptance text, since the previous grep missed the record's wording.
