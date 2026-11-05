```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F3 change set for agreement with sibling text and twins.

No findings.

- **F1:** The refusal text in `internal/boardengine/store.go:389` still names `set-deps` bare. It is the board's own refusal, and the review leaves it alone. The notice test at `internal/loomcli/commitstatus_integration_test.go:167` asserts the substring `lym board set-deps`, which the new text still contains. The `depends_on` payload matches the `set-deps` shape the review cites.
- **F2:** The grep for "before the start guard" finds no twin of the old sentence elsewhere in `internal/` or `docs/`.
- **F3:** The new row sits in the same table as its recipe siblings and follows their naming. It reuses the existing `status`, `recipe` and `wantRefused` fields.

Tool calls:
1. `grep` for "before the start guard" and "set-deps" across `internal` and `docs`. My context held the changed files but not twin wordings elsewhere in the repository.
