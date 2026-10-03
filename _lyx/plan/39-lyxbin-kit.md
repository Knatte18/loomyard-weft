# Card 39 — lyxbin-kit

**Create:**
- `testkit/lyxbin/lyxbin.go`
- `testkit/lyxbin/lyxbin_integration_test.go`

**Edit:**
- `reedcli/smoke_test.go`
- `loomcli/smoke_test.go`
- `orchcli/smoke_cycle_test.go`
- `orchcli/smoke_live_test.go`
- `//cmd/lyx/main_integration_test.go`
- `//cmd/lyx/tierpurity_test.go`
- `//CONSTRAINTS.md`

**Uses:**
- `testkit/enforcement_test.go`

**Intent:**
Replace the copies of `buildLyxBinary` with one kit (#324 §6, D2, D3).

`lyxbin.go` exports `Build(t testing.TB) string` and `BuildWithLDFlags(t testing.TB, ldflags string) string`, each running `go build -o <t.TempDir()>/lyx[.exe] ./cmd/lyx` from the module root and returning the binary path.
It finds the module root from its own source path via `runtime.Caller`, so the only process it ever starts is that `go build`; that is the whole of the `os/exec` exemption card 1's scan grants it.
A caller that builds once per test binary keeps its own `sync.Once` around `Build`, since the kit holds no cross-test state.

Delete `buildLyxBinary`/`buildLyxBinaryWithLDFlags` (`reedcli/smoke_test.go`), `buildLyxBinary` (`cmd/lyx/main_integration_test.go`, `loomcli/smoke_test.go`) and the inline `go build ./cmd/lyx` blocks in `orchcli`'s two smoke files; callers use the kit.

`tierpurity_test.go`'s `bannedTokens` gains `"lyxbin."`, so only `integration`/`smoke`-tagged files can call the kit; the rest of the token set and the allowlist stay verbatim.
`CONSTRAINTS.md`'s Test Tier Purity section names the `lyxbin.` ban beside the gitkit rule.
Prove the token: plant an untagged `lyxbin.Build` call, watch Tier Purity fail, remove it.

`lyxbin_integration_test.go` builds the binary and asserts the returned path exists and is executable; ldflags reach the build (check with `go version -m` on the result rather than running it).

What the exemption lets through: one kit importing `os/exec`; review keeps it to `go build` of `./cmd/lyx`, and the Tier Purity token keeps its callers out of Tier 1.

**ImpactSummary:** Smoke and integration tests in four packages share one binary builder; Tier Purity gains one banned token.

**Verify:** go test -tags smoke ./internal/reedcli/... ./internal/loomcli/... ./internal/orchcli/...
