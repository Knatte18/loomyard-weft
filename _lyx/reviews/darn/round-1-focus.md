---
round: 1
focus:
  - "internal/shuttleengine/enginesrun.go and run.go: does the ledger write in Start happen before strand registration, fail the start and remove the run directory on error, and is concurrent access across runners in one worktree safe (lock file, atomic write)?"
  - "internal/shuttleengine/enginesrun.go: does Runner.EnginesRun return the sorted union of ledger and surviving run records, counting a record with no engine as the engine set's default, and does the orphan sweep leave the ledger and its lock file alone?"
  - "internal/landingshed/config.go: do TrailerBlock ordering, trimming and empty-value handling match the board entry, and does the LoadConfig check against the engines modelspec knows cover the open-map case without rejecting valid keys?"
  - "internal/landingshed/config.go MigrateConfig and configreg.go: is the co_authored_by rewrite idempotent, does it keep an existing engine_trailers map, and is it wired as landing's Migrate hook so reconcile reaches it?"
  - "internal/landingshed/finalize.go and deps.go: is the EnginesRun read placed before any parent-side mutation, does its failure return an error with nothing merged, and is the seam wired in internal/loomcli/landingdeps.go with the wire-seam tests updated?"
  - "internal/summaryparser/summary.go: does LandingMessage append the block after a blank line and yield the bare message for an empty block, and does the Describe validation still refuse a Co-Authored-By line?"
  - "Compare the diff to the board entry for scope in both directions, and check each claim in _lyx/landing/summary.md against the diff, including the docs it says changed (doc.go files, docs/overview.md, loom-final-summary-spec.md) and PATTERN.md if a cross-cutting invariant was added."
  - "Test economy: do the new and rewritten tests in config_test.go, enginedispatch_test.go, startup_test.go and description_test.go duplicate existing coverage, and is any added behavior uncovered; does the test-budget.yaml change match the new top-level tests?"
---

Seed focus for round 1, set from the diff stat and the change description only.
