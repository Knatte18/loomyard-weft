# Code review reads the gate's redundancy report

Code-Review now sees the redundancy report that `lym gate test` writes, so the reviewer can fold or delete redundant tests instead of judging test economy from the diff alone.

## What changes

- `loom-rubric-code-review.md` gains a `redundancy_report` marker and a "Redundancy report" section.
  Go fills the marker for the Code-Burler-Review and Code-Bouncer call sites.
- The rubric treats each test the report lists as contained as a test-economy candidate.
  The reviewer checks the candidate's assertions against its covering tests, because coverage blocks do not see assertions.
  It files a fold-or-delete finding naming the covering tests, or accepts the test with a `//testtiming:keep <reason>` finding when the test pins what coverage cannot see.
  A test the report lists as unique or not judged raises no finding on that ground.
- `internal/shedadapters`: `RubricNotes` gets a `RedundancyReport` function, and `BouncerConfig` gets the same field.
  `ReadRubric` calls it only for a rubric that carries the marker, since reading the report matches it against a commit's tree.
  A nil function or a blank answer renders `none`, like the other optional markers.
- `internal/loomcli` (`redundancynote.go`): renders the note and is wired into the recipe's review call sites.
  When the report matches HEAD's tree, the note holds one line per test and per error.
  Otherwise it holds one line saying no current report exists and why: none written, a red tier (naming the gate key and time of the call), a report that judged nothing, a read or match error, or a different tree.
- `internal/shedrecipe`, `internal/testkit/envkit`: wiring and test support.
- `PATTERN-test-speed`: the index line and the background's "Tests run through the gate" and "Test economy" sections now name `lym gate testmap`, the gate's `redundancy` step, the package-scoped verdict with its module-wide column, the map, and the step's bounds.
  They also say a gate finding is evidence for Code-Review exactly as a report candidate is.
  `docs/overview.md` is updated to match.

## Design decisions

- The match is by content, through `gatecli.ReadRedundancyReport` and `gatecli.RedundancyReportMatches` against the HEAD SHA from `fabricengine`.
  A report therefore stays valid across commits that touch only run records or code outside the judged packages.
- A report written on a red tier records no hashes and never matches.
- The match ignores the map, so a map rebuilt since the report can leave a contained verdict stale.
  Review reading each finding bounds this.
- The rubric tells the reviewer to raise no finding about a missing report, so the feature stays advisory.

## To check by hand

- Run a Code-Review against a branch with a current report and confirm the rendered rubric lists the report's tests.
  Then confirm that a commit changing a judged package's Go files turns the note into the no-current-report line.
- No regression check ran for this task.
