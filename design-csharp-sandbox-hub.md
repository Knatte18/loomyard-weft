# [.NET] A small public NUnit repo as sandbox hub for end-to-end C# runs

`csharp-sandbox-hub` · Note · enhancement, dotnet

## Seen

The C# path is tested only in unit tests over fixtures: no loom run has gone end to end on a C# repo, with a plan in C# glyphs, Webster writing C#, and the gate running `dotnet test`.
Models, the target repo, is proprietary and is never cloned outside OSL-1046 (operator, 2026-10-22).

## Idea

A small public NUnit repo of our own as a C# sandbox hub, beside `lyx-test-LYXHUB`, shaped like Models: projects targeting `net9.0`, built with SDK 10 (operator, 2026-10-22), NUnit tests.
Runs on it prove the whole C# path on Neve before OSL-1046.

## Toolchain facts

- OSL-1046 has SDKs 8.0.32, 9.0.21 and 10.0.13 (operator, 2026-10-22).
- Neve is Ubuntu 26.04, whose apt carries .NET 10 only; SDK 10 builds `net9.0` projects, but running their tests needs the .NET 9 runtime, which Microsoft's `dotnet-install.sh --runtime dotnet --channel 9.0 --install-dir ~/.dotnet` installs without sudo.
