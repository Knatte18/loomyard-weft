# Webster-Review round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit per finding.
`go build ./...`, `go vet -tags smoke ./...` and `go test -tags integration ./...` pass after the last commit.

## Fixed

- **F1 (MEDIUM)** — `08e020835`.
  `NewPlanDirRotator` now plans the rotation first (`stalePlanRotation`: plan directory, free archive path, stale file names, no disk writes), renders the prior-plan block, and only then moves files (`rotateStalePlanDir`).
  A render failure leaves the plan in place, so the retry announces it.
  The missing-stencil subtest now asserts the plan directory is untouched after the failure and that a retry with the stencil present announces the file.
- **F2 (LOW)** — `ebdf9daff`.
  Semantic line breaks in the new `NewPlanDirRotator` and `rotateStalePlanDir` doc lines.
- **F3 (LOW)** — `631af3673`.
  Semantic line breaks in the new `singlellm.go` constructor paragraph, the `Call` doc sentence, the inline comment in `Call`, and the changed `shedadapters/doc.go` list item.
- **F4 (LOW)** — `f0841983b`.
  The edited `planWriteEntry` doc sentence is on one line.
- **F5 (LOW)** — `f3bd6c507`.
  The `loomshed/doc.go` sentence is on one line, and the `loomengine/plan.go` sentence breaks at its semicolon only.
- **F6 (LOW)** — `e29b05465`.
  Added launch rows on pwsh and posix that combine fork mode with the real assembled notice (its `:` and `;` inside the one quoted argument, inside the env wrap), and `TestBuildResumeCmd_NoticePosix` for the posix resume line.
- **F7 (NIT)** — `53262b8af`.
  The rotator wraps the render error as `loomshed: announce prior plan:`, so the chain no longer repeats `render prior-plan block`.
- **F8 (NIT)** — `5c40a6e8a`.
  `PriorPlanBlock`'s first doc sentence is on one line.

## Deferred

None.
