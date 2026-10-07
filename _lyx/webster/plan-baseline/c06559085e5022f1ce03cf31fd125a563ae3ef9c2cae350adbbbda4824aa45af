# Card 7 — lint-comments-verb

**Create:**
- `internal/loomcli/lintcomments.go`
- `internal/loomcli/lintcomments_integration_test.go`

**Edit:**
- `internal/loomcli/cli.go`
- `internal/loomcli/arm_seed_test.go`
- `internal/loomcli/wiring_test.go`
- `docs/overview.md`
- `internal/websterengine/cardgate.go`
- `internal/websterengine/cardgate_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/doc.go`
- `pattern/PATTERN-comment-line-breaks.md`
- `pattern/PATTERN-gate-self-check-parity.md`
- `PATTERN.md`

**Uses:**
- `plan:internal/commentlint#Lint`

**Intent:**
Decision D7's standalone verb and card gate, and decision D6's reading of the line-break rule (issue #421).

`lyx loom lint-comments [--base <commit>]` resolves its worktree through loom's existing wiring (`PATTERN-cliwire-sole-wiring`), calls `commentlint.Lint`, and reports one JSON envelope: success with no findings, or a failure whose `findings` key lists each finding's file, line and text.
It takes no positional arguments, and `--base` is its one flag; without it the verb lints the working tree against HEAD, which is the card gate's form.
The round gate (card 10) calls `commentlint.Lint` directly, so the gate and the verb share one function (`PATTERN-gate-self-check-parity`).
Bound: the verb only reads git and files, and a finding fails only the gate whose command runs it.
`pattern/PATTERN-gate-self-check-parity.md`'s Pairs list gains the lint pair: the card gate's command and `lint-comments` share `commentlint.Lint`, and card 10 adds the round gate to the entry.
`loom` stays covered at module level by the core sandbox suite's `**Covers:** loom` scenario, so the verb needs neither a suite row nor an entry on the sandbox-coverage exclusion list (`PATTERN-sandbox-coverage`).

`cardGateCommand` appends ` && lyx loom lint-comments` to every card's command, after the tests.
`cardgate_test.go` and `beginbatch_test.go`'s rendered-gate expectation gain the verb.

`pattern/PATTERN-comment-line-breaks.md`, per D6:

- A break inside a sentence is allowed only at an independent-clause boundary: after a semicolon, or after a comma before a coordinating conjunction whose clause has its own subject and verb.
- A sentence end includes `.`, `!`, `?` or `:` followed by a closing `)`, quote or backtick, so a line ending in a colon is an allowed break.
- A break at any other comma (a list item, a compound predicate, a relative or introductory clause) is a violation.
- The clause-boundary break is permitted, never required, so its absence is no finding; the first bullet's "with a further break at an independent-clause boundary" is reworded to say so.
- A break between two untouched lines, or one an edit carried over unchanged, is never a finding, and a fix rewraps only the sentence the edit touches.
  This replaces the clause "untouched lines are never rewrapped" and keeps "a line-width finding is never legitimate" and "the rule governs new and changed comments only".
- The background names `lyx loom lint-comments` and the card and round gates as the rule's partial enforcement, states that the rule applies `scribe:prose`'s line-break rule with these clarifications, and that inside lyx's reviewers, fixers and lint this reading governs where `scribe:prose` reads as requiring the clause break.
- "Enforcement is review discipline, not a test" gives way to the lint.

`PATTERN.md`'s `PATTERN-comment-line-breaks` line gains `(test)` and names the lint.
`websterengine/doc.go`'s card-gate paragraph names the appended verb, and `docs/overview.md`'s loom verb list and its validate-verb paragraph name `lint-comments`.
`arm_seed_test.go`'s and `wiring_test.go`'s per-verb tables gain the verb's row, with the same expectations `validate-plan`'s row carries.

`lintcomments_integration_test.go` drives the verb over a real repository: a wrapped new comment fails with its file, line and text, a clean one succeeds, and `--base` lints the committed range.

**ImpactSummary:** Adds one loom verb and lengthens every card gate's command by the verb; the card gate's existing steps are unchanged.
