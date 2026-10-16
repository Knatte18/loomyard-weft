# Fixer report — discussion round 3

## Changes

All seven findings were fixed as proposed, in `_lyx/discussion/decision-record.md`, with a Round 3 entry in `_lyx/discussion/support-log.md`.

- F1: D1's definition states that `--by` is an unverified label, citing `internal/loomcli/decision.go`, and counts a `--by operator` entry only when it is in the committed record before the escalation.
  The brief's settle condition carries the same rule.
  Step 1 says the fork never records with `--by operator`, and the Bound allows only `lyx loom decision add --by parent` and lists the late `--by operator` entry among what never counts.
- F2: The board sources are narrowed to a dated passage the entry marks as an operator decision, as `Operator decision (<date>)` does, and to a note whose body states it records an operator decision.
  `Idea` and agent-suggestion passages and unmarked operator views never count, in the definition and the Bound.
  The board entry marks these as lead-in paragraphs rather than headings, so the wording says "passage" rather than "heading".
- F3: The brief's materials gain the support log in the decision record's directory, read when the definition needs its `## Interview`.
  The brief's Acceptance criterion names it.
- F4: D1 rewords the role's design-call bullet to cover only escalations that are not review escalations.
  Scope's first line and a new Acceptance criterion name it.
- F5: D1 rewords the lead-in of `bouncer-fragment-decide-open.md` and `bouncer-fragment-decide-capped.md` to name the applied operator decision as the ground for the option.
  Scope lists both files.
- F6: D9's first bullet holds the check only while `merge_max_bounces` is above 0 and leaves the budget-0 case to the orch's workaround rule (D5).
  The Merge-Review Acceptance criterion gains the condition, and Open risks names the case.
- F7: D9's second bullet names the orch as the note's reader: on that run's `landed` notice it hands a fresh agent the removal of the `ORCHESTRATOR.md` entry.
  The matching Open risk is reworded.

## Disputed

None.

## Deferred

None.
