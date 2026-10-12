# Plan round 1 — fixer report

Every finding's premise held against the cards and the code it cites (`internal/shedadapters/archive.go:60`, `internal/fabricengine/fabric.go:134`, `internal/orchcli/cli_test.go:113`).
After the edits, `lyx loom validate-plan` returns `ok: true` with informational `create-new-unit` findings only.

## Changes

- **F1** (as proposed): card 14's `Week` gains `Arbiter`, `LLMSteps` (`landing` included), `DriveReports`, `Decisions` and `Alignment`, each summed over the week's runs holding its source, with `Lacking` naming the rest and `nil` when no run holds it; the card states these reach the JSON alone, since D12's markdown sections are the four board questions.
  `TestCompute_Weeks` gains an arbiter row (one run holding the source, one lacking it, counted in `Lacking["arbiter"]`) and an LLM-step sum row per source.
- **F2** (as proposed): card 5 creates `shedadapters.ArchivedRunDirSegment` in `archive.go` and edits `TestArchiveRunDir` to pin it against `archiveRunDir`'s naming, collision suffix included.
  Card 12 uses it: an archived sibling adds its rounds to its segment's `Rounds`, `Outcome` stays the live directory's, plus the `plan` / `plan-20261012T101500Z` row of `TestCompute_RepairsAndRounds`.
  Card 11 uses it to add a sibling's `EscalatedRounds` to the segment's `Escalations` count.
  Beyond the fix key, card 11 also gains a `TestCompute_WakesAndStops` row: an escalation file only in the archived sibling still gives a `review-escalation` wake, which pins card 11's half of the change.
- **F3** (as proposed): card 8 creates `plan:internal/gitrepo_test#TestReportReads`, and card 9 edits that glyph in place of the file path.
  Card 10 creates `allowedImports`, `fakeRepo` and `fakeRepo.BlameLines` and names `allowedImports` in its Intent.
  Cards 11, 12 and 13 edit `allowedImports` in place of `seam_enforcement_test.go`, and their Intent lines name it too.
  Card 13 edits `fakeRepo` and `fakeRepo.BlameLines` in place of `fake_test.go`.
  Cards 11 and 12 keep their `fake_test.go` path.
- **F4** (as proposed): card 10 gains the `archive/x` row; card 13 gains the exact-joined landing whose slug no board commit holds (kind `other`, one finding, not blamed).
- **F5** (as proposed, third-test option): card 15's integration test first runs from the prime's records-side worktree and expects the `RequireDrivableWorktree` refusal.
  `cli_test.go` gains `TestRefuseNonPrime`, a table shaped like orch's, and the budget row becomes `internal/reportcli: 3`.
  A separate test was chosen because `refuseNonPrime` is a pure function, while `TestConvergence_RefusesBadFlags` rows drive `RunCLIIn`.
- **F6** (as proposed): card 13's exact join walks in growing pages (`Limit` 64, doubling, re-read from the tag) until a trailer is found or a page comes back short.
- **F7** (as proposed): card 13's time join opens a never-tagged slug's window at its newest checkpoint, and the not-yet-torn-down row uses a slug with no archive tag.
- **F8** (as proposed): card 15's default-since step accepts either of the dates 84 days before the UTC dates taken just before and just after the `RunCLIIn` call.

## Disputed

None.

## Deferred

None.
