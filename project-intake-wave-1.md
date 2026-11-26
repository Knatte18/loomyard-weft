# P1 Intake wave 1: PR modes, Webster, dotnet gate, reed, creel, commits

Rendered 2026-11-26 09:16 UTC; every age below is as of then.
Under: [Board](README.md).

`intake-wave-1` · Age 57m · Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (2)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Reed reads each client's own session, and shellguard refuses the session bus** | [`reed-client-and-bus-guard`](design-reed-client-and-bus-guard.md) | darn · light | Darn-Write | neve | 2d<br>1m | reed, process |
| T6 | **creel post refusal after a persisted cd does not say why** | [`creel-refusal-after-cd`](design-creel-refusal-after-cd.md) | darn · light | Darn-Write | neve | 1h<br>1m | creel, cwd |

## Layer A (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **PR block in three modes, review, trail and off, with no agent under it** | [`pr-modes-no-agent`](design-pr-modes-no-agent.md) | darn · solid | T3 | 2d<br>56m | landing, loom, config |
| T2 | **Webster: plan fixes after begun batches, and recovery that ends asking** | [`webster-plan-recovery`](design-webster-plan-recovery.md) | darn · light | T3 | 1h<br>57m | webster, quarry |
| T4 | **Gate test fixes for dotnet hubs: hung testhost, loggers, filters, tiers** | [`dotnet-gate-fixes`](design-dotnet-gate-fixes.md) | darn · light | T3 | 1h<br>57m | gate, dotnet, darn, burler |
| T7 | **Iterative agents should commit often** | [`writers-commit-each-step`](design-writers-commit-each-step.md) | darn · light | T3 | 1h<br> | stencil, darn |
