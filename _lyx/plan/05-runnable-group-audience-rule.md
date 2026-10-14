# Card 5 — runnable-group-audience-rule

**Prosa:**
- `pattern/PATTERN-cli-cobra.md`
- `docs/overview.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Intent:** Write the runnable-group audience rule where plans and their reviewers read it (D7), since no mechanical plan check can read a command's shape from card prose and the rule lived only in `clihelp.Invocable` and `cmd/lyx/clitree_test.go`.

- `pattern/PATTERN-cli-cobra.md`'s audience bullet states `clihelp.Invocable`'s rule: a command with subcommands is invocable only when it is runnable and its `Use` carries a token after its name or it has a visible local flag.
  Any other group carries no audience, prints help on a bare call and refuses a mistyped subcommand, so behaviour meant for the group name belongs on a leaf such as `show`.
- `docs/overview.md`'s audience paragraph in "Module dispatch" points at that bullet with a relative link to `../pattern/PATTERN-cli-cobra.md`, which `PATTERN-markdown-link-integrity` checks.
- The Plan-Review rubric's "Also flag" list gains one bullet, "CLI shape", after "`Custom` is a last resort": a card that adds or reshapes a `lyx` CLI command is checked against the audience bullet of `pattern/PATTERN-cli-cobra.md`, and a command shape that bullet rules out is a finding.
  It points at the background and does not restate the rule, per `PATTERN-producer-pointer-rule`.

D6 rides on this commit: the commit message body closes #584 with no change to `internal/lyxtemp`, citing `78e31aa06`'s bounded retry and the two green runs on current code, `lyx gate test ./internal/lyxtemp -- -race -run TestSweep -count=1000` and the same with `-count=300` and no `-race`.

**Commit:** `5: the runnable-group audience rule is written where plans read it`
