# Plan fixes after begun batches block Webster; glyph-not-found on members a card creates

`webster-plan-fix-begun-batch` · Note · bug, webster, quarry

Imported from [issue #221](https://github.com/Knatte18/loomyard/issues/221).

## What happened

Two Webster halts after the plan was edited by a plan review/fix round while batches were already begun.

1. `blocked` / `escalation-to-human`:

```
webster: the auto-rebaseline of the changed plan blocks this run: webster: the edited plan changes the cards of a batch this run already begun: 00-overview.md changed outside its Card Index; it carries the plan's integration verify and only its Card Index is rebaselined; way forward: restore 00-overview.md, or fix the plan: move a done card's edit to a follow-up card after the last begun batch that carries the decision, with its Card Index line in 00-overview.md, then re-step the Webster row
```

2. After the plan was fixed and the run re-stepped, `failed` / `producer-hard-failure`, plan validation refused with four `glyph-not-found` findings, all of the form:

```
glyph-not-found/<card>[blocking]: target "<Unit>#<Member>"'s unit exists but the member is missing — check for a misspelled member
```

The members named were ones the plan's cards (5 and 6) introduce or modify, and the targets include a method signature with a generic parameter type (`IReadOnlyList<double>`) and a nullable (`TimeSpan?`).
Card 5 and card 6 cite the same new member as a target.

## Why abnormal

- The plan review loop (`Plan-Burler-Fix`) edited `00-overview.md` outside its Card Index and the run then blocked on it; the plan-fix stage gives no warning that such an edit is unrecoverable once a batch has begun, so each such fix costs an operator escalation.
- If the glyph targets name members that an earlier card of the same plan creates, validation cannot find them at plan time and refuses a valid plan. If they are misspelled, the finding text does not say which member of the unit is closest. Either way the finding should say whether the member is expected to be created by a card.
  I could not tell from the notes which case applies.

## What to do

- Have the plan-fix stage refuse or warn on edits to `00-overview.md` outside the Card Index once any batch has begun.
- Let glyph validation accept a target that an earlier card of the plan declares as created, and print the unit's actual member names beside a `glyph-not-found` finding.

Provenance: task `models-torque-drag-adapter`; notes `loom-halt-4.md`, `loom-halt-6.md`.
