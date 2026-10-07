# Round 1 fixer report

Every finding in `round-1-review.md` is fixed; `lyx loom validate-plan` exits clean (`ok: true`, informational findings only) over the edited plan.
`approved:` in `00-overview.md` is untouched (`false`).

## Changes

- **F1 (BLOCKING)** — `07-two-agent-round.md` (the former card 6) now lists `internal/burlerengine/template_test.go` as an Edit target, and its Tests paragraph replaces the single `orchestrator` row and `orchestratorMarkerValues` with a row and values map per new orchestrator.
- **F2 (BLOCKING)** — `03-per-round-models.md` states that each new loom config load error names its way forward, adds `contracts/specs/refusal-spec.md` to its Edit list with the loom-section row, and names the planned `config_test.go` rows as the tests that reach them.
- **F3 (MEDIUM)** — new `06-strand-remover.md` creates `StrandRemover`, `NewReedStrandRemover`, `remover.go` and a table-driven `remover_test.go` over a `shuttlefake.Reed`, with a stated bound.
  The engine card (now 7) uses it instead of creating it and builds it in the two CLI wirings; the resume card (now 8) keeps using it.
  Files renamed `06-two-agent-round.md` → `07-two-agent-round.md` and `07-two-half-resume.md` → `08-two-half-resume.md`; headings, card cross-references and the overview's Card Index and intro follow.
- **F4 (MEDIUM)** — the engine card's Intent gains a paragraph stating what the marker handoff, the review-only comparison and the moved gates let through, and what bounds each.
- **F5 (MEDIUM)** — card 1's test is reduced to the found-candidate-returns-an-unwaited-handle behaviour plus the nil not-found shape; the removal and refusal paths are left to `TestAttach_RemovesSupersededStrands` and `TestAttach_Multiplicity`.
- **F6 (LOW)** — card 5 now retargets `burler-step-3-fix.md`'s header comment to the fixer orchestrator.
- **F7 (LOW)** — card 3 adds `docs/overview.md` (the loom config sentence for the `fix` key and list form); card 2 adds `docs/overview.md` (the `internal/burlermarker` tree line) and its `ImpactSummary`.
- **F8 (LOW)** — card 4's test becomes one table-driven test with a missing-argument row and a stat-error row, and the nothing-written check applies to every row.
- **F9 (NIT)** — card 8's Intent and test row now separate the one-live-half case (strand removed) from the done/gone case (nothing removed); both respawn attempt 1.

Also: the engine card's Intent notes that the retained pair-file attach probe matches no run until card 8, as the review's focus verdict recorded.

## Deferred

None.
