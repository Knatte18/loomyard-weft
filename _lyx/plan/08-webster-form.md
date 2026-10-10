# Card 8 — webster-form

**Create:**
- `internal/webstercli/doc.go`

**Edit:**
- `internal/webstercli#Command`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli#websterCLI.validateCmd`
- `internal/webstercli#websterCLI.runCmd`
- `internal/webstercli#websterCLI.statusCmd`
- `internal/webstercli#websterCLI.pauseCmd`
- `internal/webstercli#websterCLI.beginBatchCmd`
- `internal/webstercli#websterCLI.awaitBatchCmd`
- `internal/webstercli#websterCLI.recordBatchCmd`
- `internal/webstercli#websterCLI.recoverBatchCmd`
- `internal/webstercli#websterCLI.acceptAuditCmd`
- `internal/webstercli#websterCLI.rebaselineCmd`
- `internal/webstercli#websterCLI.restorePlanCmd`
- `internal/webstercli#websterCLI.verifyCmd`

**Uses:**
- `internal/clihelp#AudienceAnnotation`
- `internal/clihelp#AudienceOperator`
- `internal/clihelp#AudienceRole`
- `internal/clihelp#AudienceInternal`

**Intent:** Bring `webster` to the D3, D6, D8 and D9 form.
`reset --to start|pre-fix|... [--batch NN]` becomes `reset`, with `--to` and `--batch` in `Flags:` and the five targets listed in `Long`; the verb's own refusal keeps naming the targets, so no flag is newly marked required.
`begin-batch <NN>`, `await-batch <NN>`, `record-batch <NN>` and `recover-batch <NN>` become `<nn>`, since an argument name is lower-case.
`Short`s starting upper-case ("Master's bracket call...") are reworded in lower case.
The trim set is measured, not listed: before editing, the implementer measures every webster page's `Long` (`lyx webster <verb> --help --json | jq -r .long | wc -c`, the group included) and trims every page over 2,000 bytes; when this plan was written that set was the `webster` group (which also drops its verb table, since cobra lists the subcommands), `reset` and `validate`.
Their rationale moves into the new `internal/webstercli/doc.go`, the package doc, after each trimmed page's refusals are checked to still name their way forward in the message.
Audiences: `status`, `reset`, `verify`, `validate`, `run`, `pause` and `restore-plan` are `operator` (`status` and `reset` are the record's borderline diagnosis verbs, decided operator); `begin-batch`, `await-batch`, `record-batch`, `recover-batch`, `accept-audit` and `rebaseline` are `role`, the operator's named example of Master's bracket verbs.
The `webster` group carries no audience: its `--stencils-dir`, `--plan-dir` and `--target-dir` are persistent flags for its verbs, so it is not invocable (card 1's `Invocable`).
`TestCommand_LongStringsHaveNoStaleBatchLanguage` keeps passing over the trimmed texts unchanged; every other reworded string is grepped for and any asserting test joins the card.
Coverage for the form, the audience annotations included, is card 16's tree-walk test.

**ImpactSummary:** Help text and annotations across fourteen webster builders, three pages trimmed, one package doc created; no runtime behavior changes.
