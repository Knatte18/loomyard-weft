---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: The proposal's second option was taken, and only that option. The verb refuses `--amends-plan` and PATTERN-change calls while the scope record's judge session is open, and the exemption is not extended to cover their writes. The change, in `internal/loomcli/decision.go` and `internal/loomcli/driftwiring.go`, goes beyond the proposal's wording by adding a read-failure refusal and naming the halted case in the way forward. A sound exemption of `amendment.yaml` and the PATTERN bullet would need new append-only snapshot copies and a PATTERN-aware identity text.
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

## What changed

- F1, `internal/driftengine/expectation.go`: removed the unkeyed counter and the refusal of a `lines` range over a fully keyed baseline case, and the two sentences it added to the `ParseExpectation` comment.
  Removed the two sentences it added to `internal/driftengine/doc.go`.
  `seats.go`'s format text is kept.
  `TestParseExpectation` has an accepted row, `RangeOverFullyKeyedBaselineCase`, in place of the refusal row.
  `_lym/landing/summary.md` is rewritten around the format-text change and the F3 and F4 behaviour.
- F2: settled by F1.
  `ParseExpectation` no longer refuses such a range, so a sealed expectation with one parses again and no sealed bytes are re-judged under a new rule.
- F3, `internal/driftengine/judgesnapshot.go` and `internal/discussionparser/replacecheck.go`:
  `appendedDecisionRecord` takes the records `CheckoutState`.
  It grants the exemption only when the checkout lists the record as neither changed nor untracked.
  It also requires that every carry-over block of the snapshot's copy is present and unchanged, through the new `discussionparser.CheckCarryOversNotLost`.
  That function shares its body with `CheckCarryOversKept` and leaves out the added-block finding.
  `doc.go`, the `Deps.DecisionRecord` comment and `summary.md` state what still goes unseen.
  `TestCheckCarryOversNotLost` covers the new check.
  `TestJudgeSnapshot` gains rows for an uncommitted appended entry, an edited carry-over block and an entry beside a kept block.
- F4, `internal/loomcli`: `decisionVerb` refuses a call with `--amends-plan` or a PATTERN-change entry while the run's drift scope record holds an open judge session.
  The refusal's way forward says to re-run after Drift-Verify has ended its session, and names a running judge and a halted row.
  `driftJudgeSessionOpen` reads the mark, and reads nothing when no scope record exists.
  The `decision add` help text and the `decisionVerb` comment state the refusal.
  Tests: two `TestDecisionVerb_Refusals` rows, one `TestDecisionVerb_AmendsPlan` row and one `TestDecisionVerb_AppendsAndCommits` row showing a plain entry still lands while a judge session is open.
- F5, `internal/loomcli/export_test.go` and `driftwiring_test.go`: `TestDriftDeps_CarryTheDecisionRecord` checks that `driftDeps` passes the decision record path into `Deps.DecisionRecord`.
  No existing test built `driftDeps` with a set path, so the assertion is a new small test over a new export.
- F6, `internal/driftengine/judgesnapshot_test.go`: three `TestJudgeSnapshot` rows name an added entry in the three fail-closed setups, an unset `Deps.DecisionRecord`, a record absent at the snapshot, and a second snapshot that cleared an earlier copy.

The gate over `./internal/driftengine`, `./internal/discussionparser` and `./internal/loomcli` passed after the last edit of the fix itself.
A later gate run refused the round's new comments for wrapped sentence breaks and over-long lines.
The repair breaks each of them at a sentence boundary or shortens it, in these places:
`CheckCarryOversNotLost` in `internal/discussionparser/replacecheck.go`, the decision-record exemption text in `internal/driftengine/doc.go`,
the `Changes` and `appendedDecisionRecord` comments in `internal/driftengine/judgesnapshot.go`,
the `decisionVerb` comment in `internal/loomcli/decision.go`, and the `driftJudgeSessionOpen` and `TestDriftDeps_CarryTheDecisionRecord` comments in `internal/loomcli`.
No code changed in that repair.
A later verify run failed `TestCLIForm_EveryCommand`, because the sentence this round added to the `decision add` help text put its `Long` at 2046 bytes, over the cap of 2000.
That sentence is removed again and the refusal is stated in the `internal/loomcli/doc.go` package doc instead.
This undoes the help-text part of the fix-consistency self-check item above, which the package doc now carries.

## Self-check

- fix-consistency, LOW: `Deps.DecisionRecord` comment stated the old exemption. Fixed as proposed in `internal/driftengine/deps.go`.
- fix-consistency, LOW: `decision add` help text omitted the refusal. Fixed as proposed.
- fix-proposal, MEDIUM: a carry-over settle during a judge session would be named by the F3 check. Disputed, premise false: the only writers of a carry-over block are the review loop's bouncer calls at `internal/shedadapters/bouncer.go:766`, `internal/shedadapters/bouncer.go:1206` and `internal/shedadapters/bouncer.go:1326`, which run in review rows and never during a Drift-Verify row.
  The lens itself found no overlap.
- fix-proposal, MEDIUM: the F4 refusal during a halted Drift-Verify row with an open mark gives no hint that a halt is no running judge. Fixed with an own change: the way forward names waiting for a running judge and `lym loom resume` for a halted row.
  The refusal itself stays, since a write made while halted is still compared against the snapshot at the resume.
- fix-proposal, LOW: `ReadScope` creates the ephemeral drift directory, and a corrupt `scope.json` makes the calls refuse. Fixed in part: the seam stats the scope file first and reads nothing when it is absent.
  The refusal on an unreadable scope record stays, since the verb cannot tell whether a judge session is open then.
- fix-proposal, LOW: no test pins the `seats.go` format sentence. Not acted on: the finding asks for an improvement the review never asked for and the fix did not make worse, which the rubric excludes.
- fix-code-correctness, LOW: `CheckCarryOversNotLost` comments passed 120 characters and cited `CheckCarryOversKept`. Fixed as proposed.
- fix-code-correctness, gap: no test showed a plain `decision add` succeeding while a judge session is open. Fixed with the added `TestDecisionVerb_AppendsAndCommits` row.
- fix-code-correctness, LOW, unconfirmed: the refusal's text applies to a halted row too. Covered by the fix-proposal change above.
- fix-affected-set: no findings.

## Disputed

- fix-proposal MEDIUM on the carry-over settle: ground `premise false`.
  Evidence: `internal/shedadapters/bouncer.go:766`, `internal/shedadapters/bouncer.go:1206` and `internal/shedadapters/bouncer.go:1326` are the only callers that reach `WriteCarryOver`, through `internal/loomcli/wiring.go:1156`, and they belong to the review loop.
  This disputes a fork finding, not a finding of the review.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached this round.
