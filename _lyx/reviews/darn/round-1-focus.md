---
round: 1
focus:
  - "internal/boardengine/render.go `entryCell`: does a task without `run_settings` render byte-identically to before, and do the setting lines land in the cell the board entry names (under the title/brief) with no new column?"
  - "Setting-line formatting: are nested keys, string versus non-string values, pipe escaping and sorting handled correctly, and could a value containing a newline or a backtick break the table row or code span?"
  - "Docs: do `internal/boardengine/doc.go`'s `# README` section and the `render.go` header match the rendered output, and does every claim in `_lyx/landing/summary.md` hold against the diff?"
  - "Tests: does the `TestRenderReadmeGolden` fixture and golden cover the behavior the diff adds without duplicating existing coverage, and is any test-budget row in `cmd/lyx/testdata/test-budget.yaml` affected?"
---

Seed focus for round 1 of the darn review; the diff is small and centred on `render.go`.
