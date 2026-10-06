# test-suite-prune-rest: prune the remaining Go test suites and measure

All 23 batches reached `done`.
Batch 1 recorded the before measurement, batches 2–22 pruned the in-scope packages group by group, and batch 23 reran the redundancy report and recorded the after measurement.

This session drove only batch 23, the earlier 22 were already `done` on the resume trail.
Its fork committed at `316f4659d`.
It reran every in-scope package, spliced each section into `docs/benchmarks/test-redundancy.md` and filled the After section of `docs/benchmarks/test-suite-prune-rest.md`.
The fork's own report lists these measurements.
Top-level tests went from 3377 to 1997, summed median wall time from 335.32 s to 110.33 s, and summed serial time from 350.56 s to 150.11 s.
It lists thirteen packages whose wall time rose by 0.10 s or less, with the timing runs starting at higher load than the before runs.
It also reports `cmd/lyx` `TestExitCode_HelpPaths` as a tool artifact: the tool lists it `failed`, but it passes alone and was `failed` in the before report too.

The final rerun found gaps in the shed and config packages, and the fork closed them:

- It added keep directives to `TestShedVerbTexts_GotoAdmissionRuleOnlyOnGoto`, `TestConfigRejectUnknown` and `TestBinaryChanged`.
- It deleted `TestMark_NeverDoubleWraps` and moved its assertion into `TestClassAndMark`.
- It added a process-cwd row to `TestRunCLIIn_FromNonGitDirectory` in `internal/configcli`.
- It added a params-differ case to `TestWriteSeed_RefusesDisagreeingSeed` in `internal/shedrun`.

The fork reports that a block-by-block comparison against the before coverage profile finds no lost block.

Deviations reported by `record-batch` for batch 23: test files outside the card's declared targets, all informational.
They are `internal/configcli/configcli_test.go`, `internal/shedcli/cli_test.go`, `internal/shedrecipe/config_test.go`, `internal/shedrun/seed_test.go`, `internal/shedtransient/shedtransient_test.go` and `internal/shedverbs/buildidentity_test.go`.
