# Card 23 — comment-lint-clause

**Create:**
- `internal/commentlint#opensClause`

**Edit:**
- `internal/commentlint#endsSemanticBreak`
- `internal/commentlint/lint_test.go`
- `internal/commentlint/doc.go`
- `pattern/PATTERN-comment-line-breaks.md`

**Intent:**
Stop the comment lint passing every comma-plus-conjunction break (D13).
A checked break ending in a comma whose next line starts with a coordinating conjunction now passes only when `opensClause` accepts the words after the conjunction, up to the next `,`, `;`, `:` or sentence end: they open with a subject (a subject or demonstrative pronoun, a determiner followed by a word, or a backticked identifier) and hold a finite-verb candidate after the subject's head word (a closed set of auxiliaries and modals, or a word ending in `s` or `ed`).
Any other comma-plus-conjunction break is a finding; an uncertain case errs toward flagging, since joining the two lines always passes.
The lint stays diff-scoped, inside `commentlint.Lint`, so only new breaks are checked and the gate and `lyx loom lint-comments` keep calling one function (`PATTERN-gate-self-check-parity`).
Bound: a plural noun read as a verb (`, and the gate tests`) passes, and a clause whose subject the opener set misses (`, and nothing runs`) is flagged and joined.
`TestFindNewWrappedBreaks` gains rows flagging a new break before `, and returns the value` and before ``, and `Foo` ``, passing one before `, and it returns the value` and before `, but the gate refuses it`, and leaving an untouched legacy break unchecked.
`doc.go` and the lint-bound section of `PATTERN-comment-line-breaks.md` are rewritten to this rule.

**ImpactSummary:** One predicate in the lint and its doc; new comment breaks that join a list item or a compound predicate after a comma now fail the card gate and the round gate's lint.
