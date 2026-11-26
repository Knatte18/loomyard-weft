# gate test: tier3 command already carries --filter, so a second --filter in args makes dotnet test refuse

`gate-tier3-double-filter` · Note · bug, gate, dotnet

Imported from [issue #234](https://github.com/Knatte18/loomyard/issues/234).

The tier3 command of `lym gate test` already carries a `--filter`. Passing a test filter in `{args}` adds a second `--filter`, and `dotnet test` refuses the call. An agent found that `-p:VSTestTestCaseFilter=...` works as an override, but nothing in the verb's help says so.

This is abnormal because selecting one test class is the common use of the verb. The gate should merge a caller filter into the tier's filter (or replace it) and document the way to do it.

Provenance: task models-stick-slip-floor-tune, note Darn-Write.md.