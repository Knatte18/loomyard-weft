# Card 5 — The runnable-group audience rule is written where plans read it

**Prosa:**
- `pattern/PATTERN-cli-cobra.md`
- `docs/overview.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Intent:**
D7 (#588): no mechanical plan check can read a command's shape from card prose, and the rule that decides a group's audience lived only in `clihelp.Invocable` and `cmd/lyx/clitree_test.go`.

- `pattern/PATTERN-cli-cobra.md`'s audience bullet states `clihelp.Invocable`'s rule: a command with subcommands is invocable only when it is runnable and its `Use` carries a token after its name or it has a visible local flag.
  Any other group carries no audience, prints help on a bare call and refuses a mistyped subcommand, so behaviour meant for the group name belongs on a leaf such as `show`.
- `docs/overview.md`'s audience paragraph, the one opening `Every invocable command also carries an `audience` annotation`, points at that bullet with a relative link to `../pattern/PATTERN-cli-cobra.md`.
- `contracts/stencils/loom/loom-rubric-plan-review.md`'s `## Also flag` list gains one bullet, `**CLI shape.**`: a card that adds or reshapes a CLI command is checked against the audience bullet of `pattern/PATTERN-cli-cobra.md`, which it points at without restating, per `PATTERN-producer-pointer-rule`.

D6 rides on this commit: on current code `lyx gate test ./internal/lyxtemp -- -race -run TestSweep -count=1000`, and the same with `-count=300` and no `-race`, pass, so `internal/lyxtemp` does not change.
The commit body closes #584, citing `78e31aa06`'s bounded retry and those two runs, and closes #588.

**Commit:** 5: write the runnable-group audience rule where plans read it

**Verify:** lyx gate test ./internal/lyxcwd
