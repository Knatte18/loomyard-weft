# TestScheduler_HookWake blocked-chain subtest flakes under tier_parallel 8

`hookwake-blocked-chain-flake` · Note · bug, creel

## Seen

Run 2 of the `bench-osl1046-xdr` bench, at `tier_parallel` 8 on OSL-1046, failed one `internal/creelengine` test: `TestScheduler_HookWake/MailArrivingInAWokenTurnsBlockedChain...` (the strand's report truncates the name).
The run's cap allowed no repeat, so it was not rerun and its message was not read.
It is the third timing-sensitive creel test seen on this machine, after the refused-forward chase subtest and the daemon lifecycle and `TestCreelCLI` flakes bundled in `creel-flakes-and-adopt-names`.

## Idea

Run the subtest alone repeatedly, with and without CPU load, to find which wait it assumes, and make it wait for the worker or event it depends on instead of for time.
