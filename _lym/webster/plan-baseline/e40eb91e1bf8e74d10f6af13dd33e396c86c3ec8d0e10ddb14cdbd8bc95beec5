# Card 1 — code-reviewer-fan-local

**Edit:**
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestPrepareRunSettings`
- `docs/overview.md`
- `pattern/PATTERN-config-layers.md`

**Uses:**
- `internal/loomengine#MigrateConfig`
- `internal/loomengine#retiredPerchKeyRenames`

**Intent:**
D5: reclassify loom's `code_reviewer_fan` from `configengine.CategoryGuard` to `configengine.CategoryNone` in `ConfigSpec`'s `Categories`, so a profile block, a task's `run_settings` and a pair's local layer can set it.
The other five reviewer and fixer fan keys, `discussion_reviewer_fan`, `plan_reviewer_fan`, `discussion_fixer_fan`, `plan_fixer_fan` and `code_fixer_fan`, stay guards.
`ConfigSpec`'s doc comment gains one sentence: `code_reviewer_fan` is the one reviewer fan key open to the local layer, since its value names only a fan the guarded `lenses.yaml` defines, or `""` for a solo round, and the perch's round count, rubric and whole-diff read stay.

What the change lets through, and its bound, as D5 states it: an agent can thin its own run's code review to solo, or point it at any other fan `lenses.yaml` defines, through its pair's local layer.
The write is not checked against `lenses.yaml`; a wrong name fails the pair's loom config load through `lenses.ValidateFanKeys`, whose refusal names the key and the fix.
The refusal does not name the layer holding the value; for a seeded or hand-set value that is the pair's local loom file, where setting the entry to `""` or to a fan `lenses.yaml` defines clears it, since a `run_settings` or hub write reaches no running pair.
The refusal text is shared with every fan key and stays as it is, which the record does not ask to change.
The round keeps the reviewer's whole-diff read under the code-review rubric, and `code_max_bounces: 0` in the same file was already the stronger switch.
A task's `run_settings` reach a pair only at Seed-Child, so a `run_settings` write during a run changes no running pair.

The retired keys that reach `code_reviewer_fan` are admitted with it, with no code change: `configreg`'s darn key move carries `darn_reviewer_fan` and `review_fan` to `loom.code_reviewer_fan`, and `MigrateConfig` renames `webster_fan` through `webster_reviewer_fan` onto `code_reviewer_fan` (`retiredPerchKeyRenames` runs after `retiredReviewKeyRenames`), both before `PrepareRunSettings`' check.
A hand-edited local file holding `webster_fan` or `webster_reviewer_fan` stays refused at load as a key outside the loom template until a reconcile migrates it, which this card does not change.

Tests:
- `TestLoadConfig_Refuses`' subtest "a local fan key is refused naming the local file and the key" drops `code_reviewer_fan` from its key loop and keeps the other five, so a local file holding any of them is still refused.
- `TestLoadConfig_Loads`' "hub stack" subtest gains the row "a local code_reviewer_fan of \"\" is open to the local layer": `local: "code_reviewer_fan: \"\"\n"` over the template's `["review-code", ""]`, `mutate` setting `CodeReviewerFan` to `lenses.FanList{""}`, so the local scalar replaces the hub list whole, the state a pair seeded from a solo-fan profile loads.
- `TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied["loom"]` drops `code_reviewer_fan`.
- `TestPrepareRunSettings`: the rows "a retired darn reviewer fan key moves to loom's code_reviewer_fan and is refused as a guard key" and "a retired darn review_fan moves to loom's code_reviewer_fan and is refused as a guard key" become admitted rows, renamed to "... moves to loom's code_reviewer_fan and is admitted", with `wantDocs: {"loom": "code_reviewer_fan: review-code\n"}` and `wantNoModule: "darn"`, as the darn model-key move row renders its moved keys.
  Two rows are added: `{"loom": {"webster_fan": "review-code"}}` and `{"loom": {"webster_reviewer_fan": "review-code"}}`, each admitted with `wantDocs` `code_reviewer_fan: review-code # <comment>\n`, where `<comment>` is the line comment of `code_reviewer_fan` in `internal/loomengine/template.yaml`, which a rename carries as the webster_review row shows.
  A row sets `{"loom": {"code_reviewer_fan": ""}}` directly and is admitted with `wantDocs` `{"loom": "code_reviewer_fan: \"\"\n"}`, the shape a profile block or a task's `run_settings` carries.
  The other five fan keys stay refused through run settings with no new row: the existing refused rows for `discussion_fan` and `review_fix_fan` pin the rename-then-refuse path, and `PrepareRunSettings` refuses through the same `LocalDenied` set `wantLocalDenied` pins.

Docs (D7):
- `docs/overview.md`'s loom config line replaces "all six closed to the local layer" with wording that names `code_reviewer_fan` as open to the local layer and the other five as closed.
- `pattern/PATTERN-config-layers.md`'s Guard bullet gains a sentence naming `code_reviewer_fan` as the one reviewer fan key open to the local layer, with D5's bound: its value names only a fan the guarded `lenses.yaml` defines or `""`, and the perch's rounds, rubric and whole-diff read are unchanged.
  The `PATTERN.md` overview line is unchanged.

**ImpactSummary:** one category entry in loom's spec; LocalDenied for loom loses one key, pinned by configreg's classification test; run-settings and loom load rows change or join, no production caller changes.
