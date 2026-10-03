# Card 32 — tmuxkit-testmain-b

**Edit:**
- `internal/landingshed/testmain_integration_test.go`
- `internal/logger/sink_callsite_integration_test.go`
- `internal/loomcli/testmain_test.go`
- `internal/lyxcwd/testmain_test.go`
- `internal/orchcli/testmain_test.go`
- `internal/orchcli/testmain_integration_test.go`
- `internal/planglyph/testmain_test.go`
- `internal/preflight/testmain_test.go`
- `internal/preflightshed/testmain_integration_test.go`
- `internal/reedcli/testmain_test.go`
- `internal/shedcli/testmain_integration_test.go`
- `internal/shuttlecli/testmain_test.go`
- `internal/standalonegeom/testmain_integration_test.go`
- `internal/stencilcli/testmain_test.go`
- `internal/treadleengine/testmain_test.go`
- `internal/webstercli/testmain_test.go`
- `internal/websterengine/testmain_test.go`

**Create:**
- `internal/landingshed/testmain_test.go`
- `internal/logger/testmain_test.go`
- `internal/preflightshed/testmain_test.go`
- `internal/reedengine/testmain_test.go`
- `internal/shedcli/testmain_test.go`
- `internal/standalonegeom/testmain_test.go`
- `internal/standalonestate/testmain_test.go`
- `internal/testkit/lyxbin/testmain_test.go`
- `internal/verifyrun/testmain_test.go`

**Uses:**
- `internal/testkit/tmuxkit/tmuxkit.go`

**Intent:** Run the second half of the packages that have an `integration`- or `smoke`-tagged test file through `tmuxkit.Main`, in the same shape as card 31: existing `TestMain`s exit through `tmuxkit.Main(m)` after their own setup, integration-only `TestMain`s gain a `//go:build !integration` `testmain_test.go` sibling, and packages with none gain an untagged one.
`orchcli`'s tag-split pair is covered on both sides.
`logger`'s `TestMain` in `sink_callsite_integration_test.go` keeps its own setup and exits through `tmuxkit.Main`.
Tests in these packages that relied on running inside the caller's tmux are fixed to set the variable they need.
This card changes no `smoke`-tagged file.

**ImpactSummary:** Test-binary setup only, in the second half of the tagged-test packages; no test body changes.
