# Fixer-report, code round 1

## Changes

- F1 (`90a77516`): added `internal/fabricengine/boardoverview_test.go`, a tier0 table pinning `isBoardOverview` (`README.md` and `project-x.md` are overviews; `project-.md`, `project-a/b.md`, `dir/project-x.md` and others are not).
  Added a Bolt row to `TestRecordsReplay` in `replay_integration_test.go`: a commit touching `project-x.md` beside `board.json` and `README.md` replays by entry with no stop, against an origin commit that also changed `board.json`.
  The summary's Tests line the finding mentions belongs to the handler's summary, which this round does not write.
- F3 (`c2523623`): added a `ghost` dependency, a slug on no board entry, to task T3 in the projects golden row and pinned its code span in the After cell, `T2<br>T7<br>`ghost``.
- F4 (`c2523623`): added `project-tidy.md` to the `files` map of the same row, pinning its Draft subsection after the empty Ready section.
  Extended the test's doc comment to name both pins.
- F5 (`6bfb76dc`): added a comment on `projectFilePrefix` in `internal/boardengine/render.go` and on the const block holding `boardProjectPrefix` in `internal/fabricengine/replay.go`, each saying the prefix must equal the other package's.

`lym gate test --tier 0` and `--tier 1` on `./internal/boardengine ./internal/fabricengine` are green.

## Decision-class findings

F2 went to the parent and is left out, as the fix step requires.
The parent's recorded decision (creel #116) keeps the `replay.go` change and asks for the tests F1 names, which F1 now holds.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
