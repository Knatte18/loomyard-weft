# A plainly outdated task entry goes to the parent: from the Discussion review in loom, from Darn-Write in darn

`outdated-entry-to-parent` · Note · enhancement, loom, stencil

## Seen

A task's board entry is written when it is promoted, and can sit for days while other work lands and other runs start.
Its text reaches Discussion-Write and Darn-Write as it stands, so a scope that overlaps, contradicts or has been overtaken by other planned or landed work is found late, after a whole Discussion or a whole darn change has been written from it.

## Design (operator, 2026-10-24)

No new row runs before Discussion, and the board's run lock gets no exemption: an outdated entry is caught by work the run already does, and goes to the parent.

- Loom: the Discussion review's `board-fit` lens gets a finding class "the entry is plainly outdated" (overtaken by landed work, contradicted by the board or another run).
  Such a finding escalates to the parent orch instead of going to the fixer.
  This is the escalating finding class note `discussion-alignment-into-lenses` describes; the two belong together.
- Darn: it has no Discussion review, and Darn-Write reads the entry and writes code in one go.
  So Darn-Write itself checks the entry text against the parent's recent landed commits and the board before it starts, and when the text is plainly outdated it asks the parent and waits instead of guessing.

Only a plainly outdated entry goes to the parent; an ordinary small misalignment is settled by the writer as today.

## Replaces

An earlier idea (2026-10-22): a row before Discussion-Write and Darn-Write that aligns the entry text and edits its own entry through a run-lock exemption.
Dropped: it costs an LLM call in every run while most entries are fresh, and needs a lock exemption.
