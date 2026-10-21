---
verdict: APPROVED
---

## Scope of this review

This round was cut short by a usage limit.
It read the decision record, the overview and cards 1 to 11, and settled focus items 1 to 3 against the code.
Focus items 4 and 5 were not examined, and the fidelity, granularity, test-economy, attack-surface and card-order checks got only a read-through, with no cross-checking against the code.
No defect turned up in what was checked, but this approval does not cover those unexamined parts.

## Settled focus items

- Card 9, `logcapture` under parallel tests: `TestEnsureOrWarn_WarnsWithTheWayForward` is one top-level test that never calls `t.Parallel`.
  The existing capture tests in `internal/hubreconcile/ensure_integration_test.go` are built the same way.
  Go resumes a paused parallel top-level test only after every serial top-level test has finished, so another test's Warn cannot reach the buffer.
  Not a defect.
- Card 9, loom resume row: `resume` is not in `genericShedVerb`, so `arm` does not stop at the missing-seed refusal.
  After `EnsureOrWarn` it reaches `wire`, whose first step is `loomengine.LoadConfig`.
  `configengine` words a parse failure as `config file <path>: ...`, so the load error names the file.
  The run helper uses `CombinedOutput`, so the Warn on stderr and the load error are both in the asserted output.
  The assertions the card states hold against a correct implementation, and the card claims no refusal text the load does not produce.
  Not a defect.
- Card 6, per-name drop on a present base: `yamlengine.Prune` removes keys the template lacks and never drops a per-name entry.
  `yamlengine.Reconcile` is the function that returns `dropped`, and it also adds template keys, so it cannot be used on a sparse base.
  The card's new `DropTemplateEqualNames`, sharing `sameValue` with `mergePerNameMaps`, is therefore the mechanism that can produce the `Result.Dropped` report.
  Not a defect.

## Not examined

- Card 8: whether each Edit glyph whose parameters change carries the re-sign arrow, and whether every caller that must pass the new parameter is a target.
- Card 11: what `CloneAndWire` leaves when its board commit fails after the marker write, and whether the next start verb's walk recovers it as card 8 describes.
