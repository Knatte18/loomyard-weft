# Card 5 — cli-runnable-group-rule

**Prosa:**
- `pattern/PATTERN-cli-cobra.md`
- `docs/overview.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Intent:** D7: the runnable-group audience rule is written where a plan writer and Plan-Review read it, so a plan that shapes a CLI command against `cmd/lyx/clitree_test.go`'s rules is caught before implementation (#588).

- `pattern/PATTERN-cli-cobra.md`'s audience bullet states `clihelp.Invocable`'s rule.
  A command with subcommands is invocable only when it is runnable and its `Use` carries a token after its name, or it has a visible local flag.
  Any other group carries no audience, prints help on a bare call and refuses a mistyped subcommand, so behavior meant for the group name belongs on a leaf such as `show`.
- `docs/overview.md`'s audience paragraph, the one naming `operator`, `role` and `internal`, gains a sentence pointing at that bullet through the link `../pattern/PATTERN-cli-cobra.md`, the spelling its other pattern links use, so `PATTERN-markdown-link-integrity` holds.
- The Plan-Review rubric gains one "Also flag" bullet, **CLI shape**.
  It says a card that adds or reshapes a `lyx` CLI command is checked against the audience bullet of `pattern/PATTERN-cli-cobra.md`, and it does not restate the rule (`PATTERN-producer-pointer-rule`).
  The path sits under none of the prefixes `contracts/stencils/citation_enforcement_test.go` scans.

No mechanical plan check reads a command's shape from card prose, so the rule is a review check only.
`lyx loom lint-pattern`, which the card gate runs, passes with the background's new text.

This card's commit also carries D6, which changes no file: its body adds a line closing #584, citing `78e31aa06`'s bounded retry and the two passing runs on current code, `lyx gate test ./internal/lyxtemp -- -race -run TestSweep -count=1000` and the same with `-count=300` and no `-race`.

**Commit:** 5: Write the runnable-group audience rule where plans are written and reviewed (closes #588, #584)
