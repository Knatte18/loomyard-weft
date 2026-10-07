# Round 4 fixer report

All four findings fixed, in `_lyx/discussion/decision-record.md`.
`support-log.md` needed no change.

## Changed

- **F1** (Constraints, `PATTERN-fabric-vocabulary`): the entry now states that the scan is a case-insensitive substring match over identifiers, literals and comments, covers every test file anywhere (`tools/tokencount`'s included) and not non-test `.go` under `tools/`, and passes only a closed exempt list, of which only `lyx weft sync`, `lyx warp checkout` and `lyx.exe weft push` are spellings D13 refuses.
  It tells the plan writer to take spelling fixtures from `lyx fabric …` or those three, build weft-named paths with the fabric side's helpers, and take tokencount's flag name from a non-test constant.
- **F2** (Goal, D13, Acceptance): the Goal names the plan-level verify section instead of a card's.
  D13 scans fenced blocks and inline spans of every plan file, cards and overview alike, and the finding names the card, or the overview section for a hit there; the acceptance criterion says the same.
- **F3** (D13, Constraints): the shared rule is stated as the three patterns plus the heredoc stripping and quote blanking before them, and the seam is stated to expose the path rule and the spelling rule separately and return the matched text.
- **F4** (D11, Acceptance): a run contributes findings when its batches name one profile, it was not restarted with `--fresh` and its review files were read; a profile line with no contributing run shows findings per card as not available, never zero.

## Deferred

None.
