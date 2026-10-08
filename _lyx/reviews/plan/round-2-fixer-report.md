# Plan-Review round 2 — fixer report

Each finding was checked against the code before it was fixed; every premise held.

## Fixed

- **F1 (BLOCKING, scope)** — confirmed: `internal/ideengine/menu.go:109` calls `Spawn(l, chosenSlug)` in the one-value form.
  Card 15 now lists `internal/ideengine/menu.go`.
  Its Intent states that `Menu` takes the new return and logs a skip with its reason via `logger.Warn`, and that `menu_test.go` needs no change.
  The ImpactSummary names both production callers.
- **F2 (BLOCKING, scope)** — confirmed: `internal/orchengine` has 122 top-level tests against a row of 122, and `watcher_test.go` uses one function per behavior.
  Card 14 now lists `cmd/lyx/testdata/test-budget.yaml` and sets the `internal/orchengine` row to the package's count after the card.
  It also states that `start_test.go` adds no function.
- **F3 (MEDIUM, scope)** — confirmed: `watchdog_integration_test.go:153,221,255` and `attachgeometry_integration_test.go:218,427,519` assume one status row.
  Card 5 now lists both files.
  Its Tests section says those height checks expect the client's rows minus the two reserved status rows, and that their comments say `status` is pinned to `2`.
- **F4 (LOW, consistency)** — confirmed: M19 in `tools/sandbox/SANDBOX-REED-SUITE.md:342` requires the `#{status-left}` identity text, and `docs/shared-libs/yamlengine.md:116` cites `status_line`.
  Card 5 now lists the sandbox suite and rewrites M19's check to the two-line bar.
  Card 6 now lists `docs/shared-libs/yamlengine.md` and swaps its example to `segment_colors`.
- **F5 (LOW, design)** — card 2 drops the `validateSegmentColors` rows.
  Its new test file keeps only the `segmentColor` table, and the `TestUp_BootValidation` row covers the refusal through `Up`.
- **F6 (NIT, consistency)** — confirmed: `probe.go` holds only `requiredSubcommands` before card 7.
  Card 4's `doc.go` text now says only that `SwitchClient` issues `switch-client`, and leaves the optional-verb wording to card 7.

## Disputed

None.

## Deferred

None.
