# Fixer report — plan, round 2

## Changes

- **F1** (BLOCKING), fixed as proposed, with two further members of the same set and a different remedy for two sweep rows.
  Card 2 gains the Edit target `internal/creelengine_test#TestScheduler_Injects`.
  Its Tests paragraph now names the rows the card changes:
  - `InjectsAreTypedBeforeRemindersInQueueOrder`
  - the rewritten `Inject_SelfInjectPassesExceptForOrch` and `Inject_AndInterruptIntoOrchAreRefused`, whose gained rows no longer include a program's inject, since the rewritten rows cover it
  - the three `TestSweep_RemovedHolders` rows
  - `Read_SetsAndInbox`'s `inbox.Withheld` check, found at `internal/creelengine/verbs_test.go:187`
  - `ReminderSubjectIsCutAndItsDelimitersReplaced`, which tests the removed `reminderSubject`
  - `TestJournal_Fold`'s `p.withheld` assertion at `internal/creelengine/journal_test.go:151`, where the card had said the row "keeps passing"; it cannot compile once `pool` loses `withheld`

  The ImpactSummary names the seven functions.
  Fixed differently: the review proposed that the two sweep rows "post to a non-orch strand or read only sweep waits".
  The card keeps only "read only sweep waits through `e.sweepWaits`", because under card 2 every strand gets a delivery worker, so mail to a non-orch strand puts probe waits on the clock too (self-check, fix-claims).
- **F2** (BLOCKING), fixed as proposed.
  Card 11 gains the Edit target `internal/orchcli#TestStart_EnsuresTheRunsStrandBeforeTheOrch`.
  Its Tests paragraph says each row's `wantCalls` gains:
  - the first withdraw, with no first release, since no orch strand is listed in any row
  - the second withdraw and release after the start

  The runs-strand ordering assertions stay.
- **F3** (MEDIUM), fixed as proposed.
  Card 2 gains the Edit target `tools/sandbox/SANDBOX-CREEL-SUITE.md` and an Intent bullet that rewrites scenario C9's Goal, Watch and Verdict.
  Its ImpactSummary names the rewrite.
- **F4** (LOW), fixed as proposed.
  Card 8 drops the `PATTERN-plan-generation` cut and cuts `PATTERN-documentation-lifecycle` to "Design docs: none for unbuilt work; …".
  The arithmetic, measured from `PATTERN.md`'s entry texts, is stated exactly:
  - the cuts total 159 bytes
  - `PATTERN.md` ends the card 86 bytes under its base
  - after card 10 it is 15558 against a base of 15560

  A sentence says why the plan-generation line keeps its qualifier, and the ImpactSummary is reworded.
  Non-gerund triggers such as "A hub-level config module:" already exist in `PATTERN.md`, so the new trigger fits.
- **F5** (LOW), fixed differently in one clause.
  Card 1 gains:
  - the Edit targets `internal/testkit/shuttlefake/shuttlefake.go` and `cmd/lyx/testdata/test-budget.yaml`
  - the Create target `TestSignalEngine_PairsStampsAndHoldsTrailingStamps`
  - its Intent sentence
  - the budget raise for `internal/testkit/shuttlefake`

  Not made: the proposed clause "a plain `Engine` yields no signals".
  `Engine` implements no `SessionSignalParser`, and `TestEngine_InertDefaults` already asserts that per the card, so the clause would duplicate a covered behavior (`PATTERN-test-economy`).
- **F6** (LOW), fixed as proposed.
  Card 10 drops the Create target `TestWatcher_RestartAdoptsThePendingStep`.
  Its coverage becomes rows of these tests:
  - `TestWatcher_RestartLandedInjectionNoDuplicateSend`
  - `TestWatcher_RestartUnlandedInjectionResendsOnce`
  - `TestWatcher_ReloadRestartAroundTheRetryStep`
  - `TestWatcher_ReloadRestartRetypesOnlyTheUnconfirmedStep`

  The rows are stated after the turn-end test's paragraph.
  The soft-handoff restart sentence names the adoption rows.
- **F7** (LOW), fixed as proposed and widened.
  Card 10 states the stall reason's clearing rule.
  The reason sets no `StuckByCreel` and clears when the step advances, whether its inject reached `injected` or `unconfirmed` or it was skipped at its timeout, and otherwise at `enter` and `toIdle`, which clear `Stuck` today (`internal/orchengine/watcher.go:452,471`).
  The rule also covers a creel failure that overwrote a stall reason and then cleared.
  The proposal named only `injected` and `unconfirmed`; the skip-at-timeout advance reaches neither (self-check, fix-proposal).
  `TestWatcher_StepOutcomes`' expired row and skip row assert the clearing.
  Card 11's parenthetical on card 10's clearing rule now names both rules.

## Self-check

- fix-claims, MEDIUM: card 2's "post to a non-orch strand" option for the sweep rows does not restore the no-worker premise. Fixed as proposed: only reading sweep waits remains.
- fix-consistency, LOW: card 11's "card 10's clearing rule covers `creelFailed`'s reasons only" went stale after F7. Fixed as proposed.
- fix-consistency, LOW: card 2's ImpactSummary left out the C9 rewrite. Fixed as proposed.
- fix-affected-set, LOW: `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry` already asserts the under-hold reminder and changes to name `--from <owner>`, so card 2's gained hold row duplicated it and `reminderFor` needs the owner. Fixed as proposed: the existing row changes, the gained row is gone, and `reminderFor` takes the hold owner.
- fix-proposal, MEDIUM: the sweep-row option. A duplicate of the fix-claims finding, fixed there.
- fix-proposal, MEDIUM: F7's rule missed a step skipped at its timeout. Fixed as proposed: the rule and the skip row of `TestWatcher_StepOutcomes` cover it.

## Disputed

None.

## Deferred

None.
