---
verdict: APPROVED
---

## Measure

The board entry `csharp-fixtures-compile` asks for a test that compiles every `.cs` fixture under `testdata/` with `dotnet build` in a throwaway project, skips when `dotnet` is not on `PATH`, and sits in a gate tier the machines with .NET run, so a Go-only machine stays green.
The decision record has no `Added after Discussion` entries, so the board body is the measure unamended.

## Diff against the entry

Pull request 830 at `0d0b4f9ee` adds one file, `cmd/lyx/csharpfixtures_integration_test.go`, and changes nothing else.

- Every `.cs` file whose module-relative directory path has a `testdata` component is found; the walk skips only version-control and lyx state directories.
- Each fixture directory is built with `dotnet build` as a class library in a `t.TempDir()` project, so nothing is written into the repository.
- The test skips when `dotnet` is not on `PATH`, and also when `dotnet --version` fails on a host with no SDK, which serves the entry's aim of keeping a machine without .NET green.
- The file carries `//go:build integration`, the tag the gate's integration tier compiles, and `//lyx:guard`, so the round gate runs it on every machine and the skip keeps a Go-only machine green.

Every point of the entry is covered, and the diff makes no change the entry does not call for.

## Open risks carried over

The decision record carries two open risks: a future intentionally invalid fixture would need an exclusion, and an SDK older than 7 rejects `--disable-build-servers`.
Neither names an unrecorded parent correction, and neither is a scope deviation.

## Overlap

- Peer `run-process-ownership` (row 8, own row 21): it changes run-process ownership records and teardown, and lists no changed paths; this run adds one test file in `cmd/lyx`.
  No behaviour, decision or PATTERN entry overlaps, so there is no overlap to order.
- Parent commits: `lyx loom peers` lists none.

## Superseded

No board entry is made obsolete; `csharp-sandbox-hub` and `dotnet-verify-protocol` touch .NET but ask for different work.
