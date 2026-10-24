---
round: 1
focus:
  - "Scope against the board entry: each of its eight bullets (guard files' `|| integration` term and the one-tag scan allowance in `cmd/lyx/tiertags_test.go`, hub tier commands, `internal/impactset` and `internal/planglyph` package loads, `cmd/testtiming` spawn scan and `defaultRedundancyTags`, the `tier2`/`tier3` refusal guards, tier purity and tmux isolation guards, `allow_tmux.go` constraint, `glyphchain/shapes/tagged.go`) must be landed by the diff; check none is missing."
  - "Leftovers: grep the tree at HEAD for any remaining old tag spelling (`integration`, `tmux`, `llm` as build tags, `|| integration`) outside the entry's list; settle whether each hit is a needed spelling or a missed removal."
  - "Over-reach: the diff touches many test files and `internal/shuttleengine/claudeengine/testdata/settings-golden.json`; settle whether each change is licensed by the entry, and whether the golden-file churn follows from a code change in the diff."
  - "Correctness of the dropped terms: in the guards, scans and package loads that lose a spelling (`internal/impactset/guards.go`, `internal/planglyph/typesload.go`, `internal/planparser/verifymodule.go`, `internal/shellguard/rows.go`), settle whether a tagged file can now go unscanned or a tier be misclassified."
  - "Docs and comments: `doc.go` and `PATTERN.md`/`pattern/` text that names the removed spellings must change in the same diff; check no doc still describes the transitional state."
  - "Test economy: the removed assertions in `internal/impactset/guards_test.go` and `internal/shellguard/rows_integration_test.go`; settle whether remaining coverage still pins the one-tag rule."
---

Seed focus from the board entry and the diff stat; no round has been judged.
