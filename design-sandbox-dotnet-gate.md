# Prove the dotnet scoper, shared build and Drift on the C# sandbox hub

`sandbox-dotnet-gate` · Note · enhancement, dotnet, gate

## Seen (operator and hub orch, 2026-10-29)

The sandbox hub `lym-csharp-LYMHUB` (shortname `cs`, anchor `lym-csharp/dotnet`, over `Knatte18/lym-csharp`) ran its first darn run, `friction-colebrook`, end to end, and it landed as `7f50ffe`.
Its gate is the simple form: `tier0` runs `dotnet test` without `Category=Slow` and `tier1` with it, with no `scoper`, no `build` step and no `--no-build`.
So the dotnet scoper, the shared build and Drift are not proved on a C# repository.
The first `tier1` gate run on Neve died with "Internal CLR error (0x80131506)" after 900 s and passed on a rerun; the cause is unknown.

## Idea

Switch the sandbox gate to `scoper: dotnet`, a `build` step and `--no-build`, so the scoper and the shared build are proved there, then turn on `drift.enabled` there to prove Drift, before any C# hub that holds proprietary code relies on them.
The hub's setup is in `docs/dotnet-hub.md`; the sandbox stays the place to reproduce a lym fault another C# hub shows.
