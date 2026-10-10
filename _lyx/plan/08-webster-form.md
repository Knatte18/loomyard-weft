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
- `internal/webstercli/cli_test.go`

**Uses:**
- `plan:internal/clihelp#AudienceAnnotation`
- `plan:internal/clihelp#AudienceOperator`
- `plan:internal/clihelp#AudienceRole`
- `plan:internal/clihelp#AudienceInternal`

**Intent:** Bring `webster` to the D3, D6, D8 and D9 form.
`reset --to start|pre-fix|... [--batch NN]` becomes `reset`, with `--to` and `--batch` in `Flags:` and the five targets listed in `Long`; the verb's own refusal keeps naming the targets, so no flag is newly marked required.
`begin-batch <NN>`, `await-batch <NN>`, `record-batch <NN>` and `recover-batch <NN>` become `<nn>`, since an argument name is lower-case.
`Short`s starting upper-case ("Master's bracket call...") are reworded in lower case.
`Long` texts over the cap are trimmed: the `webster` group (which also drops its verb table, since cobra lists the subcommands), `reset` and `validate` today; their rationale moves into the new `internal/webstercli/doc.go`, the package doc, after each trimmed page's refusals are checked to still name their way forward in the message.
Audiences: `status`, `reset`, `verify`, `validate`, `run`, `pause` and `restore-plan` are `operator` (`status` and `reset` are the record's borderline diagnosis verbs, decided operator); `begin-batch`, `await-batch`, `record-batch`, `recover-batch`, `accept-audit` and `rebaseline` are `role`, the operator's named example of Master's bracket verbs.
`TestCommand_LongStringsHaveNoStaleBatchLanguage` keeps passing over the trimmed texts, and gains the assertion that every verb carries an audience from the closed set; every other reworded string is grepped for and any asserting test joins the card.
Coverage for the form is card 16's tree-walk test.

**ImpactSummary:** Help text and annotations across fourteen webster builders, three pages trimmed, one package doc created; no runtime behavior changes.
