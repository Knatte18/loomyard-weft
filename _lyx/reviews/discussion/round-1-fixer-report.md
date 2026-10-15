# Round 1 fixer report — discussion

Every finding's premise held against the code read (`webster-template-master.md` on `summary_path`, `render.go`'s `RenderRecoveryPrompt` and `RenderMasterPrompt` directive roles, the two pattern directives' filters, and the three live FRESH-READ fragments).
All five are fixed as proposed.

## Changes

- F1 (BLOCKING): `decision-record.md` D8 rewritten.
  No card implements it; the orch files a board note through `lyx board` at landing that carries the `tools/forkaudit` re-measure and the Master-read compliance check.
  D8 now states that forks explore more by construction, so a rise in per-fork reads is not a regression.
  The rationale says why the task summary cannot carry it.
  `support-log.md`: the Q9 ledger line names the board note, and the accepted-cost line points at D8.
- F2 (MEDIUM): D4 names the set the recovery prompt's own constraints section points at (implementer directive, filtered by what the strand is about to change) and drops "the same set the Master reads".
  D1 adds that, with card files banned, the Master applies the orchestrator filter from `00-overview.md`'s Card Index and `## Shared Decisions` alone.
  The first Auto-mode assumptions bullet is deleted.
- F3 (MEDIUM): the third acceptance criterion now checks that an empty `pattern_directive` render has no constraints section, conditions every `pattern/` mention on that section, and names no read beyond the plan directory and a named gate findings file.
- F4 (MEDIUM): D3 names the three replaced spans exactly, states the one replacement statement, and changes "re-read" to "read" in the plain fragment.
- F5 (LOW): the second Open risks bullet states the consequence (stale context to later forks), the bound (D3's fresh-read rule), and that D8 detects it.

## Disputed

None.

## Deferred

None.
