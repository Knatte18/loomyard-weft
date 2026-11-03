---
round: 2
focus:
  - "internal/fabricengine/originreconcile.go, originTargets: with scope.PairMayBeAbsent set, does a task worktree whose directory is gone but whose git registration is still listed as prunable take the absent-pair path, so the slug is classified and the run's entry is judged for owned-elsewhere, ended or stopped before the pair is re-adopted; and without PairMayBeAbsent does it still return the earlier error?"
  - "internal/fabricengine/originreconcile_integration_test.go, PairMayBeAbsent subtest: is there a row for a deleted-but-unpruned task worktree that fails if the prunable case is treated as present?"
  - "internal/battencli/readoptstop.go and internal/fabricengine/archive.go: is the archive tag name formed in one place only, with no second declaration of the tip length?"
---

Round 2 re-checks the areas the round 1 fixes touched; it is no mandate to skip the rest of the range.
