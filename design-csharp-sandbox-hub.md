# A small public NUnit repo as sandbox hub for end-to-end C# runs

`csharp-sandbox-hub` · Note · enhancement, dotnet

## Seen

The C# path was tested only in unit tests over fixtures until the first sandbox run.
Models, the target repo, is proprietary and is never cloned outside OSL-1046 (operator, 2026-10-22).

## Idea

A small public NUnit repo of our own as a C# sandbox hub, beside `lym-test-LYMHUB`, with NUnit tests, built with SDK 10 like Models.
Its projects target `net10.0`, not Models' `net9.0`: what the sandbox proves (build, `dotnet test`, filters, incremental builds) depends on the SDK, not the target runtime (operator, 2026-10-22).

## Toolchain facts

- OSL-1046 has runtimes 8.0.32, 9.0.21 and 10.0.13 and SDK 10.0.402.
- Neve has SDK 10.0.113 from apt (`dotnet-sdk-10.0`, installed 2026-10-22).

## Set up and first run

The hub exists on Neve at `/home/knatte/Code/lym-csharp-LYMHUB` and on OSL-1046 at `~/Code/Knatte18/lym-csharp-LYMHUB` (shortname `cs`, anchor `lym-csharp/dotnet`), over the public repos `Knatte18/lym-csharp` and `Knatte18/lym-csharp-weft`: `LymCsharp.sln` with three libraries and two NUnit test projects, `lens_set: csharp`, gate `tier0` running `dotnet test` without `Category=Slow` and `tier1` with it.
The first darn run, `friction-colebrook`, ran end to end on OSL-1046 and landed as commit `7f50ffe` (operator, 2026-10-29): plan in C# glyphs, Webster writing C#, both gate tiers, anchor in a subdirectory.

## Not yet proved

The sandbox gate is the simple form with no `scoper`, no `build` step and no `--no-build`, so the dotnet scoper, the shared build and Drift are not proved.
The first `tier1` gate run on Neve died with "Internal CLR error (0x80131506)" after 900 s and passed on a rerun; cause unknown.

## Next

Models on OSL-1046 is the target; the operator names what to test there.
Its hub is cloned under the `HenrikNORCE` account at `~/Code/NORCE-DrillingAndWells/Models-LYMHUB` (shortname `mo`) and is not adopted yet.
Setup follows `docs/dotnet-hub.md`, and a `lym gate report` baseline is taken on OSL-1046 first.
The sandbox hub stays as the place to reproduce a lym fault that Models shows.

## Regression harness (operator, 2026-10-25)

The sandbox gets a small capture/compare regression harness on the pattern of Models' `NORCE.Models.NumericalBaseline`: a capture writes scrubbed output into a gitignored baseline directory, and a compare mode fails on the first deviation.
It lets the Drift rows, landed in 1c344094a and off by default behind the hub-wide `drift.enabled` config flag, be proved on Neve, so Models on OSL-1046 is only the final confirmation.
