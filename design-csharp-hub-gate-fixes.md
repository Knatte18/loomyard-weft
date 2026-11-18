# C# hub: silent dotnet test exit, and runs failing in a hub with no tier0 or only inert changes

`csharp-hub-gate-fixes` · Task · bug, dotnet, process, gate, config

Depends on: [`orch-ignores-deny-python`](design-orch-ignores-deny-python.md), [`leftover-strands-and-runs`](design-leftover-strands-and-runs.md)

Two bugs met in the C# hub, bundled because both sit in the gate and its guard around a hub test command.

## Part: dotnet test exits 0 with no output

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


## Part: a hub with no tier0 fails at shed build, and Finalize ignores inert_paths

Imported from [issue #133](https://github.com/Knatte18/loomyard/issues/133).

### Seen

In a new hub with no tests yet, a darn task with profile `docs` (drift scope off, a docs-only change) failed at once, before Darn-Write ran:

```
loomrecipe: shedbuild: producer 5 "Darn-Write": shedrecipe: Darn-Write: gate fixer: loomcli: render the Darn-Write fixer's gate call: gate config: the hub declares no tier0, so no gate has a tier to run
```

The template's gate keys all name tier `[0]`, while the template declares `tiers: {}`.
So every run in a fresh hub fails at shed build, whatever its recipe or diff.
The tackler could not fix it, since the gate config is written only from the prime, and it handed the stop back.

### Workaround

From the prime, declare a tier0 and mark docs as inert:

```
lym config gate --layer default --set 'tiers.tier0={command: "dotnet test {targets} {args}", all_targets: ., timeout_min: 30, counter: vstest}' --set scoper=dotnet --set 'inert_paths=["*.md", "docs/*"]'
```

### Expected

A hub with no declared tiers can run a docs-only task: either the gate call is rendered only when a tier is needed,
or a gate key naming no declared tier passes with nothing to run.
At least, hub setup or the first release should flag the missing tier0 before a run starts, instead of a failed run and a tackler attempt.


### Issue comment, 2026-11-16

### Seen: Finalize ignores `inert_paths`

After the workaround above, the same docs-only darn run reached Finalize, and its landing gate failed with no failing test.
The branch held only `README.md` and a file under `docs/`, both covered by `inert_paths: ["*.md", "docs/*"]`.
Finalize still ran tier0 over all targets (gate key `finalize`, scope `all`), and in a repository with no .NET project yet,
`dotnet test .` exits with MSB1003 "Specify a project or solution file".
The Finalize fixer could not fix it on the task branch without adding code outside the task's bounds.

### Workaround

From the prime, make tier0 pass when no project exists:

```
lym config gate --layer default --set 'tiers.tier0.command=bash -c '"'"'if ls *.sln *.slnx *.csproj >/dev/null 2>&1; then dotnet test {targets} {args}; else echo no .NET project, nothing to test; fi'"'"''
```

### Expected

A change whose files are all inert runs no tier at any gate key, Finalize included,
or a scope-`all` gate in a repository with nothing to test passes instead of failing the landing.

