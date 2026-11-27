---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: deferred
    reason: "Partly done: fakePRSessions and envkit's noopSessions are gone, and the tests run the real shuttleengine.Runner over shuttlefake. Reed, creel and the LLM session are still stood in for: no kit holds a recorded session stand-in, a real reed needs tmux while these tests are tier0, and shuttlefake is itself a standing module stand-in. The parent decides whether to accept this or to build the infrastructure."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "Took the finding's alternative, a halt with the gate's reason when the post fails, instead of leaving MailedHead unset: an unset mailed head makes every wake gate the head again at the one-second floor."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "A pr-reviewer strand is admitted only when it equals the record's StrandName, with no slug match: a matching slug would also admit another hub's session of that slug. The verb needs no slug parameter."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
---

# Fixer report, round 1

## Changes

- F1: the `LoadConfig` doc comment writes the model-spec keys in full.
- F2 (deferred in part): the PR-Review tests drive the real session runner.
  Added `shuttlefake.ResumeEngine`, since the kit's engine cannot resume a session.
  `envkit` builds a real runner over `shuttlefake`.
  What is still hand-written:
  the `SessionLive` closures over the kit's strand table and in `envkit`,
  the `RegisterSession` closures,
  the `MailAgent` closures, including one that returns a made-up "no such strand" error,
  and `ResumeEngine` itself.
- F3: the gate mail, the session prompt, the `pr-continue` long help and `docs/overview.md` say the retry also ends the review.
- F4: `sessionStopWanted` follows the endings `judge` takes.
- F5: a gate failure whose mail fails halts the row.
- F6: a gate failure in an ended review halts the row.
- F7: `pr-continue` admits only the run's recorded PR-Review session.
- F8, F12: rows added for the stop, the re-read, the resume and the listed session behaviours.
  F12's empty-strand branch of `mailFailure` was removed by the self-check fix, since no kept session lacks a strand name.
- F9: the `wireLanding` test asserts both session seams are filled.
- F10: a transient error leaves the session running.
- F11: a closed pull request gets a session only under the continue flag.
- F13: the review-mode comments say the review ends on the operator's word.
- F14: `rows.go` is gofmt-clean.
- F15, F17, F18: the session prompt has the validate step, names `stencil.Fill`, and says about two minutes.
- F16: the change description's blast radius is widened, and its behaviour bullets follow the fixes.
  It is a record file and is left uncommitted.

Three `Loosened-Check:` trailers are on the F2 group commit.

After the round's gate reported comment lines wrapped mid-sentence, the comments of F4 and F13 it named
in `beamsetting.go`, `dobbyengine/config.go` and `prreview.go` were broken at sentence boundaries, in their own commit.

The gate passed on tier0 for the changed packages,
and after the self-check fixes on tier1 for `internal/loomcli`, `internal/landingshed` and `internal/dobbyengine`.
After the comment repair it passed again on tier0 for `internal/beamsetting`, `internal/dobbyengine` and `internal/landingshed`.

## Self-check

- fix-code-correctness, BLOCKING: resume after an ended-review halt pushed the refused head and recorded it as gated. Fixed as proposed in substance: the row now gates that head before any push.
- fix-consistency, BLOCKING: the same defect. Fixed with it.
- fix-proposal, MEDIUM: the same defect. Fixed with it.
- fix-code-correctness, MEDIUM: the failed-record-write row passed without the stop it pins. Fixed: the commit failure is now transient.
- fix-proposal, MEDIUM: a pull request closed with no flag could mail a stopped strand and never gate that head again. Fixed: it halts and records no mailed head.
- fix-affected-set, MEDIUM: F2 is only partly met and adds stand-ins. Fixed differently: F2 is recorded as deferred with the remainder named above.
- fix-affected-set, LOW: the renamed-mode reinterpretation notice kept the old wording. Fixed.
- fix-consistency, LOW: the `shuttlefake` package doc did not name `ResumeEngine`. Fixed.

No fork saw the self-check fixes.

## Disputed

None.

## Deferred

- F2, for the reason in the frontmatter.

## Rulings

None.
