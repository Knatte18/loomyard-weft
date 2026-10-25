# Discussion scope in lenses, parent-only questions to the parent, Parent-Align

All nine batches recorded `done`.
No batch needed recovery, and no digest listed a `Pattern-Departure:` trailer.

- **01-decision-note-title** (head `119d1f59`): `DecisionNoteTitle` moved into `internal/orchengine`, and `internal/reportengine` gained read-only decoders of archived parent-review and alignment records.
  Deviation: `internal/reportengine/archivedrecords.go`.
- **03-drop-parent-review-gate** (head `73081967`): the parent-review gate kind, `Env.ParentReview` and the loom wiring of the gate and the aligner are gone.
  Deviations: `internal/loomrecipe/fixture_test.go`, `internal/shedrecipe/entries_discussionwrite_test.go`.
- **04-drop-review-verbs** (head `6d8122ec`): `lyx loom review`, the status waiting note and batten's `ReviewWait` seam are removed.
  Deviations: `internal/loomcli/cli.go`, `internal/loomcli/review_test.go`, `internal/loomcli/reviewstatus_test.go`.
- **05-delete-alignment-and-parent-review** (head `7dbcc200`): the two packages, the loomengine prompts and the five stencils are deleted, and the four loom.yaml keys are retired through `MigrateConfig`.
  No deviations.
- **07-scope-lenses** (head `e1d55c84`): the scope lenses, the `decision` class in the rubric, the writer and darn-writer rules, the carry-over `AddedSince` and the Bouncer's `decision` escalation.
  Deviations: `internal/shedadapters/bouncer.go`, `bouncer_decision_test.go`, `bouncercarryover.go`, `bouncerdecision.go`, `escalation.go`.
- **13-discussion-decision-switches** (head `5a372b2a`): the `escalate_decisions` and `decisions_escalated` row keys, the arbiter's refusal of a decision escalation and the circling verbs' naming of the cause, and `merge_fix_attempts` renamed to `parent_align_fix_attempts` with its migration.
  Deviation: `docs/overview.md`.
- **16-parent-align-row** (head `c7c794b8`): the Merge-In row, producer, engine key, fixer mode and gate key are renamed, and runs parked on the old row carry across.
  Deviations: `internal/gateslot/config.go`, `internal/landingshed/{deps,finalize,fixer,fixerattempts,verifygate}.go`, `internal/loomcli/{landingdeps,start,subjects,wiring}.go`, `internal/loomcli/peers_test.go`, `internal/loomcli/start_midmerge_test.go`, `internal/shedrecipe/entries_simple_test.go`.
- **17-parent-align-prose** (head `c4306bce`): Merge-In renamed in docs, specs, pattern background files and comments.
  No deviations.
- **18-orch-stencils** (head `acbdc78b`): parent review dropped from the orch prompts, which now name decision escalations and Parent-Align.
  No deviations.

The scope guard reported only the deviations above, all informational.

## Verify gate findings

The plan-level verify failed on these identities:

- `internal/loomengine.TestDiscussionTable/lens_review/template_fan_without_lenses.yaml`
- `internal/loomengine.TestDiscussionTable/lens_review/the_fan_lives_only_in_the_repo_default`
- `internal/loomengine.TestDiscussionTable/lens_review/the_key_names_a_fan_of_lenses.yaml's_own`
- `internal/loomengine.TestDiscussionTable/lens_review/lenses.yaml_redefines_the_fan`
- `internal/loomcli.TestSmokeBootstrapLifecycle/malformed_status_proceeds_to_the_success_envelope_and_logs_why`

One fixer fork ran in answer.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `19147b1df514f1ec2e346e5188762ab2bae61531` fix: keep the twill directive right after the mode rules, and leave an undecodable status file to the driver's read gate
