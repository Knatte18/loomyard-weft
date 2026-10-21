# run-reentry: webster run summary

All 13 batches reached `done`.
Batches 01, 05, 07 and 09 were already done when this session resumed; this session ran batches 12 to 20.
Every `record-batch` envelope carried a "worktree is dirty" warning, and none carried a `batch_failed`, so no recovery was needed.

## Batches this session

- 12 `bouncer-amendment` (`4091019f`): a changed subject starts an amendment generation instead of clearing the settle record.
  Deviations: `internal/shedadapters/amendment.go`, `bouncer_amendment_test.go`, `bouncer_reentry_test.go`, `rubric.go`.
  Departure: the note says "since commit" instead of "weft commit", per `PATTERN-fabric-vocabulary`.
  The fork reported a known gap: after the case-3 settle clear, a code segment gets the document wording.
- 13 `recipe-bouncer-subject` (`e1e8f37e`): the Bouncer row takes a `subject` key and the segment's amendment allowance.
  Deviations: `internal/shedrecipe/entries_bouncer_test.go`, `internal/testkit/envkit/envkit.go`.
  Departure: the Env seam `WeftHead` is named `RecordHead`.
- 14 `loomshed-rework-records` (`75ed3668`): PR-Rework keeps the review records and the reject's findings.
  Deviation: `internal/loomshed/prrework_test.go`.
  No departures.
- 15 `loom-subject-wiring` (`ed73eacf`): loom and darn tell each review segment its subject, allowance and reject file.
  Deviations: `internal/fabricengine/doc.go`, `headsha_integration_test.go`, `taskpatch.go`, and `internal/loomcli/subjects.go`, `subjects_integration_test.go`, `subjects_test.go`.
  Departure: `Fabric.WeftHeadSHA` is named `RecordsHeadSHA`.
- 16 `loom-goto-guard` (`dc9747f0`): a goto onto a settled review segment is refused.
  Deviations: `internal/loomcli/arbiter_test.go`, `settledguard.go`.
  No departures.
- 17 `budget-way-forward-texts` (`823738e2`): a budget halt names the resume verb, and no text calls a goto a fresh budget.
  No deviations or departures.
- 18 `friction-no-per-stop` (`ab05b565`): a halt writes its friction note and starts no reflection.
  Deviation: `internal/loomcli/cli_test.go`.
  No departures.
- 19 `burler-forkcount-retry` (`d47f1a88`): a review round with the wrong fork count is retried, and the failure names how the chair ended.
  Deviation: `internal/shedadapters/burlerreview.go`.
  No departures.
- 20 `status-spec-docs` (`25674a05`): the status spec and overview follow the re-entry rules.
  No deviations or departures.

## Scope guard

The scope guard's deviation lists are informational and are given per batch above.

## Verify gate findings

Attempt 1 of 3 failed before any verify command ran: the worktree was not clean, because the untracked `.scratch_task.json` remained.


## Plan rebaselined

webster: the plan changed since the run recorded it and was rebaselined on entry, keeping 4 begun batch(es) and accepting 12-bouncer-amendment.md, 13-recipe-bouncer-subject.md, 15-loom-subject-wiring.md, 17-budget-way-forward-texts.md, 18-friction-no-per-stop.md
