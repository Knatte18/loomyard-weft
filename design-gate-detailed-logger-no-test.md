# gate test: reports 'ran no test' when the detailed console logger's summary is used

`gate-detailed-logger-no-test` · Note · bug, gate, dotnet

Imported from [issue #235](https://github.com/Knatte18/loomyard/issues/235).

With `--logger "console;verbosity=detailed"` passed to `lym gate test`, the gate reported that it ran no test, although the test log shows `Total tests: 1, Passed: 1`. The gate's test counter appears to misread the detailed logger's summary format.

This is abnormal because a passing run is reported as a run of nothing, which an agent may read as a failure or retry needlessly. The counter should parse the summary lines of every console logger verbosity, or read the result from a structured logger (trx) instead.

Provenance: task models-stick-slip-floor-tune, note Darn-Write.md.