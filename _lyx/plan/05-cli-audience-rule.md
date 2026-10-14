# Card 5 — cli-audience-rule

**Prosa:**
- `pattern/PATTERN-cli-cobra.md`
- `docs/overview.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Intent:** D7: the runnable-group audience rule is written where plans read it, so a plan that shapes a CLI command against the `cmd/lyx/clitree_test.go` rules is caught at Plan-Review (#588).

`pattern/PATTERN-cli-cobra.md`'s audience bullet states `clihelp.Invocable`'s rule: a command with subcommands is invocable only when it is runnable and its `Use` carries a token after its name or it has a visible local flag.
Any other group carries no audience, prints help on a bare call and refuses a mistyped subcommand, so behaviour meant for the group name belongs on a leaf such as `show`.

`docs/overview.md`'s audience paragraph (the "Every invocable command also carries an `audience` annotation" sentence under "Module dispatch") points at that bullet with an inline link to `../pattern/PATTERN-cli-cobra.md`, which `PATTERN-markdown-link-integrity` checks.

`contracts/stencils/loom/loom-rubric-plan-review.md`'s "Also flag" list gains, after the `Custom` item, one item that points at the bullet without restating it:

```
- **CLI shape.**
  A card that adds or reshapes a `lyx` CLI command is checked against the audience bullet of `pattern/PATTERN-cli-cobra.md`; a command shape that bullet rules out is a finding.
```

D6 lands no code: `internal/lyxtemp`'s `TestSweep` does not reproduce on `78e31aa06`'s bounded retry, so this card's commit body closes #584 citing that retry and the two green runs the decision record names (`-race -count=1000` and `-count=300`).

**Commit:** `5: Write the runnable-group audience rule where plans read it (#588); closes #584`
