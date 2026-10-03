# Card 43 — lyxbin-callers

**Edit:**
- `reedcli/smoke_test.go`
- `reedcli/smoke_attach_test.go`
- `reedcli/smoke_coldstart_test.go`
- `reedcli/smoke_dotfill_test.go`
- `reedcli/smoke_lifecycle_test.go`
- `reedcli/smoke_removebyname_test.go`
- `reedcli/smoke_statuslineseed_test.go`
- `reedcli/smoke_warmpath_test.go`
- `loomcli/smoke_test.go`
- `loomcli/smoke_bootstrapwiring_test.go`
- `loomcli/smoke_driverstrand_test.go`
- `loomcli/smoke_drivingsurface_test.go`
- `loomcli/smoke_starttail_test.go`
- `orchcli/smoke_cycle_test.go`
- `orchcli/smoke_live_test.go`
- `//cmd/lyx/main_integration_test.go`

**Uses:**
- `testkit/lyxbin/lyxbin.go`

**Intent:**
Replace the copies of `buildLyxBinary` with card 42's `lyxbin` kit (#324 §6).

Delete `buildLyxBinary`/`buildLyxBinaryWithLDFlags` (`reedcli/smoke_test.go`), `buildLyxBinary` (`cmd/lyx/main_integration_test.go`) and the inline `go build ./cmd/lyx` blocks in `orchcli`'s two smoke files; every caller, in the defining file or a sibling file listed here, calls `lyxbin.Build` or `lyxbin.BuildWithLDFlags`.
`loomcli/smoke_test.go`'s builder builds once per test binary and shares the result across tests, so it cannot use `Build`, whose binary is deleted when the first test ends.
It shrinks to a `sync.Once` wrapper that calls `lyxbin.BuildInto` on its `os.MkdirTemp` directory and keeps recording the error for every later caller.
It is renamed so no `buildLyxBinary` copy remains, and its callers in the sibling loomcli smoke files follow the rename.

Every smoke and integration test keeps its assertions; only how it obtains the binary changes.
Apply D10 to every touched test.

**ImpactSummary:** Smoke tests in reedcli, loomcli and orchcli and one cmd/lyx integration file drop their binary builders for the kit; loomcli keeps its once-per-binary cache over `BuildInto`, and no assertion changes.

**Verify:** go test -tags smoke ./internal/reedcli/... ./internal/loomcli/... ./internal/orchcli/...
