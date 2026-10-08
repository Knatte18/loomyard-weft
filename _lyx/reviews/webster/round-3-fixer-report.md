# Webster round 3 — fixer report

All four findings were checked against the code and hold; each is fixed in its own commit.
`go build ./...`, `go vet ./internal/planglyph`, `lyx loom lint-comments` and the full `go test ./...` pass.

## Fixed

- **F1** (`e040ef633`) — put the three edited sentences on one line: the `restampFingerprint` comment in `internal/websterengine/beginbatch.go`, the "It is this package's one call to `(*quarry.Repo).Resolve`" sentence in the `resolveTargets` doc in `internal/planglyph/repo.go`, and the "Folding a Rename pair's New side" sentence in the `cardOwnHandles` doc in `internal/planglyph/handle.go`.
  The untouched first sentence of `resolveTargets` keeps its old break, per `PATTERN-comment-line-breaks`.
- **F2** (`be04080f7`) — `editTarget`'s doc now reads "one Edit-group ref of a card, a member glyph or a file, with its shape and disk path", and `collectEditTargets`'s now reads "every Edit-group target of cards, limited to member glyphs and files".
- **F3** (`4541f0f20`) — `internal/planglyph/doc.go` lists `deleteorder.go` among the passes that raise `glyph-rejected`.
  That sentence says each pass is named at its own bullet, so the `delete-before-reference` bullet now also says that `glyph-rejected` is its fail-closed arm for a Delete or Edit member whose answer it cannot read.
  This matches the two `unreadableAnswerFinding` call sites in `deleteorder.go`.
- **F4** (`b61f48043`) — removed the unused `want` field from `TestGlyphChain_CrossCard` and passed `nil` to `assertPlanGate`, because every case expects a clean plan.

## Disputed

None.

## Deferred

None.
