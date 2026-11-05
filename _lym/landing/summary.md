# The arbiter settles decision escalations and records its calls

A `decision` review escalation now goes to the arbiter before the run's parent, as `hard-cap` and `circling` do.
The arbiter records the calls the board entry's design and the decision record already settle, so the orch hears only the outcome.
Before, a fixer's decision findings or a bouncer wanting a ruling reached the orch every time, and an eligible arbiter could only resume or hand on, even when it had found the exact correction.

## What changes

- `internal/arbiterstop`: a pending `decision` escalation is eligible; only `budget`, an unknown cause and an unreadable record still go to the parent.
- `contracts/stencils/loom/loom-template-arbiter.md`:
  - When the run's repair verbs include `lym loom decision add`, the arbiter records each call it settles with `--by arbiter`, worded so the run's own agents act on it.
  - It makes at most the one edit its stop asks for, recorded as a decision naming the clause and its exact replacement; anything wider is recorded for the run's agents to carry out.
  - It hands on only a question the board entry and the record leave open (what to build, scope, a choice between open designs), and records every call it could settle before it does.
  - The brief's "What to do" section is the parent's procedure; the stencil lists the parts that do not bind the arbiter and the lines that still do.
  - It has rules for `hard-cap`, `circling` and `decision`, summarised under the decisions below.
- `internal/sheddriver`: when the arbiter's resume at an awaiting stop has been consumed by the step, the driver posts the parent one line naming the run, the stop and the verdict file, and asking nothing.
  The line is skipped when the step refused the attempt, and with no orch strand recorded it is logged and posted nowhere.
- `contracts/orchrole/orch-role.md`: the orch takes no action on that line, and a `decision` escalation reaches it only as the arbiter's hand-on.
- Bouncer stencils, `shed-recipe-spec.md` and doc comments no longer say a `decision` finding is one "only the run's parent can settle".

## Design decisions

- `--by arbiter` is an existing author, already used for `--amends-plan` amendments; no new author kind.
- At `hard-cap` the arbiter accepts only when a finding is wrong or the fix in place already answers it.
  An accept ends the review segment, so a fix still to be made goes to the parent as a hand-on question instead of a recorded decision no agent would carry out.
- At `circling` it records the ruling and continues below the hard cap; at the hard cap the `hard-cap` rule applies.
- At `decision` it reads the brief's Refusal section first.
  When a refusal is named, a held accept meets it again, so it records the answers and continues below the hard cap, and hands on at or past it.
- `budget` stays parent-only, since the arbiter has nothing to judge when an input could not be read.
- The outcome line is held until the step has consumed the resume, so the parent is never told of a settlement the step refused.

## To check by hand

- `hard-cap` record-and-accept: the arbiter stencil, at `hard-cap`, must not let the arbiter record a decision and accept over a fix that still has to be made.
- Outcome notice order: in `internal/sheddriver/drive.go`, the notice must go out only after the step consumed the arbiter's resume.
- Outcome notice paths: its skip paths, guard, refusal case and text are pinned by tests in `internal/sheddriver/drive_test.go`; confirm they cover each.
- Parent-only prose: the bouncer stencils, `shed-recipe-spec.md` and the orch role must no longer read as if a decision finding goes to the parent alone.
- No end-to-end run of an arbiter settling a `decision` escalation was made; the behaviour of the stencil's new rules rests on review of its text.
