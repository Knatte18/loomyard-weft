---
round: 1
focus:
  - "Card 4, `internal/websterengine` and `internal/webstercli`: does every caller that formerly formed batches (`Run`, begin-batch, await-batch, record-batch, recover-batch, rebaseline, validate, render, suspect) now read the recorded partition and fall back to `batcher.Identity()` only for a state without one? Settle whether any path still recomputes batches from the disk sizes mid-run."
  - "Card 4, `CheckBatchOrder`: does the order assertion refuse a partition that reorders cards or runs a batch before a card it depends on, with the refusal text and way forward the plan and `refusal-spec.md` name?"
  - "Card 5, `internal/batcher/cost.go`: does the DP match the plan's feasibility rules (a one-card segment always feasible; `AloneAbove`, `MaxCards` and `Budget` apply only to segments of two or more cards; ties go to more batches), and does `cost_test.go` carry the brute-force comparison over a deterministic input set?"
  - "Card 6, `Active` and `config.go`: do the empty `active:` and `active: identity` resolve to identity with no `profiles:` block, does a configured `identity` profile win, and does each listed load error name `batcher.yaml` and the offending profile or key? Settle whether `ConfigOpenMaps` is wired through `configreg.Modules()` so reconcile keeps operator profiles."
  - "Card 6, `template.yaml`: is `active: \"\"` still the first line and do the `cautious` values match the card's numbers, with `active` left on identity?"
  - "Card 7, `rebaseline.go`: is the kept prefix computed from `State.Batches` records, does a card set change at or before the last begun batch refuse with `ErrRebaselineCardSetChanged`, and does every refusal leave `State` and `Partition` unchanged because `Partition` is replaced only after all checks pass?"
  - "Card 8 has no commit by plan; confirm `internal/batcher/template.yaml` was not changed by card 8 and that no `tokencount` calibration output or report was committed."
  - "Per-card mechanical check for each card that carries a Rename or Delete group (card 6 deletes `batcher_test.go` and `registry_test.go`): did the type-specific check run against every target of the group, and do the deleted tests' behaviors survive in `config_test.go` or another test?"
  - "Docs in the same commit: do `doc.go`, `docs/overview.md`, `PATTERN.md`, `refusal-spec.md`, `loom-plan-spec.md` and `webster-spec.md` changes land in the cards that the plan assigns them to, and do new doc comments follow the repository's own constraints document?"
  - "Test economy: new tests in `partition_test.go`, `cost_test.go`, `config_test.go` and the rebaseline tests; name the existing test that covers any duplicated behavior, and name any added behavior with no coverage."
---

Seed focus from the plan's Card Index and the seven committed cards.
Review range: `git merge-base main HEAD` to `HEAD`, since no rework round exists.
