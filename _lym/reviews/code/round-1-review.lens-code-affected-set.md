```yaml
lens: code-affected-set
outcome: reported
```

Task: list, per change, the set that reads or asserts it, and report any member the diff leaves wrong.

**Per-change list**

| Change | Set found | Covered? |
|---|---|---|
| `PATTERN-sole-parsers` text (declared change) | `PATTERN.md` line and `pattern/PATTERN-sole-parsers.md` | Covered. The line carries the declared text. The background file gains the `JoinParts` write path and a parts-format section. |
| Pattern-Departure trailers | All 20 card commits and the fix commit (`377fd58d`) | None carries a trailer, so nothing to check. |
| New tapestry rows and `entry_gate` edges | `contracts/recipes/tapestry-recipe.yaml`, `internal/loomrecipe/plansplit_test.go`, `internal/loomrecipe/fixture_test.go`, `sequence_test.go`, `shape_test.go`, `docs/overview.md`, `internal/reportengine/timingspans.go`, `internal/loomshed/interruptpolicy.go` | Covered. I found no other doc listing the old `Drift-Baseline` → `Plan-Write` edge. The hits in `internal/sheddriver/tackler_test.go:129` and `internal/shedarbiter/call_test.go:139` use their own fixtures. |
| `partsparser` as a new sole parser | `partsparser/*_test.go`, `internal/loomshed/seam_enforcement_test.go`, `internal/loomcli/validateparts_test.go` | Covered. The seam-enforcement test is touched. |
| `Seat.Writer`, `Seat.Timeout` and `SeatResult` fields | `internal/seatengine/*`, `internal/shedadapters/multillm.go`, `doc.go` in `seatengine` and `shedadapters` | Covered. I found no seat-engine spec under `contracts/specs`, so no spec is stale. |
| `verbUsesLightweightWiring` gains `validate-parts` | `internal/loomcli/cli.go`, `wiring_commitstatus_test.go` | Covered. |
| `join-parts` is not in `verbUsesLightweightWiring` | `internal/loomcli/cli.go:150` | See the one finding below. |
| `settledSegmentGuard` signature, now with a history reader | `internal/loomcli/settledguard.go` and `settledguard_test.go` | Covered. `segmentWriter` has one caller. |
| `planGate` signature, now taking `planDir` | `NewPlanGate`, `NewLensReviewPlanGate`, `NewPartPlanGate` | Covered. |
| `planparser.Plan.Part` and `Reparse` | `overlay.go`, `validate.go`, `planglyph` | Covered. `Reparse` keeps the part scope. |
| Plan-format blocks, `Plan-Write` prompt | `loom-template-plan.md`, the four blocks, `govalues_test.go`, `markers_test.go`, `citation_enforcement_test.go`, `loom-rubric-plan-review.md` | Covered. I expanded the includes against the old template. The only difference is blank lines at block boundaries, from my emulation not trimming a block's leading blank line. I expect the stencil loader's comment strip to trim that, but I did not run the real loader, so the line-for-line claim is unverified. |
| Plan spec and webster spec edits | `loom-plan-spec.md`, `webster-spec.md`, `specs.go`, `specs_test.go` | Covered. |

**Finding**

- **LOW, consistency.** Location: `internal/loomcli/cli.go:287-296` (the `verbUsesLightweightWiring` doc and switch).
  - The diff adds `validate-parts` to the lightweight-wiring set but not `join-parts`.
  - `join-parts` therefore takes the full `wire()` path, including its early config refusal, so a broken module config blocks the chair's join.
  - `join-parts` reads `c.recipe`, `c.shedPaths`, `c.env` and `c.planContext`, and calls the full `wire()`.
  - D8 says nothing requiring lightweight wiring, so I could not tell whether this is deliberate.
  - If it is deliberate, the doc comment should say so. I read nothing that confirms it.

No other member of the affected set is left wrong.

**Tool calls**
1. One `grep` and `git diff --stat` call. It covered the stale row-order mentions, which test files reference `partsparser`, whether any seat-engine spec or doc mentions `Required`, and the diffstat of docs and specs. My context lacked a repo-wide search for those.
