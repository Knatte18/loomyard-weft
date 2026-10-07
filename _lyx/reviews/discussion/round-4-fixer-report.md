# Round 4 fixer report

All seven findings fixed, in `_lyx/discussion/decision-record.md` only.
`support-log.md` needed no change.

## Changed

- F1 (D6): the halted wait reason is now the text batten logs and writes to the row's stuck-reason file.
  The bullet says `lyx batten status` does not show it while the child is halted, that no status surface is added, and that the orch learns the verb from its role stencil and D7's refusals.
- F2 (D6, acceptance): the awaiting hints keep `lyx loom start` and `lyx loom approve`/`lyx loom reject`.
  The batten acceptance row now limits the `lyx loom resume` wording to a halted child.
- F3 (acceptance): the batten row states both cases as D6 does, a failed revive and a child with no driver strand in reed state each also naming `lyx loom start --no-attach`.
- F4 (D7, acceptance): the dead-strand refusal names both `lyx batten run <slug>` (no batten run watching) and `lyx loom start --no-attach` (a batten run watching, or a failed revive), and the `lyx loom resume` row requires both.
- F5 (D7, acceptance): a retiring driver strand counts as not live and falls under the dead-strand refusal; the acceptance row names it.
- F6 (D3, acceptance): "deadline" is the one term, including in the findings text, the rationale and the acceptance row, and D3 says `verifytree` declares the one 60-minute constant.
- F7 (D8): `docs/overview.md` is on the doc list, for the loom verb list and the batten halted-child paragraph.

## Deferred

Nothing.
