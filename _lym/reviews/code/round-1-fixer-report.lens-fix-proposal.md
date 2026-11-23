```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals F1, F3 and F4 that the fixer carried out as proposed.

**Findings**

1. **LOW** — `contracts/stencils/friction/friction-template-reflection.md:48`
   - The rewritten sentence is one line of about 138 characters of text: "A `build-skew` halt is dropped too: the run's lym was older than a board stencil it read, which is a binary to update and not a lym defect;".
   - The prose skill caps a line at 119 characters and says to split at a semicolon, colon or clause boundary.
   - The reviewer's replacement was a single clause with no break point, and the fixer copied it without breaking it.
   - Put a line break after "it read," so the `which is a binary to update…` clause starts its own line.

No other proposal broke anything.

- **F1:**
  - The wording in `doc.go` and `anomaly.go` now matches the reviewer's cited hand-over row, `binaryEligible(stop) && BuildChanged && !HandedOver`.
  - The `doc.go` lines are about 100 characters or fewer, so they are within the limit.
- **F3:**
  - "does not hand the run over, and its tackler settles the stop" holds for the failed-producer path the review traced.
  - With `tackler_max_spawns` at 0 the tackler is off, so the stop would not be settled there.
  - That is a corner case, and the doc says a missing verdict reads as a hand-back.
  - The wording is a change-description sentence and no code depends on it, so I raise no finding.
- **F4:**
  - The new row has `StateRunning`, `buildSkew` true and an empty reason.
  - In `ClassifyHalt` at `anomaly.go:99-113`, removing the `state == shedengine.StateFailed` guard from the build-skew case makes that row return `AnomalyBuildSkew, true`.
  - That differs from the wanted `"", false`, so the row fails with the defect in place.
  - The `interrupted` case sits earlier but is false in the row, so it does not mask the defect.

**Tool calls**
- Read `internal/loomengine/anomaly.go:85-129`: my context lacked the `ClassifyHalt` body, which I needed to confirm the F4 row fails with the guard removed.
