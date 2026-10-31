# loom names the general run, and Discussion becomes a block every variant can turn on

`run-verbs-shared` · Task · enhancement, shed, loom, darn · draft

Depends on: [`tapestry-rename`](design-tapestry-rename.md)

## Seen (operator, 2026-10-14)

Since `0850f76e` a darn run has a decision record, and `lym loom decision add` works on it.
The operator finds it absurd that a `loom` command acts on something that is not a loom run.
The same holds for the other run verbs the orch uses on darn runs: `lym loom status`, `goto`, `resume`, `approve`, `reject`, `start`, `review approve`.
A third recipe is planned, `Hardener` (the note `hardener`), which would make three recipes steered with `lym loom`.

## Direction (operator, 2026-10-29)

`loom` becomes the name of the general run in Shed, the generic driver of a run, and `lym loom ...` stays as it is today.
What is called loom today is one variant of it, `tapestry`, darn is another, and `Hardener` becomes a third.
The mechanical rename of today's variant to `tapestry` is task `tapestry-rename`; this note is the design work after it: moving the generic parts under `loom`, built once that rename has landed.

## Discussion as a shared block (operator, 2026-10-31)

Today the two variants keep the same decision record, in the same format and checked by the same parser, but make it differently:
- loom's Discussion-Write writes it alone, and Discussion-Review's perch reviews it before Plan starts;
- darn's one writer turns the board entry into the record after a coarse reading of the code, asks the orch only when the entry is outdated, and then writes the code in the same session; no perch reviews the record itself.

The design:
- Discussion, a writer and its perch, becomes one block of the general run that a variant turns on or off: always on in `tapestry`, off by default in darn.
- Every run starts by turning the board entry's body into the record's format, a cheap step for a small model such as Haiku, checked by `lym loom validate-discussion`: the task text is the record's first draft.
- With Discussion on, the writer takes that draft as a first version and makes it thorough, and the perch reviews it; Plan and Webster work from the final record.
- With Discussion off, the draft is the final record: darn treats the board entry as already discussed and plans and writes the code from it in one session.
  This fits `loom-beam-darns`, whose darns are pieces a loom Discussion already cut.

## Open

- The conversion only translates: it adds, interprets and fills in nothing, and what the entry lacks stays empty or open in the record.
- Where the check that the entry is not outdated (landed work or another entry contradicting it) lives once the record comes ready-made: in the conversion step, or in darn's writer as today.
- How `loom` as the general driver relates to `shed`: `lym shed` already carries `run`, `step`, `status`, `pause` and `goto`, which the general `lym loom` verbs would overlap.
