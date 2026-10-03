# Webster-Review round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit each.
After the fixes, `go build ./...`, `go test ./...`, the plan's integration run, and its smoke-tagged `go vet` all pass, and `gofmt -l` over the changed Go files is empty.

## Fixed

- **F1 (BLOCKING)** — `273745f04`.
  Added `SegmentBounces` to `nilLegal` in `internal/testkit/envkit/envkit.go`, since its nil is a documented default.
  `internal/testkit/envkit` and `internal/battencli` pass again.
- **F2 (MEDIUM)** — `f9b0a9288`.
  Ran `gofmt -w` on `internal/loomcli/wiring.go` and `internal/loomengine/config.go`.
- **F3 (MEDIUM)** — `8e7defc94`.
  Reflowed every listed new or changed Go comment to one sentence per line, breaking only at independent-clause boundaries.
  This covers `internal/shedadapters/{bouncer,bouncerfacts,bouncerfiles,circling,doc,round}.go`, `internal/burlerengine/{doc,verdict}.go`, `internal/loomengine/review.go` and `internal/shedrecipe/{recipe,entries_bouncer}.go`.
  Where a changed line sat inside a legacy wrapped paragraph (`NewBouncer`'s budget rule, `Call`'s clear rule, `settle`, the `Finding` doc, the `Env.Slug` doc), the whole paragraph was reflowed so no sentence straddles a fixed-column break.
  The text is unchanged apart from two `--` asides, which became a semicolon and a sentence break so that no line starts with a dash.
- **F4 (LOW)** — `cb3cd41ee`.
  The circling verbs' status reader is now `circlingStatusReader` in `internal/loomcli/circling.go`.
  It reports not-found only when the status file is absent, and otherwise creates the lock's scratch directory before reading.
  `TestCirclingStatusReader` covers both cases, including a status file whose scratch directory is missing, which the old reader refused as absent.
- **F5 (NIT)** — `4b3f38814`.
  Removed the blank line between the two new Review Round Invariant bullets in `CONSTRAINTS.md`.

## Deferred

None.
