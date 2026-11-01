No findings.

- F2 text: "the steers of the raw go and refuse-raw rows each carry the full rule" matches `rows.go:161` and `rows.go:176`.
- F2 and the raw go notice: the notice is `rawGoSteer + "."` (`rows.go:210`), so the rule is stated once in the system prompt as the doc says.
- F1 text: the new comment at `rows.go:183` stands alone; the cross-row relationship sits in the package doc paragraph.
- Twins: a grep for the old phrasings (`each row's steer`, `states the gate rule`, `carries the full rule`) found no other copy.
- Related comments: the `Rows` doc comment (`rows.go:158`) and the `rows_test.go:27` comment agree with the new text.

Tool calls:
1. `grep` over `internal` and `docs` for the old and new phrasings; the context lacked `rows_test.go` and other files that might repeat the wording.
