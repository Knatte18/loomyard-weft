---
round: 1
focus:
  - "Cards 1 and 2 both edit `darn-template-write.md`, `landing-template-describe.md` and `TestDarnSpec`: does card 2's Intent still match what card 1 leaves in those files, and does card 1 only depend on earlier work?"
  - "Cards 6 and 7 turn a destructive verb into a dry run behind `--apply`, and card 9 adds a refuse/error path: does each Intent state what the changed behavior can still skip or let through and what bounds it?"
  - "Cards 5, 6 and 9 each add a new top-level test per the `test-budget-raises` decision: does an existing test or table already cover the behavior, and which one?"
  - "Card 5 (`Migrate` hooks) and card 9 (`WriteCarryOver`): where the Intent changes parameters, results or receiver of an `Edit` member, is the re-sign arrow present; where Intent names symbols, are member glyphs listed rather than bare Go files?"
  - "Every Decision and Constraint in `_lyx/discussion/decision-record.md` (D1 to D6, assumptions, Open risks outside carry-over markers): is each carried by some card, and does any card add scope the record does not license?"
  - "Granularity of card 2 (one `Rename`, one `Create`, ten `Edit` targets) and of card 8 versus its consumers: is each an independently reviewable unit?"
  - "The `## verify:` section: does it run the tests of every package the cards target, including `cmd/lyx`, `internal/cliwire`, `internal/darnengine` and the board and fabric packages, and is it in the template's order?"
---

Seed focus for a 9-card plan of small independent fixes, no rework generation.
Entries name where to look; none pre-states a verdict.
Gate-checked items and missing `DependsOn`/`Produces` were left out per the rubric.
