# Writing the map run's green-record subtest

Card 3 asks for a tier0 command of `true` beside a `--tier 0 steady` run.
The gate refuses targets for a command with no `{targets}`, and then refuses `{targets}` with no `all_targets`,
so the fixture needed `true {targets}` with `all_targets: ./...`.
It took two extra tier1 runs, because the card gate runs no tier1 test of `internal/gatecli`.
