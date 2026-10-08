# Webster-Burler round 1 — fixer report

Both findings were valid against the code and are fixed, one commit each.
`go build ./...`, `lyx loom lint-comments`, `go test ./...` and `go test -tags integration ./...` are green after the fixes.

## Fixed

- **F1** (LOW, consistency) — `fe2bdd373 websterengine: fix F1 — PlanOverviewFrameHash and overview doc state the baseline-copy fallback`.
  `internal/websterengine/state.go`: the `PlanOverviewFrameHash` comment now says a state without the field takes the frame from the stored baseline copy of its recorded overview until the first restamp, and refuses any overview change only when that copy is absent or has no parseable Card Index.
  `internal/websterengine/doc.go`: the overview paragraph's acceptance sentence now judges the file's frame hash against "the run's recorded overview frame", agreeing with the paragraph's later sentences on the baseline-copy fallback.
- **F2** (LOW, consistency; a coverage gap) — `80f1a3466 tokencount: fix F2 — cover a launch line in list content and a transcript ending at the prompt Read's result`.
  `tools/tokencount/count_test.go`, `TestCountRun`'s Merriam fixture gains two sessions rather than a new test:
  one whose launch line is a text item of list content, which measures its start context;
  one whose transcript ends at the prompt Read's result, which carries the `NoStart` reason `no assistant message after the prompt Read's result`.

## Disputed

None.

## Deferred

None.
