# No change: died-driver smoke case already passes on main

This task changes no code and no test.

`TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` in `internal/loomcli` failed on a run based before `380101d7`.
There the driver log carried `shedverbs: step interrupted (loop-exited)`, from `d1535412`, where the test waits for the poisoned status file's `invalid state`.
`380101d7` puts the stop's detail into the driver log line, so the log names the invalid state again and the test's expectation holds.
The test passes on `main` as it stands, and the failure was seen only on bases that predate `380101d7`.

The decision the board entry asked for, which message the died-driver case surfaces, needs no answer: the log keeps the reason the run halted.

Nothing to check or run by hand beyond the existing `internal/loomcli` suite.
