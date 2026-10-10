# Card 7 — loom-form

**Create:**
- `internal/loomcli/doc.go`

**Edit:**
- `internal/loomcli#loomVerbTexts`
- `internal/loomcli#Command`
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli#loomCLI.resumeCmd`
- `internal/loomcli#loomCLI.circlingCmd`
- `internal/loomcli#loomCLI.decisionCmd`
- `internal/loomcli#loomCLI.reviewCmd`
- `internal/loomcli#loomCLI.rejectCmd`
- `internal/loomcli#loomCLI.approveCmd`
- `internal/loomcli#loomCLI.commitRecordsCmd`
- `internal/loomcli#loomCLI.lintCommentsCmd`
- `internal/loomcli#loomCLI.validatePlanCmd`
- `internal/loomcli#loomCLI.validateDiscussionCmd`
- `internal/loomcli#loomCLI.validateDescriptionCmd`
- `internal/loomcli/approve_awaiting_test.go`

**Uses:**
- `internal/clihelp#AudienceAnnotation`
- `internal/clihelp#AudienceOperator`
- `internal/clihelp#AudienceRole`
- `internal/clihelp#AudienceInternal`
- `internal/loomcli#StartAliasCommand`

**Intent:** Bring `loom` to the D3, D6, D8 and D9 form.
`loomVerbTexts` rewrites `run [run-id]`, `step [run-id]`, `status [run-id]`, `pause [run-id]` to the `[<run-id>]` form and `goto [run-id] --to <producer>` to `goto [<run-id>]`, and sets each verb's `Audience`: `status`, `pause` and `goto` are `operator`, `run` and `step` are `role` (the driver's verbs).
`circling continue|accept [slug]`, `decision add [slug]`, `review notify|delivered|approve|reject [slug]` adopt `[<slug>]`, and `review delivered` and `review approve` drop their optional flags from `Use`; `reject <review-file>` and `review reject [<slug>] <review-file>` already fit.
Audiences: `start` (and so the root `start` alias, which shares the same builder), `resume`, `approve`, `reject`, `circling continue|accept`, `decision add` and `validate-plan`, `validate-discussion`, `validate-description`, `lint-comments` are `operator`, the last four because an operator or a writer agent runs them standalone; `commit-records`, `review notify` and `review delivered` are `role`; `review approve` and `review reject` are `operator` (the parent-review fork submits with them, on the orch's behalf).
The trim set is measured, not listed: before editing, the implementer measures every loom page's `Long` (`lyx loom <path> --help --json | jq -r .long | wc -c`, the group and nested groups included) and trims every page over 2,000 bytes to what the verb does, when to use it, its arguments and flags and one or two examples; when this plan was written that set was `start`, the `loom` group (which also stops restating its subcommands in prose) and `resume`.
Their rationale and refusal narratives move into the new `internal/loomcli/doc.go`, the package doc, after each trimmed page's refusals are checked to still name their way forward in the message.
`TestApproveCmd_HelpNamesAwaitingOrBlocked` is updated to the trimmed text if its asserted sentence moves; every other reworded string is grepped for across the repository and any asserting test joins the card.
Coverage for the form is card 16's tree-walk test.

**ImpactSummary:** Help text and annotations across the loom verb texts and thirteen builders, three pages trimmed, one package doc created; no runtime behavior changes.
