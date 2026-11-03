# Prove Drift, an anchor other than ".", the dotnet scoper and the shared build on the C# sandbox hub

`sandbox-dotnet-gate` · Task · enhancement, dotnet, gate, drift · draft

## Seen (operator and hub orch, 2026-10-29)

The sandbox hub `lym-csharp-LYMHUB` (shortname `cs`, anchor `lym-csharp/dotnet`, over `Knatte18/lym-csharp`) ran its first darn run, `friction-colebrook`, end to end, and it landed as `7f50ffe`.
Its gate is the simple form: `tier0` runs `dotnet test` without `Category=Slow` and `tier1` with it, with no `scoper`, no `build` step and no `--no-build`.
So the dotnet scoper, the shared build and Drift are not proved on a C# repository.
The first `tier1` gate run on Neve died with "Internal CLR error (0x80131506)" after 900 s and passed on a rerun; the cause is unknown.

## Seen (operator, 2026-11-03)

The agents and orchs that drive Drift hit many problems, because Drift was never tested for real.
Drift's capture and compare start in the directory `.lym-anchor` names (`drift-anchor-and-skip`), but that was tested with fakes and integration tests only, never against a real hub whose anchor is not `.`.

## Goal

The `cs` sandbox hub is the proving ground for Drift and for an anchor other than `.`:

- Drift: baseline capture, scope, compare and waive, end to end on real runs.
- An anchor other than `.`: every row of a run, Drift's included, works from `lym-csharp/dotnet` and not from the repository root.
- The dotnet scoper and the shared build.

## Steps

1. Switch the `cs` gate to `scoper: dotnet`, a `build` step and `--no-build`, and set `drift.enabled`; the setup is in `docs/dotnet-hub.md`.
2. Run 3 darn tasks on `cs` that change code under the anchor, so each run captures a Drift baseline and compares against it.
3. File each lym fault seen with `lym selfreport create`; each fix becomes a task in the loomyard hub.

The trial runs are started by the `cs` hub's own orch.
The sandbox stays the place to reproduce a lym fault another C# hub shows.
