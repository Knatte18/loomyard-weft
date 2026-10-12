# Fixer report — plan, round 1

## Changes

- **F1** (BLOCKING, fixed as proposed): the premise holds.
  `envkit.nilLegal` (`internal/testkit/envkit/envkit.go:38`) lists `InnerRun.Now`, `Land.Notices.Sleep` and `Teardown.Notices.Sleep`, but none of the new `NoticeLog.Now` paths, and `envkit_test.go`'s `TestNilSeams_SkipsNilLegalPaths` shows that the skip list is the only exemption.
  Card 8 now has `internal/testkit/envkit#nilLegal` under **Edit:**.
  Its **Intent:** says `nilLegal` gains `InnerRun.NoticeLog.Now`, `Land.Notices.NoticeLog.Now` and `Teardown.Notices.NoticeLog.Now`, each nil-legal because a nil `Now` reads `time.Now`.
  Its **ImpactSummary:** names envkit's nil-legal list.
  The test budget is unchanged.
- **F2** (LOW, fixed as proposed): card 1's **Intent:** now says that no mechanics in `layout.go` or `checksum.go` change, and that `layout.go` changes only the comment of `placement.strip`.
  `internal/reedengine/render#placement` stays on **Edit:**.
- **F3** (LOW, fixed as proposed): card 5 creates `plan:internal/statuspane#FormatMinutes` -> `func FormatMinutes(d time.Duration) string`.
  The **Intent:** paragraph places it in `pane.go`, ties it to the `minute-resolution-times` decision and says it has no test of its own.
  Card 5's opening line now names the shared time format.
  Card 7's `renderChecklist` bullet and card 9's `renderOverview` bullet now call `statuspane.FormatMinutes` in place of the inline format, and each card has a **Uses:** section naming it.
  Card 5's import list is unchanged, since `time` is standard library.
- **F4** (LOW, fixed as proposed): card 9 drops `TestFitOverview`.
  Its two cases are now `TestRenderOverview` rows that pass `renderOverview`'s output through `fitOverview` with a told row count.
  The `test-budget.yaml` bullet is lowered to `internal/battencli` 69 → 72.
  No later card raises the battencli row.

## Disputed

None.

## Deferred

None.
