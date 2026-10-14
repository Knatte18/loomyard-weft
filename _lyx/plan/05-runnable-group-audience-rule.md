# Card 5 — runnable-group-audience-rule

**Prosa:**
- `pattern/PATTERN-cli-cobra.md`
- `docs/overview.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Edit:**
- `contracts/stencils#wordingClaims`

**Intent:** D7: the runnable-group audience rule is written where plans read it (#588), and D6's lyxtemp disposition rides on this commit's message.

- `pattern/PATTERN-cli-cobra.md`'s audience bullet states `clihelp.Invocable`'s rule: a command with subcommands is invocable only when it is runnable and its `Use` carries a token after its name or it has a visible local flag; any other group carries no audience, prints help on a bare call and refuses a mistyped subcommand, so behaviour meant for the group name belongs on a leaf such as `show`.
- `docs/overview.md`'s audience paragraph (the one naming the `operator`, `role` and `internal` annotation) gains one sentence pointing at that bullet of `pattern/PATTERN-cli-cobra.md` for which commands are invocable, without restating it.
- `contracts/stencils/loom/loom-rubric-plan-review.md`'s `## Also flag` list gains one bullet, `**CLI shape.**`, after `Custom` is a last resort: a card that adds or reshapes a `lyx` CLI command is checked against the audience bullet of `pattern/PATTERN-cli-cobra.md`, and a command shape that bullet rules out is a finding; it points at the bullet and never restates the rule (`PATTERN-producer-pointer-rule`).
- `wordingClaims`' `loom-rubric-plan-review.md` entry gains one claim, `must: "audience bullet of `pattern/PATTERN-cli-cobra.md`"`, with the why "a card shaping a CLI command is checked against the cobra pattern's audience rule, by pointer".

`lyx loom lint-pattern` passes over the PATTERN text of this card and card 1.

D6: `internal/lyxtemp` is not changed.
This card's commit message body closes #584, citing `78e31aa06`'s bounded retry and that `lyx gate test ./internal/lyxtemp -- -race -run TestSweep -count=1000` and the same with `-count=300` and no `-race` pass on current code.

**ImpactSummary:** three prose files gain the rule or a pointer to it and one rubric claim pins the pointer; no code path changes.

**Commit:** `5: write the runnable-group audience rule where plans read it (#588; closes #584)`
