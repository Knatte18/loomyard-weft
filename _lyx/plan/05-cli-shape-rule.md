# Card 5 — cli-shape-rule

**Prosa:**
- `pattern/PATTERN-cli-cobra.md`
- `docs/overview.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Uses:**
- `internal/clihelp#Invocable`

**Intent:** D7, #588: the runnable-group audience rule, which lived only in `clihelp.Invocable` and `cmd/lyx/clitree_test.go`, is written where a plan writer and Plan-Review read it.

`pattern/PATTERN-cli-cobra.md`'s audience bullet states `clihelp.Invocable`'s rule: a command with subcommands is invocable only when it is runnable and its `Use` carries a token after its name or it has a visible local flag.
Any other group carries no audience, prints help on a bare call and refuses a mistyped subcommand, so behaviour meant for the group name belongs on a leaf such as `show`.
`docs/overview.md`'s audience paragraph gains a sentence pointing at that bullet with a relative link, `../pattern/PATTERN-cli-cobra.md`, which `PATTERN-markdown-link-integrity` checks.
The Plan-Review rubric's `## Also flag` list gains, after `Custom` is a last resort, one check that points at the bullet without restating it, per `PATTERN-producer-pointer-rule`:

```
- **CLI shape.**
  A card that adds or reshapes a `lyx` CLI command is checked against the audience bullet of `pattern/PATTERN-cli-cobra.md`; a command shape that bullet rules out is a finding.
```

No mechanical plan check reads a command's shape, since card prose carries no `Use`, flags or annotations to check.

D6 lands with this card: `internal/lyxtemp` is not changed, and this commit closes #584, since on current code `lyx gate test ./internal/lyxtemp -- -race -run TestSweep -count=1000` and the same with `-count=300` and no `-race` pass over `78e31aa06`'s bounded retry.

**Commit:** `5: The runnable-group audience rule is written where plans read it; closes #584 (78e31aa06's bounded retry passes TestSweep -race -count=1000 and -count=300)`
