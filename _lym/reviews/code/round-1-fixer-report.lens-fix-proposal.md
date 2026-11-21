No findings.

Task: challenge the F1 proposal (Door line changed to one-way, plus one sentence) as fixed in `_lym/landing/summary.md`.

- **Door value:** the spec at `contracts/specs/loom-final-summary-spec.md:39` makes the door one-way for a record format another binary reads. `verify-gate.yaml` is read back by `readVerifyGateReport`, so one-way is correct.
- **Parsing:** `labelledValue` in `internal/summaryparser/description.go:170` matches lines by prefix. It reads `**Door:** one-way` as `one-way`, and the extra sentence on its own line below does not affect it. `**Blast radius:** gate` is still found and is one word.
- **Section boundary:** the section runs to the next level-1 or level-2 heading, and the added sentence introduces none.
- **Prose:** the sentence matches the review's wording and makes no claim the code contradicts.

Tool calls:
1. Read `contracts/specs/loom-final-summary-spec.md` (grep for Door): my context lacked the door and Merge danger format rules.
2. Read `internal/summaryparser/description.go` (grep for `labelledValue`) and `_lym/landing/summary.md` lines 28–40: my context lacked the parser behaviour and the edited section as it now stands.
