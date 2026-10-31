```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set: I reran F1's two searches against the cards as the fixer changed them.

Unchecked findings: F2 and F3. Each concerns a single test row, not a set, so there was no search to rerun.

no findings

- **Entry-number consumers:** every numbered-row string in a test, golden or json file is in `internal/battencli/overview_test.go`, `internal/battencli/overviewdobby_test.go` or `internal/loomcli/checklist_test.go`. Only `checklist_test.go` and `recipe_test.go` build numbers from a shipped recipe.
  - The overview hits include the `6.7 Plan-Burler-Review` row, which the fixer added to card 9.
  - In `checklist_test.go`, only the darn rows come from a recipe, and card 11 now names them.
  - `TestDobbyLines` takes cells that are already built, so leaving it off card 9 is right.
  - `docs/overview.md:654` already says `6.5 Plan-Write`.
- **Plan-field readers:** the only test files that read a plan field after loading are `config_test.go` (`TestLoadConfig_Loads` and `TestMigrateConfig`), `review_test.go` (`TestResolveReview`) and `wiring_test.go` (`TestWire_DefaultConfig`). Card 6 now edits all of them.

Tool calls:
1. Repo-wide search for numbered-row strings and recipe-derived numbers. My context held only the fixer's search, which covered `internal` test files.
2. Search of the docs, contracts and pattern files for numbered rows, plus a rerun of the plan-field readers without the variable-name filter. My context held neither.
