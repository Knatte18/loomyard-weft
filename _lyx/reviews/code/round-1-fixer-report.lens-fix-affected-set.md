```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F1 and F2 fixes cover every member of the sets they touch.

**Findings**

- LOW. Changed text: `internal/websterengine/report.go`, where the reason test was dropped. A stale twin remains at `internal/websterengine/doc.go:63`. It reads "ParseReport refuses an entry missing any of the three or naming one glyph twice", where "the three" means `old`, `new` and `reason`. After the fix `ParseReport` refuses only a missing `old` or `new`, or the same glyph twice. The module doc now overstates what is refused, which a reader would notice. The fixer's edits to `report.go` and `report_test.go` left it unchanged. It should say that `ParseReport` refuses an entry missing `old` or `new`, or naming one glyph twice, and that `reason` is informational.

- No findings for F1. `insertCardVerb` and `removeCardVerb` both build their envelope through the single `renumberEnvelope` mapping (`renumber.go:194-207`). The one assertion on `rewritten_refs` therefore covers the whole mapping, and the review asked for only one case.

- No findings for F2's other reference sites:
  - `report_test.go` has no other rows that depend on a missing reason.
  - The stencils `webster-body-implementer.md:47` and `webster-template-master.md:112` still ask for a reason, as the review intended.

**Unchecked:** none.

**Tool calls**
1. `grep -rniE` across the repo's `.go`, `.md` and `.yaml` files for reason/rename wording. I searched because my context lacked the other sites that describe the reason requirement, and it found `doc.go:63`.
