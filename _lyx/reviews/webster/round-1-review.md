---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/selfreportengine/inbox.go:23
    summary: New doc comments in inbox.go are wrapped mid-sentence, breaking PATTERN-comment-line-breaks
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/selfreportengine/selfreport.go:4
    summary: Changed and newly written comments in selfreport.go (package doc, targetRepo, callTimeout, repoClient, classifyCallError) keep the old fixed-column wrapping
  - id: F3
    severity: LOW
    class: design
    location: internal/boardengine/store.go:132
    summary: Load normalizes depends_on and issues but not labels, so an entry stored with kind and no labels key lists and persists labels as null
---

Range: `main..HEAD` (`product.parent` is `main`, no rework round exists).
Build, vet and the untagged tests of the touched packages pass.

Focus checks that came out clean:

- Card 2: `notes`, `promote-note`, `Board.PromoteNote` and the two `board notes#…` `cliTreeFindings` entries are gone, and a repo grep finds no caller left.
- Cards 3 and 4: no `tier` or `type` field survives outside the decode shape in `migrate.go`/`legacy.go`, the retired-key refusals in `validateUpsertFields` and the migration prose in `doc.go`.
  Old-shape entries convert in `migrateEntries` and drop the retired fields on the next `Save`.
- Shared decisions: `Vocabulary` is set in both `boardCriticalSection` and `loadStore`; `checkFilterLabels` uses the `b.out.Readme == ""` predicate `boardCriticalSection` uses and skips validation on a path-only Board.
  The refusals in cards 3, 5, 6, 8 and 9 each name the way forward.
- Cards 7 to 9: import checks run in the specified order, a fold closes the issue with the pointer comment, a close requires a reason, and the board is written before GitHub is touched.
- Card 10: `doc.go`, `docs/overview.md`, the skill, the sandbox doc and the D13 label snippet (in `doc.go` and the commit body) are present.

### [BLOCKING] inbox.go comments wrapped mid-sentence

**Location:** `internal/selfreportengine/inbox.go:23`, `:75`, `:76`
**Issue:** `ListOpenIssues`, and `CommentAndClose`'s two paragraphs, break lines inside a sentence ("…following pagination to the / last page.").
`PATTERN-comment-line-breaks` requires one sentence per line for new comments.
**Fix:** Rewrite those comments one sentence per line.

### [BLOCKING] selfreport.go changed comments keep fixed-column wrapping

**Location:** `internal/selfreportengine/selfreport.go:4`, `:26`, `:39`, `:66`
**Issue:** The package doc, `targetRepo` and `callTimeout` comments were edited in this diff but still wrap mid-sentence, and the inline comments in the new `repoClient` and `classifyCallError` bodies carry the old wrapping.
Changed and new comments fall under `PATTERN-comment-line-breaks`.
**Fix:** Rewrite those comments with semantic line breaks, keeping the wording.

### [LOW] Load leaves labels nil for entries stored with kind and no labels

**Location:** `internal/boardengine/store.go:132`
**Issue:** `Load` defaults `DependsOn` and `Issues` to empty lists, and `migrateEntries` gives kind-less entries `[]`, but an entry with `kind` and no `labels` key keeps nil.
`list` then prints `"labels":null` and the next `Save` writes `null`, while `NewTask` and `ApplyPatch` always produce `[]`.
**Fix:** Default `Labels` to an empty list in the same loop in `Load`, with a store test.
