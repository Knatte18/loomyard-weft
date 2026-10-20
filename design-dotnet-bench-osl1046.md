# [.NET 001] Measure NORCE Models build and test on OSL-1046 with Cortex XDR

`dotnet-bench-osl1046` · Note · audit, dotnet

## Goal

Measure what building and testing NORCE Models costs on OSL-1046 with Cortex XDR before any .NET support is built, so the measurement decides which `[.NET]` entry comes first, as the Go measurement in `docs/benchmarks/gate-test-xdr.md` did for Go.

## Seen (orch, 2026-10-20)

- On OSL-1046 a freshly built .NET apphost (`./out/dnbench`) was held 10.1 s by XDR, while `dotnet out/dnbench.dll` ran in 0.07 s: XDR holds new executables, and a DLL started through the known `dotnet` host was not held.
- `dotnet test` starts its test host through `dotnet`, so the XDR cost may land on build output and MSBuild nodes rather than on test start; this is not measured.
- For Go, most of the cost was new binaries with unseen hashes and test-cache misses, and `-trimpath` cut a cached unit run from 98.6 s to 20.3 s.

## Measure

- `dotnet restore`, `dotnet build` and `dotnet test` of `NORCE.Models.sln` from `src/csharp/NORCE.Models/`, cold (fresh clone or clean `bin/` and `obj/`) and warm, wall and CPU time.
- The same in a second checkout of the same commit, to see whether identical output is reused or held again.
- Whether deterministic builds (`ContinuousIntegrationBuild=true`, `PathMap`) make two checkouts' DLLs byte-identical, the .NET counterpart of `-trimpath`.
- The slow tier (`-p:RunSlowTests=true`) and the baseline tier (`-p:RunBaseline=true`) once each.
- Log the numbers in `docs/benchmarks/` beside the Go ones, with the hardware block of `gate-test-xdr.md`.
