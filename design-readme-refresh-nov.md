# Refresh the README with what landed since 2026-11-11

`readme-refresh-nov` · Task · enhancement, docs · running · Worktree-Create

Bring the root `README.md` up to what has landed since its last refresh (`efda1928`, 2026-11-11).
The operator will point to LoomYard from job applications, so every claim must hold against the code and the reports it links.

## Changes (operator, 2026-11-15)

1. **Coverage-map test selection**, in "Beyond the run", under Mechanical gates:
   the gate selects the tests of importing packages from per-package coverage maps, so a change runs the tests that reach it;
   `lym gate redundant` lists tests other tests already cover.
   Read `internal/testcover`, `internal/gatecli` and the pilot report `docs/research/speed/coverage-map-pilot-2026-11-12/report.md` for what holds today.
2. **The orch in the decision-record review:** a Discussion-Review round can ask the parent orch, with a deadline, and keeps its findings (commits `32e8e32e`, `4946e478`).
   One sentence where the review rounds are described.
3. **Lens reviews in forks:** replace the current "Review lenses in forks" bullet with this text, the operator's wording:

   > - **Lens reviews in forks.** Writers and reviewers alike hand each lens, a focus such as missing tests, to a forked subagent.
   >   A fork inherits its parent's context and prompt cache, so it starts without re-reading what the parent read,
   >   and a lens costs far less than a new review.

4. **Measurements table:** add rows, in the table's existing form and neutral wording, for
   the coverage-map pilot (`docs/research/speed/coverage-map-pilot-2026-11-12/report.md`),
   the plan-writer bench (`docs/research/loom/plan-writer-bench-2026-11-12/report.md`)
   and the advisor bench (`docs/research/loom/advisor-model-bench-2026-11-12/report.md`).
   State each finding with its sample size; a negative result is stated as such.
5. **Several machines:** the "Parallel tasks, beams and machines" bullet claims several machines work on different projects at once.
   Moving a hub between machines is being redesigned; word the claim so it promises no more than the code delivers today.

No other section changes. Follow the scribe prose rules the README already follows.
