# Fixer report — plan, round 1

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  Card 8 lists `internal/testkit/envkit#nilLegal` under **Edit:**.
  Its **Intent:** says `nilLegal` gains `InnerRun.NoticeLog.Now`, `Land.Notices.NoticeLog.Now` and `Teardown.Notices.NoticeLog.Now`, each nil-legal because a nil `Now` reads `time.Now`, and names the seam tests that fail without them.
  Its ImpactSummary names envkit's nil-legal list.
  The test budget is unchanged.
  Premise confirmed against `internal/testkit/envkit/envkit.go` (`nilLegal`, `walkNil`), `internal/battencli/wire_seams_test.go` (`TestWire_EverySeamFilled`) and `internal/testkit/envkit/envkit_test.go` (`TestFullEnv_FillsEverySeam`).
- **F2 (LOW), fixed as proposed.**
  Card 1's **Intent:** parenthetical now says no mechanics in `layout.go` or `checksum.go` change, and that `layout.go` changes only the comment of `placement.strip`.
  `internal/reedengine/render#placement` stays on **Edit:**.
  Premise confirmed: `placement` is declared in `internal/reedengine/render/layout.go`.
- **F3 (LOW), fixed as proposed.**
  Card 5 creates `plan:internal/statuspane#FormatMinutes` -> `func FormatMinutes(d time.Duration) string`, and its **Intent:** gives it a bullet with no test of its own, since `TestRenderChecklist` and `TestRenderOverview` assert the exact time strings.
  Cards 7 and 9 render times through `statuspane.FormatMinutes` instead of describing the format inline, and each lists it under a new **Uses:** section.
- **F4 (LOW), fixed as proposed.**
  Card 9 drops `TestFitOverview` and adds its two cases to `TestRenderOverview` as rows that pass `renderOverview`'s output through `fitOverview` with a told row count.
  Its budget bullet is now `internal/battencli` 69 → 72.
  No later card builds on the old 73.

## Self-check

- consistency, LOW: `00-overview.md`'s card 5 summary line did not name the new time format. Fixed as proposed: the line now ends "two-strike status reader and minute-resolution time format".
- claims: no findings.
- way-forward: no findings.

## Disputed

None.

## Deferred

None.
