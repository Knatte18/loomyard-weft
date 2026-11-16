# Round 1 fixer-report

## Changes

- **F1** (fixed as proposed), commit `5d3ca088`.
  `internal/boardengine/task.go` declares `pullRequestKey` and `onLandKey` beside the `Task` type, each with a one-line comment.
  `render.go` `runFieldLines`, `replay.go` (both `Name` values), `front.go` (three sites) and `store.go` (both entries of the key list) use them.
  The struct tags stay literal.
- **F2** (fixed as proposed), commit `8add579e`.
  `internal/boardengine/doc.go` now reads "The README's Recipe cell names only a profile the task carries."

`go build`, `go vet` and `lym gate test ./internal/boardengine` pass.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW, `front.go:57` mixes `onLandKey` with the literal `"front"`: disputed, see below.
- fix-proposal, LOW, `store.go:500` mixes the constants with bare literals in the key list: disputed, see below.

Every fork returned a report.

## Disputed

- fix-proposal LOW on `front.go:57` and LOW on `store.go:500`, merged as one.
  Ground: premise false.
  The finding asks for constants of the other keys, which F1 never named, and the rubric says a further improvement the review never asked for, and the fix did not make worse, is not a finding.
  The remaining literals are keys that no review finding or `PATTERN` hit named, and the fix made no site worse.

## Deferred

None.

## Rulings

None.
