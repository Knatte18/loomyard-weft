# Card 16 — cli-form-guard

**Edit:**
- `cmd/lyx#newRoot`
- `cmd/lyx/clitree_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `PATTERN.md`
- `pattern/PATTERN-cli-cobra.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/clihelp#Invocable`
- `plan:internal/clihelp#Audiences`
- `plan:internal/clihelp#AudienceAnnotation`
- `plan:internal/clihelp#AudienceInternal`
- `plan:internal/clihelp#IndexNoteAnnotation`

**Intent:** Pin the form every earlier card produced with one tree-walking test, and record the three obligations in PATTERN (D3, D6, D8, D9, D10).
`clitree_test.go` gains `TestCLIForm_EveryCommand`, which walks `newRoot()` with the shared walker, runs no invocation and `t.Parallel`s, and reports every problem per command path from four pure checks, each also run as subtests over small synthetic cobra trees so the negative cases are proven: the audience check fails on an invocable command (per `clihelp.Invocable`) without the annotation, a value outside `clihelp.Audiences`, a non-invocable group carrying one, and a hidden invocable command not `internal`, with a passing row for a `GroupRunE` group whose only flags are its own persistent ones, which is not invocable and carries none; the `Use` check splits on whitespace except that a brace-balanced `{…}` span is one token, requires the first token to equal the command's name, every further token to match `<arg>`, `[<arg>]`, `<arg>...`, `[<arg>...]` or `--flag <value>` with the arg name lower-case letters, digits and hyphens, a `--flag` token to name a flag cobra marks required (the `cobra.BashCompOneRequiredFlag` annotation), and a `{…}` token only to balance its braces; the `Short` check requires a lower-case first letter, one line and no final period; the `Long` check caps the byte length at one test constant, `maxLongBytes = 2000`, with the root exempt from nothing but the D9 subcommand-list rule, which is review discipline.
The `completion` subtree is skipped as the walker already does, and `help` is checked like any command.
`newRoot`'s `Short` ("Loomyard task-tracker CLI") is reworded in lower case to say when to reach for `lyx`, the one command the earlier cards left; its `Long` keeps its "Available modules" list.
`PATTERN.md`'s `PATTERN-cli-cobra` line and `pattern/PATTERN-cli-cobra.md` grow the three obligations: every invocable command carries an `audience` from the closed set, `Use` and `Short` follow the form above, and `Long` sits under the cap, all pinned by the `cmd/lyx` tree walk; no new entry is added.
`docs/overview.md`'s module dispatch paragraph names the form and the index beside the three steps of adding a module.
`cmd/lyx/testdata/test-budget.yaml` raises `cmd/lyx` by one.

**ImpactSummary:** One root `Short`, one new guard test over the whole tree and the PATTERN text; the guard fails any later command that breaks the form.
