# Card 5 — cli-shape-rule

**Prosa:**
- `pattern/PATTERN-cli-cobra.md`
- `docs/overview.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Uses:**
- `internal/clihelp#Invocable`

**Intent:** D7: the runnable-group audience rule is written where plans and their reviewers read it, since no mechanical plan check can read a command's shape from card prose.

- `pattern/PATTERN-cli-cobra.md`'s audience bullet states `clihelp.Invocable`'s rule: a command with subcommands is invocable only when it is runnable and its `Use` carries a token after its name or it has a visible local flag.
  Any other group carries no audience, prints help on a bare call and refuses a mistyped subcommand, so behaviour meant for the group name belongs on a leaf such as `show`.
- `docs/overview.md`'s audience paragraph (the `audience` annotation sentence under Module dispatch) points at that bullet.
- The Plan-Review rubric gains, under `## Also flag` after `Custom` is a last resort, the check: "**CLI shape.** A card that adds or reshapes a `lyx` CLI command is checked against the audience bullet of `pattern/PATTERN-cli-cobra.md`; a command shape that bullet rules out is a finding."
  It points at the bullet and restates no part of it (`PATTERN-producer-pointer-rule`).

D6 lands no code: this card's commit message body closes #584, citing `78e31aa06`'s bounded retry and the two passing runs on current code, `lyx gate test ./internal/lyxtemp -- -race -run TestSweep -count=1000` and the same with `-count=300` and no `-race`.

**Commit:** `5: cli-shape-rule`
