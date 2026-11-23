# Review rounds ask the parent instead of settling past its rulings

A review round no longer settles past a held decision finding, an open BLOCKING finding at a spent budget, or an undriven hardener focus entry.
Each case now stops and asks the run's parent (or, for the hardener focus, the arbiter first), which accepts or orders another round.
`per-task-settings-one-lookup` landed with its review unsettled through the first two faults; a hardener round approved with its focus undriven through the third.

## What changes

- **Held decision findings gate.**
  The Code-Bouncer rows of the darn, tapestry and sampler recipes now set `escalate_decisions`, and their Code-Burler-Fix rows set `decisions_escalated`.
  A round holding a `decision`-class finding escalates and waits for the parent's decision record entry instead of passing with the finding listed as not gating.
- **A spent budget with an open BLOCKING finding escalates.**
  Those same Code-Bouncer rows set `blocking_escalates`.
  In `internal/shedadapters/bouncer.go`, the check now runs first in the soft-cap window.
  A round whose ledger has an open BLOCKING entry, or cannot be read, escalates with cause `budget`; the detail names the open keys.
  Neither the `code_auto_accept` auto-accept nor the soft-cap rule's accept can pass it.
- **A recorded accept no longer passes open BLOCKING findings of another class.**
  An `accept` recorded for a decision escalation over a CONTINUE or CIRCLING verdict rules only on the decision findings.
  Outside the soft-cap window the round runs another round; inside it the accept is retired and the round escalates with cause `budget`.
  The decision escalation also goes out as cause `budget` when the same round has open BLOCKING entries of another class, its detail naming the decision findings too.
- **A hardener round cannot approve with its focus undriven.**
  `harden-rubric-review.md` now has the reviewer raise a `decision`-class, BLOCKING, `origin: focus` finding for each focus entry it left undriven or marked WARN.
  `burlerengine.CheckReviewRound` exempts such a finding from the changed-file rule when `RoundContext.FocusFindings` is set, since an undriven entry has no changed hunk.
  The Harden-Bouncer row sets `escalate_decisions`, so the arbiter rules first and the parent after it.
  A review holding such a finding never converges.
- **`code_auto_accept: true` means less.**
  It no longer accepts a round with an open BLOCKING finding.
  The `template.yaml` comment says so, and `internal/loomengine/reinterpret.go` adds a reinterpretation, `loom-code-auto-accept-escalates-blocking`.
  It flags a hub `loom.yaml` that sets the key to true, sets none, or is absent, and leaves the file unchanged.
- The shed-recipe spec, the bouncer escalation and decision stencils, the burler review format and `docs/overview.md` carry the new rules.

## Design decisions

- The escalation cause for a spent budget over BLOCKING findings is `budget`, not `decision`, because only the parent may accept it.
  The arbiter's scope for cause `budget` stays "decide nothing"; the stencil now tells the parent what the cause names.
- A `CONVERGED` verdict keeps cause `decision` for a decision escalation, since the refusal check already shows the open entry.
- `blocking_escalates` is set on the Code-Bouncer rows only; the discussion and split perches keep their auto-accept behaviour.
- The reinterpretation flags and does not rewrite: the value `true` stays, because the new behaviour is carried by the recipe rows.

## Check by hand

These findings were left open when the code review closed:

- `below-window-decision-accept-passes-open-blocking`: a decision accept recorded below the soft-cap window over an open BLOCKING ledger entry of another class, in `internal/shedadapters/bouncerdecision.go`.
- `budget-escalation-decision-detail-unasserted`: the decision-findings detail appended to a budget escalation has no test asserting it, in `internal/shedadapters/bouncer_escalation_test.go`.
- `code-auto-accept-meaning-change-no-reinterpretation`: the narrowed meaning of `code_auto_accept` reaches hubs only through the flagging reinterpretation in `internal/loomengine/reinterpret.go`; confirm a flag, with no rewrite, is enough.
- `hardener-focus-permission-wiring-untested`: the wiring that lets a hardener round raise a focus finding (`FocusFindings` into the round context) is not covered end to end, in `internal/burlercli` and `internal/shedadapters/roundcontext.go`.

No integration suite or regression check ran for this task.

## Merge danger

**Door:** two-way
**Blast radius:** shedadapters
