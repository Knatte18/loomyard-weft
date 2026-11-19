# lym gate test: unexplained test-host crash and false 'ran no test' verdict with a detailed console logger

`gate-verdict-from-trx` · Note · bug, gate, dotnet

Imported from [issue #163](https://github.com/Knatte18/loomyard/issues/163).

During diagnostic runs of `lym gate test --tier 1 -- --filter ...`:

1. The first run aborted with "Test host process crashed" and the gate log gave no cause. The identical command passed on the next try.
2. With a detailed console logger among the pass-through args, the gate reported "ran no test" although the run's own output showed `Passed: 1`. The verdict line was wrong for that call.

This is abnormal because the gate verdict is what agents rely on; a verdict contradicting the test output, and a crash with no captured cause, force agents to distrust it.
Fix: parse the test result from the TRX or a logger the gate controls instead of console text, so user-supplied loggers cannot change the verdict; and capture the test host's crash dump or stderr in the gate log when the host crashes.

Provenance: task bed-erosion-rate-gap, note Darn-Write.md.