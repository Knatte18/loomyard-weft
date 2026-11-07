# Tests wait real time and repeat builds every tier 1 pass

`test-real-time-waits` · Task · enhancement, git, gate, termwindow, process

Depends on: [`gate-tier-binaries-only`](design-gate-tier-binaries-only.md)

The operator wants tests faster as a priority (2026-11-07).
Items 7 and 9 of `docs/research/speed/tier1-test-revision-2026-11-06/report.md`, read before starting.

## Seen

- Tests wait real time: `internal/gitexec`'s `TestRun_RemoteBounds` waits real deadlines and a literal `sleep 2` (3.5 s of the package); `internal/gatecli`'s `TestGateTest_DetachedRun` has stand-in delays of 2 to 5 s and leases of 3 s; `internal/termwindow` has a 3 s row; `internal/verifyrun` has 1 s expiries.
- Tests repeat builds every pass: `TestBuildInto_UnwritableDirReturnsError` builds a binary only to fail writing it (2.3 s); `TestCrossCompileLinux` and the dotnet fixture build cost about 5 s of `cmd/lym`.

## Wanted

- Shorten each real-time wait, through an injected clock or a shorter bound, keeping a margin that does not flake on a loaded machine; each package's comments name its margins.
- Have `TestBuildInto_UnwritableDirReturnsError` fail before building, and keep `TestCrossCompileLinux` and the dotnet build in the whole pass while a card or scoped gate runs them only when a file they guard changes, such as a `_linux.go` file.
- No test loses what it pins.
