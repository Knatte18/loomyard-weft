# dotnet test exits 0 with no output in a darn session while lym gate test is off limits

`dotnet-test-silent-exit` · Note · bug, dotnet, process

Imported from [issue #135](https://github.com/Knatte18/loomyard/issues/135).

A darn-writer agent measuring a golden test ran `dotnet test` on a C# test project.
The command exited 0 and printed nothing, not even a test count, while `lym gate test` was off limits to that agent.
The agent could not tell that no test had run and spent several rounds before noticing.
Running the built test assembly through `dotnet vstest` with a `--TestCaseFilter` ran the tests normally.

Why this is abnormal: a silent exit 0 reads as a passing run.
If a lym guard refuses or neutralises `dotnet test`, it should refuse with a message and a nonzero exit.
The darn prompt also names no way to run a single test class while measuring.

What to do:
- Make a refused or neutered `dotnet test` say so.
- Have the darn prompt name a permitted way to run one test class, for example the `dotnet vstest` route above.

Provenance: task `td-stick-slip-surface-torque`; notes Darn-Write.md.
