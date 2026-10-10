# Card 4 — small-modules-form

**Edit:**
- `internal/burlercli#Command`
- `internal/burlercli#burlerCLI.runCmd`
- `internal/burlercli#burlerCLI.awaitReviewCmd`
- `internal/burlercli#burlerCLI.validateReviewCmd`
- `internal/configcli#Command`
- `internal/configcli#buildConfigLong`
- `internal/gatecli#newCommand`
- `internal/idecli#Command`
- `internal/quarrycli#Command`
- `internal/quarrycli#newExpandCmd`
- `internal/quarrycli#newGlyphsCmd`
- `internal/quarrycli#newResolveCmd`
- `internal/quarrycli#newTOCCmd`
- `internal/selfreportcli#Command`
- `internal/shuttlecli#Command`
- `internal/shuttlecli#shuttleCLI.runCmd`
- `internal/shuttlecli#shuttleCLI.sendCmd`
- `internal/shuttlecli#shuttleCLI.interruptCmd`
- `internal/shuttlecli#shuttleCLI.stateCmd`
- `internal/stencilcli#Command`
- `internal/stencilcli#newDiffCmd`
- `internal/stencilcli#newPromoteCmd`

**Uses:**
- `internal/clihelp#AudienceAnnotation`
- `internal/clihelp#AudienceOperator`
- `internal/clihelp#AudienceRole`
- `internal/clihelp#AudienceInternal`

**Intent:** Bring the eight small modules to the D3, D6, D8 and D9 form, one package at a time inside the card: every leaf and runnable group gets its `audience` annotation, every `Short` starts lower-case, is one line, ends without a period and says when to reach for the command, every `Use` is the name, then `<arg>`, `[<arg>]`, `<arg>...` or `[<arg>...]` tokens, with no optional flag, and every `Long` stays under the cap with its rationale moved to the package doc.
The trim set is measured, not listed: before editing, the implementer measures every page's `Long` in the eight packages (`lyx <path> --help --json | jq -r .long | wc -c`, groups included) and trims every page over 2,000 bytes; when this plan was written none was over.
The `burler` group carries no audience: its `--stencils-dir` and `--target-dir` are persistent flags for its verbs, so it is not invocable (card 1's `Invocable`).
`config [module]` becomes `config [<module>]`; the config command is a runnable group (its own `RunE` plus `--print` and `--set`) and carries `operator`, as do `reconcile` and `menu`; `buildConfigLong` is measured and trimmed under the cap if it exceeds it, keeping the editor variables, the `--set` form and the module names `TestConfigLong` asserts.
`gate test` becomes `test <packages>...`, with `-C` and `--tags` left to `Flags:` and the `--` pass-through explained in `Long`; it carries `operator`.
`quarry` and `stencil` lower-case their `Short`s (every one is capitalised today), `stencil diff [name]` becomes `diff [<name>]`, and `ide`'s three `Short`s are lower-cased.
Audiences: `burler run`, `burler validate-review` and `burler await-review` are `role` (named only by the burler stencils); `quarry` verbs, `stencil` verbs, `ide` verbs, `selfreport create`, `shuttle state` and `shuttle run` are `operator`; `shuttle send` and `shuttle interrupt` are `role`, since only spawned agents and lyx drive another strand's pane.
The implementer confirms each value once against who calls the verb, taking the board entry's Part 1 table (`lyx board get '{"slug":"cli-surface"}'`) as the starting classification; the value lives in the annotation and nowhere else.
Every reworded `Short` or `Use` is grepped for across the repository, and any test asserting the old text joins the card.
Coverage for the form itself is the tree-walk test card 16 adds; this card changes help text only and relies on each package's existing tests.

**ImpactSummary:** Help text and annotations in twenty-two command builders across eight packages; no runtime behavior changes.
