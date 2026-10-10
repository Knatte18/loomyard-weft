# Plan-Burler round 3 — fixer report

Every finding was checked against the code and the decision record, and every premise held, so all fifteen are fixed.
`lyx loom validate-plan` passes after the edits, with only the existing informational `create-new-unit` finding.

## Fixed

- **F1** (card 4): the `MerriamBase` failure joins the `ErrAutoRebaseline` wrapped set with the re-step clause, because its own `re-run the verb` clause names no step the row can take.
  `internal/websterengine#MerriamBase` is added to `Uses:`.
  `TestRun_AutoRebaseline` gains a row that forces the failure with a directory in place of the worktree's `CLAUDE.md`.
  `merriambase.go` refuses any read error except `ErrNotExist`, and `Run` reads no `CLAUDE.md` before the auto path.
- **F2** (card 17): `TestVerify_DropsStrandName` now has one row per environment branch of `Verify`.
  The unslotted branch runs over a nil pool, and the slotted branch over a free one-slot `gateslot.Pool` built as `TestVerify_SlotGate` builds its pool.
- **F3** (card 14): rejected rows are added for `git log *`, `git log a?`, `git log [a]` and `git log ~/x`.
  The help form gains `lyx webster record-batch --help` accepted and `lyx Fabric --help` rejected.
  The Intent now defines a command word as lowercase letters and inner hyphens.
- **F4** (card 4): the Intent names the two D1 clauses that license the widening.
  The first is D1's "never to run `lyx webster rebaseline` by hand" on the auto path, together with the acceptance row that no auto-path refusal text contains it.
  The second is that a stuck outcome's reason opens with the warning.
  A bound is added: the widening reaches only a run whose auto-rebaseline was accepted, and every other text and disposition stays as it is today.
- **F5** (card 3): the no-partition branch of `rebaselineBatches` no longer calls `formBatches`.
  It calls `batchCards` with the identity batchifier, then `CheckBatchOrder` with the deps' step.
  `internal/websterengine#batchCards` is added to `Uses:`.
  `TestRebaseline_CardSet` gains a step-set order-violation row over a record with no partition.
- **F6** (card 22): `Runner.start` now sets the started-fresh field only when `ResumeSessionID` is empty.
  `Runner.attach` is dropped from `Edit:`: `Runner.reconstruct` (`attach.go`) builds the attached `Run` and leaves the field unset without any edit.
  The acceptance rows add a run started with a `ResumeSessionID` that waits on the shell.
- **F7** (card 24): the single pre-`Start` HEAD read now supplies both `RecoveryStartSHA` and the `start` fallback.
  The post-`Start` `HeadSHA` call in `recoverSpawn` (`recoverbatch.go`) is removed.
- **F8** (card 2): a new top-level `TestCommitsNamedBy_OneBatchCheck` puts a recording `git` first on `PATH`.
  It asserts exactly one `cat-file --batch-check` call and no `cat-file -t`.
  It runs without `t.Parallel` under a comment naming `PATH`.
  `test-budget.yaml` joins `Edit:` and the websterengine row rises by one.
- **F9** (card 3): the existing `TestRebaseline_EditedPlan` row for a state without the overview frame hash and without the baseline copy moves into `TestRebaseline_OverviewFrameUnreadable`, which can capture the log.
  It becomes the absent-copy cause row, and its wanted text gains the cause and the path.
  New rows cover only the other two causes.
  The no-frame text keeps the `recorded no overview frame` phrase, so the recorded-frame row's `wantNotText` still bites.
- **F10** (card 6): the unreadable-card-file row is dropped from `TestEditedDoneCards`.
  The case is pinned once, by card 7's `TestNewPlanGate` row.
- **F11** (card 18): the subcommand walk runs every subcommand with `--help` under a webster strand name and asserts that it passes the guard.
  The `add` refusal row asserts that the text names the `hubforge` fixture and the `status: FAILED` step.
- **F12** (card 4): the `SaveState` failure is forced by making the webster dir read-only while `plan-baseline` stays writable, which `runlevel_test.go` already does.
  That row asserts the fake starter saw no `Start`.
  The Intent states that `Run` returns before starting anything on a failed save.
  The accepted row asserts exactly one Warn line.
  Card 3's cause rows assert exactly one Info line each.
- **F13** (card 24): `TestRecoveryRetry` gains a row for a terminal `stuck` recovery prior and a row for a terminal `failed` one, each with commits past its start and each expecting false.
- **F14** (card 1): `doc.go` now names `RunStdin` as a second checked form beside `Run`, and `RunGit` stays the raw form.
- **F15** (card 18): `internal/fabriccli#Command` is replaced with the file target `internal/fabriccli/fabric.go`.
  Two regions of that file change: the package comment and `Command`'s `add` help.
  Listing the member beside the file would trip `redundant-file-target`.

## Disputed

None.

## Deferred

None.
