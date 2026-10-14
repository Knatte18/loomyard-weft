# Card 5 — cli-audience-rule

**Prosa:**
- `pattern/PATTERN-cli-cobra.md`
- `docs/overview.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Intent:**
D7: the runnable-group audience rule is written where a plan writer and Plan-Review read it, since no mechanical plan check can read a command's shape from card prose.

`pattern/PATTERN-cli-cobra.md`'s audience bullet states `clihelp.Invocable`'s rule: a command with subcommands is invocable only when it is runnable and its `Use` carries a token after its name or it has a visible local flag.
Any other group carries no audience, prints its help on a bare call and refuses a mistyped subcommand, so behaviour meant for the group name belongs on a leaf such as `show`.

`docs/overview.md`'s audience paragraph (the one opening "Every invocable command also carries an `audience` annotation") gains a sentence pointing at that bullet, as the link `[PATTERN-cli-cobra](../pattern/PATTERN-cli-cobra.md)` the file already uses, so the docs link-integrity test still resolves it.

`contracts/stencils/loom/loom-rubric-plan-review.md`'s "Also flag" list gains one bullet after `**`Custom` is a last resort.**`, pointing at that bullet without restating it, per `PATTERN-producer-pointer-rule`, in the wording the hub's deployed rubric already carries:

```
- **CLI shape.**
  A card that adds or reshapes a `lyx` CLI command is checked against the audience bullet of `pattern/PATTERN-cli-cobra.md`; a command shape that bullet rules out is a finding.
```

D6 makes no code change: on current code `lyx gate test ./internal/lyxtemp -- -race -run TestSweep -count=1000` and the same with `-count=300` and no `-race` pass.
This card's commit body closes #584, citing `78e31aa06`'s bounded sweep retry and those two runs.

**Commit:** 5: docs: state the runnable-group audience rule where plans read it
