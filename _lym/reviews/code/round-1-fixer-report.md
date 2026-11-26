---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The proposed counterpart rule counts only a file of the request's files, which lists unruled files alone: a check carried by a file whose ruling is reused would be ruled uncaused again. The judge also searches the repository's tests, names the carrying file under the new carried_by key, and such a ruling is judged again once the range changes."
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
---

# Fixer report, code round 1

`lym gate test ./internal/driftengine ./contracts/stencils` passed after the review fixes, after the self-check fixes and after the comment repair below.
The round gate then found three over-long comment lines, in `doc.go`, `judgement.go` and `verify.go` under `internal/driftengine`.
Each is split into shorter sentences, with no code change.
The new `carried_by` test row was not run against the code without its fix.

## Changes

- F1 (`9041bf93a`, `685e3268f`): fixed differently.
  The judge stencil and the `driftengine` package doc say that a removed assertion another test still holds, no looser, is not loosened.
  Instead of the proposed counterpart limited to the request's `files`, the judge also searches the repository's tests.
  The change not made is that limit: `listFiles` leaves a file with a reused ruling out of the request, so a check carried there would look dropped.
  The judge lists the carrying file under `carried_by` (`FileRuling`, `JudgedFile`), and `listFiles` reuses such a ruling only while the range key is unchanged.
  The Code-Review rubric and the `Loosened-Check` commit fragment carry the same moved-check rule.
  This changes what Code-Review and every committing session treat as a removed assertion.
  The fake judge writes `carried_by` for the new `TestDriftVerify` row; there is no assertion on stencil prose.
- F2, F6 (`0a1f9254c`): the `TestDriftVerify` row now changes the diff of a file already in the range and asserts that a third file's `none` ruling is not listed again and stays as stored.
- F3 (`d64effcea`): `docs/overview.md` names the three ways forward and the re-judge.
- F4 (`92052fdf4`): `checkAndJudge`'s doc comment states when it halts with no judge.
- F5 (`0a1f9254c`): halt text, comments and `_lym/landing/summary.md` name a change to a file of the range as the trigger.
- F7 (`0a1f9254c`): `rangeRulingKey` no longer hashes the trailer lines.

`_lym/landing/summary.md` was edited in place, with no git command, as F5 asks.

## Self-check

- fix-code-correctness, BLOCKING (a ruling relying on a carried check is reused after the check is dropped): fixed as proposed, through `carried_by`.
- fix-consistency 1, MEDIUM (same defect): fixed with the above.
- fix-consistency 2, MEDIUM (Code-Review rubric and fragment disagree with the moved-check rule): fixed.
- fix-consistency 3, LOW (`Call` doc comment states the old reuse rule): fixed.
- fix-consistency 4, LOW (summary title and design decision still say any new commit): fixed.
- fix-consistency 5, LOW (stencil says to decide from the file's diff): fixed.
- fix-consistency 6, LOW (overview defines a loosened check without removal): fixed.
- fix-affected-set 1, MEDIUM (Code-Review disagrees): fixed with consistency 2.
- fix-affected-set 2, LOW (summary wording): fixed with consistency 4.
- fix-affected-set 3, LOW (`doc.go` "whole range"): fixed.
- fix-proposal 1, LOW (summary wording): fixed with consistency 4.
- fix-proposal 2, LOW ("range unchanged" ignores trailers in `checkAndJudge`'s comment): fixed.
- fix-proposal 3, LOW (stencil's diff-only sentence): fixed with consistency 5.
- fix-proposal 4, LOW (Code-Review disagrees): fixed with consistency 2.

## Disputed

## Deferred

## Rulings
