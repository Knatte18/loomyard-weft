# Round 1 fixer-report — plan

## Changes

- **F1** (BLOCKING, fixed as proposed): card 8 adds `internal/testkit/envkit#nilLegal` to **Edit:**.
  Its **Intent:** says `nilLegal` gains `InnerRun.NoticeLog.Now`, `Land.Notices.NoticeLog.Now` and `Teardown.Notices.NoticeLog.Now`, each nil-legal because a nil `Now` reads `time.Now`, and names the seam tests that fail without them.
  The ImpactSummary names the three new nil-legal paths.
  The test budget is unchanged.
  Premise checked: `walkNil` recurses into nested structs and reports nil func fields not in `nilLegal` (`internal/testkit/envkit/envkit.go`), and `nilLegal` already lists `InnerRun.Now` and `Land.Notices.Sleep`.
- **F2** (LOW, fixed as proposed): card 1's first **Intent:** paragraph now says no mechanics in `layout.go` or `checksum.go` change, and that `layout.go` changes only the comment of `placement.strip`.
  `internal/reedengine/render#placement` stays on **Edit:**.
- **F3** (LOW, fixed as proposed): card 5 creates `plan:internal/statuspane#FormatMinutes` (`func FormatMinutes(d time.Duration) string`), with an Intent bullet that gives it no test of its own, since `TestRenderChecklist` and `TestRenderOverview` assert its exact strings.
  The `renderChecklist` bullet in card 7 and the `renderOverview` bullet in card 9 now call `statuspane.FormatMinutes` instead of describing the format inline.
  Each of those two cards gains a **Uses:** section naming `plan:internal/statuspane#FormatMinutes`.
- **F4** (LOW, fixed as proposed): card 9 drops `TestFitOverview` and moves its two cases into `TestRenderOverview` as rows that pass `renderOverview`'s output through `fitOverview`.
  Card 9's budget bullet is now `internal/battencli` 69 → 72.

## Disputed

None.

## Deferred

None.
