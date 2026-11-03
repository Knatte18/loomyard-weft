# Drive the C# sandbox hub through a Drift and subdirectory-anchor trial and report what breaks

`sandbox-dotnet-gate` · Task · enhancement, dotnet, gate, drift · draft

## Seen (operator, 2026-11-03)

The agents and orchs that drive Drift hit many problems, because Drift was never tested for real.
Drift's capture and compare start in the directory `.lym-anchor` names (`drift-anchor-and-skip`), but that was tested with fakes and integration tests only, never against a real hub whose anchor is not `.`.
The dotnet scoper and the shared build are not proved on a C# repository either.

## Goal

This darn run drives the C# sandbox hub `lym-csharp-LYMHUB` (shortname `cs`, anchor `lym-csharp/dotnet`, over `Knatte18/lym-csharp`) through a fixed trial and reports what breaks:

- Drift: baseline capture, scope, compare, verify and waive, end to end on real runs.
- An anchor other than `.`: every row of a run, Drift's included, works from `lym-csharp/dotnet` and not from the repository root.
- The dotnet scoper and the shared build.

## Set up already (hub orch, 2026-11-03)

- The hub's Bolt checkout was moved from `_board` to `_bolt` by hand, and Neve took the hub over with `lym adopt root`.
- `gate.yaml`: `scoper: dotnet`, a `build` step, tiers with `{targets}`, `{filter}` and `--no-build`, and `drift-verify` among the gate keys.
- `drift.yaml`: `enabled: true`, with capture and compare through `dotnet run --project src/LymCsharp.Regression`.
- Three darn `light` tasks on the `cs` board, chained by `depends_on` so each Drift outcome stands alone:
  1. `pipeflow-reynolds-helper`: a behavior-preserving refactor; Drift must compare clean.
  2. `churchill-friction`: a new friction model and one new regression case; shows how Drift treats a case the baseline lacks.
  3. `report-significant-digits`: an intended output change; Drift must report it and its verify accept it with a reason.

## Steps

1. Read the `cs` hub's adopt check result; a red check is the first finding.
2. Release the three `cs` tasks, one at a time as each lands, with `lym board set-status` run through `env -C /home/knatte/Code/lym-csharp-LYMHUB/lym-csharp/dotnet`; the `cs` hub's dobby starts them and its orch settles their stops.
3. Follow each run to its landing or a stop it cannot pass, reading `lym batten status`, `lym loom status`, `lym drift status` and the run's drive reports.
4. Run exactly these three trial tasks once each; start no other task on `cs`.

## Deliverable

- A report `docs/research/drift-csharp-trial-2026-11-03.md`: per trial task, what each Drift row and each gate did, with the anchor and the scoper's scope, and every fault seen with its evidence.
- Each lym fault becomes a bug note on this hub's board, one finding per note, with the module labels it touches.
- No code change to lym in this run: fixes are tasks of their own.
