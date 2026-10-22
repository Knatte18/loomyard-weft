# [.NET] A small public NUnit repo as sandbox hub for end-to-end C# runs

`csharp-sandbox-hub` · Note · enhancement, dotnet

## Seen

The C# path is tested only in unit tests over fixtures: no loom run has gone end to end on a C# repo, with a plan in C# glyphs, Webster writing C#, and the gate running `dotnet test`.
Models, the target repo, is proprietary and is never cloned outside OSL-1046 (operator, 2026-10-22).

## Idea

A small public NUnit repo of our own as a C# sandbox hub, beside `lyx-test-LYXHUB`, with NUnit tests, built with SDK 10 like Models.
Its projects target `net10.0`, not Models' `net9.0`: what the sandbox proves (build, `dotnet test`, filters, incremental builds) depends on the SDK, not the target runtime, and Neve then needs no 9.0 runtime (operator, 2026-10-22).
Runs on it prove the whole C# path on Neve before OSL-1046.

## Toolchain facts

- OSL-1046 has SDKs 8.0.32, 9.0.21 and 10.0.13 (operator, 2026-10-22).
- Neve has SDK 10.0.113 from apt (`dotnet-sdk-10.0`, installed 2026-10-22).
