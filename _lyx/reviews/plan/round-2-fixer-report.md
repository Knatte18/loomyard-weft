# Fixer report — plan, round 2

## Changes

- **F1** (BLOCKING, card 17), fixed as proposed.
  Added Edit targets `TestShellWaitMark`, `TestWait_PayloadShellLogsPastWaitMinOnceAndFinishesAtTheNextTurnEnd` and `TestWait_HeldTurnEndNotice`, and stated each one's new expectation in the Tests paragraph: one held-turn-end notice in the two Wait tests, and `TestShellWaitMark`'s three single-shell rows rewritten over two shells at staggered first-seen times.
  The ImpactSummary now says five existing tests change.
- **F2** (MEDIUM, card 17), fixed as proposed.
  Added Edit targets `Spec`, `Run.shellWaitActive` and `websterengine#masterAwaitedShellPrefix`.
  The Docs paragraph now rewords `pollEventsTick`'s doc (already a target), `Spec.AwaitedShellPrefixes`, `masterAwaitedShellPrefix` and `shellWaitActive`.
  The Intent's "display threshold of every other wait" now names the `background shells` mark and the staggered-shell state it shows in.
- **F3** (MEDIUM, card 11), fixed as proposed.
  `removeVerdict` returns the empty verdict for a stop made only by gone branches of the slug, and states the loop as its reason.
  Added the two `TestRemoveVerdict` rows and the `TestRemover` row.
  The help-text and doc line names the exception.
- **F4** (MEDIUM, card 11), fixed as proposed.
  Removed the live-parent-branch clause and its `TestRemoveVerdict` row, and kept the parent-pair-entry row.
- **F5** (MEDIUM, card 02), fixed as proposed.
  Added `TestEnd` rows for the group check failing at each phase (no live member, or only members that started earlier), and for a reused non-leader record pid between the SIGTERM and the SIGKILL.
- **F6** (LOW, card 10), **fixed differently**.
  Dropped the `registered` parameter and "the slug's live owner records" from `awaitRunLockOrPass`'s reads.
  Reworded the row to "a marker of another pid".
  Stated that the paused pass covers known and unknown holders alike, and added the unknown-paused-no-parent-lock row.
  I did **not** drop the Uses lines `plan:internal/runproc#List` and `plan:internal/runproc#Record`.
  The card's `busyError` change "adds the slug's live owned processes (role and pid)", and that still needs both.
- **F7** (LOW, card 10), fixed as proposed, then narrowed by the self-check (see below).
  An unpaused refusal with no parent lock record names `lyx tender pause <slug>` and the removal first, then the retry, and states the reason.
  The wording assertions now cover an unpaused known holder and an unpaused unknown holder.

## Self-check

- fix-consistency, MEDIUM: card 11's gone-branch exception departs from D5's "a `stopped` verdict refuses" (`decision-record.md:106`, `:267`). **Deferred**, see below.
- fix-consistency, LOW: card 10's unpaused pause way forward departs from D5's `:115` wording. **Deferred**, see below.
- fix-consistency, LOW: D11's "display threshold of every other wait" (`decision-record.md:195`) is left as it was. **Deferred**, see below.
- fix-consistency, LOW: `TestPollEventsTick_OutstandingBackgroundWork`'s doc comment still states the old rule. **Fixed as proposed**: card 17's Tests paragraph now rewords it beside the `//testtiming:keep` reason.
- fix-claims, MEDIUM, merged with fix-proposal, LOW: the gone-branch exception counted a busy shared branch, and the run scope's busy records branch of the slug, as stopping, unlike `classifyEntries` (`originreconcile.go:744`, `:756`). **Fixed as proposed**: the exception leaves both out, and a `TestRemoveVerdict` row covers a gone stop beside a busy Bolt branch.
- fix-proposal, MEDIUM: F7's pause way forward loops when a parent lock record names the slug. Pause leads to the paused refusal, its resume leads back to the unpaused refusal. **Fixed as proposed**: the pause way forward now appears only with no parent lock record. An unpaused known holder under a parent lock record names its pid and the retry only. The wording rows are split to match.
- fix-affected-set, MEDIUM: `websterengine/recoverbatch_test.go:698` waits past the bound on a non-awaited shell. **Disputed**, see below.
- fix-affected-set, LOW: `attach_asking_test.go`'s row "running record whose waiting line was notified: no new notice once the shell bound elapsed" is not mentioned. **Fixed as proposed**: card 17 names it as reaching the expired held turn end and staying green. `notifyHeld` skips a held offset at or below `NotifiedOffset` (`internal/shuttleengine/hold.go:138`).
- fix-affected-set listed `TestWait_DialogNotice`'s shell row as unchecked; it was not raised as a finding.

## Disputed

- fix-affected-set's `recoverbatch_test.go:698` finding, ground `premise false`.
  That row drives the recovery strand through `RecoverAwait`, and `RecoverAwait` uses webster's own classification, `Classify`/`PollUntilTerminal`/`TurnEndedAfter`/`StrandLive` (`internal/websterengine/recoverbatch.go:1`).
  It does not use shuttle's `Wait`, so card 17's change to `pollEventsTick` and `expiredTurnEnd` never reaches it, and the row still holds.

## Deferred

These fork findings concern `_lyx/discussion/decision-record.md`, which is outside this round's write surface (`_lyx/plan` and this report):
- D5 `:106` and `:267`: the gone-branch exception to "a `stopped` verdict refuses".
- D5 `:115`: the unpaused run-lock refusal's `lyx tender pause <slug>` way forward, with no parent lock record.
- D11 `:195`: "stays the display threshold of every other wait".

The cards now follow the review's findings.
The decision record's matching sentences need the same change from whoever owns that file.
