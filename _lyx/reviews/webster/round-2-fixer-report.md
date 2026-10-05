# Round 2 fixer report

## Fixed

- F1: `loomengine.LoadConfig`'s refusal for `review_circling_checkpoint` and `review_max_bounces` below 1 now ends with "set it to a positive integer in loom.yaml".
  `contracts/specs/refusal-spec.md` gained a `config: review key below 1` row in the loom section.
  `TestLoadConfig_RejectsReviewKeysBelowOne` now asserts the way forward as well as the key name.
  `go test` over `internal/loomengine` and `contracts/...` passes.

## Deferred

None.
