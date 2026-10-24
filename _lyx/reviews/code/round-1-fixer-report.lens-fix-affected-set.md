```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each finding's own search to check the fix covers every member of its set.

- **F1 (verbatim vs raw string).** `rawBody` has one caller, at `lex.go:190`, and `stringBody` has one caller, at `lex.go:161`. The fix gates the only raw-string entry on `!verbatim`, so the whole set is covered. No other site takes the `quotes >= 3` path.
- **F2 (`record` as a declaration).** `typeKeywords` gains its type declaration only through `startsTypeDecl` at `source.go:116`, and `declaredName` is used only by `startsTypeDecl` and `typeDecl`. The new follower check sits in `startsTypeDecl`, so every path that declares a record goes through it. `typeDecl` at `source.go:278` and `source.go:281` runs only after that check passes.
- **F3 (whole-entry-only wording).** The only other sentence in `internal/gatecli/doc.go` that names scopers with whole-entry carrying is `doc.go:181`, which is the fixed line. Lines 61 and 86 describe scoper behaviour that does not concern carrying. `internal/gatecli/green_test.go:32`, `green_test.go:118` and `green_test.go:131` mention only the `none` scoper. Those are test descriptions of existing rows and are not made false by the doc change. No `.md` file states the carry rule.

Findings left unchecked: none.

no findings

Tool calls:
1. One combined `grep` across `internal/` for `rawBody` and `stringBody` callers. Context lacked the callers of `rawBody` and `stringBody`.
2. The same call also searched for `declaredName`, `startsTypeDecl` and `"record"` in `internal/dotnetimpact`. Context lacked these callers.
3. The same call also searched `.go` and `.md` files for the whole-entry carry wording. Context lacked any other statement of the rule.
