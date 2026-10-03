# Card 42 — lyxbin-kit

**Create:**
- `testkit/lyxbin/lyxbin.go`
- `testkit/lyxbin/lyxbin_integration_test.go`

**Edit:**
- `//cmd/lyx/tierpurity_test.go`
- `//cmd/lyx/spawnobservability_test.go`
- `//CONSTRAINTS.md`

**Uses:**
- `testkit/enforcement_test.go`

**Intent:**
Provide one shared `lyx` binary builder (#324 §6, D2, D3) for the copies card 43 converts.

`lyxbin.go` exports:

- `BuildInto(dir, ldflags string) (string, error)`, which runs `go build -o <dir>/lyx[.exe] [-ldflags <ldflags>] ./cmd/lyx` from the module root, omitting `-ldflags` when ldflags is empty, and returns the binary path or the build error with its combined output;
- `Build(tb testing.TB) string` and `BuildWithLDFlags(tb testing.TB, ldflags string) string`, which call `BuildInto` on `tb.TempDir()` and `tb.Fatalf` on its error.

`Build`'s binary lives only as long as the calling test's temp dir, so it is for per-test callers.
A caller that builds once per test binary calls `BuildInto` inside its own `sync.Once` on a directory that outlives every test (an `os.MkdirTemp` directory, as `loomcli`'s smoke builder uses today) and records the error rather than fataling inside the `Once`; the kit holds no cross-test state.
It finds the module root from its own source path via `runtime.Caller`, so the only process it ever starts is that `go build`; that is the whole of the `os/exec` exemption card 1's scan grants it.
Its package doc (D11) says the kit builds `./cmd/lyx` for `integration`/`smoke` tests, is the one kit exempt from the spawn-import rule, runs nothing but that `go build`, and is callable only from tagged test files.

`lyxbin.go` is a non-test file under `internal/` containing an `exec.Command`, so `cmd/lyx/spawnobservability_test.go` would fail it.
`CONSTRAINTS.md` puts test-fixture machinery outside the Live-Substrate Spawn Observability rule, so `spawnObservabilityAllowedSpawners` gains one "not governed" row for `internal/testkit/lyxbin/lyxbin.go`, worded like the `internal/hubforge/hub.go` row: a test-fixture builder, not a code path reachable from a `lyx` command.
That row is the only change to that guard; its other rows, floor and scan stay verbatim.

`tierpurity_test.go`'s `bannedTokens` gains `"lyxbin."`, so only `integration`/`smoke`-tagged files can call the kit; the rest of the token set and the allowlist stay verbatim.
`CONSTRAINTS.md`'s Test Tier Purity section names the `lyxbin.` ban beside the gitkit rule.
Prove the token: plant an untagged `lyxbin.Build` call, watch Tier Purity fail, remove it.

`lyxbin_integration_test.go` builds the binary and asserts the returned path exists and is executable; ldflags reach the build (check with `go version -m` on the result rather than running it); and `BuildInto` returns an error, not a fatal, for an unwritable directory.

What the exemption lets through: one kit importing `os/exec`, plus one spawn-observability row; review keeps it to `go build` of `./cmd/lyx`, card 1's scan bars every other kit from importing it, and the Tier Purity token keeps its callers out of Tier 1.

**ImpactSummary:** Two cmd/lyx guards each gain one entry (a Tier Purity token, a not-governed spawn-observability row) and CONSTRAINTS.md names the token; no existing test changes until card 43 converts the callers.
