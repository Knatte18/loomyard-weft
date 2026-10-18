---
round: 1
focus:
  - "internal/reportengine/runs.go and weeks.go: does run selection admit exactly finalized loom and darn seeds, and keep unfinalized or other-recipe (batten) seeds partial, in both `lyx report timing` and `lyx report convergence`?"
  - "internal/reportengine/timingspans.go: does each recipe's closed phase table cover every row its runs can write, with the Darn-Write, Darn, Describe, Darn-Review and Merge-In mappings the summary claims, and does the producer lookup for a Run-Shed change use the run's own recipe table?"
  - "internal/reportengine/timing.go: is `Phases` keyed by recipe then phase consistently in the computation, the JSON output and the medians, with sleep and wait accounting unchanged for loom and applied to darn runs?"
  - "Board entry scope: does the diff cover timing, convergence and the recipe tag, and does it stay within what the entry licenses, including the closed table per recipe instead of a recipe-derived phase set?"
  - "Docs in the same diff: do `internal/reportengine/doc.go` and `docs/overview.md` match the changed behavior and the Recipe columns?"
  - "Test economy: do the new `TestCompute_SelectsRuns` and `TestComputeTiming` rows and the markdown tests cover the added behavior without duplicating an existing test, and is any added behavior left uncovered?"
---

Seed focus from the board entry, the change description and the diff stat; the reviewer judges the full diff beyond these entries.
