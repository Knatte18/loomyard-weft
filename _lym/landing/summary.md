# Refresh the README with what landed since 2026-11-11

The root `README.md` now matches the code and reports that landed after its last refresh.
Only the sections the task names change.

## What changes

- Mechanical gates: a gate with the coverage scope selects tests from per-package coverage maps, and `lym gate redundant` lists tests other tests already cover.
- Review rounds: a decision-record round can ask the parent orch for a review, with a deadline, and keeps its findings.
- Lens reviews in forks: the bullet carries the operator's wording.
- Measurements table: rows for the coverage-map pilot, the plan-writer bench and the advisor bench, each with its sample size and the negative result stated as one.
- Several machines: the claim that several machines work on different projects at once is replaced by what the code delivers, one owner machine per board entry and `lym adopt` taking a project's runs over.

## Design decisions

- The coverage claim says "a gate with the coverage scope", since the scope is a per-gate key in `internal/gatecli`'s config, not a hub-wide default.
- The advisor row's finding opens with "No", as that bench shows no match of the Opus writer's first-pass quality.
- The names table keeps its "lens" row unchanged, as no other section changes.
