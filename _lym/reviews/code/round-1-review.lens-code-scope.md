```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each diff change to a decision (D1–D7 and the arbiter's F13 entry) and each decision to the changes that build it.

**Per change: the decision it traces to**

- Rubric bullets in `loom-rubric-code-review.md`: D1.
- `loom-fragment-loosened-check.md`, the `stencils.go` registry entry, `markers_test.go`, and the `{{template "loom-fragment-loosened-check"}}` includes in the following briefs: D2 (card 11). The briefs are burler fix, darn write, gatefix darn, landing fixer, pr-review, sampler agent, weaver batch and verify-fix, and webster implementer and verify-fix.
- Plain-include resolution in `stencilstore` (`engineblock.go`, `doc.go`) and the switch to `ReadForEngine` in `prsession.go` and `samplerengine/spec.go`: forced by the D2 fragment (card 6).
- `fabricengine` `CommittedFileDiffs` and `CommitMessages`: D3 (card 1).
- `driftengine`: `JudgeFile`, `JudgeRequest`, `FileRuling`, `LoosenedRuling` and `filesFindings` in `judgement.go`, plus `loosened.go`, `records.go`, `verify.go`, `verdict.go`, `deps.go` and `doc.go`: D3 (cards 2 and 3).
- The judge stencil, `driftwiring.go`, `Env.Drift.ReviewBase = verifyMergeBase`, the `shedrecipe` drift entry seam checks, and the `envkit` and `battencli` nil-seam entries: D3.
- `Config.PRReviewSmallMax`, `template.yaml`, the `config_test.go` row, and the `docs/overview.md` hunks: D5 (card 4). The overview lines on the verb and the friction note are forced docs for D4.
- Send-back mark, fix-here count, send-back count and counted-sets fields in `pullrequest/record.go`: D4 (card 5).
- `prfindings.go`, `cli.go`, `arm.go` (verb not read as a run-id), the `loomcli/doc.go` text, and `sendBackEdge`: D4 (card 8).
- `prfindings.go` friction note (`writeSendBackNote`): D6.
- `landingshed` `settleSendBack`, `reroutePersisted`, `sendBack`, the wake hook, the `ShedStatus` seam and its wiring in `landingdeps.go`: D4 and the F13 entry (card 9).
- `on_reroute: Code-Bouncer` in the three recipes, the `testdata` yaml, the recipe comments, and the `loomrecipe` doc and tests: D4 (card 10).
- `ReviewInput` replaced by `ReviewInputs` and `Full` in `amendment.go`, `bouncer.go`, `settlerecord.go`, `recipe.go`, `entries_bouncer.go` and `subjects.go`, with `ReviewInputNote`, the `review_input` rubric marker, and the `shed-recipe-spec.md` text: D4 (card 7).
- `Rubric` seam in `burlerreview.go` and `reviewRubricNotes` in `entries_burler.go`: forced by D4's review-input note. The note is written after the rubric is built at row construction, so the rubric needs a per-round refill.
- Rubric `## Review input` text: D6.
- PR-Review brief's pr-findings rule: D4.
- Describe brief and darn writer figures rule: D7.
- Test-only edits (`fix_test.go`, `template_test.go`, `commitrecords_test.go` `DeepEqual`, `wiring_*_test.go`): forced by the traced changes.

**Per decision: the changes that build it**

- D1: built.
- D2: built through the fragment and every code-committing brief. The plan's shared decision says no Go-written role commit exists, so the Go-appends-trailers clause has no site and is not a gap.
- D3: built (cards 1–3).
- D4: built (cards 5 and 7–10). This covers the verb, the row's deferred clear, the full generation, the `ReviewInputs` seam, the review-input note, the PR-Review brief rule and the recipe edges.
- D5: built.
- D6: built (friction note, rubric reading).
- D7: built.
- F13 entry: built (`settleSendBack`, matching the file in the newest own history entry).

**Findings**

None.

Two minor additions beyond the decision text, both forced and not findings: `CountedSets` in the record makes a re-run over the same file count once, and the generation record still reads an older single `review_input` key, which `PATTERN-tolerant-records` requires.

**Tool calls:** none. My context already held every diff piece, so I read nothing further.
