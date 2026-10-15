# Lens review

Every lens fork returned a report.

- criteria-coverage, MEDIUM, `decision add` ignoring a malformed list has no test: fixed as proposed; card 3 adds a `relevant` field and a malformed-list row to `TestAppendDecision_AppendsAtEndOfDecisionsSection`.
- criteria-coverage, LOW, D1's gate re-prompt update of the list is unworded: fixed as proposed; card 6 has Step 6's gate re-prompt fix say a design change may update `{{.relevant_path}}`.
- criteria-coverage LOW, branch-coverage MEDIUM and row-reachability MEDIUM (merged), `CheckRelevant`'s read error through `ValidateLoomDiscussion` reaches no gate or verb row: fixed as proposed; card 4 adds the parity case `Error_RelevantIsDirectory` (verdict error) and the `TestValidateDiscussionCmd` row `IOFault_RelevantIsDirectory`.
- branch-coverage, LOW, the verb's I/O error envelope names the decision record path when the relevant list's read fails: fixed as proposed; card 4 makes the prefix `loom: validate discussion: `, and `IOFault_RelevantIsDirectory` asserts the message contains `relevant.md`.
- branch-coverage LOW and row-reachability LOW (merged), short-circuit and combined-findings order unstated and untested: fixed as proposed; card 4 states `CheckRelevant` runs after `Validate`'s findings, and its new `TestNewDiscussionGate` subtest pairs a missing heading with a stray-line list and asserts both check names in order.
- branch-coverage, LOW, a bare `#` line and a bare `- ` line have no `TestCheckRelevant` row: fixed as proposed; card 3 adds both rows.
- row-reachability, MEDIUM, card 5's three-path assertion stops guarding `RelaunchFiles` once card 6 puts the relevant path in the prompt: fixed as proposed; card 5 asserts `finishes writing ` immediately followed by the `RelaunchFiles` phrase.
- row-reachability LOW and test-economy LOW (merged), the `TestValidate_CleanRecords` row needs an unlisted field and cannot fail since `Validate` takes no relevant path: fixed as proposed; card 3 drops the row and leaves `TestValidate_CleanRecords` unedited, with darn covered by card 4's `Darn_MalformedRelevantIgnored` and `decision add` by the new `AppendDecision` row.
- test-economy, LOW, card 4's `TestNewDiscussionGate` malformed-list subtest repeats parity's `Stuck_RelevantMalformed`: fixed differently; the proposed drop would leave the combined-findings order unpinned, so the subtest is rewritten to the missing-heading plus stray-line case above, which no other row reaches.
- test-economy, LOW, parity's `Darn_Clean_RelevantMalformedIgnored` adds nothing since `NewDarnDiscussionGate` takes no relevant path (`internal/loomcli/parity_test.go:159-160`): fixed as proposed; card 4 drops the case.
- forced-edits, MEDIUM, `pattern/PATTERN-gate-self-check-parity.md:7` still pairs the loom discussion gate with `discussionparser.Validate`: fixed as proposed; card 4 lists the file and names `loomshed.ValidateLoomDiscussion`.
- forced-edits, LOW, `docs/overview.md:446-448` and `:488` go stale: fixed as proposed; card 4 lists `docs/overview.md` and rewords the discussionparser entry and the `validate-discussion` paragraph.
- forced-edits, LOW, `resolveGateSpec`'s doc comment (`internal/shedrecipe/entries_gate.go:29`) omits the new requirement: fixed as proposed; card 4 lists `internal/shedrecipe#resolveGateSpec` and adds `env.RelevantPath`.
- forced-edits, LOW, the `AdvisorNotesAreNotRead` comment in `TestNewDiscussionGate` (`internal/loomshed/gates_test.go:132`) turns false: fixed as proposed; card 4's Intent rewords it.
