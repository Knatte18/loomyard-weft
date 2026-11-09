```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set searches behind F3, F4 and F5 and found no uncovered member.

**F4** (recipe lists in docs): no list of recipe names that omits `arras` remains. The grep over `docs/`, the README and `contracts/specs` hit only research data. The three `docs/overview.md` lists now all name `arras`. The Board-task `recipe` rule at line 1597 ("`tapestry`, `darn` or empty") is a different set and correctly left alone.

**F5** (split_* keys in `docs/overview.md`): the set members are the places that list `discussion_*`/`code_*` perch keys. In `docs/overview.md`, the judge/seed (1319), fan (1327), hard-cap (1349) and auto-accept (1351) lines each have a matching split line now. `internal/loomengine/template.yaml` already carries `split_reviewer_fan`, `split_fixer_fan`, `split_judge` and `split_auto_accept` with comments. `contracts/specs/llm-model-spec.md:149` lists only `discussion_judge` and `code_judge` and also omits `plan_judge`, so it was never a complete list and is not a gap of this change.

**F3** (table rows for split_* keys): the test files that cover the `code_*` keys with no `split_` coverage are `configsync_test`, `configreg/runsettings_test`, `battencli/runsettings_test`, `hubreconcile/ensure_integration_test`, `configcli_integration_test` and `loomcli/circling_test`. They test run-settings and config-sync plumbing and the renamed or retired `code_*` keys, not per-perch behavior. The review named five places for rows, and all five got them: `ResolveJudge`, `ResolveReview`, the `LoadConfig` budget refusal, `TestBouncerRunSubdir` and the arras blocks of `runprofile/template.yaml`.

**F1 and F2** are not set findings, so I did not search them.

Tool calls (3 of 6):
1. Grepped docs, the README and `contracts/specs` for recipe-name lists lacking `arras`, and for perch-key lists lacking `split_`, because my context held only the `docs/overview.md` spans.
2. Counted `code_*` against `split_*` mentions per test file to find test sites outside the review's named five.
3. Read the key lines of `internal/loomengine/template.yaml` and the `split_` references in `configreg`, because my context did not show them.
