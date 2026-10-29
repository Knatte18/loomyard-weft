# A small public NUnit repo as sandbox hub for end-to-end C# runs

`csharp-sandbox-hub` · Note · enhancement, dotnet

## Seen

The C# path is tested only in unit tests over fixtures: no loom run has gone end to end on a C# repo, with a plan in C# glyphs, Webster writing C#, and the gate running `dotnet test`.
Models, the target repo, is proprietary and is never cloned outside OSL-1046 (operator, 2026-10-22).

## Idea

A small public NUnit repo of our own as a C# sandbox hub, beside `lym-test-LYMHUB`, with NUnit tests, built with SDK 10 like Models.
Its projects target `net10.0`, not Models' `net9.0`: what the sandbox proves (build, `dotnet test`, filters, incremental builds) depends on the SDK, not the target runtime, and Neve then needs no 9.0 runtime (operator, 2026-10-22).
Runs on it prove the whole C# path on Neve before OSL-1046.

## Toolchain facts

- OSL-1046 has SDKs 8.0.32, 9.0.21 and 10.0.13 (operator, 2026-10-22).
- Neve has SDK 10.0.113 from apt (`dotnet-sdk-10.0`, installed 2026-10-22).

## Set up (2026-10-22)

The hub exists on Neve at `/home/knatte/Code/lym-csharp-LYMHUB` (shortname `cs`), over the public repos `Knatte18/lym-csharp` and `Knatte18/lym-csharp-weft`: `LymCsharp.sln` with three libraries and two NUnit test projects, `lens_set: csharp`, gate `tier0` running `dotnet test` without `Category=Slow` and `tier1` with it.
No run has gone through it yet; the first run there is the end-to-end proof this note asks for.
The first `tier1` gate run died with "Internal CLR error (0x80131506)" after 900 s and passed on a rerun; cause unknown.

## Order before Models (2026-10-26)

1. `beam-start-and-ownership` lands (done 2026-10-26).
2. `rename-binary-lym` runs alone.
3. One darn run on this sandbox hub proves the whole C# path, with the hub anchored in a subdirectory (anchor not `.`) so the per-worktree `.lym-anchor` path is proved end to end in the same run.
4. Models, on OSL-1046; nothing is done there until the operator names what to test.

## Decision (operator, 2026-10-29)

Skip the sandbox run and test directly on Models, a real C# repo on OSL-1046.
Models' subpath is not `.`, so the anchor-in-a-subdirectory proof of step 3 is covered there.
What the sandbox would have separated is lost: a failure on Models can be a lym fault or a Models fault, and Models cannot be shared for a reproduction.
The sandbox hub stays as the place to reproduce a lym fault that Models shows.
Setup follows `docs/dotnet-hub.md`, and a `lym gate report` baseline is taken on OSL-1046 first.
The operator names what to test first.

## Regression harness (operator, 2026-10-25)

The sandbox gets a small capture/compare regression harness on the pattern of Models' `NORCE.Models.NumericalBaseline`: a capture writes scrubbed output into a gitignored baseline directory, and a compare mode fails on the first deviation.
It lets the Drift rows, landed in 1c344094a and off by default behind the hub-wide `drift.enabled` config flag, be proved on Neve, so Models on OSL-1046 is only the final confirmation.
