---
round: 1
focus:
  - "Declared PATTERN changes: does the diff to `PATTERN.md` and `pattern/` land `PATTERN-gate-slot` as a change and `PATTERN-test-tiers` as an add, each with its declared text?"
  - "Retired inputs: does any reference to `go_parallel`, `publish_verify`, darn's `verify`, `verify_tiers` or the plan's `## verify:` section remain in code, stencils, prompts, config or docs after cards 13, 16 and 19?"
  - "Stencil wording: for each stencil the diff changes (cards 15 and 16), does it still keep every rule its card's Intent did not ask to change, and name a gate's tier list and scope form rather than a test command or target list?"
  - "Delete and Rename groups: for each, does `assert-no-callers` hold, and is no reference to an old name left?"
  - "Test economy: do the new tests in `internal/gateslot`, `internal/gatecli`, `internal/impactset` and the shell guard duplicate an existing test's coverage, and does any added behavior, such as the `refuse_raw` guard or the plan gate's `--wait` check, lack coverage?"
  - "Refusal wording: does every refusal that names `lyx config gate` carry the role clause `withConfigWayForward` carries, per the plan's Shared Decisions?"
---

Focus covers the plan's cross-card seams, where a late card can leave an earlier card's retired input behind.
