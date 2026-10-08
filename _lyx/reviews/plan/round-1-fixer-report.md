# Fixer report — plan round 1

Every finding was checked against the code and the decision record, and every finding holds.
All nine are fixed.
`lyx loom validate-plan` passes after the fixes: `ok: true`, with only the informational `create-new-unit` note for card 7's `stop_test.go`.

## Fixed

- **F1** (card 5): `awaitStartup`'s three status-cap exits now return `run.abandonStartup(OutcomeDied, <existing Errorf>)`, as D4's "every other start failure past `AddStrand`" requires.
  The card names what follows from that:
  - `TestStartup_MechanismFailure`'s assertions are corrected instead of adding a test.
    They previously pinned the old no-teardown behaviour.
  - The audited key `awaitStartup [Errorf]` x3 becomes `awaitStartup [Errorf,OutcomeDied]` x3 in `completionsignal_enforcement_test.go`, which is now a card 5 target.
  - The doc comments of `awaitStartup` (its return mapping) and `Runner.RunGated` (now a target) no longer describe a no-teardown mechanism failure.
  - A Bound line states that these exits now reach the agent-start transient class, and only before any prompt was delivered.
- **F2** (card 9): `RunDeps.Reed` and `BeginDeps.Reed` are deleted, with their fills in `webstercli`, `loomcli` and the two websterengine fixtures.
  The sentence now says only `RecoverDeps.Reed` stays.
  Checking the code showed the gap was wider than the finding: `RunDeps.Reed` is also required by `shedrecipe`'s `websterEntry` seam check, filled by `shedfake.WebsterSeams`, and named in `entries_simple_test.go`, `shedfake_test.go`, `battencli`'s `intentionallyNil` table and `loomcli`'s wiring subtest.
  None of these were targets.
  A new `Stopper` field left out of `websterEntry`'s check would also panic on a nil stopper at a reclaim, and batten's `TestWire_EverySeamFilled` would fail on it.
  Card 9 now does the following:
  - `websterEntry`'s check requires `WebsterDeps.Stopper` instead of `Reed`.
  - `WebsterSeams` fills a new `strandStopper` placeholder, and its `reedOps` is deleted.
  - Each of those tests names `Stopper` in place of `Reed`, and all of them are listed as targets.
- **F3** (cards 8 and 9): each card's Intent now has a Bound paragraph covering what the rerouted stop can let through: a record-write failure that leaves a live strand behind `running`, and a removal failure that leaves `stopped` over a live strand.
  It also says how each call site handles that and what bounds it.
  For `awaitTerminal`, a stop error stays a warning, decided and justified in the card.
  That is no wider than today's failed-removal residual, since the terminal digest is already on disk, and the bound is the guid in the warning plus webster's reset.
- **F4** (card 6): `boundaryIdle` is keyed on the same capability pair as `awaitIdleSession`, the idle reading plus session signals.
  An engine lacking either answers idle at once.
  `SessionCycler` is added to Uses, and the ImpactSummary and test-engine wording follow.
- **F5** (card 4): the confirm test's `inputBoxEngine` gains, per row, card 3's idle-reading fake, so the clear rows can reach the clear.
  Those rows script the idle wait's captures, and the row for an engine without the idle reading omits it.
- **F6** (card 3): the duplicate row is dropped, and the card names `TestRun_InterruptAndSend_RefuseAgentlessShellPane` as the test that pins the unchanged guard.
- **F7** (card 8): the card now names the four test files and counts seven callers of `burlerengine.New`.
- **F8** (card 5): `abandonStartup`'s doc comment and closing Warn widen to cover a start that never took its first input.
  The card now says which cause each existing call passes: nil for `awaitStartup`'s three not-ready answers and the died load turn, and the loader's error for the `skillLoader` failure.
- **F9** (card 3): the comment on `driverResumeSendAttempts`/`driverResumeSendInterval` is added to card 3's loomcli doc update, and both constants are added as targets.

## Disputed

None.

## Deferred

None.
