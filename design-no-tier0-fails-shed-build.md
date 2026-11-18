# A run in a hub with no declared tier0 fails at shed build, even a docs-only darn

`no-tier0-fails-shed-build` · Note · bug, gate, config

Imported from [issue #133](https://github.com/Knatte18/loomyard/issues/133).

## Seen

In a new hub with no tests yet, a darn task with profile `docs` (drift scope off, a docs-only change) failed at once, before Darn-Write ran:

```
loomrecipe: shedbuild: producer 5 "Darn-Write": shedrecipe: Darn-Write: gate fixer: loomcli: render the Darn-Write fixer's gate call: gate config: the hub declares no tier0, so no gate has a tier to run
```

The template's gate keys all name tier `[0]`, while the template declares `tiers: {}`.
So every run in a fresh hub fails at shed build, whatever its recipe or diff.
The tackler could not fix it, since the gate config is written only from the prime, and it handed the stop back.

## Workaround

From the prime, declare a tier0 and mark docs as inert:

```
lym config gate --layer default --set 'tiers.tier0={command: "dotnet test {targets} {args}", all_targets: ., timeout_min: 30, counter: vstest}' --set scoper=dotnet --set 'inert_paths=["*.md", "docs/*"]'
```

## Expected

A hub with no declared tiers can run a docs-only task: either the gate call is rendered only when a tier is needed,
or a gate key naming no declared tier passes with nothing to run.
At least, hub setup or the first release should flag the missing tier0 before a run starts, instead of a failed run and a tackler attempt.


## Issue comment, 2026-11-16

## Seen: Finalize ignores `inert_paths`

After the workaround above, the same docs-only darn run reached Finalize, and its landing gate failed with no failing test.
The branch held only `README.md` and a file under `docs/`, both covered by `inert_paths: ["*.md", "docs/*"]`.
Finalize still ran tier0 over all targets (gate key `finalize`, scope `all`), and in a repository with no .NET project yet,
`dotnet test .` exits with MSB1003 "Specify a project or solution file".
The Finalize fixer could not fix it on the task branch without adding code outside the task's bounds.

## Workaround

From the prime, make tier0 pass when no project exists:

```
lym config gate --layer default --set 'tiers.tier0.command=bash -c '"'"'if ls *.sln *.slnx *.csproj >/dev/null 2>&1; then dotnet test {targets} {args}; else echo no .NET project, nothing to test; fi'"'"''
```

## Expected

A change whose files are all inert runs no tier at any gate key, Finalize included,
or a scope-`all` gate in a repository with nothing to test passes instead of failing the landing.
