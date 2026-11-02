# Fixer report — plan, round 1

## Changes

- **F1** (card 9), fixed differently in part.
  The goto path and both consume paths now end the fixers inside a wrapper of loom's told `GotoGuard`, as proposed.
  `Goto` runs that guard last, after its running, done and target checks.
  `Relaunch` runs it only for a non-empty `Target`.
  So a refused goto and a target-less relaunch end no fixer.
  The calls to `endRunFixers` before the two consumes are gone.
  `TacklerDeps.GotoGuard` and the arbiter's `GotoGuard` keep the bare `settledGuard`, because they only check a verdict.
  - The wrapper is a free function, `fixerEndingGuard(inner, endFixers)`, so a test can record whether the end ran.
    The proposal re-signed `loomCLI.loomPreGoto` as the wrapper.
    `loomPreGoto` and `TestSpecFor_PreGotoEndsFixers` are replaced by `fixerEndingGuard` and `TestFixerEndingGuard`.
  - **Not made as proposed:** the Intent sentence saying that a consume `AdmitRelaunch` refuses ends no fixer.
    That claim is false against the code: `Relaunch` runs `Guard` before `Admit` (`internal/shedengine/goto.go`, the `checkGuard` call before `req.Admit(cur)`).
    The card instead states the case as an accepted bound, with the reason.
    Ending the fixers only after `AdmitRelaunch` would need an end inside the `Admit` closures of `sheddriver.ConsumeTacklerAttempt` and `shedarbiter.ConsumeAttempt`, which batten's runs share.
  - The overview's Card Index line for card 9 now reads "an admitted goto or targeted relaunch".
- **F2** (card 1), fixed as proposed: `internal/runproc/doc.go` is under **Edit:** and has an Intent bullet.
  The self-check trimmed the bullet to name the group of fixer roles, not the role names.
- **F3** (card 3), fixed as proposed, with the set widened.
  The new Intent bullet lists the review's four comment sites plus three more found by rerunning its search:
  - `doc.go` lines 423 and 560;
  - the `ClassifyInputs.TurnEnded` field comment, added under **Edit:** as `internal/websterengine#ClassifyInputs` because the file had no target;
  - the `WebsterBodyVerifyFix` doc comment, added under **Edit:** as `contracts/stencils#WebsterBodyVerifyFix`.
- **F4** (card 7), fixed as proposed: `contracts/specs/shed-recipe-spec.md` is under **Edit:** and has an Intent bullet.
  The self-check added that `""` turns the fixer off.

`lym loom validate-plan` answers `ok: true` with informational findings only.

## Self-check

- fix-consistency, LOW: `specFor`'s wiring lost its test row. Fixed with my own change: `TestFixerEndingGuard` gains a row that builds the guard as `specFor` does, over `c.endRunFixers` on a receiver with no location, and checks that it admits and ends nothing. `TestSpecFor_WaitingHookNilWithoutLocation` is back under **Uses:**.
- fix-consistency, LOW: `endGateFixers`'s goto way-forwards did not fit the consume path. Fixed: its errors now name no way forward of their own. `shedengine`'s guard check adds goto's way forward, and on a consume the driver reports the error with the stop. The `TestEndGateFixers` rows are updated to match.
- fix-claims, MEDIUM: the time bound under the locks was understated. Fixed as proposed: the bound now names `runproc.EndRole`'s grace, kill bound and passes for each fixer role.
- fix-proposal, LOW: runproc's doc named the role strings of a higher package. Fixed as proposed: the bullet names the group.
- fix-proposal, LOW: the spec bullet left out that an empty value turns the fixer off. Fixed as proposed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
