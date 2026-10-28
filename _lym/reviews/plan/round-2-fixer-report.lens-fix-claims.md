```yaml
lens: fix-claims
outcome: reported
```

Task: check, through the fix-claims lens, that every claim the fix added or rewrote holds against HEAD.

no findings

- Card 7's added sentence "A line that names two created test files gives each the tags of both" holds. It follows from the card's own pass rule, which collects "the card's Intent lines that name the file" and "the tier tags those lines name".
- Card 7's new test row also holds under that rule. A line naming both files gives each file `tier0` and `tier1`, which are not joined by `||`, so both files fail. Split over two lines, each file's naming line holds one tag, so both pass.
- Card 9's move rests on the claim that the Long help rewrites an existing sentence, and it holds. `internal/loomcli/validate.go:181-182` reads "Every mode but --require-approved also runs rename-tool-missing and pattern-changes-mismatch, which read language.yaml and the decision record". Card 8 lists `internal/loomcli#loomCLI.validatePlanCmd` under `**Edit:**`, which agrees with the move. `lym loom validate-plan` returned `ok: true` after the edits.

Tool calls:
- Read `_lym/plan/08-seam-test-check.md` lines 1-25 and `internal/loomcli/validate.go` lines 175-190. My context did not show which group card 8 puts `validatePlanCmd` in, or the full Long help sentence.
