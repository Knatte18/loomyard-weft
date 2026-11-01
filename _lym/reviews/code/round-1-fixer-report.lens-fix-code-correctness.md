No findings.

Both commits (`a7655295` for F1, `7f24080d` for F2) changed only comment text:
- `internal/shellguard/rows.go:183` is a `//` line comment inside the `Row` literal in `refuseRawRow`. The `Notice` expression on the next line is untouched.
- `internal/shellguard/doc.go:18` is package doc text.

No signature, branch, string value or test changed. Callers of `Rows` and `refuseRawRow` behave as before, and `TestCheckTexts`'s `wantNotice` still matches the unchanged `Notice` string. The edit added no test, so no test needs to fail against the old defect.

Tool calls made: 0. Everything needed was already in the context.
