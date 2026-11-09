# Code review reads the gate's redundancy report

The Code-Review rubric now carries the gate's redundancy report, so a reviewer sees which of the diff's new and changed tests the coverage maps find contained.
`PATTERN-test-speed` names the coverage map, `lym gate testmap` and the gate's `redundancy` step.

## What changes

- `contracts/stencils/loom/loom-rubric-code-review.md` gets a `## Redundancy report` section with a Go-filled `redundancy_report` marker.
  Its test-economy text treats each contained test as a candidate.
  The reviewer checks its assertions against its covering tests.
  It then files a finding to fold or delete it, naming the covering tests,
  or a `//testtiming:keep <reason>` finding when it pins what coverage cannot see.
  A `none` or no-current-report note raises no finding of its own.
- `internal/shedadapters`: `RubricNotes.RedundancyReport` and `BouncerConfig.RedundancyReport` are `func() string`.
  `ReadRubric` calls it only for a rubric that carries the marker, since the read matches the report against a commit's tree.
  A nil or blank answer renders `none`.
- `internal/shedrecipe`: `Env.RedundancyReport` reaches both call sites, the BurlerReview entry (Code-Burler-Review) and the Bouncer entry (Code-Bouncer).
- `internal/loomcli`: `redundancyReportNote` reads the worktree's report through `gatecli.ReadRedundancyReport`
  and matches it against HEAD's SHA through `gatecli.RedundancyReportMatches`.
  A matching report renders one line per test, with its verdict and either its covering tests or its reason, then one line per error.
  Any other case renders one line starting `No current redundancy report:` and naming why:
  no report written; a red tier, naming the gate key; a report that judged nothing, with its note; a failed match; or a report written for another tree.
  A red or note-only report renders without a match, so it costs no git read.
- `contracts/stencils/rubric_test.go` allows the new marker and pins that only the Code-Review rubric carries it.
- `PATTERN.md`'s `PATTERN-test-speed` line now reads "it pins behavior no other test covers, which the gate's redundancy report checks".
  The background's "Tests run through the gate" section adds `lym gate testmap` and the report step as runs that take a tier's `env` and a gate slot.
  A new subsection of "Test economy" states the map, the step's package-scoped verdict, its bounds,
  the match Code-Review reads it under, and that a gate finding is evidence exactly as a report candidate is.
- `docs/overview.md` and the `loomcli` and `shedrecipe` package docs describe the marker.

## Design decisions

- The note is a `func() string` that never fails, unlike `ReviewBase`.
  The report is advisory, so a read or match failure becomes the note's own reason line and never blocks a review row's build.
- The read is lazy, keyed on the rubric's top-level markers.
  Every Bouncer and BurlerReview row gets the closure, and only the Code-Review rubric carries the marker.
  So Plan-Review and Discussion-Review never pay for the merge-base diff and tree reads.
- The rendering lives in `internal/loomcli` beside the wiring.
  The data and its match stay in `internal/gatecli`.
- `TestRedundancyStep`'s match subtest commits its stand-in run record as `records/notes.md`, a commit touching no Go file.
  The fixture pair's `_lym` is a link git cannot add through, and the matcher gives `_lym` no special case,
  so that commit pins the acceptance case of a commit touching only run records.
  The fixture's `drops` package already carries a `tier2` test in the matched diff.

## To check by hand

- The rubric's new paragraphs in `## Also flag`'s test-economy bullet and the `## Redundancy report` section, against D8's wording.
- The new "The coverage map and the gate's redundancy report" subsection of `pattern/PATTERN-test-speed.md`, against D9.
