# Card 10 — remove-await-review

**Delete:**
- `internal/burlercli/await.go`

**Edit:**
- `internal/burlercli#burlerCLI.resolvePersistentPreRun`
- `internal/burlercli#Command`
- `internal/burlercli/cli_test.go`
- `internal/burlermarker/doc.go`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-BURLER-SUITE.md`

**Intent:**
Remove `lyx burler await-review` (decision D7), now that the fixer waits with `lyx shuttle await` (card 9).

`internal/burlercli/await.go` is deleted with the verb's name constant and builder.
`Command` stops registering the verb, and `resolvePersistentPreRun` drops it from the verbs that skip hub resolution, leaving `burler` and `validate-review`; its doc comment drops `await-review`.
`burlermarker.Path` still derives the marker path; `internal/burlermarker/doc.go` says the fixer's `lyx shuttle await` waits on the marker.

The verb's removal is the open risk the record names: a fixer mid-round across the deploy that calls the removed verb fails, and the round then ends on the skipped-handoff error or the timeout, which the row's retry or a re-step recovers.

Tests: `TestAwaitReviewVerb` leaves `internal/burlercli/cli_test.go`; `lyx shuttle await` keeps its own coverage in `internal/shuttlecli`.

Docs: `docs/overview.md`'s burler entry drops the `await-review` clause and names the fixer's wait on `lyx shuttle await`;
`tools/sandbox/SANDBOX-BURLER-SUITE.md` drops scenario S4 and its intro sentence's S4 clause, so S1-S3 keep the burler module exercised through `lyx burler run` (`PATTERN-sandbox-coverage`).

**ImpactSummary:** Deletes one debug-CLI verb and its test; its only caller was the fixer stencil, already moved off it in card 9.
