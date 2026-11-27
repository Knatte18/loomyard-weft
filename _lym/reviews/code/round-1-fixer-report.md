---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "A task member with no pair and no run status no longer refuses under --remove, as proposed, but a sub-beam member still does. A sub-beam can be open on another machine with no local pair and no run status (internal/beamcli/create.go records no claim), so passing it would delete its parent branch under it."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
---

# Fixer report, code round 1

## Changes

- F1: `refuseBeamCancel` takes `remove`; with it, `workingBlockers` drops idle task members in both the held and unheld branches.
  The unheld refusal then names only what still blocks, and the held refusal names no move off the beam for a member it says to cancel.
  Not made from the proposal: passing a sub-beam member (`BeamLandMember.Beam` keeps it a blocker).
  New table rows cover each branch.
- F2: the fake `EndBeam` records the call and returns false; the row with the made-up board error is gone.
  `TestCancelIntegration_RemoveOfABeamLeavesAPlainProject` runs the real cancel over a hub board.
- F3: option (b). `boardengine`'s doc names `Board.EndBeam` as the one other write of an owner.
- F4: the integration test's beam has a pair with no beam mark and asserts no orch-stop warning.
- F5: the door is now one-way, and the blast radius includes boardengine.
- F6: the sentence is back in `--help` and out of the package doc; the beam paragraph is shorter.
- F7: the Output paragraph names `beam_ended`.
- F8: a row for a lone flag without `--remove`.
  Both `isBeamProject` error branches are provoked in the integration test by making `board.json` unreadable.
- F9: the flag clear now runs after the records push.

`_lym/landing/summary.md` is edited for F1, F2, F4, F5 and F9 and left uncommitted for the loop.

## Self-check

- fix-code-correctness: no findings; its note outside the lens (the package doc's order sentence omits the flag clear): fixed.
- fix-consistency 1, MEDIUM (`PATTERN-owner-machine` still says only the setter writes an owner): fixed as proposed.
  This edits a PATTERN background line; the rule line ("set only by `boardengine`'s setter") is unchanged, since `EndBeam` only clears.
- fix-consistency 2, LOW (refusal text says a board write never changes an owner): fixed as proposed.
- fix-affected-set 1, MEDIUM (same PATTERN line): fixed, merged with fix-consistency 1.
- fix-affected-set 2, MEDIUM (`ownerset.go` header calls itself the only writer): fixed as proposed.
- fix-proposal 1, MEDIUM (a sub-beam live on another machine passes `--remove`): fixed as proposed, by exempting task members only.
- fix-proposal 2, LOW (the `push_spawned` help note inverted its meaning): fixed as proposed.

## Gate

The round's gate found comment lines wrapped mid-sentence in `cancel_integration_test.go`, `cli.go` and boardengine's `doc.go`.
They are rewritten as one sentence per line, in a commit of their own that changes comments only.

`lym gate test` on `./internal/battencli`, `./internal/boardengine` and `./cmd/lym` passed at tier 0 after that repair.
Tier 1 passed on the same packages before it.

## Disputed

None.

## Deferred

None.

## Rulings

None.
