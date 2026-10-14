# Fixer report — discussion round 1

Every finding was checked against the code and fixed; none disputed, none deferred.

## Fixed

- **F1 (BLOCKING)** — confirmed: the loom `raw I/O` row in `contracts/specs/refusal-spec.md` reads "re-run the refused verb; nothing is mutated", and D2's own precondition refuses the partial state a mid-apply failure leaves.
  D2 now reverses applied steps from the in-memory move list on a mid-apply I/O failure, so the grouped row's contract holds.
  A failure of the reversal is its own correctness-halt row, whose envelope lists the unreversed steps and whose way forward routes through status FAILED to the orch, since the stencils forbid a plan agent to rename or delete a card file.
  Acceptance criteria gained the injected-failure line.
- **F2 (BLOCKING)** — confirmed: `cardIndexLineRe` in `internal/planparser/parse.go` takes the slug as `\S+`, and `cardFileName` formats it unchecked.
  D2 now defines the verb's own slug rule (lowercase ASCII letter and digit runs joined by single hyphens) and states it keeps the new file inside `planparser.PlanDir`; the Acceptance criteria refusal list names a slug holding `/` or `..`.
- **F3 (MEDIUM)** — confirmed: `begin-batch` records under `websterengine.AcquireStateMutation` (`internal/webstercli/beginbatch.go`).
  D2 now holds that lease from the run-record read through the last apply step; Acceptance criteria gained the waiting-begin-batch line.
- **F4 (LOW)** — confirmed: `cardIndexLineRe`'s `(.+)` rejects an empty intent, and a line break splits the index line.
  D2 now refuses an empty, whitespace-only or multi-line `--intent`; added to the Acceptance criteria refusal list.
- **F5 (LOW)** — confirmed against `internal/websterengine/autorebaseline.go`, `Rebaseline` and `changedPlanFiles` in `internal/websterengine/fingerprint.go`.
  The open risk is replaced by a Constraint stating the answer, including that the Card Index-only bound is what keeps rebaseline accepting the edit; Q9 is marked resolved in the support log.
- **F6 (LOW)** — confirmed: `ValidatePlan` reads `websterengine.DoneCards`, the terminal-done set.
  D2 now defines the begun set as every batch record, terminal or not; D3 points at `LoadState` for loading only; the Notes bullet no longer restates the bound and points at `DispatchScope` and `rebaselineBatches`.
  The Acceptance criteria begun-card line now names a begun batch that is not yet terminal.
- **F7 (LOW)** — D5's writer rule and rubric bullet are narrowed to refusals and guard branches, with the grouped raw-I/O row exempt; the matching Acceptance criteria line says so.
- **F8 (LOW)** — the residual-risk bullet moved to the support log's Interview turn 5.
  It carried no requirement beyond D5, so nothing was extracted first.
  `## Open risks` keeps its required heading and now reads "None."

## Disputed

None.

## Deferred

None.
